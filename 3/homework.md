
# Установка и настройка PostgreSQL

## поставьте на нее PostgreSQL 14 через sudo apt

```bash
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14
```

## проверьте что кластер запущен через sudo -u postgres pg_lsclusters

```bash
admekb@postgres:~$ sudo pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```

## зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым postgres=# create table test(c1 text); postgres=# insert into test values('1'); \q

```bash
postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
postgres=#  \q
```

## остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop

```bash
admekb@postgres:~$ sudo pg_ctlcluster 14 main stop
admekb@postgres:~$ sudo pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```

## Добавление диска

```bash
root@postgres:~# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   1   0 wz--n- <30.50g 15.25g
root@postgres:~# vgextend ubuntu-vg /dev/sdb
  Physical volume "/dev/sdb" successfully created.
  Volume group "ubuntu-vg" successfully extended
root@postgres:~# vgs
  VG        #PV #LV #SN Attr   VSize  VFree
  ubuntu-vg   2   1   0 wz--n- 40.49g <25.25g
root@postgres:~# mkdir /mnt/data
root@postgres:~# lvcreate -n postgres -L 10G ubuntu-vg
  Logical volume "postgres" created.
root@postgres:~# sudo mkfs.ext4 /dev/ubuntu-vg/postgres
mke2fs 1.45.5 (07-Jan-2020)
Discarding device blocks: done
Creating filesystem with 2621440 4k blocks and 655360 inodes
Filesystem UUID: f806a84e-fcc0-485d-b896-f2edd661508f
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
root@postgres:~# cat >> /dev/ubuntu-vg/postgres /mnt/data ext4 defaults 0 1
root@postgres:~# mount -all
root@postgres:~# df -h|grep mnt
/dev/mapper/ubuntu--vg-postgres    9.8G   37M  9.3G   1% /mnt/data

```
> смотрим текущию vg, добавляем в нее диск /dev/sdb, убеждаемся что размер vg увеличился, создаем логический том postgres, размечаем его, пописываем в fstab, монтируем

## сделайте пользователя postgres владельцем /mnt/data

```bash
root@postgres:~# chown -R postgres:postgres /mnt/data/
```

## перенесите содержимое /var/lib/postgres/14 в /mnt/data

```bash
root@postgres:~# mv /var/lib/postgresql/14 /mnt/data
```

## попытайтесь запустить кластер

```bash
root@postgres:~# sudo -u postgres pg_ctlcluster 14 main start
Error: /var/lib/postgresql/14/main is not accessible or does not exist
```
> кластер не запускается, потому что каталога с бд по пути который указан в конфигурационных файлах - нет.

## задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его

```bash
admekb@postgres:~$ vi /etc/postgresql/14/main/postgresql.conf
```

## напишите что и почему поменяли

```bash
data_directory = '/mnt/data/14/main/'            # use data in another directory
                                        # (change requires restart)
```
> потому что в предыдущим шагом мы перенесли туда бд

## попытайтесь запустить кластер

```bash
admekb@postgres:~$ sudo -u postgres pg_ctlcluster 14 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@14-main
```
> получилось, потому что я молодец и поправил файл конфигурации на путь в который мы перенесли бд

## зайдите через через psql и проверьте содержимое ранее созданной таблицы

```bash
admekb@postgres:~$ sudo -u postgres psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# select * from test;
 c1
----
 1
(1 row)

postgres=#
