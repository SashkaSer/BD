# Домашнее задание к занятию 1"`PostgreSQL`" - `Сергиенко А`

### Задание 1

#### Вывод списка БД
```sql
\l[+]   [PATTERN]      list databasesdt
```
#### подключения к БД
\c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
                         connect to new database (currently "postgres")

#### вывода списка таблиц
\dt[S+] [PATTERN]      list tables

#### вывода описания содержимого таблиц
\d[S+]  NAME           describe table, view, sequence, or index

#### Выход из SQL
\q                     quit psql


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

