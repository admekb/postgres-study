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
testdb=# grant select on all tables in schema testnm to readonly;
GRANT
```
