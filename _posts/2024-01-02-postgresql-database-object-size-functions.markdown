---
layout: post
title:  "PostgreSQL Database Object Size Functions"
date:   2024-01-02 22:55:00 +0800
---

# PostgreSQL Database Object Size Functions

PG provides us several database object size functions to calculate the disk space usage of database objects. Such as:
- pg_database_size()
- pg_indexes_size()
- pg_relation_size()
- pg_table_size()
- pg_total_relation_size()

The image below shows very clear about the definition of those functions. Thanks the answer from Satish Patro in this stack overflow question.

[What's the difference between pg_table_size, pg_relation_size & pg_total_relation_size? (PostgreSQL)](https://stackoverflow.com/questions/41991380/whats-the-difference-between-pg-table-size-pg-relation-size-pg-total-relatio)

![pg-size-functions.png](/assets/img/in-post/2024-01-02-postgresql-database-object-size-functions/pg-size-functions.png)