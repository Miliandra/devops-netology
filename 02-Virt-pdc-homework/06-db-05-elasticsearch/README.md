1. Dockerfile манифест:
```dockerfile
FROM centos:7
RUN cd /opt &&\
    groupadd elasticsearch && \
    useradd -c "elasticsearch" -g elasticsearch elasticsearch &&\
    mkdir /var/lib/data && chmod -R 777 /var/lib/data &&\
    yum -y install wget perl-Digest-SHA && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.1.0-linux-x86_64.tar.gz && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.1.0-linux-x86_64.tar.gz.sha512 && \
    shasum -a 512 -c elasticsearch-8.1.0-linux-x86_64.tar.gz.sha512 && \
    tar -xzf elasticsearch-8.1.0-linux-x86_64.tar.gz && \
    rm elasticsearch-8.1.0-linux-x86_64.tar.gz elasticsearch-8.1.0-linux-x86_64.tar.gz.sha512 && \
    chown -R elasticsearch:elasticsearch /opt/elasticsearch-8.1.0

USER elasticsearch
WORKDIR /opt/elasticsearch-8.1.0
ENV PATH=$PATH:/opt/elasticsearch-8.1.0/bin
COPY elasticsearch.yml config/
EXPOSE 9200
EXPOSE 9300
ENTRYPOINT ["elasticsearch"]
```
Ссылка на образ в репозитории dockerhub:\
https://hub.docker.com/r/miliandra/centos7_elastic \
Проверяем ответ:
```bash
curl -X GET "localhost:9200/"
{
  "name" : "netology_test",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "5TTMaRhyTZ6wmK6DfvNrkw",
  "version" : {
    "number" : "8.1.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "3700f7679f7d95e36da0b43762189bab189bc53a",
    "build_date" : "2022-03-03T14:20:00.690422633Z",
    "build_snapshot" : false,
    "lucene_version" : "9.0.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```
2. Добавили индексы:
```bash
curl -X PUT "localhost:9200/ind-1" -H 'Content-Type: application/json' -d'{"settings": {"index": {"number_of_shards": 1,"number_of_replicas": 0}}}'
curl -X PUT "localhost:9200/ind-2" -H 'Content-Type: application/json' -d'{"settings": {"index": {"number_of_shards": 2,"number_of_replicas": 1}}}'
curl -X PUT "localhost:9200/ind-3" -H 'Content-Type: application/json' -d'{"settings": {"index": {"number_of_shards": 4,"number_of_replicas": 2}}}'
```
Получили список индексов и их статусы:
```bash
[root@dz elasticsearch]# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   ind-1 9vdvmNShTmC5xM4OrU1MwA   1   0          0            0       225b           225b
yellow open   ind-3 tofTMIFER1uv-qnTQNRjTQ   4   2          0            0       413b           413b
yellow open   ind-2 gwgskMegTsuwbD4y507fgA   2   1          0            0       450b           450b
```
Получили состояние кластера:
```bash
[root@dz elasticsearch]# curl -X GET 'http://localhost:9200/_cluster/health/?pretty=true'
{
  "cluster_name" : "elasticsearch",
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
Так как нод в нашем кластере всего 1, а у 2 и 3 индекса мы указали 1 и 2 реплики соответственно, то реплицировать их некуда. Из-за этого и состояние кластера `yellow`.\
Удаляем наши индексы:
```
curl -X DELETE http://localhost:9200/ind-1
curl -X DELETE http://localhost:9200/ind-2
curl -X DELETE http://localhost:9200/ind-3
```
3. Используя `API` регистрируем директорию:
```bash
[root@dz elasticsearch]# curl -X PUT "http://localhost:9200/_snapshot/netology_backup?verify=false&pretty" -H 'Content-Type: application/json' -d'{"type": "fs","settings": {"location": "/opt/elasticsearch-8.1.0/snapshot"}}'
{
  "acknowledged" : true
}
```
Создаем индекс:
```bash
curl -X PUT "localhost:9200/test" -H 'Content-Type: application/json' -d'{"settings": {"index": {"number_of_shards": 1,"number_of_replicas": 0}}}'
```
Список индексов:
```bash
[root@dz elasticsearch]# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test  vBsAS4QMSh-sN2UFKn1aPg   1   0          0            0       225b           225b
```
Создаем `snapshot`:
```
curl -X PUT "http://localhost:9200/_snapshot/netology_backup/test_snapshot?wait_for_completion=true&pretty"
```
Приводим список файлов директории со `snapshot'ами`:
```bash
[root@dz elasticsearch]# docker exec reverent_fermat bash -c "ls -la /opt/elasticsearch-8.1.0/snapshot/"
total 32
drwxr-xr-x. 3 elasticsearch elasticsearch   134 Mar 14 02:02 .
drwxr-xr-x. 1 elasticsearch elasticsearch    48 Mar 13 23:37 ..
-rw-r--r--. 1 elasticsearch elasticsearch   846 Mar 14 02:02 index-0
-rw-r--r--. 1 elasticsearch elasticsearch     8 Mar 14 02:02 index.latest
drwxr-xr-x. 4 elasticsearch elasticsearch    66 Mar 14 02:02 indices
-rw-r--r--. 1 elasticsearch elasticsearch 18264 Mar 14 02:02 meta-AgofKjvwTX-Pk0wf72VRlA.dat
-rw-r--r--. 1 elasticsearch elasticsearch   355 Mar 14 02:02 snap-AgofKjvwTX-Pk0wf72VRlA.dat
```
Удалили индекс `test` и добавили `test-2`:
```bash
curl -X DELETE http://localhost:9200/test && curl -X PUT "localhost:9200/test-2" -H 'Content-Type: application/json' -d'{"settings": {"index": {"number_of_shards": 1,"number_of_replicas": 0}}}'
```
Список индексов:
```bash
[root@dz elasticsearch]# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 EePJ3t_DTnauNLZatdROow   1   0          0            0       225b           225b
```
Запросили доступные `snapshot'ы`:
```bash
[root@dz elasticsearch]# curl -X GET 'http://localhost:9200/_snapshot?pretty'
{
  "netology_backup" : {
    "type" : "fs",
    "uuid" : "WZ_wnC74QrqcRhBrh2yrDw",
    "settings" : {
      "location" : "/opt/elasticsearch-8.1.0/snapshot"
    }
  }
}
```
Произвели восстановление кластера и отобразили итоговый	список индексов:
```bash
[root@dz elasticsearch]# curl -X POST "http://localhost:9200/_snapshot/netology_backup/test_snapshot/_restore?pretty" -H 'Content-Type: application/json' -d'{"indices": "*","include_global_state": true}'
{
  "accepted" : true
}

[root@dz elasticsearch]# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 EePJ3t_DTnauNLZatdROow   1   0          0            0       225b           225b
green  open   test   0lHUeri9TNS3BoASA4rrPQ   1   0          0            0       225b           225b
```
