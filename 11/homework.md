## Секционировать большую таблицу из демо базы flights

* создаем партиционированную таблицу аналогичной структуры таблицы, которую хотим секционировать

```sql
CREATE TABLE bookings_part (
  "book_ref" char(6) COLLATE "pg_catalog"."default" NOT NULL,
  "book_date" timestamptz(6) NOT NULL,
  "total_amount" numeric(10,2) NOT NULL
) partition by range (book_date);
```
* создаем партиции декларативным способом по месячно за последний год (который есть в демо бд) и делаем вставку из таблицы которую партиционируем

```sql
create table bookings_2017_08 partition of bookings_part for values from (date'2017-08-01') to (date'2017-09-01');
insert into bookings_2017_08 select * from bookings where book_date between date'2017-08-01' and date'2017-09-01' -1;

create table bookings_2017_07 partition of bookings_part for values from (date'2017-07-01') to (date'2017-08-01');
insert into bookings_2017_07 select * from bookings where book_date between date'2017-07-01' and date'2017-08-01' -1;

create table bookings_2017_06 partition of bookings_part for values from (date'2017-06-01') to (date'2017-07-01');
insert into bookings_2017_06 select * from bookings where book_date between date'2017-06-01' and date'2017-07-01' -1;

create table bookings_2017_05 partition of bookings_part for values from (date'2017-05-01') to (date'2017-06-01');
insert into bookings_2017_05 select * from bookings where book_date between date'2017-05-01' and date'2017-06-01' -1;

create table bookings_2017_04 partition of bookings_part for values from (date'2017-04-01') to (date'2017-05-01');
insert into bookings_2017_04 select * from bookings where book_date between date'2017-04-01' and date'2017-05-01' -1;

create table bookings_2017_03 partition of bookings_part for values from (date'2017-03-01') to (date'2017-04-01');
insert into bookings_2017_03 select * from bookings where book_date between date'2017-03-01' and date'2017-04-01' -1;

create table bookings_2017_02 partition of bookings_part for values from (date'2017-02-01') to (date'2017-03-01');
insert into bookings_2017_02 select * from bookings where book_date between date'2017-02-01' and date'2017-03-01' -1;

create table bookings_2017_01 partition of bookings_part for values from ('2017-01-01') to (date'2017-02-01');
insert into bookings_2017_01 select * from bookings where book_date between date'2017-01-01' and date'2017-02-01' -1;
```
* не забываем создать дефолтную партицию, чтобы не потерять данные

```sql
create table default_partition_bookings partition of bookings_part default;
```
* пробуем найти данные за предпоследний месяц
```sql
explain
select * from bookings_part where book_date > '2017-07-30%'
```
```console
Append  (cost=0.00..5286.28 rows=93852 width=21)
  ->  Seq Scan on bookings_2017_07 bookings_part_1  (cost=0.00..3136.90 rows=5731 width=21)
        Filter: (book_date > '2017-07-30 00:00:00+00'::timestamp with time zone)
  ->  Seq Scan on bookings_2017_08 bookings_part_2  (cost=0.00..1657.38 rows=87781 width=21)
        Filter: (book_date > '2017-07-30 00:00:00+00'::timestamp with time zone)
  ->  Seq Scan on default_partition_bookings bookings_part_3  (cost=0.00..22.75 rows=340 width=52)
        Filter: (book_date > '2017-07-30 00:00:00+00'::timestamp with time zone)
```
*план запроса выполнился по трем партиционированным секциям, две их которых за последние месяца и дефолтная*
