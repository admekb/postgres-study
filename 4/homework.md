# Работа с базами данных, пользователями и правами

## создайте новый кластер PostgresSQL 13 (на выбор - GCE, CloudSQL)

```bash
Success. You can now start the database server using:

    pg_ctlcluster 13 main start

Processing triggers for postgresql-common (241.pgdg20.04+1) ...
Building PostgreSQL dictionaries from installed myspell/hunspell packages...
Removing obsolete dictionary files:
admekb@postgres:~$ sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 online postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
```

## зайдите в созданный кластер под пользователем postgres

```bash
admekb@postgres:~$ sudo -u postgres psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1), server 13.7 (Ubuntu 13.7-1.pgdg20.04+1))
Type "help" for help.

postgres=# SHOW server_version;
          server_version
----------------------------------
 13.7 (Ubuntu 13.7-1.pgdg20.04+1)
(1 row)

postgres=#
```

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
GRANT ROLE
```
## получилось?; напишите что именно произошло в тексте домашнего задания у вас есть идеи почему?; ведь права то дали?;посмотрите на список таблиц; а почему так получилось с таблицей?
> Нет, потому что все что ранее создали находится в схеме public, т.к. полный путь мы не указывали и не меняли search_path

## вернитесь в базу данных testdb под пользователем postgres

```bash
postgres=# \c testdb
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1), server 13.7 (Ubuntu 13.7-1.pgdg20.04+1))
You are now connected to database "testdb" as user "postgres".
```
## удалите таблицу t1

```bash
testdb=# drop table t1;
DROP TABLE
```
## создайте ее заново но уже с явным указанием имени схемы testnm

```bash
testdb=# create table testnm.t1(c1 integer);
CREATE TABLE
```
## вставьте строку со значением c1=1

```bash
testdb=# insert into testnm.t1 values(1);
INSERT 0 1
```
## зайдите под пользователем testread в базу данных testdb

```bash
postgres=# \c testdb testread
Password for user testread:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1), server 13.7 (Ubuntu 13.7-1.pgdg20.04+1))
You are now connected to database "testdb" as user "testread".
testdb=>
```
## сделайте select * from testnm.t1;

```bash
testdb=> select * from testnm.t1;
ERROR:  permission denied for table t1
```
## получилось?; есть идеи почему? ;как сделать так чтобы такое больше не повторялось?
> Не получилось, оказывается grant select дает права только на существующие таблицы, а мы их только что пересоздали

## как сделать так чтобы такое больше не повторялось?
```bash
testdb=# ALTER default privileges in SCHEMA testnm grant SELECT on TABLEs to readonly;
ALTER DEFAULT PRIVILEGES
```
## сделайте select * from testnm.t1;
```bash
testdb=# select * from testnm.t1;
 c1
----
  1
(1 row)

```
## теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
```bash
testdb=# create table t2(c1 integer); insert into t2 values (2);
CREATE TABLE
INSERT 0 1
```
## а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly
>создали в схеме publiс в которой есть доступ по умолчанию

## есть идеи как убрать эти права?
```bash
postgres=# revoke CREATE on SCHEMA public FROM public;
REVOKE
postgres=# revoke all on DATABASE testdb FROM public;
REVOKE
```
## теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
```bash
Password for user testread:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1), server 13.7 (Ubuntu 13.7-1.pgdg20.04+1))
You are now connected to database "testdb" as user "testread".
testdb=> create table t3(c1 integer); insert into t2 values (2);
CREATE TABLE
ERROR:  permission denied for table t2
testdb=>
```
> отозвали все права с схемы public, а у действующей роли есть права только на select
