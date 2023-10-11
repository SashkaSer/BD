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
