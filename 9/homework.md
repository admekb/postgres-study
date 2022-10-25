# Виды и устройство репликации в Postgres

## изменяем тип репликации на логическую на первом сервере, создаем бд, задаем пароль пользователю, перезагружаем кластер (повторяем данные процедуры еще на двух серверах)

```console
postgres=# show wal_level;
 wal_level
-----------
 replica
(1 row)

postgres=# ALTER SYSTEM SET wal_level = logical;
ALTER SYSTEM
postgres=# create database test;
CREATE DATABASE
postgres=# alter user postgres password 'postgres';
ALTER ROLE
postgres=# \q
root@admekb:~# pg_ctlcluster 14 main restart
root@admekb:~# sudo -u postgres psql
psql (14.5 (Ubuntu 14.5-2.pgdg22.04+2))
Type "help" for help.

postgres=# show wal_level;
 wal_level
-----------
 logical
(1 row)
```

## на первом сервере подключаемся к бд test, создаем таблицу test, наполяем ее данными, создаем публикацию этой таблицы

```console
postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# CREATE TABLE test (acc_no integer PRIMARY KEY, amount numeric);
CREATE TABLE
test=# INSERT INTO test VALUES (1,1000.00), (2,2000.00), (3,3000.00);
INSERT 0 3
test=# CREATE PUBLICATION test_pub FOR TABLE test;
CREATE PUBLICATION
test=# \dRp+
                            Publication test_pub
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test"
```

## на втором сервере подключаемся к бд test, создаем таблицу test, создаем подписку на таблицу test

```console
ostgres=# \c test
You are now connected to database "test" as user "postgres".
test=# CREATE TABLE test (acc_no integer PRIMARY KEY, amount numeric);
CREATE TABLE
test=# select * from test;
 acc_no | amount
--------+--------
(0 rows)

test=# CREATE SUBSCRIPTION test_sub CONNECTION 'host=172.16.60.136 port=5432 user=postgres password=postgres dbname=test' PUBLICATION test_pub WITH (copy_data = true);
NOTICE:  created replication slot "test_sub" on publisher
CREATE SUBSCRIPTION
test=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 16392
subname               | test_sub
pid                   | 4298
relid                 |
received_lsn          | 0/9BCDC38
last_msg_send_time    | 2022-10-25 19:58:25.625718+00
last_msg_receipt_time | 2022-10-25 19:58:25.614332+00
latest_end_lsn        | 0/9BCDC38
latest_end_time       | 2022-10-25 19:58:25.625718+00

test=# select * from test;
 acc_no | amount
--------+---------
      1 | 1000.00
      2 | 2000.00
      3 | 3000.00
(3 rows)
```
> проверим на первом сервере статус
```console
test=# SELECT * FROM pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 2381
usesysid         | 10
usename          | postgres
application_name | test_sub
client_addr      | 172.16.60.137
client_hostname  |
client_port      | 41784
backend_start    | 2022-10-25 19:57:25.503084+00
backend_xmin     |
state            | streaming
sent_lsn         | 0/9BCDC38
write_lsn        | 0/9BCDC38
flush_lsn        | 0/9BCDC38
replay_lsn       | 0/9BCDC38
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2022-10-25 20:01:45.848971+00
```
## на третьем сервере подключаемся к бд test, создаем таблицу test, создаем подписку на таблицу test

```console
postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# CREATE TABLE test (acc_no integer PRIMARY KEY, amount numeric);
CREATE TABLE
test=# CREATE SUBSCRIPTION test_sub_s3 CONNECTION 'host=172.16.60.136 port=5432 user=postgres password=postgres dbname=test' PUBLICATION test_pub WITH (copy_data = true);
NOTICE:  created replication slot "test_sub_s3" on publisher
CREATE SUBSCRIPTION
test=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 16393
subname               | test_sub_s3
pid                   | 1888
relid                 |
received_lsn          | 0/9BCDCA8
last_msg_send_time    | 2022-10-25 20:22:05.776895+00
last_msg_receipt_time | 2022-10-25 20:22:05.77294+00
latest_end_lsn        | 0/9BCDCA8
latest_end_time       | 2022-10-25 20:22:05.776895+00

test=# select * from test;
 acc_no | amount
--------+---------
      1 | 1000.00
      2 | 2000.00
      3 | 3000.00
(3 rows)
```

## на втором сервере подключаемся к бд test, создаем таблицу test2, создаем публикацию таблицы test2

```console
test=# CREATE TABLE test2 (acc_no integer PRIMARY KEY, amount numeric);
CREATE TABLE
test=# INSERT INTO test2 VALUES (50,70.00), (60,80.00), (70,90.00);
INSERT 0 3
test=# CREATE PUBLICATION test_pub_ser2 FOR TABLE test2;
CREATE PUBLICATION
test=# \dRp+
                         Publication test_pub_ser2
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test2"
```
## на первом сервере подключаемся к бд test, создаем таблицу test2, создаем подписку на таблицу test2

```console
postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# CREATE TABLE test2 (acc_no integer PRIMARY KEY, amount numeric);
CREATE TABLE
test=# CREATE SUBSCRIPTION test_sub_sr2_1 CONNECTION 'host=172.16.60.137 port=5432 user=postgres password=postgres dbname=test' PUBLICATION test_pub_ser2 WITH (copy_data = true);
NOTICE:  created replication slot "test_sub_sr2_1" on publisher
CREATE SUBSCRIPTION
test=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 16428
subname               | test_sub_sr2_1
pid                   | 2627
relid                 |
received_lsn          | 0/1747E88
last_msg_send_time    | 2022-10-25 20:48:18.711079+00
last_msg_receipt_time | 2022-10-25 20:48:18.729293+00
latest_end_lsn        | 0/1747E88
latest_end_time       | 2022-10-25 20:48:18.711079+00

test=# select * from test2;
 acc_no | amount
--------+--------
     50 |  70.00
     60 |  80.00
     70 |  90.00
(3 rows)
```

## на третьем сервере подключаемся к бд test, создаем таблицу test2, создаем подписку на таблицу test2

```console
test=# CREATE TABLE test2 (acc_no integer PRIMARY KEY, amount numeric);
CREATE TABLE
test=# CREATE SUBSCRIPTION test_sub_sr2_3 CONNECTION 'host=172.16.60.137 port=5432 user=postgres password=postgres dbname=test' PUBLICATION test_pub_ser2 WITH (copy_data = true);
NOTICE:  created replication slot "test_sub_sr2_3" on publisher
CREATE SUBSCRIPTION
test=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 16393
subname               | test_sub_s3
pid                   | 1888
relid                 |
received_lsn          | 0/9BEF130
last_msg_send_time    | 2022-10-25 20:48:39.154428+00
last_msg_receipt_time | 2022-10-25 20:48:39.151551+00
latest_end_lsn        | 0/9BEF130
latest_end_time       | 2022-10-25 20:48:39.154428+00
-[ RECORD 2 ]---------+------------------------------
subid                 | 16401
subname               | test_sub_sr2_3
pid                   | 1990
relid                 |
received_lsn          | 0/1747E88
last_msg_send_time    | 2022-10-25 20:48:18.717209+00
last_msg_receipt_time | 2022-10-25 20:48:18.732622+00
latest_end_lsn        | 0/1747E88
latest_end_time       | 2022-10-25 20:48:18.717209+00

test=# select * from test2;
 acc_no | amount
--------+--------
     50 |  70.00
     60 |  80.00
     70 |  90.00
(3 rows)
```
*в итоге получили что и хотели, а именно обмен данными между разными таблицами на двух серверах, содержимое которых можно посмотреть на любом из серверов. А так же сервер который будет хранить как резервную копию содержимое этих таблиц. Либо использовать для отчетов, чтобы не нагружать основные сервера*
