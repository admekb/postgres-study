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
CREATE TRIGGER t_sales
AFTER INSERT OR UPDATE OR DELETE ON sales FOR EACH ROW EXECUTE FUNCTION insert_sum();
```
* делаем запрос в таблицу

```sql
select * from pract_functions.good_sum_mart;
```
```console
Спички хозайственные	86.00
Автомобиль Ferrari FXX K	1110000000.06
```
* делаем вставку

```sql
INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (2, 1);
```
```console
Спички хозайственные	91.00
Автомобиль Ferrari FXX K	1295000000.07
```
* пробуем удалить

```sql
select * from pract_functions.good_sum_mart;
```
```console
Спички хозайственные	91.00
Автомобиль Ferrari FXX K	1110000000.06
```
```sql
DELETE from sales where sales_id = 3;
```
```sql
Спички хозайственные	86.00
Автомобиль Ferrari FXX K	1110000000.06
```
* пробуем обновить

```sql
select * from pract_functions.good_sum_mart;
```
```console
Спички хозайственные	540.50
Автомобиль Ferrari FXX K	1295000000.07
```
```sql
UPDATE sales SET sales_qty = 900 where sales_id = 2;
```
```sql
Спички хозайственные	540.50
Автомобиль Ferrari FXX K	167610000009.06
```
*дочитав до конца задание я понимаю, что при изменении стоимости товара у нас от витрины смысла нет, но это вроде как задание уже со звездочкой. Так что если у меня было бы время, я бы сделал еще один столбец в sales, в который вставлял бы функцией сумму проданного товара и уже складывал бы ее с привязкой к товару в общую сумму, которую записывал бы в витрину*
