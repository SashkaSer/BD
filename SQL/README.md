# Домашнее задание к занятию 1"`SQL`" - `Сергиенко А`

### Задание 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

Docker-compose файл
```yaml
version: '3.9'

services:

  postgres:
    image: postgres:12
    container_name: psql

    environment:
      POSTGRES_USER: "sasha"
      POSTGRES_PASSWORD: "Qwerty!@3"
      PGDATA: "/var/lib/postgresql/data"

    volumes:
      - ./data:/var/lib/postgresql/data
      - ./backup:/var/lib/postgresql/backup
  
    ports:
      - "5432:5432"

```
Команда для запуска: 
*docker compose up -d*

### Задача 2

* Список БД  
![DBlist](https://github.com/SashkaSer/BD/blob/main/SQL/img/DBlist.png)  

* Описание таблиц
![Tables](https://github.com/SashkaSer/BD/blob/main/SQL/img/tables.png)  

* SQL-запрос для выдачи списка пользователей с правами над таблицами test_db  
*SELECT grantee, privilege_type, table_name FROM information_schema.role_table_grants WHERE table_name IN ('orders', 'clients');*

Права над таблицами  
![Grants](https://github.com/SashkaSer/BD/blob/main/SQL/img/grants.png)