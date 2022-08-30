# Работа с журналами

## Настройте выполнение контрольной точки раз в 30 секунд.

```bash
postgres=# ALTER SYSTEM SET log_checkpoints = on;
ALTER SYSTEM
postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# show log_checkpoints
postgres-# ;
 log_checkpoints
-----------------
 on
(1 row)

postgres=# ALTER SYSTEM SET checkpoint_timeout = 30;
ALTER SYSTEM
postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# show checkpoint_timeout;
 checkpoint_timeout
--------------------
 30s
(1 row)

```

## 10 минут c помощью утилиты pgbench подавайте нагрузку.

```bash
postgres@postgres:~$ pgbench -i postgres
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.08 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.39 s (drop tables 0.00 s, create tables 0.02 s, client-side generate 0.23 s, vacuum 0.07 s, primary keys 0.07 s).
postgres@postgres:~$ pgbench -c50 -P 60 -T 600 -j 2 -U postgres postgres
pgbench (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
starting vacuum...end.
progress: 60.0 s, 297.4 tps, lat 167.427 ms stddev 172.861
progress: 120.0 s, 297.0 tps, lat 168.203 ms stddev 162.474
progress: 180.0 s, 287.6 tps, lat 173.762 ms stddev 180.216
progress: 240.0 s, 298.2 tps, lat 167.744 ms stddev 170.562
progress: 300.0 s, 300.0 tps, lat 166.598 ms stddev 161.212
progress: 360.0 s, 298.6 tps, lat 167.432 ms stddev 164.664
progress: 420.0 s, 293.1 tps, lat 170.526 ms stddev 171.278
progress: 480.0 s, 289.2 tps, lat 172.739 ms stddev 168.482
progress: 540.0 s, 299.0 tps, lat 167.118 ms stddev 171.767
progress: 600.0 s, 297.9 tps, lat 167.975 ms stddev 177.483
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
duration: 600 s
number of transactions actually processed: 177539
latency average = 168.947 ms
latency stddev = 170.191 ms
initial connection time = 79.004 ms
tps = 295.828772 (without initial connection time)

```

## Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

```bash
postgres@postgres:~$ du -sh /var/lib/postgresql/14/main/*|grep pg_wal
33M	/var/lib/postgresql/14/main/pg_wal
postgres@postgres:~$ du -sh /var/lib/postgresql/14/main/*|grep pg_wal
65M	/var/lib/postgresql/14/main/pg_wal
2022-08-22 20:18:17.032 UTC [3290477] LOG:  checkpoint complete: wrote 526 buffers (3.2%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.809 s, sync=0.014 s, total=26.861 s; sync files=14, longest=0.006 s, average=0.001 s; distance=9570 kB, estimate=18165 kB
```
>32мб сгенерировался
>19 мегабайт примерно на одну

## Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?

```bash
2022-08-22 19:58:50.401 UTC [3290477] LOG:  checkpoint complete: wrote 3 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.011 s, sync=0.006 s, total=0.042 s; sync files=2, longest=0.003 s, average=0.003 s; distance=0 kB, estimate=0 kB
2022-08-22 20:05:20.716 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:05:47.071 UTC [3290477] LOG:  checkpoint complete: wrote 1710 buffers (10.4%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.306 s, sync=0.015 s, total=26.355 s; sync files=46, longest=0.006 s, average=0.001 s; distance=12821 kB, estimate=12821 kB
2022-08-22 20:06:50.132 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:07:17.156 UTC [3290477] LOG:  checkpoint complete: wrote 2077 buffers (12.7%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.913 s, sync=0.048 s, total=27.025 s; sync files=23, longest=0.010 s, average=0.003 s; distance=18886 kB, estimate=18886 kB
2022-08-22 20:07:20.158 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:07:47.062 UTC [3290477] LOG:  checkpoint complete: wrote 1906 buffers (11.6%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.816 s, sync=0.025 s, total=26.905 s; sync files=12, longest=0.007 s, average=0.003 s; distance=19964 kB, estimate=19964 kB
2022-08-22 20:07:50.065 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:08:17.061 UTC [3290477] LOG:  checkpoint complete: wrote 1959 buffers (12.0%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.908 s, sync=0.027 s, total=26.997 s; sync files=18, longest=0.010 s, average=0.002 s; distance=19013 kB, estimate=19869 kB
2022-08-22 20:08:20.064 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:08:47.076 UTC [3290477] LOG:  checkpoint complete: wrote 1885 buffers (11.5%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.920 s, sync=0.030 s, total=27.013 s; sync files=10, longest=0.011 s, average=0.003 s; distance=18933 kB, estimate=19775 kB
2022-08-22 20:08:50.077 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:09:17.111 UTC [3290477] LOG:  checkpoint complete: wrote 2002 buffers (12.2%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.924 s, sync=0.028 s, total=27.034 s; sync files=18, longest=0.011 s, average=0.002 s; distance=18580 kB, estimate=19656 kB
2022-08-22 20:09:20.112 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:09:47.124 UTC [3290477] LOG:  checkpoint complete: wrote 1943 buffers (11.9%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.936 s, sync=0.015 s, total=27.013 s; sync files=10, longest=0.011 s, average=0.002 s; distance=18724 kB, estimate=19563 kB
2022-08-22 20:09:50.125 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:10:17.113 UTC [3290477] LOG:  checkpoint complete: wrote 2016 buffers (12.3%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.902 s, sync=0.021 s, total=26.988 s; sync files=16, longest=0.013 s, average=0.002 s; distance=18989 kB, estimate=19505 kB
2022-08-22 20:10:20.115 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:10:47.102 UTC [3290477] LOG:  checkpoint complete: wrote 1946 buffers (11.9%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.903 s, sync=0.021 s, total=26.987 s; sync files=11, longest=0.012 s, average=0.002 s; distance=19157 kB, estimate=19471 kB
2022-08-22 20:10:50.104 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:11:17.089 UTC [3290477] LOG:  checkpoint complete: wrote 2080 buffers (12.7%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.901 s, sync=0.018 s, total=26.985 s; sync files=19, longest=0.006 s, average=0.001 s; distance=19426 kB, estimate=19466 kB
2022-08-22 20:11:20.090 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:11:47.082 UTC [3290477] LOG:  checkpoint complete: wrote 1993 buffers (12.2%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.914 s, sync=0.018 s, total=26.992 s; sync files=10, longest=0.010 s, average=0.002 s; distance=19580 kB, estimate=19580 kB
2022-08-22 20:11:50.085 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:12:17.090 UTC [3290477] LOG:  checkpoint complete: wrote 2123 buffers (13.0%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.902 s, sync=0.027 s, total=27.005 s; sync files=17, longest=0.013 s, average=0.002 s; distance=19260 kB, estimate=19548 kB
2022-08-22 20:12:20.092 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:12:47.097 UTC [3290477] LOG:  checkpoint complete: wrote 1983 buffers (12.1%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.916 s, sync=0.023 s, total=27.005 s; sync files=9, longest=0.011 s, average=0.003 s; distance=19073 kB, estimate=19500 kB
2022-08-22 20:12:50.097 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:13:17.091 UTC [3290477] LOG:  checkpoint complete: wrote 2064 buffers (12.6%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.905 s, sync=0.029 s, total=26.994 s; sync files=14, longest=0.007 s, average=0.003 s; distance=18844 kB, estimate=19434 kB
2022-08-22 20:13:20.094 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:13:47.072 UTC [3290477] LOG:  checkpoint complete: wrote 1974 buffers (12.0%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.923 s, sync=0.013 s, total=26.978 s; sync files=9, longest=0.007 s, average=0.002 s; distance=18950 kB, estimate=19386 kB
2022-08-22 20:13:50.074 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:14:17.074 UTC [3290477] LOG:  checkpoint complete: wrote 2119 buffers (12.9%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.908 s, sync=0.028 s, total=27.000 s; sync files=16, longest=0.010 s, average=0.002 s; distance=18493 kB, estimate=19297 kB
2022-08-22 20:14:20.077 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:14:47.072 UTC [3290477] LOG:  checkpoint complete: wrote 2004 buffers (12.2%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.901 s, sync=0.029 s, total=26.995 s; sync files=9, longest=0.014 s, average=0.004 s; distance=18976 kB, estimate=19265 kB
2022-08-22 20:14:50.075 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:15:17.070 UTC [3290477] LOG:  checkpoint complete: wrote 2154 buffers (13.1%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.912 s, sync=0.023 s, total=26.996 s; sync files=15, longest=0.007 s, average=0.002 s; distance=18847 kB, estimate=19223 kB
2022-08-22 20:15:20.073 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:15:47.056 UTC [3290477] LOG:  checkpoint complete: wrote 1958 buffers (12.0%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.902 s, sync=0.011 s, total=26.984 s; sync files=8, longest=0.010 s, average=0.002 s; distance=18890 kB, estimate=19190 kB
2022-08-22 20:15:50.058 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:16:17.135 UTC [3290477] LOG:  checkpoint complete: wrote 2191 buffers (13.4%); 0 WAL file(s) added, 0 removed, 1 recycled; write=27.010 s, sync=0.020 s, total=27.078 s; sync files=17, longest=0.007 s, average=0.002 s; distance=18769 kB, estimate=19148 kB
2022-08-22 20:16:20.137 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:16:47.111 UTC [3290477] LOG:  checkpoint complete: wrote 2011 buffers (12.3%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.921 s, sync=0.012 s, total=26.974 s; sync files=9, longest=0.007 s, average=0.002 s; distance=18869 kB, estimate=19120 kB
2022-08-22 20:17:50.172 UTC [3290477] LOG:  checkpoint starting: time
2022-08-22 20:18:17.032 UTC [3290477] LOG:  checkpoint complete: wrote 526 buffers (3.2%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.809 s, sync=0.014 s, total=26.861 s; sync files=14, longest=0.006 s, average=0.001 s; distance=9570 kB, estimate=18165 kB
```
> все точно по расписанию, потому что максимальный размер wal файла в 1Гб (в настройках задано) не превышался за 30 сек.

## Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.

```bash
postgres@postgres:~$ pgbench -c50 -P 60 -T 600 -j 2 -U postgres postgres
pgbench (14.5 (Ubuntu 14.5-1.pgdg20.04+1))
starting vacuum...end.
progress: 60.0 s, 2138.7 tps, lat 23.251 ms stddev 23.425
progress: 120.0 s, 2158.7 tps, lat 23.091 ms stddev 22.933
progress: 180.0 s, 2204.1 tps, lat 22.600 ms stddev 22.454
progress: 240.0 s, 2211.6 tps, lat 22.534 ms stddev 22.645
progress: 300.0 s, 2143.0 tps, lat 23.251 ms stddev 23.573
progress: 360.0 s, 2191.1 tps, lat 22.743 ms stddev 23.283
progress: 420.0 s, 2159.3 tps, lat 23.074 ms stddev 22.436
progress: 480.0 s, 2152.3 tps, lat 23.150 ms stddev 22.966
progress: 540.0 s, 2161.6 tps, lat 23.050 ms stddev 22.829
progress: 600.0 s, 2138.2 tps, lat 23.301 ms stddev 22.850
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
duration: 600 s
number of transactions actually processed: 1299577
latency average = 23.003 ms
latency stddev = 22.944 ms
initial connection time = 69.863 ms
tps = 2165.864982 (without initial connection time)
```
>Запись идет через wal_writer_delay, с уменьшенным временем отлика, производительность выше в несколько раз

## Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?

```bash
[root@localhost ~]# su - postgres -c '/usr/pgsql-14/bin/pg_checksums --enable -D "/var/lib/pgsql/14/data/"'
Checksum operation completed
Files scanned:  931
Blocks scanned: 3297
pg_checksums: syncing data directory
pg_checksums: updating control file
Checksums enabled in cluster
[root@localhost ~]# systemctl start postgresql-14
[root@localhost ~]# su - postgres -c 'psql -c "SHOW data_checksums;"'
 data_checksums
----------------
 on
(1 row)

[root@localhost ~]# sudo -u postgres psql
psql (14.5)
Type "help" for help.

postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
postgres=# insert into test values('2');
INSERT 0 1
postgres=# select * from test;
 c1
----
 1
 2
(2 rows)
postgres=# SELECT pg_relation_filepath('test');
 pg_relation_filepath
----------------------
 base/14487/16384
(1 row)
postgres=# \q
[root@localhost ~]# systemctl stop postgresql-14
[root@localhost ~]# echo "test" >> /var/lib/pgsql/14/data/base/14487/16384
[root@localhost ~]# systemctl start postgresql-14
postgres=# select * from test;
 c1
----
 1
 2
(2 строки)
postgres=# SHOW data_checksums;
 data_checksums
----------------
 on
(1 строка)
```
>не хочет у меня ломаться
