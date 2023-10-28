# Домашнее задание к занятию 1"`SQL`" - `Сергиенко А`

### Задание 1

Не получилось собрать образ, использую готовый через docker compose.

### Задание 2

Создание индексов

```yml
curl -X PUT localhost:9200/ind-1 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
curl -X PUT localhost:9200/ind-2 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 2,  "number_of_replicas": 1 }}'
curl -X PUT localhost:9200/ind-3 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 4,  "number_of_replicas": 2 }}'
```

Список индексов
```yml
[sasha@localhost Elastic]$ curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases azJk9uZFRPqQauaeBke2IA   1   0         40            0     37.8mb         37.8mb
green  open   ind-1            XZopX-YoRPOC5xlomphypA   1   0          0            0       226b           226b
yellow open   ind-3            JOS0pyBjQXCp7ySgFC931w   4   2          0            0       904b           904b
yellow open   ind-2            u9qVuhixRse93Jt3FyZhrA   2   1          0            0       452b           452b
```

Состояние индексов
```yml
[sasha@localhost Elastic]$ curl -X GET 'http://localhost:9200/_cluster/health/ind-1?pretty'
{
  "cluster_name" : "sasha-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
[sasha@localhost Elastic]$ curl -X GET 'http://localhost:9200/_cluster/health/ind-2?pretty'
{
  "cluster_name" : "sasha-cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 2,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}
[sasha@localhost Elastic]$ curl -X GET 'http://localhost:9200/_cluster/health/ind-3?pretty'
{
  "cluster_name" : "sasha-cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 4,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 8,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}
[sasha@localhost Elastic]$ 
```

Состояние кластера
```sql
[sasha@localhost Elastic]$ curl -X GET localhost:9200/_cluster/health/?pretty
{
  "cluster_name" : "sasha-cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 8,
  "active_shards" : 8,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}
```

Индексы находятся в желотом статусе, так как их попросту некуда реплицировать, имеем всего одну ноду Эластика

Удаление индексов
```sql
[sasha@localhost Elastic]$ curl -X DELETE http://localhost:9200/ind-1
{"acknowledged":true}
[sasha@localhost Elastic]$ curl -X DELETE http://localhost:9200/ind-2
{"acknowledged":true}
[sasha@localhost Elastic]$ curl -X DELETE http://localhost:9200/ind-3
{"acknowledged":true}
```