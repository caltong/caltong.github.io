---
layout: post
title:  "Be Careful With Executor In Springboot"
date:   2024-01-18 20:31:00 +0800
---

# Be Careful With Executor In Springboot

When using a thread pool executor as a component like below codes, 
Spring will shut down this executor when Spring is shutting down.
```java
@Bean
public ThreadPoolExecutor executor() {
    return new ThreadPoolExecutor(4, 4,
        60, TimeUnit.SECONDS,
        new ArrayBlockingQueue<>(10),
        new ThreadPoolExecutor.DiscardPolicy());
    }
```
But if creating executor like below codes, there will be a problem.
```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(4, 4,
                            60, TimeUnit.SECONDS,
                            new ArrayBlockingQueue<>(10),
                            new ThreadPoolExecutor.DiscardPolicy());
executor.execute(()->{
    //sample runnable
        });
```
In this case, Spring won't be shut down successfully due to this executor can't be shut down automatically.
Spring will wait this executor endlessly.

Therefore, remember shut down executor explicitly by calling `executor.shutdown()`.
This method will let Spring shut down smoothly.

