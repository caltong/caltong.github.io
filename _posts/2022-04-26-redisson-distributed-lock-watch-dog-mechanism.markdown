---
layout: post
title:  "Redisson distributed lock watch dog mechanism"
date:   2022-04-26 12:05:48 +0800
#categories: jekyll update
---

# Redisson distributed lock watch dog mechanism

Found this exception when using Redisson to implement the distributed lock:

```
java.lang.IllegalMonitorStateException: attempt to unlock lock, not locked by current thread by node id: d8fa7ae5-3152-468a-8793-6102b512df68 thread-id: 1
```

It happens when trying to unlock a lock that is already unlocked.
It can be reproduced by the codes below.

```
@Test
public void reproduce() throws InterruptedException {
    RLock lock = redissonClient.getLock("reproduce");
    lock.lock(5,TimeUnit.SECONDS);
    Thread.sleep(6000);
    try{
        lock.unlock();
        Assert.fail();
    }catch (IllegalMonitorStateException e){
        System.out.println(e);
    }
}
```

For the sake of code robustness, we need to catch this exception when unlocking a lock.

```
@Test
public void fix() throws InterruptedException {
    RLock lock = redissonClient.getLock("fix");
    lock.lock(5,TimeUnit.SECONDS);
    Thread.sleep(6000);
    try {
        lock.unlock();
    }catch (IllegalMonitorStateException e){
        System.out.println("Lock has expired already.");
    }
}
```

The code above shows us the problem when setting a lock with a specific timeout value.
But if we don't set a timeout value, then this lock won't be released by default from redisson.
The Redis locks with no expiration time may cause deadlock issues when the thread has some exceptions.
Redisson provides us with a watchdog to avoid the deadlock issue.
It makes codes a lot more simple for us.
No need to worry about the deadlock issue.
The code below shows us the source code of the mechanism of the watchdog.

```
@Test
public void watchDog() throws InterruptedException {
    RLock lock = redissonClient.getLock("watchDog");
    lock.lock();
    for (int i = 0; i < 30; i++) {
        Thread.sleep(1000);
        System.out.println(lock.remainTimeToLive());
    }
}
```

The output is:

```
28978
27958
26936
25914
24891
23872
22854
21835
20816
29883
28864
27845
26827
25818
24810
23792
22784
21772
20758
29845
28831
27821
26801
25783
24766
23756
22739
21721
20712
29791
```

We can see that although the code does not explicitly specify the lock expiration time,
Redisson sets it to 30 seconds by default, and then every 10 seconds,
the lock expiration time will be renewed for 10 seconds,
that is, as long as the lock-holding thread does not release the lock voluntarily or an exception occurs,
then the lock will not be released.
This is guaranteed by Redisson's watchdog mechanism.

We can get this code from Redisson config file:

```
/**
    * This parameter is only used if lock has been acquired without leaseTimeout parameter definition. 
    * Lock expires after <code>lockWatchdogTimeout</code> if watchdog 
    * didn't extend it to next <code>lockWatchdogTimeout</code> time interval.
    * <p>  
    * This prevents against infinity locked locks due to Redisson client crush or 
    * any other reason when lock can't be released in proper way.
    * <p>
    * Default is 30000 milliseconds
    * 
    * @param lockWatchdogTimeout timeout in milliseconds
    * @return config
    */
public Config setLockWatchdogTimeout(long lockWatchdogTimeout) {
    this.lockWatchdogTimeout = lockWatchdogTimeout;
    return this;
}
```

As you can see, the watchdog mechanism only intervenes when `leaseTimeout` is not set,
as we can see from the source code below:

```
private void renewExpiration() {
    ExpirationEntry ee = EXPIRATION_RENEWAL_MAP.get(getEntryName());
    if (ee == null) {
        return;
    }

    Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
        @Override
        public void run(Timeout timeout) throws Exception {
            ExpirationEntry ent = EXPIRATION_RENEWAL_MAP.get(getEntryName());
            if (ent == null) {
                return;
            }
            Long threadId = ent.getFirstThreadId();
            if (threadId == null) {
                return;
            }
            
            RFuture<Boolean> future = renewExpirationAsync(threadId);
            future.whenComplete((res, e) -> {
                if (e != null) {
                    log.error("Can't update lock " + getRawName() + " expiration", e);
                    EXPIRATION_RENEWAL_MAP.remove(getEntryName());
                    return;
                }
                
                if (res) {
                    // reschedule itself
                    renewExpiration();
                } else {
                    cancelExpirationRenewal(null);
                }
            });
        }
    }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);

    ee.setTimeout(task);
    }
```

By default, the lock will be renewed after `internalLockLeaseTime / 3, TimeUnit.MILLISECONDS`,
which is 30/3 seconds after the default setting.