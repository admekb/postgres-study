# Настройка autovacuum с учетом оптимальной производительности

## установить на него PostgreSQL 13 с дефолтными настройками

```bash
admekb@postgres:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 online postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
```

## применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла

```bash
sudo vi /etc/postgresql/13/main/postgresql.conf
```
>max_connections = 40
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 500
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 6553kB
min_wal_size = 4GB
max_wal_size = 16GB

## создайте новую базу данных testdb

```bash
postgres=# create database testdb;
CREATE DATABASE
```

## зайдите в созданную базу данных под пользователем postgres

```bash
postgres=# \c testdb
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1), server 13.7 (Ubuntu 13.7-1.pgdg20.04+1))
You are now connected to database "testdb" as user "postgres".
testdb=#
```

## создайте новую схему testnm

```bash
testdb=# create schema testnm;
CREATE SCHEMA
```

## создайте новую таблицу t1 с одной колонкой c1 типа integer

```bash
testdb=# CREATE TABLE t1(c1 integer);
CREATE TABLE
```

## вставьте строку со значением c1=1

```bash
testdb=# INSERT INTO t1 values(1);
INSERT 0 1
```

## создайте новую роль readonly

```bash
testdb=# CREATE ROLE readonly;
CREATE ROLE
```
## дайте новой роли право на подключение к базе данных testdb

```bash
testdb=# grant connect on database testdb to readonly;
GRANT
```

## дайте новой роли право на использование схемы testnm

```bash
testdb=# grant usage on schema testnm to readonly;
GRANT
```

## дайте новой роли право на select для всех таблиц схемы testnm

```bash
testdb=# grant select on all tables in schema testnm to readonly;
GRANT
```

## создайте пользователя testread с паролем test123

```bash
testdb=# create user testread with password 'test123';
CREATE ROLE
```
## дайте роль readonly пользователю testread

```bash
testdb=# grant readonly TO testread;
GRANT ROLE
```
## зайдите под пользователем testread в базу данных testdb

```bash
postgres=# \c testdb testread
Password for user testread:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1), server 13.7 (Ubuntu 13.7-1.pgdg20.04+1))
You are now connected to database "testdb" as user "testread".
testdb=>
```
## сделайте select * from t1;

```bash
testdb=# grant readonly TO testread;
GRANT ROL
