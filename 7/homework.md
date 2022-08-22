# Работа с журналами

## Настройте выполнение контрольной точки раз в 30 секунд.

```bash
postgres=# ALTER SYSTEM SET checkpoint_timeout = 30;
ALTER SYSTEM
postgres=# show checkpoint_timeout;
 checkpoint_timeout
--------------------
 5min
(1 row)

postgres=# \q
admekb@postgres:~$ sudo -u postgres pg_ctlcluster 14 main stop
admekb@postgres:~$ sudo -u postgres pg_ctlcluster 14 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@14-main
admekb@postgres:~$ sudo -u postgres psql
psql (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
Type "help" for help.

postgres=# show checkpoint_timeout;
 checkpoint_timeout
--------------------
 30s
(1 row)

postgres=#
```

## Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.

```bash
postgres=# CREATE TABLE accounts(
postgres(#   acc_no integer PRIMARY KEY,
postgres(#   amount numeric
postgres(# );
CREATE TABLE
postgres=# INSERT INTO accounts
postgres-#   VALUES (1,1000.00), (2,2000.00), (3,3000.00);
INSERT 0 3
postgres=# select * from accounts;
 acc_no | amount
--------+---------
      1 | 1000.00
      2 | 2000.00
      3 | 3000.00
(3 rows)

postgres=# BEGIN;
BEGIN
postgres=*# UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;
UPDATE 1
postgres=*# SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for
postgres-*# FROM pg_locks WHERE relation = 'accounts'::regclass;
 locktype |       mode       | granted |   pid   | wait_for
----------+------------------+---------+---------+-----------
 relation | RowExclusiveLock | t       | 3379184 | {}
 relation | RowExclusiveLock | t       | 3390345 | {3390060}
 relation | RowExclusiveLock | t       | 3390060 | {3379184}
 tuple    | ExclusiveLock    | t       | 3390060 | {3379184}
 tuple    | ExclusiveLock    | f       | 3390345 | {3390060}
(5 rows)

postgres=*# COMMIT;
COMMIT
postgres=# SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for
FROM pg_locks WHERE relation = 'accounts'::regclass;
 locktype |       mode       | granted |   pid   | wait_for
----------+------------------+---------+---------+-----------
 relation | RowExclusiveLock | t       | 3390345 | {3390060}
 relation | RowExclusiveLock | t       | 3390060 | {}
(2 rows)

postgres=# SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for
FROM pg_locks WHERE relation = 'accounts'::regclass;
 locktype |       mode       | granted |   pid   | wait_for
----------+------------------+---------+---------+----------
 relation | RowExclusiveLock | t       | 3390345 | {}
(1 row)

postgres=# SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for
FROM pg_locks WHERE relation = 'accounts'::regclass;
 locktype | mode | granted | pid | wait_for
----------+------+---------+-----+----------
(0 rows)

```
>Еще в двух окнах был запущен UPDATE и постепенно выполнялся COMMIT.
>Здесь мы видимо экслюзивную блокировку строки первой транзакции 3379184.
>Транзакцию 3390060 эксклюзивной блокировки стройки ожидающую завершение транзакции 3379184.
>Экслюзивную блокировку самой транзакции 3390060.
>Транзакцию 3390345 эксклюзивной блокировки строки ожидающую завершение транзакции 3390060 без доступа, потому есть вторая транзакция которая ожидает UPDATE.
>Экслюзивную блокировку самой транзакции 3390345.

## Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

```bash
2022-08-09 13:04:27.193 UTC [3390060] postgres@postgres LOG:  process 3390060 acquired ShareLock on transaction 2426128 after 2959399.613 ms
2022-08-09 13:04:27.193 UTC [3390060] postgres@postgres CONTEXT:  while updating tuple (0,6) in relation "accounts"
2022-08-09 13:04:27.193 UTC [3390060] postgres@postgres STATEMENT:  UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;
2022-08-09 13:04:27.193 UTC [3390345] postgres@postgres LOG:  process 3390345 acquired ExclusiveLock on tuple (0,6) of relation 17311 of database 13414 after 2892719.279 ms
2022-08-09 13:04:27.193 UTC [3390345] postgres@postgres STATEMENT:  UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;
2022-08-09 13:04:27.393 UTC [3390345] postgres@postgres LOG:  process 3390345 still waiting for ShareLock on transaction 2426129 after 200.231 ms
2022-08-09 13:04:27.393 UTC [3390345] postgres@postgres DETAIL:  Process holding the lock: 3390060. Wait queue: 3390345.
2022-08-09 13:04:27.393 UTC [3390345] postgres@postgres CONTEXT:  while rechecking updated tuple (0,7) in relation "accounts"
2022-08-09 13:04:27.393 UTC [3390345] postgres@postgres STATEMENT:  UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;
2022-08-09 13:04:35.003 UTC [3390345] postgres@postgres LOG:  process 3390345 acquired ShareLock on transaction 2426129 after 7809.647 ms
2022-08-09 13:04:35.003 UTC [3390345] postgres@postgres CONTEXT:  while rechecking updated tuple (0,7) in relation "accounts"
2022-08-09 13:04:35.003 UTC [3390345] postgres@postgres STATEMENT:  UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;
```
>Изучая журнал четко видно что были блокировки с ожиданием больше 200mc и видно содержимое самих запросов, три запроса UPDATE c номерами транзакций и какая транзация какую ожидала.
