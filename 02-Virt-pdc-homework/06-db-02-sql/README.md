1. Docker-compose манифест:
```yaml
version: '3.5'

services:
  postgres:
    container_name: postgres
    image: postgres:12
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}
      PGDATA: /data/postgres
    volumes:
       - postgres_data:/data/postgres
       - postgres_backup:/data/backup/postgres
    ports:
      - "5432:5432"
    restart: always
    
volumes:
  postgres_data:
  postgres_backup:
```
Поднимаем контейнер с `PostgreSQL 12`:
```bash
[root@dz docker-compose]# docker-compose up -d
Creating network "dockercompose_default" with the default driver
Creating volume "dockercompose_postgres_data" with default driver
Creating volume "dockercompose_postgres_backup" with default driver
Pulling postgres (postgres:12)...
12: Pulling from library/postgres
5eb5b503b376: Already exists
daa0467a6c48: Pull complete
7cf625de49ef: Pull complete
bb8afcc973b2: Pull complete
c74bf40d29ee: Pull complete
2ceaf201bb22: Pull complete
1255f255c0eb: Pull complete
12a9879c7aa1: Pull complete
f7ca80cc6dd3: Pull complete
6714db455645: Pull complete
ee4f5626bf60: Pull complete
621bb0c2ae77: Pull complete
a19e980f0a72: Pull complete
Digest: sha256:505d023f030cdea84a42d580c2a4a0e17bbb3e91c30b2aea9c02f2dfb10325ba
Status: Downloaded newer image for postgres:12
Creating postgres ... done
[root@dz docker-compose]# docker-compose ps
  Name                Command              State                    Ports
-------------------------------------------------------------------------------------------
postgres   docker-entrypoint.sh postgres   Up      0.0.0.0:5432->5432/tcp,:::5432->5432/tcp
[root@dz docker-compose]# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
fe1ead476660   postgres:12   "docker-entrypoint.s…"   26 seconds ago   Up 25 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres
```
2. Вариантов работы с базой очень много, можно выбрать клиент PgAdmin, либо DBeaver, можно, конечно, через docker работать с базой, но так как-то приятнее.\
Создание пользователя:
```sql
CREATE ROLE "test-admin-user" PASSWORD 'test' SUPERUSER CREATEDB CREATEROLE INHERIT LOGIN;
```
Создание БД:
```sql
CREATE DATABASE test_db;
```
Создание таблиц:
```sql
CREATE TABLE orders (
    id serial primary key,
    Name varchar(80),
    Price int
);
CREATE TABLE clients (
    id serial primary key,
    Last_Name varchar(20),
    Country varchar(20),
    OrderID int REFERENCES orders (id)
);
```
Создание индекса:
```sql
CREATE INDEX ON clients (country);
```
Добавили комментарии к таблице orders:
```sql
COMMENT ON COLUMN orders.name IS 'Наименование';
COMMENT ON COLUMN orders.price IS 'Цена';
```
Добавили комментарии к таблице clients:
```sql
COMMENT ON COLUMN clients.last_name IS 'Фамилия';
COMMENT ON COLUMN clients.country IS 'Страна проживания';
COMMENT ON COLUMN clients.orderid IS 'Заказ';
```
Добавляем привилегии:
```sql
GRANT ALL ON TABLE public.clients, public.orders TO "test-admin-user";
```
Создание второго пользователя:
```sql
CREATE ROLE "test-simple-user" PASSWORD 'test1' NOSUPERUSER NOCREATEDB NOCREATEROLE INHERIT LOGIN;
```
Добавляем привилегии для 2 пользователя:
```sql
GRANT INSERT, SELECT, UPDATE, DELETE ON TABLE public.clients, public.orders TO "test-simple-user";
```
Итоговый список БД:
```sql
postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
(4 rows)
```
Описание таблиц:
```sql
test_db=# \d+ clients
                                                            Table "public.clients"
  Column   |         Type          | Collation | Nullable |               Default               | Storage  | Stats target |    Description
-----------+-----------------------+-----------+----------+-------------------------------------+----------+--------------+-------------------
 id        | integer               |           | not null | nextval('clients_id_seq'::regclass) | plain    |              |
 last_name | character varying(20) |           |          |                                     | extended |              | Фамилия
 country   | character varying(20) |           |          |                                     | extended |              | Страна проживания
 orderid   | integer               |           |          |                                     | plain    |              | Заказ
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "clients_country_idx" btree (country)
Foreign-key constraints:
    "clients_orderid_fkey" FOREIGN KEY (orderid) REFERENCES orders(id)
Access method: heap

test_db=# \d+ orders
                                                        Table "public.orders"
 Column |         Type          | Collation | Nullable |              Default               | Storage  | Stats target | Description
--------+-----------------------+-----------+----------+------------------------------------+----------+--------------+--------------
 id     | integer               |           | not null | nextval('orders_id_seq'::regclass) | plain    |              |
 name   | character varying(80) |           |          |                                    | extended |              | Наименование
 price  | integer               |           |          |                                    | plain    |              | Цена
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_orderid_fkey" FOREIGN KEY (orderid) REFERENCES orders(id)
Access method: heap
```
SQL-запрос на выдачу списка пользователей с правами над таблицами:
```sql
SELECT table_catalog, table_name, array_agg(privilege_type), grantee
FROM information_schema.table_privileges
WHERE (table_schema NOT IN ('information_schema','pg_catalog')) and (grantee !='postgres')
GROUP BY grantee, table_catalog, table_name;
```
Вывод SQL-запроса:
```sql
 table_catalog | table_name |                         array_agg                         |     grantee
---------------+------------+-----------------------------------------------------------+------------------
 test_db       | clients    | {INSERT,SELECT,UPDATE,DELETE,TRUNCATE,REFERENCES,TRIGGER} | test-admin-user
 test_db       | orders     | {INSERT,SELECT,UPDATE,DELETE,TRUNCATE,REFERENCES,TRIGGER} | test-admin-user
 test_db       | clients    | {INSERT,SELECT,UPDATE,DELETE}                             | test-simple-user
 test_db       | orders     | {INSERT,SELECT,UPDATE,DELETE}                             | test-simple-user
(4 rows)
```
3. Наполняем таблицу:
```sql
INSERT INTO public.clients (last_name,country) VALUES
('Иванов Иван Иванович','USA'),
('Петров Петр Петрович','Canada'),
('Иоганн Себастьян Бах','Japan'),
('Ронни Джеймс Дио','Russia'),
('Ritchie Blackmore','Russia');

INSERT INTO public.orders (name,price) VALUES
('Шоколад',10),
('Принтер',3000),
('Книга',500),
('Монитор',7000),
('Гитара',4000);
```
Вычисляем кол-во записей для каждой таблицы:
```sql
select 'clients' as name_table,  count(1) as all_table  from clients
union all
select 'orders' as name_table, count(1) as all_table from orders;

 name_table | all_table
------------+-----------
 clients    |         5
 orders     |         5
(2 rows)
```
4. Связываем записи из таблиц:
```sql
UPDATE clients SET orderid=3 WHERE id=1;
UPDATE clients SET orderid=4 WHERE id=2;
UPDATE clients SET orderid=5 WHERE id=3;
```
SQL-запрос и вывод пользователей, которые совершили заказ:
```sql
test_db=# select * from clients where orderid is not null;
 id |      last_name       | country | orderid
----+----------------------+---------+---------
  1 | Иванов Иван Иванович | USA     |       3
  2 | Петров Петр Петрович | Canada  |       4
  3 | Иоганн Себастьян Бах | Japan   |       5
(3 rows)
```
Но такой подход не верен, правильно будет сделать отдельную таблицу и с помощью нее и складировать заказы:
```sql
CREATE TABLE orders_to_clients (
  id serial PRIMARY KEY,
  id_clients integer,
  id_product integer,
  FOREIGN KEY (id_clients) REFERENCES clients (id),
  FOREIGN KEY (id_product) REFERENCES orders (id)
);
```
Наполняем ее нашими данными:
```sql
insert into orders_to_clients (id_clients,id_product) values
(3,1),
(4,2),
(5,3);
```
И смотрим что получилось:
```sql
select last_name, country, name, price from orders_to_clients 
join clients on clients.id=orders_to_clients.id_clients
join orders on orders.id=orders_to_clients.id_product;

      last_name       | country |  name   | price
----------------------+---------+---------+-------
 Иоганн Себастьян Бах | Japan   | Шоколад |    10
 Ронни Джеймс Дио     | Russia  | Принтер |  3000
 Ritchie Blackmore    | Russia  | Книга   |   500
(3 rows)
```
5. Вывод команды:
```sql
EXPLAIN SELECT * FROM clients WHERE orderid IS NOT null;

                         QUERY PLAN
------------------------------------------------------------
 Seq Scan on clients  (cost=0.00..15.30 rows=527 width=124)
   Filter: (orderid IS NOT NULL)
(2 rows)
```
Вывод данной команды говорит, что мы последовательно читаем данные из нашей таблицы. Затраты на получение первой строки составляет `0.00` (cost=0.00). Затраты на получение всех строк составили `(15.30)`. Приблизительно вернется `527` строк (rows=527) и средний размер каждой строки в байтах составит `124` (width=124).
6. Делаем резервную копию данных БД:
```bash
pg_dump -h 192.168.1.113 -U postgres -W -f /data/backup/postgres/test_db.dump -Fc test_db -Z 5 -v
```
Останавливаем наш контейнер и удаляем его:
```bash
[root@dz docker-compose]# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
fe1ead476660   postgres:12   "docker-entrypoint.s…"   53 minutes ago   Up 53 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres
[root@dz docker-compose]# docker stop postgres
postgres
[root@dz new_postgres]# docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED             STATUS                     PORTS     NAMES
fe1ead476660   postgres:12   "docker-entrypoint.s…"   About an hour ago   Exited (0) 4 minutes ago             postgres
[root@dz new_postgres]# docker rm postgres
postgres
```
Удаляем Volume с data:
```bash
[root@dz new_postgres]# docker volume rm dockercompose_postgres_data
dockercompose_postgres_data
```
Поднимаем новый контейнер:
```bash
[root@dz docker-compose]# docker-compose up -d
Creating volume "dockercompose_postgres_data" with default driver
Creating postgres ... done
[root@dz docker-compose]# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
b1ca4d2bffcf   postgres:12   "docker-entrypoint.s…"   16 seconds ago   Up 16 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres
```
Заходим и восстанавливаем БД:
```bash
[root@dz docker-compose]# docker exec -it postgres bash
root@b1ca4d2bffcf:/# cd /data/backup/postgres/
root@b1ca4d2bffcf:/data/backup/postgres# ls
test_db.dump
```
Убеждаемся, что нету нашей базы:
```sql
root@b1ca4d2bffcf:/data/backup/postgres# su postgres
postgres@b1ca4d2bffcf:/data/backup/postgres$ psql
psql (12.10 (Debian 12.10-1.pgdg110+1))
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)
```
Перед восстановлением, создаем наших пользователей из задачи 2 и восстанавливаем БД:
```bash
postgres@b1ca4d2bffcf:/$ pg_restore -U postgres -C -d postgres /data/backup/postgres/test_db.dump -v
```
Проверяем, что все у нас получилось:
```sql
postgres@b1ca4d2bffcf:/$ psql
psql (12.10 (Debian 12.10-1.pgdg110+1))
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
(4 rows)

postgres=# \c test_db
You are now connected to database "test_db" as user "postgres".
test_db=# \d+
                                 List of relations
 Schema |           Name           |   Type   |  Owner   |    Size    | Description
--------+--------------------------+----------+----------+------------+-------------
 public | clients                  | table    | postgres | 8192 bytes |
 public | clients_id_seq           | sequence | postgres | 8192 bytes |
 public | orders                   | table    | postgres | 8192 bytes |
 public | orders_id_seq            | sequence | postgres | 8192 bytes |
 public | orders_to_clients        | table    | postgres | 8192 bytes |
 public | orders_to_clients_id_seq | sequence | postgres | 8192 bytes |
(6 rows)

test_db=# \d+ clients
                                                            Table "public.clients"
  Column   |         Type          | Collation | Nullable |               Default               | Storage  | Stats target |    Description
-----------+-----------------------+-----------+----------+-------------------------------------+----------+--------------+-------------------
 id        | integer               |           | not null | nextval('clients_id_seq'::regclass) | plain    |              |
 last_name | character varying(20) |           |          |                                     | extended |              | Фамилия
 country   | character varying(20) |           |          |                                     | extended |              | Страна проживания
 orderid   | integer               |           |          |                                     | plain    |              | Заказ
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "clients_country_idx" btree (country)
Foreign-key constraints:
    "clients_orderid_fkey" FOREIGN KEY (orderid) REFERENCES orders(id)
Referenced by:
    TABLE "orders_to_clients" CONSTRAINT "orders_to_clients_id_clients_fkey" FOREIGN KEY (id_clients) REFERENCES clients(id)
Access method: heap

test_db=# \d+ orders
                                                        Table "public.orders"
 Column |         Type          | Collation | Nullable |              Default               | Storage  | Stats target | Description
--------+-----------------------+-----------+----------+------------------------------------+----------+--------------+--------------
 id     | integer               |           | not null | nextval('orders_id_seq'::regclass) | plain    |              |
 name   | character varying(80) |           |          |                                    | extended |              | Наименование
 price  | integer               |           |          |                                    | plain    |              | Цена
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_orderid_fkey" FOREIGN KEY (orderid) REFERENCES orders(id)
    TABLE "orders_to_clients" CONSTRAINT "orders_to_clients_id_product_fkey" FOREIGN KEY (id_product) REFERENCES orders(id)
Access method: heap
```
