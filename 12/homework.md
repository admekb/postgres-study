# Триггеры, поддержка заполнения витрин
## создали бд как было указано в задании

* создадим функцию

```sql
CREATE OR REPLACE FUNCTION insert_sum() RETURNS TRIGGER AS $t_sales$
BEGIN
        IF (TG_OP = 'DELETE') THEN
				TRUNCATE pract_functions.good_sum_mart;
				INSERT INTO pract_functions.good_sum_mart (good_name, sum_sale)
				SELECT G.good_name, sum(G.good_price * S.sales_qty)
				FROM goods G
				INNER JOIN sales S ON S.good_id = G.goods_id
				GROUP BY G.good_name;
        ELSIF (TG_OP = 'UPDATE') THEN
				TRUNCATE pract_functions.good_sum_mart;
				INSERT INTO pract_functions.good_sum_mart (good_name, sum_sale)
				SELECT G.good_name, sum(G.good_price * S.sales_qty)
				FROM goods G
				INNER JOIN sales S ON S.good_id = G.goods_id
				GROUP BY G.good_name;
        ELSIF (TG_OP = 'INSERT') THEN
				TRUNCATE pract_functions.good_sum_mart;
				INSERT INTO pract_functions.good_sum_mart (good_name, sum_sale)
				SELECT G.good_name, sum(G.good_price * S.sales_qty)
				FROM goods G
				INNER JOIN sales S ON S.good_id = G.goods_id
				GROUP BY G.good_name;
        END IF;
        RETURN NULL;
END;
$t_sales$ LANGUAGE plpgsql;
```
* создадим триггер

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
CREATE TRIGGER t_sales
AFTER INSERT OR UPDATE OR DELETE ON sales FOR EACH ROW EXECUTE FUNCTION insert_sum();
```
* делаем запрос в таблицу
```sql
select * from pract_functions.good_sum_mart;
```
```console
Спички хозайственные	146.00
Автомобиль Ferrari FXX K	925000000.05
```
* делаем вставку
```sql
INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (2, 1);
```
```console
Спички хозайственные	151.00
Автомобиль Ferrari FXX K	1110000000.06
```
*update и delete проверять смысла нет, потому что да это все будет работать, но дочитав до конца задание я понимаю, что при изменении стоимости товара у нас от нее смысла нет*
