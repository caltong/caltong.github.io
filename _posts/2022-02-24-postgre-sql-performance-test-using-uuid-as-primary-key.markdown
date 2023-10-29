---
layout: post
title:  "PostgreSQL Performance Test Using UUID as Primary Key"
date:   2022-02-24 12:05:48 +0800
#categories: jekyll update
---

# PostgreSQL Performance Test Using UUID as Primary Key

## Environmental preparation

In order to compare the difference between HDD and SSD,
the HDD and SSD are tested separately,
and the machine is an ESXi virtual machine with the same configuration:

- CPU：2 cores
- RAM：4G
- Disk：32G(HDD/SSD)from H710 card with 1G cache
- PostgreSQL：psql (10.20 (Ubuntu 10.20-1.pgdg20.04+1))

Create table

```
--Unordered uuid
pgbenchdb=# create table test_uuid_v4(id char(32) primary key);
CREATE TABLE
--Ordered uuid
pgbenchdb=# create table test_time_nextval(id char(32) primary key);
CREATE TABLE
--Increasing sequence
pgbenchdb=# create table test_seq_bigint(id int8 primary key);
CREATE TABLE
--Creating sequence
create sequence test_seq start with 1 ;
```

Test file

```
--Testing the unordered uuid
vi pgbench_uuid_v4.sql
insert into test_uuid_v4 (id) values (replace(uuid_generate_v4()::text,'-',''));
--Testing ordered uuid
vi pgbench_time_nextval.sql
insert into test_time_nextval (id) values (replace(uuid_time_nextval()::text,'-',''));
--Test sequence
vi pgbench_seq_bigint.sql
insert into test_seq_bigint (id) values (nextval('test_seq'::regclass));
```

Run test

```
pgbench -M prepared -r -n -j 8 -c 8 -T 60 -f ./pgbench_uuid_v4.sql -U sa pgbenchdb 
```

```
pgbenchdb=# insert into test_uuid_v4 (id) select  replace(uuid_generate_v4()::text,'-','') from generate_series(1,1000000);
INSERT 0 1000000
Time: 43389.817 ms (00:43.390)
pgbenchdb=# insert into test_time_nextval (id) select replace(uuid_time_nextval()::text,'-','') from generate_series(1,1000000);
INSERT 0 1000000
Time: 30585.134 ms (00:30.585)
pgbenchdb=#  insert into test_seq_bigint select generate_series (1,1000000);
INSERT 0 1000000
Time: 9818.639 ms (00:09.819)
Unordered uuid insertion for 100w takes 43s, ordered takes 30s, and sequential takes 10s.
```