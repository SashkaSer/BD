# Домашнее задание к занятию 1"`PostgreSQL`" - `Сергиенко А`

### Задание 1

#### Вывод списка БД
```sql
\l[+]   [PATTERN]      list databasesdt
```
#### Подключения к БД
```sql
\c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
                         connect to new database (currently "postgres")
```
#### Вывода списка таблиц
```sql
\dt[S+] [PATTERN]      list tables
```
#### Вывода описания содержимого таблиц
```sql
\d[S+]  NAME           describe table, view, sequence, or index
```
#### Выход из SQL
```sql
\q                     quit psql
```

### Задание 2

Восстановленный бэкап БД
```sql
postgres=# \c test_database 
psql (13.11, server 13.12 (Debian 13.12-1.pgdg120+1))
You are now connected to database "test_database" as user "postgres".
test_database=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | orders | table | postgres
(1 row)
```

ANALYZE таблицы orders
```sql
test_database=# ANALYZE VERBOSE orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
test_database=# 
```

столбец таблицы orders с наибольшим средним значением размера элементов в байтах.
```sql
test_database=# SELECT attname, avg_width FROM pg_stats WHERE tablename = 'orders' order by avg_width desc limit 1;
 attname | avg_width 
---------+-----------
 title   |        16
(1 row)
```
### Задание 3 

Шардирование таблицы orders на orders_1 - price>499 и orders_2 - price<=499

```sql
BEGIN;
ALTER TABLE orders RENAME TO orders_old;

CREATE TABLE orders (
    like public.orders_old INCLUDING ALL
)

CREATE TABLE orders_after_499 (
    CHECK (price>499)
    INHERITS (orders)
)

CREATE TABLE orders_before_499 (
    CHECK (price<=499)
    INHERITS (orders)
)

CREATE RULE orders_after_499 AS ON INSERT TO orders
WHERE (price>499)
DO INSTEAD INSERT INTO orders_after_499 VALUES (NEW.*)

CREATE RULE orders_before_499 AS ON INSERT TO orders
WHERE (price<=499)
DO INSTEAD INSERT INTO orders_before_499 VALUES (NEW.*)

INSERT INTO orders (id, title, price) SELECT id, titile, price FROM orders_old;

DROP TABLE orders_old

END;
```