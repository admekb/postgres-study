# Установка и настройка PostgteSQL в контейнере Docker

## сделать в GCE инстанс с Ubuntu 20.04 или развернуть докер любым удобным способом

```bash
admekb@postgres:~$ cat /etc/*-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.4 LTS"
NAME="Ubuntu"
VERSION="20.04.4 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.4 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

## поставить на нем Docker Engine

```bash
admekb@postgres:~$ curl -fsSL https://get.docker.com -o get-docker.sh
admekb@postgres:~$ sudo sh get-docker.sh
# Executing docker install script, commit: b2e29ef7a9a89840d2333637f7d1900a83e7153f
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
+ sh -c mkdir -p /etc/apt/keyrings && chmod -R 0755 /etc/apt/keyrings
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" | gpg --dearmor --yes -o /etc/apt/keyrings/docker.gpg
+ sh -c chmod a+r /etc/apt/keyrings/docker.gpg
+ sh -c echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu focal stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq --no-install-recommends docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-scan-plugin >/dev/null
+ version_gte 20.10
+ [ -z  ]
+ return 0
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq docker-ce-rootless-extras >/dev/null
+ sh -c docker version
Client: Docker Engine - Community
 Version:           20.10.17
 API version:       1.41
 Go version:        go1.17.11
 Git commit:        100c701
 Built:             Mon Jun  6 23:02:57 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.17
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.17.11
  Git commit:       a89b842
  Built:            Mon Jun  6 23:01:03 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.6
  GitCommit:        10c12954828e7c7c9b6e0ea9b0c02b01407d3ae1
 runc:
  Version:          1.1.2
  GitCommit:        v1.1.2-0-ga916309
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================

```

## сделать каталог /var/lib/postgres

```bash
admekb@postgres:~$ sudo mkdir /var/lib/postgres
```

## развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres

```bash
admekb@postgres:~$ sudo docker network create pg-net
8c9fa734bba8b66e8545e33569e507c6ee2551fa7b80b59f7ffa35862992557c
admekb@postgres:~$ sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
Unable to find image 'postgres:14' locally
14: Pulling from library/postgres
461246efe0a7: Pull complete
8d6943e62c54: Pull complete
558c55f04e35: Pull complete
186be55594a7: Pull complete
f38240981157: Pull complete
e0699dc58a92: Pull complete
066f440c89a6: Pull complete
ce20e6e2a202: Pull complete
c0f13eb40c44: Pull complete
3d7e9b569f81: Pull complete
2ab91678d745: Pull complete
ffc80af02e8a: Pull complete
f3a57056b036: Pull complete
Digest: sha256:3e2eba0a6efbeb396e086c332c5a85be06997d2cf573d34794764625f405df4e
Status: Downloaded newer image for postgres:14
bfe45de79c2763265457d04524e6035318b30d8cb980c6c517d7b843da121c13
```
## развернуть контейнер с клиентом postgres

```bash
admekb@postgres:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres
Password for user postgres:
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=#
```

## подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

```bash
postgres=# CREATE DATABASE study;
CREATE DATABASE
postgres=# \c study
You are now connected to database "study" as user "postgres".
study=# CREATE TABLE test (i serial, amount int);
INSERT INTO test(amount) VALUES (100);
INSERT INTO test(amount) VALUES (500);
SELECT * FROM test;
CREATE TABLE
INSERT 0 1
INSERT 0 1
 i | amount
---+--------
 1 |    100
 2 |    500
(2 rows)
```

## подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/места установки докера

```bash
(base) admekb@MacBook-Air-Aleksandr ~ % sudo docker run -it --rm --name pg-client postgres:14 psql -h 192.168.1.20 -U postgres
Password:
Unable to find image 'postgres:14' locally
14: Pulling from library/postgres
60197a4c18d4: Pull complete
e2fad7962cc3: Pull complete
ac096a3cfeb6: Pull complete
6505ea08ff74: Pull complete
273081b4002f: Pull complete
7e1b3627e162: Pull complete
f56b0e9d7500: Pull complete
6db829696ec4: Pull complete
ea61b4d4558b: Pull complete
682e50b7dd31: Pull complete
07d44ce41d1a: Pull complete
908fe291a87e: Pull complete
62c216491082: Pull complete
Digest: sha256:3e2eba0a6efbeb396e086c332c5a85be06997d2cf573d34794764625f405df4e
Status: Downloaded newer image for postgres:14
Password for user postgres:
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 study     | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(4 rows)

postgres=#
```

## удалить контейнер с сервером

```bash
admekb@postgres:~$ sudo docker rm -f pg-docker
[sudo] password for admekb:
pg-docker
```

## создать его заново

```bash
admekb@postgres:~$ sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
30718515fc8b0de0d021ba7d8b1555c1f0c83f647bd51405867d867aabf3386d
```

## подключится снова из контейнера с клиентом к контейнеру с сервером

```bash
admekb@postgres:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres
Password for user postgres:
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=#
```

## проверить, что данные остались на месте

```bash
postgres=# \c study
You are now connected to database "study" as user "postgres".
study=# select * from test;
 i | amount
---+--------
 1 |    100
 2 |    500
(2 rows)

study=#
```
