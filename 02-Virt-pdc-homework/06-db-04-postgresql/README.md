1. Подняли инстанс PostgrSQL 13 с помощью docker-compose:
```yaml
version: '3.5'

services:
  postgres:
    container_name: postgres_13
    image: faa0ddfa0c0f
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
       - /data/postgresql:/var/lib/postgresql/data
       - /data/postgresql/backup:/var/lib/postgresql/backup
    ports:
      - "5432:5432"
    restart: always
```
Вывод списка БД:
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
(3 rows)
```
Подключение к БД:
```sql
postgres=# \c postgres
You are now connected to database "postgres" as user "postgres".
```
Команды на вывод списка таблиц:
```sql
\dt
\dt+
\dS
```
Вывод описания содержимого таблиц:
```sql
\d [ pattern ]
```
Выход из Psql:
```sql
\q
```
2. Создали БД `test_database`:
```sql
postgres=# create database test_database;
CREATE DATABASE
postgres=# \l
                                   List of databases
     Name      |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
---------------+----------+----------+------------+------------+-----------------------
 postgres      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            | postgres=CTc/postgres
 template1     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            | postgres=CTc/postgres
 test_database | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
(4 rows)
```
Восстановили бэкап БД в `test_databes`:
```sql
postgres@b71646060326:~$ psql test_database < /var/lib/postgresql/backup/test_dump.sql
SET
SET
SET
SET
SET
 set_config
------------

(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
COPY 8
 setval
--------
      8
(1 row)

ALTER TABLE
```
Выполнили операцию `ANALYZE`:
```sql
postgres=# \c test_database
You are now connected to database "test_database" as user "postgres".
test_database=# \d+
                                   List of relations
 Schema |     Name      |   Type   |  Owner   | Persistence |    Size    | Description
--------+---------------+----------+----------+-------------+------------+-------------
 public | orders        | table    | postgres | permanent   | 8192 bytes |
 public | orders_id_seq | sequence | postgres | permanent   | 8192 bytes |
(2 rows)

test_database=# analyze verbose public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
```
Результат и команда для вычисления:
```sql
test_database=# select tablename, attname, avg_width from pg_stats where avg_width in (select max(avg_width) from pg_stats where tablename = 'orders');
 tablename | attname | avg_width
-----------+---------+-----------
 orders    | title   |        16
(1 row)
```
3. SQL команда для проведения разбиения таблицы `orders` на 2:
```sql
alter table orders rename to orders_old;
create table orders as table orders_old with no data;

Create table orders_1 (
    Check ( price > 499 )
    )
Inherits (orders);

Create table orders_2 (
    Check (price <= 499 )
    )
Inherits (orders);

Create Rule orders_insert_to_1 as on insert to orders
where ( price > 499 )
Do instead insert into orders_1 values (new.*);

Create Rule orders_insert_to_2 as on insert to orders
where ( price <= 499 )
Do instead insert into orders_2 values (new.*);
Insert into orders select * from orders_old;
```
Изначально можно было сделать так:
```sql
Create table orders (
    id int not null,
    title varchar(80) not null,
    price int default 0
)
Partition by range(price);
```
И конечно же не забыть, создать `orders_1` и `orders_2` с нужным для нас диапазоном для столбца `price`.

4. Создали бэкап БД:
```bash
pg_dump -h 192.168.1.113 -U postgres -W -f /var/lib/postgresql/backup/backup_bd.dump test_database -v
```
Добавили ограничение `UNIQUE`:
```sql
CREATE TABLE public.orders (
    id integer,
    title character varying(80) UNIQUE,
    price integer
);
```
