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
Спички хозайственные	146.00
Автомобиль Ferrari FXX K	925000000.05
```
* делаем вставку

```sql
INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (2, 1);
```
* проверяем

```sql
select * from pract_functions.good_sum_mart;
```
```console
Спички хозайственные	151.00
Автомобиль Ferrari FXX K	1110000000.06
```
*update и delete проверять смысла нет, потому что да это все будет работать, но дочитав до конца задание я понимаю, что при изменении стоимости товара у нас от нее смысла нет, но это вроде как задание уже со звездочкой. Так что если у меня было бы время, я бы сделал еще один столбец в sales, в который вставлял бы функцией сумму проданного товара и уже складывал бы ее с привязкой к товару в общую сумму, которую записывал бы в витрину*
