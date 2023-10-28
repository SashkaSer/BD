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
```yml
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

Индексы находятся в желотом статусе, так как их попросту некуда реплицировать, имеем всего одну ноду Эластика. Лучше использовать как минимум больше двух нод

Удаление индексов
```yml
[sasha@localhost Elastic]$ curl -X DELETE http://localhost:9200/ind-1
{"acknowledged":true}
[sasha@localhost Elastic]$ curl -X DELETE http://localhost:9200/ind-2
{"acknowledged":true}
[sasha@localhost Elastic]$ curl -X DELETE http://localhost:9200/ind-3
{"acknowledged":true}
```

Регистрация репозитория
```sql
[sasha@localhost Elastic]$ curl -XPOST localhost:9200/_snapshot/netology_backup?pretty -H 'Content-Type: application/json' -d'{"type": "fs", "settings": { "location":"repo" }}'
{
  "acknowledged" : true
}

[sasha@localhost Elastic]$ curl -X GET 'http://localhost:9200/_snapshot/netology_backup?pretty'
{
  "netology_backup" : {
    "type" : "fs",
    "settings" : {
      "location" : "repo"
    }
  }
}
```

Создание индекса
```sql
[sasha@localhost Elastic]$ curl -X PUT localhost:9200/test -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{"acknowledged":true,"shards_acknowledged":true,"index":"test"} 
[sasha@localhost Elastic]$ curl -X GET http://localhost:9200/test?pretty
{
  "test" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "test",
        "creation_date" : "1698511662021",
        "number_of_replicas" : "0",
        "uuid" : "A5CiXfNSRomSytZ_uzaIFA",
        "version" : {
          "created" : "7170999"
        }
      }
    }
  }
}
```

Создание снапшота
```yml
[sasha@localhost Elastic]$ curl -X PUT localhost:9200/_snapshot/netology_backup/elasticsearch?wait_for_completion=true
{"snapshot":{"snapshot":"elasticsearch","uuid":"yfG3_Z_iQHeS9W3YdSJVVA","repository":"netology_backup","version_id":7170999,"version":"7.17.9","indices":[".geoip_databases","test"],"data_streams":[],"include_global_state":true,"state":"SUCCESS","start_time":"2023-10-28T16:49:16.057Z","start_time_in_millis":1698511756057,"end_time":"2023-10-28T16:49:17.263Z","end_time_in_millis":1698511757263,"duration_in_millis":1206,"failures":[],"shards":{"total":2,"failed":0,"successful":2},"feature_states":[{"feature_name":"geoip","indices":[".geoip_databases"]}]}}
```

Снапшот
```yml
root@7a79fad92987:/usr/share/elasticsearch/snapshots/repo# ll
total 44
drwxrwxr-x. 3 elasticsearch root   134 Oct 28 16:49 ./
drwxr-xr-x. 3 elasticsearch root    18 Oct 28 16:43 ../
-rw-rw-r--. 1 elasticsearch root   847 Oct 28 16:49 index-0
-rw-rw-r--. 1 elasticsearch root     8 Oct 28 16:49 index.latest
drwxrwxr-x. 4 elasticsearch root    66 Oct 28 16:49 indices/
-rw-rw-r--. 1 elasticsearch root 28757 Oct 28 16:49 meta-yfG3_Z_iQHeS9W3YdSJVVA.dat
-rw-rw-r--. 1 elasticsearch root   440 Oct 28 16:49 snap-yfG3_Z_iQHeS9W3YdSJVVA.dat
```

Удаление индекса test и создание test2
```yml
[sasha@localhost Elastic]$ curl -X DELETE 'http://localhost:9200/test?pretty'
{
  "acknowledged" : true
}
[sasha@localhost Elastic]$ curl -X PUT localhost:9200/test-2?pretty -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}
[sasha@localhost Elastic]$
```

Восстановление из снапшота
```yml
[sasha@localhost Elastic]$ curl -X POST localhost:9200/_snapshot/netology_backup/elasticsearch/_restore?pretty -H 'Content-Type: application/json' -d'{"include_global_state":true}'
{
  "accepted" : true
}
[sasha@localhost Elastic]$ curl -X GET http://localhost:9200/_cat/indices?v
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2           BGlzLed3SduySFvYn79_jw   1   0          0            0       226b           226b
green  open   .geoip_databases ArzKHCakRd29lBM2Z0Fz6A   1   0         40            0     37.8mb         37.8mb
green  open   test             Xy4bMhzMQqihqf3y7BMFSw   1   0          0            0       226b           226b
```

