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

## выполнить pgbench -i postgres

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
done in 0.36 s (drop tables 0.00 s, create tables 0.02 s, client-side generate 0.20 s, vacuum 0.08 s, primary keys 0.07 s).
```

## запустить pgbench -c8 -P 60 -T 3600 -U postgres postgres

```bash
postgres@postgres:~$ pgbench -c8 -P 60 -T 3600 -U postgres postgres
starting vacuum...end.
progress: 60.0 s, 296.8 tps, lat 26.916 ms stddev 16.995
progress: 120.0 s, 306.4 tps, lat 26.085 ms stddev 16.310
progress: 180.0 s, 307.3 tps, lat 26.013 ms stddev 15.995
progress: 240.0 s, 307.7 tps, lat 25.979 ms stddev 15.812
progress: 300.0 s, 307.3 tps, lat 26.010 ms stddev 15.838
progress: 360.0 s, 296.7 tps, lat 26.943 ms stddev 16.715
progress: 420.0 s, 306.0 tps, lat 26.125 ms stddev 16.105
progress: 480.0 s, 307.3 tps, lat 26.014 ms stddev 15.987
progress: 540.0 s, 307.1 tps, lat 26.029 ms stddev 15.918
progress: 600.0 s, 306.7 tps, lat 26.063 ms stddev 15.845
progress: 660.0 s, 297.7 tps, lat 26.851 ms stddev 16.291
progress: 720.0 s, 306.9 tps, lat 26.048 ms stddev 15.609
progress: 780.0 s, 308.0 tps, lat 25.952 ms stddev 15.393
progress: 840.0 s, 307.5 tps, lat 25.991 ms stddev 15.909
progress: 900.0 s, 308.0 tps, lat 25.955 ms stddev 15.432
progress: 960.0 s, 297.0 tps, lat 26.920 ms stddev 16.692
progress: 1020.0 s, 306.0 tps, lat 26.120 ms stddev 16.056
progress: 1080.0 s, 308.5 tps, lat 25.914 ms stddev 15.374
progress: 1140.0 s, 308.4 tps, lat 25.922 ms stddev 15.382
progress: 1200.0 s, 307.9 tps, lat 25.965 ms stddev 15.678
progress: 1260.0 s, 297.2 tps, lat 26.898 ms stddev 16.176
progress: 1320.0 s, 307.0 tps, lat 26.037 ms stddev 15.569
progress: 1380.0 s, 306.9 tps, lat 26.042 ms stddev 15.947
progress: 1440.0 s, 307.5 tps, lat 26.001 ms stddev 15.492
progress: 1500.0 s, 307.6 tps, lat 25.989 ms stddev 15.927
progress: 1560.0 s, 297.3 tps, lat 26.891 ms stddev 16.728
progress: 1620.0 s, 307.0 tps, lat 26.044 ms stddev 15.559
progress: 1680.0 s, 308.2 tps, lat 25.936 ms stddev 15.562
progress: 1740.0 s, 307.8 tps, lat 25.973 ms stddev 15.673
progress: 1800.0 s, 308.0 tps, lat 25.959 ms stddev 15.553
progress: 1860.0 s, 298.7 tps, lat 26.769 ms stddev 16.120
progress: 1920.0 s, 307.1 tps, lat 26.032 ms stddev 15.357
progress: 1980.0 s, 307.8 tps, lat 25.975 ms stddev 15.789
progress: 2040.0 s, 307.7 tps, lat 25.983 ms stddev 15.776
progress: 2100.0 s, 307.9 tps, lat 25.965 ms stddev 16.065
progress: 2160.0 s, 297.0 tps, lat 26.922 ms stddev 16.515
progress: 2220.0 s, 306.8 tps, lat 26.054 ms stddev 16.020
progress: 2280.0 s, 308.4 tps, lat 25.925 ms stddev 15.684
progress: 2340.0 s, 294.5 tps, lat 27.150 ms stddev 18.712
progress: 2400.0 s, 308.3 tps, lat 25.924 ms stddev 15.544
progress: 2460.0 s, 297.4 tps, lat 26.876 ms stddev 16.466
progress: 2520.0 s, 307.0 tps, lat 26.038 ms stddev 15.965
progress: 2580.0 s, 308.1 tps, lat 25.952 ms stddev 15.665
progress: 2640.0 s, 307.8 tps, lat 25.970 ms stddev 15.803
progress: 2700.0 s, 308.2 tps, lat 25.933 ms stddev 15.624
progress: 2760.0 s, 297.0 tps, lat 26.918 ms stddev 16.353
progress: 2820.0 s, 306.7 tps, lat 26.061 ms stddev 15.358
progress: 2880.0 s, 308.1 tps, lat 25.950 ms stddev 15.695
progress: 2940.0 s, 307.9 tps, lat 25.967 ms stddev 15.762
progress: 3000.0 s, 308.4 tps, lat 25.921 ms stddev 15.757
progress: 3060.0 s, 296.7 tps, lat 26.939 ms stddev 16.774
progress: 3120.0 s, 306.9 tps, lat 26.051 ms stddev 15.910
progress: 3180.0 s, 307.0 tps, lat 26.034 ms stddev 15.735
progress: 3240.0 s, 307.9 tps, lat 25.968 ms stddev 15.693
progress: 3300.0 s, 308.1 tps, lat 25.947 ms stddev 15.956
progress: 3360.0 s, 296.7 tps, lat 26.938 ms stddev 16.419
progress: 3420.0 s, 306.4 tps, lat 26.091 ms stddev 15.799
progress: 3480.0 s, 307.9 tps, lat 25.967 ms stddev 15.842
progress: 3540.0 s, 307.8 tps, lat 25.969 ms stddev 15.533
progress: 3600.0 s, 307.5 tps, lat 25.998 ms stddev 15.404
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 3600 s
number of transactions actually processed: 1098810
latency average = 26.191 ms
latency stddev = 15.951 ms
tps = 305.222317 (including connections establishing)
tps = 305.222502 (excluding connections establishing)
```

## а дальше настроить autovacuum максимально эффективно

```bash
postgres-# FROM pg_settings WHERE name like 'autovacuum%';
                 name                  |  setting  |  context   |                                        short_desc
---------------------------------------+-----------+------------+-------------------------------------------------------------------------------------------
 autovacuum                            | on        | sighup     | Starts the autovacuum subprocess.
 autovacuum_analyze_scale_factor       | 0.1       | sighup     | Number of tuple inserts, updates, or deletes prior to analyze as a fraction of reltuples.
 autovacuum_analyze_threshold          | 50        | sighup     | Minimum number of tuple inserts, updates, or deletes prior to analyze.
 autovacuum_freeze_max_age             | 200000000 | postmaster | Age at which to autovacuum a table to prevent transaction ID wraparound.
 autovacuum_max_workers                | 3         | postmaster | Sets the maximum number of simultaneously running autovacuum worker processes.
 autovacuum_multixact_freeze_max_age   | 400000000 | postmaster | Multixact age at which to autovacuum a table to prevent multixact wraparound.
 autovacuum_naptime                    | 10        | sighup     | Time to sleep between autovacuum runs.
 autovacuum_vacuum_cost_delay          | 10        | sighup     | Vacuum cost delay in milliseconds, for autovacuum.
 autovacuum_vacuum_cost_limit          | 1000      | sighup     | Vacuum cost amount available before napping, for autovacuum.
 autovacuum_vacuum_insert_scale_factor | 0.2       | sighup     | Number of tuple inserts prior to vacuum as a fraction of reltuples.
 autovacuum_vacuum_insert_threshold    | 1000      | sighup     | Minimum number of tuple inserts prior to vacuum, or -1 to disable insert vacuums.
 autovacuum_vacuum_scale_factor        | 0.05      | sighup     | Number of tuple updates or deletes prior to vacuum as a fraction of reltuples.
 autovacuum_vacuum_threshold           | 20        | sighup     | Minimum number of tuple updates or deletes prior to vacuum.
 autovacuum_work_mem                   | -1        | sighup     | Sets the maximum memory to be used by each autovacuum worker process.
(14 rows)

```

## так чтобы получить максимально ровное значение tps на горизонте часа

```bash
postgres@postgres:~$ pgbench -c8 -P 60 -T 3600 -U postgres postgres
starting vacuum...end.
progress: 60.0 s, 297.5 tps, lat 26.859 ms stddev 16.477
progress: 120.0 s, 287.0 tps, lat 27.855 ms stddev 18.235
progress: 180.0 s, 306.2 tps, lat 26.110 ms stddev 15.925
progress: 240.0 s, 307.9 tps, lat 25.963 ms stddev 15.773
progress: 300.0 s, 308.1 tps, lat 25.945 ms stddev 16.003
progress: 360.0 s, 297.9 tps, lat 26.839 ms stddev 16.453
progress: 420.0 s, 307.6 tps, lat 25.991 ms stddev 15.980
progress: 480.0 s, 306.2 tps, lat 26.105 ms stddev 15.716
progress: 540.0 s, 307.8 tps, lat 25.972 ms stddev 15.748
progress: 600.0 s, 307.7 tps, lat 25.973 ms stddev 15.440
progress: 660.0 s, 297.4 tps, lat 26.887 ms stddev 16.722
progress: 720.0 s, 307.5 tps, lat 25.995 ms stddev 16.195
progress: 780.0 s, 306.9 tps, lat 26.050 ms stddev 15.611
progress: 840.0 s, 307.9 tps, lat 25.963 ms stddev 15.885
progress: 900.0 s, 307.4 tps, lat 26.005 ms stddev 15.907
progress: 960.0 s, 297.2 tps, lat 26.906 ms stddev 16.728
progress: 1020.0 s, 307.4 tps, lat 26.006 ms stddev 16.244
progress: 1080.0 s, 306.6 tps, lat 26.070 ms stddev 16.012
progress: 1140.0 s, 308.0 tps, lat 25.958 ms stddev 15.674
progress: 1200.0 s, 307.9 tps, lat 25.957 ms stddev 15.982
progress: 1260.0 s, 297.3 tps, lat 26.893 ms stddev 16.702
progress: 1320.0 s, 308.3 tps, lat 25.935 ms stddev 15.956
progress: 1380.0 s, 307.2 tps, lat 26.018 ms stddev 15.724
progress: 1440.0 s, 307.9 tps, lat 25.959 ms stddev 16.071
progress: 1500.0 s, 307.6 tps, lat 25.987 ms stddev 15.726
progress: 1560.0 s, 297.7 tps, lat 26.855 ms stddev 16.814
progress: 1620.0 s, 307.8 tps, lat 25.968 ms stddev 15.960
progress: 1680.0 s, 306.1 tps, lat 26.118 ms stddev 15.902
progress: 1740.0 s, 307.8 tps, lat 25.975 ms stddev 15.937
progress: 1800.0 s, 308.0 tps, lat 25.955 ms stddev 15.712
progress: 1860.0 s, 298.0 tps, lat 26.824 ms stddev 16.657
progress: 1920.0 s, 308.7 tps, lat 25.902 ms stddev 15.693
progress: 1980.0 s, 306.1 tps, lat 26.118 ms stddev 16.024
progress: 2040.0 s, 308.4 tps, lat 25.917 ms stddev 15.743
progress: 2100.0 s, 307.6 tps, lat 25.985 ms stddev 15.855
progress: 2160.0 s, 298.0 tps, lat 26.821 ms stddev 16.481
progress: 2220.0 s, 308.7 tps, lat 25.906 ms stddev 15.818
progress: 2280.0 s, 306.5 tps, lat 26.083 ms stddev 16.066
progress: 2340.0 s, 308.0 tps, lat 25.956 ms stddev 15.531
progress: 2400.0 s, 307.8 tps, lat 25.971 ms stddev 15.936
progress: 2460.0 s, 298.0 tps, lat 26.826 ms stddev 16.791
progress: 2520.0 s, 307.7 tps, lat 25.980 ms stddev 15.490
progress: 2580.0 s, 306.9 tps, lat 26.043 ms stddev 15.953
progress: 2640.0 s, 307.9 tps, lat 25.974 ms stddev 15.617
progress: 2700.0 s, 308.2 tps, lat 25.937 ms stddev 16.018
progress: 2760.0 s, 297.1 tps, lat 26.907 ms stddev 16.528
progress: 2820.0 s, 307.7 tps, lat 25.984 ms stddev 15.642
progress: 2880.0 s, 307.8 tps, lat 25.973 ms stddev 15.912
progress: 2940.0 s, 307.5 tps, lat 25.994 ms stddev 15.930
progress: 3000.0 s, 308.1 tps, lat 25.941 ms stddev 16.053
progress: 3060.0 s, 298.3 tps, lat 26.809 ms stddev 16.517
progress: 3120.0 s, 308.7 tps, lat 25.897 ms stddev 15.747
progress: 3180.0 s, 306.9 tps, lat 26.043 ms stddev 15.708
progress: 3240.0 s, 308.0 tps, lat 25.955 ms stddev 15.968
progress: 3300.0 s, 308.4 tps, lat 25.919 ms stddev 15.927
progress: 3360.0 s, 297.3 tps, lat 26.888 ms stddev 16.764
progress: 3420.0 s, 309.4 tps, lat 25.828 ms stddev 16.103
progress: 3480.0 s, 306.9 tps, lat 26.056 ms stddev 15.769
progress: 3540.0 s, 308.0 tps, lat 25.949 ms stddev 15.964
progress: 3600.0 s, 308.2 tps, lat 25.940 ms stddev 16.008
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 3600 s
number of transactions actually processed: 1099116
latency average = 26.184 ms
latency stddev = 16.062 ms
tps = 305.306974 (including connections establishing)
tps = 305.307162 (excluding connections establishing)
```
> как мы видим производительность даже после изменения настроек остается максимально эффективной и ровной. Я считаю что это связано с очень высокой производительностью жесткого диска для данного вида тестирования(одна бд с небольшим количеством данных)
