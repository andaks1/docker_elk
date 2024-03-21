# Домашнее задание к занятию 5. «Elasticsearch»

## Задача 1

В этом задании вы потренируетесь в:

- установке Elasticsearch,
- первоначальном конфигурировании Elasticsearch,
- запуске Elasticsearch в Docker.

Используя Docker-образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для Elasticsearch,
- соберите Docker-образ и сделайте `push` в ваш docker.io-репозиторий,
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины.

Требования к `elasticsearch.yml`:

- данные `path` должны сохраняться в `/var/lib`,
- имя ноды должно быть `netology_test`.

В ответе приведите:

- текст Dockerfile-манифеста,
- ссылку на образ в репозитории dockerhub,
- ответ `Elasticsearch` на запрос пути `/` в json-виде.

Подсказки:

- возможно, вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum,
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml,
- при некоторых проблемах вам поможет Docker-директива ulimit,
- Elasticsearch в логах обычно описывает проблему и пути её решения.

Далее мы будем работать с этим экземпляром Elasticsearch.

#### Ответ на задание 1.

- текст докер манифеста:
```JSON
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1578,
         "digest": "sha256:58265e52c95e47fc572f252362d61d3eaa4aa2fc4d48339f11257e649bdd7d90",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      }
   ]
}
``` 

- [ссылка](https://hub.docker.com/repository/docker/andaks/c7elk/general) на образ в репозитории dockerhub
Для запуска контейнера и ластика в нем:
```BASH
# docker run -it -p 9200:9200 -p 9300:9300 --name c7elk -d andaks/c7elk:manifest-latest
# docker exec -it c7elk bash
# service elasticsearch start
# Ctrl+D
```

- ответ `Elasticsearch` на запрос пути `/` в json-виде:
```JSON
{
  "name" : "netology_test",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "NVa2Xs2bTM6_clDmUO8Edw",
  "version" : {
    "number" : "7.17.18",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "8682172c2130b9a411b1bd5ff37c9792367de6b0",
    "build_date" : "2024-02-02T12:04:59.691750271Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```


## Задача 2

В этом задании вы научитесь:

- создавать и удалять индексы,
- изучать состояние кластера,
- обосновывать причину деградации доступности данных.

Ознакомьтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `Elasticsearch` 3 индекса в соответствии с таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API, и **приведите в ответе** на задание.

Получите состояние кластера `Elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находятся в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера Elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

#### Ответ на задание 2.

- список созданных индексов и их статусов:
```BASH
# curl -X GET "127.0.0.1:9200/_cat/indices/"
green  open .geoip_databases LOiKcomCSrib0ntT2GVcJA 1 0 37 0 34.2mb 34.2mb
green  open ind-1            xUW8qOBvR-WUphnrE4yLSA 1 0  0 0   227b   227b
yellow open ind-3            Ddjq-5c9QO2d23vKgHrMtg 4 2  0 0   417b   417b
yellow open ind-2            MXFs5Y5wQ9qUFaQZ8C196g 2 1  0 0   454b   454b
```

- состояние кластера:
```JSON
# curl -X GET "127.0.0.1:9200/_cluster/health?pretty"
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 10,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}
```

Часть индексов и сам статус кластера в нашем случае находятся в желтой зоне. Все потому, что эластик считается отказоустойчивой системой, которая априори исключает инсталляцию на одной ноде. В парадигме отказоустойчивости шарды должны иметь возможность реплицироваться на других участников кластера. В нашем случае этого не происходит.

- Удаление индексов:
```BASH
# curl -X DELETE "127.0.0.1:9200/ind-3"
# curl -X GET "127.0.0.1:9200/_cat/indices/"
green open .geoip_databases LOiKcomCSrib0ntT2GVcJA 1 0 37 0 34.2mb 34.2mb
```

## Задача 3

В этом задании вы научитесь:

- создавать бэкапы данных,
- восстанавливать индексы из бэкапов.

Создайте директорию `{путь до корневой директории с Elasticsearch в образе}/snapshots`.

Используя API, [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
эту директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `Elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `Elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:

- возможно, вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `Elasticsearch`.

#### Ответ на задание 3.

- запрос API и результат вызова API для создания репозитория:
```JSON
# curl -X PUT "127.0.0.1:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "/etc/elasticsearch/snapshots"
  }
}
'
{
  "acknowledged" : true
}
```

- список индексов:
```BASH
# curl -X GET "127.0.0.1:9200/_cat/indices/"
green open .geoip_databases LOiKcomCSrib0ntT2GVcJA 1 0 37 0 34.2mb 34.2mb
green open test             KgjpiUuTTz6QGTQHD_FLqw 1 0  0 0   227b   227b
```

- список файлов в директории со snapshot:
```BASH
# ll snapshots/
total 48
-rw-r--r-- 1 elasticsearch elasticsearch  1425 Mar 21 11:53 index-0
-rw-r--r-- 1 elasticsearch elasticsearch     8 Mar 21 11:53 index.latest
drwxr-sr-x 6 elasticsearch elasticsearch  4096 Mar 21 11:53 indices
-rw-r--r-- 1 elasticsearch elasticsearch 29240 Mar 21 11:53 meta-UFVOop3ESYGEq9VJpUHfcQ.dat
-rw-r--r-- 1 elasticsearch elasticsearch   711 Mar 21 11:53 snap-UFVOop3ESYGEq9VJpUHfcQ.dat
```

- список индексов после удаления test и создания test-2:
```BASH
# curl -X GET "127.0.0.1:9200/_cat/indices/"
green open test-2           1nHImMy5TWWujjxszYUCKw 1 0  0 0   227b   227b
green open .geoip_databases LOiKcomCSrib0ntT2GVcJA 1 0 37 0 34.2mb 34.2mb
```

- запрос к API восстановления:
```JSON
# curl -X POST "127.0.0.1:9200/_snapshot/netology_backup/elk_snapshot/_restore?pretty" -H 'Content-Type: application/json' -d'
{
  "indices": "test"
}
'
{
  "accepted" : true
}
```

- итоговый список индексов:
```BASH
# curl -X GET "127.0.0.1:9200/_cat/indices?pretty"
green open test-2           1nHImMy5TWWujjxszYUCKw 1 0  0 0   227b   227b
green open .geoip_databases LOiKcomCSrib0ntT2GVcJA 1 0 37 0 34.2mb 34.2mb
green open test             cxuAaNdPSJa-nbMXOXVazA 1 0  0 0   227b   227b
```


---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---


