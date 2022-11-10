# Сбор и использование статистики

## Создать индекс к какой-либо из таблиц вашей БД
*создаем индекс

```sql
create index book_ref_idx on tickets(book_ref);
```
## Прислать текстом результат команды explain, в которой используется данный индекс
* смотрим план запроса

```sql
explain
select * from tickets where book_ref = '1A40A1';
```
```console
Index Scan using book_ref_idx on tickets  (cost=0.43..12.46 rows=2 width=104)
Index Cond: (book_ref = '1A40A1'::bpchar)
``` 
*видим что запрос использовал сканирование по индексу*

## Реализовать индекс для полнотекстового поиска
* добавляем столбец с типом tsvector для полнотекстового поиска
```sql
alter table tickets add column passenger_name_lexeme tsvector;
```
* заполняем вновь созданный стоблец по полю passenger_name конвектируя его в тип to_tsvector
```sql
update tickets set passenger_name_lexeme = to_tsvector(passenger_name);
``` 
* создаем индекс по созданному полю типа GIN
```sql
CREATE INDEX search_index_passenger_name ON tickets USING GIN (passenger_name_lexeme);
``` 
* пробуем найти всех пассажиров с именем
```sql
explain
select * from tickets where passenger_name_lexeme @@ to_tsquery('ELENA');
``` 
```console
Gather  (cost=1867.62..289828.05 rows=92822 width=140)
  Workers Planned: 2
  ->  Parallel Bitmap Heap Scan on tickets  (cost=867.62..279545.85 rows=38676 width=140)
        Recheck Cond: (passenger_name_lexeme @@ to_tsquery('ELENA'::text))
        ->  Bitmap Index Scan on search_index_passenger_name  (cost=0.00..844.42 rows=92822 width=0)
              Index Cond: (passenger_name_lexeme @@ to_tsquery('ELENA'::text))
```
*видим что запрос использовал сканирование по индексу*

## Реализовать индекс на часть таблицы

* создаем индекс на id больше 70000
```sql
create index idx_id on ticket_flights(flight_id) where flight_id > 70000;  
``` 
* выполним поиск по id равному 89752
```sql
explain
select * from ticket_flights where flight_id = 89752; 
``` 
```console
Index Scan using idx_id on ticket_flights  (cost=0.43..397.25 rows=103 width=32)
  Index Cond: (flight_id = 89752)
```
*видим что запрос использовал сканирование по индексу*

## Создать индекс на несколько полей

* создаем индекс по полям номер самолета и аэропорт отправления
```sql
create index indx_many on flights(flight_no, departure_airport);
```
* делаем запрос содержащий два этих поля
```sql
explain
select * from flights where flight_no = 'PG0416' and departure_airport = 'DME';
```
```console
Bitmap Heap Scan on flights  (cost=4.70..104.89 rows=27 width=63)
  Recheck Cond: ((flight_no = 'PG0416'::bpchar) AND (departure_airport = 'DME'::bpchar))
  ->  Bitmap Index Scan on indx_many  (cost=0.00..4.69 rows=27 width=0)
        Index Cond: ((flight_no = 'PG0416'::bpchar) AND (departure_airport = 'DME'::bpchar))
```
*видим что запрос использовал сканирование по индексу*

## Описать что и как делали и с какими проблемами столкнулись
```sql
ANALYSE ticket_flights;
explain
select * from ticket_flights where flight_id > 89752;
```
```console
Seq Scan on ticket_flights  (cost=0.00..174829.35 rows=3700700 width=32)
  Filter: (flight_id > 89752)
```
*если сделать больше а не ровно, то выполняется Seq Scan а не Index Scan, я так подозреваю что полей таких много и планировщик считает дешевле выполнить Seq Scan*
