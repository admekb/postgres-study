
# Работа с уровнями изоляции транзакции в PostgreSQL

## сделать в первой сессии новую таблицу и наполнить ее данными

```bash
iso=# create table persons(id serial, first_name text, second_name text);
CREATE TABLE
iso=*# insert into persons(first_name, second_name) values('ivan', 'ivanov');
INSERT 0 1
iso=*# insert into persons(first_name, second_name) values('petr', 'petrov');
INSERT 0 1
iso=*# commit;
```

## посмотреть текущий уровень изоляции

```bash
iso=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
```

## в первой сессии добавить новую запись

```bash
iso=*# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
```

## сделать select * from persons во второй сессии

```bash
iso=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
## Клонируем наш репозиторий с git
- на хосте с ansible соответственно

```bash
cd /
git clone git@github.com:crowdtesting-ru/ufrpayroll-devops.git
```

## Создание и добавление jenkins как службы в автозагрузку
- создаем файл службы /etc/systemd/system/jenkins.service с следующим содержимым

```
> далее руками будем его по всем хостам разносить
