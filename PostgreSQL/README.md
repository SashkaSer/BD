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
begin;

  alter table orders rename to orders_old;
 
  create table orders (
      like orders_old
  );
  
  create table orders_after_500 (
      check (price>499)
  ) inherits (orders);

  create table orders_before_500 (
      check (price<=499)
  ) inherits (orders);

  create rule orders_after_500 as on insert to orders
  where (price>499)
  do instead insert into orders_after_500 values(NEW.*);

  create rule orders_before_500 as on insert to orders
  where (price<=499)
  do instead insert into orders_before_500 values(NEW.*);

  insert into orders (id,title,price) select id,title,price from orders_old;

  drop table orders_old;
end;

END;
```

Проверка

```sql
test_database=# \dt
               List of relations
 Schema |       Name        | Type  |  Owner   
--------+-------------------+-------+----------
 public | orders            | table | postgres
 public | orders_after_500  | table | postgres
 public | orders_before_500 | table | postgres
(3 rows)

test_database=# SELECT * from orders_before_500;
 id |        title         | price 
----+----------------------+-------
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  7 | Me and my bash-pet   |   499
(5 rows)

test_database=# SELECT * from orders_after_500;
 id |       title        | price 
----+--------------------+-------
  2 | My little database |   500
  6 | WAL never lies     |   900
  8 | Dbiezdmin          |   501
(3 rows)

test_database=# 
```
Вставка данных для проверки

![backup](https://github.com/SashkaSer/BD/blob/main/PostgreSQL/img/checking2.png)  

Ручное разбиение можно исключить при изначальном создании таблицы с учетом шардирования

### Задание 4

```sql
[sasha@localhost postgresql]$ pg_dump -h 127.0.0.1 -p 5432 -U postgres test_database > test_database.dump
Password: 
[sasha@localhost postgresql]$ ls -lah
total 20K
drwxr-xr-x.  3 sasha            sasha   91 Oct 25 01:59 .
drwx------. 22 sasha            sasha 4.0K Oct 19 23:11 ..
drwx------. 19 systemd-coredump sasha 4.0K Oct 25 01:24 data
-rw-r--r--.  1 sasha            sasha  310 Oct 25 00:20 docker-compose.yml
-rw-r--r--.  1 sasha            sasha 2.6K Oct 25 01:59 test_database.dump
-rw-r--r--.  1 sasha            sasha 2.1K Oct 24 22:29 test_dump.sql
[sasha@localhost postgresql]$ 
```

Уникалььность значение столбца title можно достичь наложением ограничения уникальности
```sql
ALTER TABLE orders ADD CONSTRAINT unique_title UNIQUE (title);
```