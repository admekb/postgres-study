# Настройка postgres

## поставить на неё PostgreSQL 14 любым способом

```console
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14
postgres@admekb:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```

## настроить кластер PostgreSQL 14 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины
> проверим производительность до изменения параметров
```console
postgres@admekb:~$ psql
psql (14.5 (Ubuntu 14.5-2.pgdg22.04+2))
Type "help" for help.

postgres=# create database test;
CREATE DATABASE
postgres=# alter user postgres password 'postgres';
ALTER ROLE
postgres-# \q
postgres@admekb:~$ pgbench -i test
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.05 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.15 s (drop tables 0.00 s, create tables 0.00 s, client-side generate 0.08 s, vacuum 0.04 s, primary keys 0.02 s).
postgres@admekb:~$ pgbench -c 50 -C -j 2 -P 10 -T 60 -M extended test
pgbench (14.5 (Ubuntu 14.5-2.pgdg22.04+2))
starting vacuum...end.
progress: 10.0 s, 318.2 tps, lat 149.553 ms stddev 171.096
progress: 20.0 s, 305.5 tps, lat 160.355 ms stddev 159.230
progress: 30.0 s, 311.8 tps, lat 156.803 ms stddev 171.501
progress: 40.0 s, 312.8 tps, lat 155.865 ms stddev 156.706
progress: 50.0 s, 300.8 tps, lat 161.420 ms stddev 153.768
progress: 60.0 s, 317.9 tps, lat 152.947 ms stddev 157.169
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: extended
number of clients: 50
number of threads: 2
duration: 60 s
number of transactions actually processed: 18721
latency average = 156.261 ms
latency stddev = 162.084 ms
average connection time = 4.017 ms
tps = 311.634317 (including reconnection times)
```

> подбираем оптимальные параметры через pgtune
* max_connections = 100
* shared_buffers = 1GB
* effective_cache_size = 3GB
* maintenance_work_mem = 256MB
* checkpoint_completion_target = 0.9
* wal_buffers = 16MB
* default_statistics_target = 100
* random_page_cost = 1.1
* effective_io_concurrency = 200
* work_mem = 5242kB
* min_wal_size = 2GB
* max_wal_size = 8GB

> проводим тестирование и видимо что особо ничего не изменилось (я так подозреваю что данных очень мало для того чтобы заметить разницу с данными параметрами)
```console
root@admekb:~# pg_ctlcluster 14 main restart
root@admekb:~# su - postgres
postgres@admekb:~$ pgbench -c 50 -C -j 2 -P 10 -T 60 -M extended test
pgbench (14.5 (Ubuntu 14.5-2.pgdg22.04+2))
starting vacuum...end.
progress: 10.0 s, 309.2 tps, lat 155.222 ms stddev 156.909
progress: 20.0 s, 309.9 tps, lat 156.562 ms stddev 166.604
progress: 30.0 s, 310.8 tps, lat 157.005 ms stddev 164.674
progress: 40.0 s, 304.5 tps, lat 158.976 ms stddev 192.776
progress: 50.0 s, 306.3 tps, lat 157.868 ms stddev 189.765
progress: 60.0 s, 296.1 tps, lat 165.637 ms stddev 193.361
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: extended
number of clients: 50
number of threads: 2
duration: 60 s
number of transactions actually processed: 18420
latency average = 158.820 ms
latency stddev = 178.539 ms
average connection time = 4.100 ms
tps = 306.549536 (including reconnection times)
postgres@admekb:~$ psql
psql (14.5 (Ubuntu 14.5-2.pgdg22.04+2))
Type "help" for help.
```
> проводим тестирование выключив синхронный коммит (приняв риск потери данных) и видимо небольшой прирост
```console
postgres=# alter system set synchronous_commit='off';
ALTER SYSTEM
postgres=# exit
postgres@admekb:~$ exit
logout
root@admekb:~# pg_ctlcluster 14 main restart
root@admekb:~# su - postgres
postgres@admekb:~$ pgbench -c 50 -C -j 2 -P 10 -T 60 -M extended test
pgbench (14.5 (Ubuntu 14.5-2.pgdg22.04+2))
starting vacuum...end.
progress: 10.0 s, 316.7 tps, lat 151.232 ms stddev 163.197
progress: 20.0 s, 322.1 tps, lat 150.648 ms stddev 152.974
progress: 30.0 s, 320.8 tps, lat 151.410 ms stddev 151.729
progress: 40.0 s, 318.7 tps, lat 153.166 ms stddev 152.437
progress: 50.0 s, 320.7 tps, lat 152.465 ms stddev 158.941
progress: 60.0 s, 319.2 tps, lat 151.751 ms stddev 157.095
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: extended
number of clients: 50
number of threads: 2
duration: 60 s
number of transactions actually processed: 19232
latency average = 151.840 ms
latency stddev = 156.111 ms
average connection time = 4.160 ms
tps = 320.205704 (including reconnection times)
```
