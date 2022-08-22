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

## 10 минут c помощью утилиты pgbench подавайте нагрузку.

```bash
postgres=# CREATE TABLE test(i int);
CREATE TABLE
postgres=# INSERT INTO test SELECT s.id FROM generate_series(1,1000000) AS s(id);
INSERT 0 1000000

```

## Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

```bash
postgres@postgres:~/14/main$ du -sh *|grep pg_wal
81M	pg_wal
```
