1. Реализовали свой форк образ:
Собрали свой образ:
```
[root@dz dockerbuilds]# docker build -t miliandra/nginx:1.0 .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM nginx:latest
 ---> c316d5a335a5
Step 2/2 : COPY index.html /usr/share/nginx/html/index.html
 ---> a06f725ac183
Successfully built a06f725ac183
Successfully tagged miliandra/nginx:1.0
```
Проверили работоспособность и нашу стартовую страницу:
```
[root@dz dockerbuilds]# docker run -d -p 80:80 miliandra/nginx:1.0
6f81c54de9d8630e3e0941f5a5efb53563b4bb421e82139ad8fc407c0972d723
[root@dz dockerbuilds]# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                               NAMES
6f81c54de9d8   miliandra/nginx:1.0   "/docker-entrypoint.…"   13 seconds ago   Up 13 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   loving_allen
```
И отправили в наш репозиторий:
```
[root@dz dockerbuilds]# docker push miliandra/nginx:1.0
The push refers to repository [docker.io/miliandra/nginx]
7f461c1e21ae: Pushed
762b147902c0: Mounted from library/nginx
235e04e3592a: Mounted from library/nginx
6173b6fa63db: Mounted from library/nginx
9a94c4a55fe4: Mounted from library/nginx
9a3a6af98e18: Mounted from library/nginx
7d0ebbe3f5d2: Mounted from library/nginx
1.0: digest: sha256:6292bfecb5cdd0b7bbbfa372face088221998ab8aec47219a3cad83f89d33e7a size: 1777
```
Ссылка на наш образ:
```
https://hub.docker.com/r/miliandra/nginx
```
2. Рассмотрели для данных сценариев использование Docker-контейнеров или виртуальных/физических машин:
* Высоконагруженное монолитное java веб-приложение;
  * Так как это приложение одно и оно высоконагруженное, то для него лучше использовать физику или виртуальную машину.
* Nodejs веб-приложение;
  * Так как тут не такая большая нагрузка и + к этому приложений может быть несколько, то docker-контейнер хорошее решение. Сможем спокойно изолировать друг от друга их и масштабировать тоже.
* Мобильное приложение c версиями для Android и iOS;
  * Лучше использовать ВМ, но можно и docker-контейнер. Все зависит от нагрузки и наших задач.
* Шина данных на базе Apache Kafka;
  * Тут подходят все наши варианты, зависит только от нагрузки и наших задач.
* Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
  * Вероятно, лучшим решением будет использование docker-контейнера, так как проще разворачивать и масштабировать, но так же можно использовать ВМ.
* Мониторинг-стек на базе Prometheus и Grafana;
  * Docker-контейнеры подойдут сюда прекрасно. 
* MongoDB, как основное хранилище данных для java-приложения;
  * Тут можно использовать все три варианта, но можно и разделить, приложение в контейнер, а СУБД на физику или ВМ, все зависит от нагрузки.
* Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.
  * В данном случае лучше физика или виртуализация.
3. Запустили и создали файл в centos контейнере:
```
[root@dz data]# docker run -it -v /opt/dockerdata/data:/data centos
[root@5cf6511d29f3 /]# cd /data/
[root@5cf6511d29f3 data]# touch centos.txt
[root@5cf6511d29f3 data]# ls
centos.txt
[root@5cf6511d29f3 data]#
```
Создали файл на хостовой машине:
```
[root@dz data]# touch hostfile.txt
[root@dz data]# ls -la
total 0
drwxr-xr-x. 2 root root 44 Jan 30 02:06 .
drwxr-xr-x. 3 root root 18 Jan 30 02:02 ..
-rw-r--r--. 1 root root  0 Jan 30 02:04 centos.txt
-rw-r--r--. 1 root root  0 Jan 30 02:06 hostfile.txt
```
Запустили и проверили через Debian контейнер наличие файлов:
```
[root@dz dockerdata]# docker run -it -v /opt/dockerdata/data:/data debian
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
0c6b8ff8c37e: Pull complete
Digest: sha256:fb45fd4e25abe55a656ca69a7bef70e62099b8bb42a279a5e0ea4ae1ab410e0d
Status: Downloaded newer image for debian:latest
root@da74167f8167:/# ls -la /data
total 0
drwxr-xr-x. 2 root root 44 Jan 29 23:06 .
drwxr-xr-x. 1 root root 18 Jan 29 23:03 ..
-rw-r--r--. 1 root root  0 Jan 29 23:04 centos.txt
-rw-r--r--. 1 root root  0 Jan 29 23:06 hostfile.txt
```
4. Ссылка на Docker образ с Ansible:
```
https://hub.docker.com/r/miliandra/ansible
```
