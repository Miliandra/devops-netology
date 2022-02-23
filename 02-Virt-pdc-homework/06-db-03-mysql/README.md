1. Подняли инстанс MySQL с помощью docker-compose:
```yaml
version: '3.5'       
       
services:

  mysql:
    container_name: mysql
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root1234
    volumes:
      - /data/mysql:/var/lib/mysql
    ports:
      - "3306:3306"
```
Перешли в  управляющую консоль `mysql` и получили статус БД:
```sql
mysql> \s
--------------
mysql  Ver 8.0.28 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:          17
Current database:
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.0.28 MySQL Community Server - GPL
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    latin1
Conn.  characterset:    latin1
UNIX socket:            /var/run/mysqld/mysqld.sock
Binary data as:         Hexadecimal
Uptime:                 17 min 31 sec

Threads: 2  Questions: 41  Slow queries: 0  Opens: 137  Flush tables: 3  Open tables: 56  Queries per second avg: 0.039
```
Проверили список БД и восстановили наш тестовый бэкап БД, предварительно создав БД `test_db`:
```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> create database test_db;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test_db            |
+--------------------+
5 rows in set (0.00 sec)

root@5d4439e5b902:/var/lib/mysql# mysql -uroot -p test_db < test_dump.sql
Enter password:
```
Подключились к восстановленной БД и получили список таблиц:
```sql
mysql> \u test_db
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)
```
Количество записей    с `price` > 300:
```sql
mysql> select * from orders where price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)
```
2. Создаем пользователя `test`:
```sql
create user 'test'@'localhost' 
identified with mysql_native_password by 'test-pass'
with max_queries_per_hour 100
password expire interval 180 day
failed_login_attempts 3 password_lock_time 2
attribute '{"fname": "James", "lname": "Pretty"}';
```
Даем ему привилегию на операцию с базой `test_db`:
```sql
grant select on test_db.* to 'test'@'localhost';
```
Получили данные по пользователю `test`:
```sql
mysql> select * from information_schema.user_attributes where user='test';
+------+-----------+---------------------------------------+
| USER | HOST      | ATTRIBUTE                             |
+------+-----------+---------------------------------------+
| test | localhost | {"fname": "James", "lname": "Pretty"} |
+------+-----------+---------------------------------------+
1 row in set (0.00 sec)
```
3. Установили профилирование `SET profiling = 1` и изучили вывод:
```sql
mysql> \u test_db
Database changed
mysql> SET profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select * from orders;
+----+-----------------------+-------+
| id | title                 | price |
+----+-----------------------+-------+
|  1 | War and Peace         |   100 |
|  2 | My little pony        |   500 |
|  3 | Adventure mysql times |   300 |
|  4 | Server gravity falls  |   300 |
|  5 | Log gossips           |   123 |
+----+-----------------------+-------+
5 rows in set (0.00 sec)

mysql> show profiles;
+----------+------------+----------------------+
| Query_ID | Duration   | Query                |
+----------+------------+----------------------+
|        1 | 0.00018425 | select * from orders |
+----------+------------+----------------------+
1 row in set, 1 warning (0.00 sec)
```
Узнаем какой `ENGINE` используется в нашей таблице:
```sql
select engine from information_schema.tables where table_schema = 'test_db' and table_name = 'orders';

+--------+
| ENGINE |
+--------+
| InnoDB |
+--------+
1 row in set (0.00 sec)
```
Изменили `ENGINE`:
```sql
mysql> alter table orders engine = myisam;
Query OK, 5 rows affected (0.02 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> alter table orders engine = innodb;
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> show profiles;
+----------+------------+--------------------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                                  |
+----------+------------+--------------------------------------------------------------------------------------------------------+
|        1 | 0.00018425 | select * from orders                                                                                   |
|        2 | 0.00018425 | SHOW CREATE TABLE orders                                                                               |
|        3 | 0.00020425 | SHOW CREATE TABLE orders                                                                               |
|        4 | 0.00061675 | select engine from information_schema.tables where table_schema = 'test_db' and table_name = 'orders'  |
|        5 | 0.01545375 | alter table orders engine = myisam                                                                     |
|        6 | 0.00874100 | alter table orders engine = innodb                                                                     |
+----------+------------+--------------------------------------------------------------------------------------------------------+
6 rows in set, 1 warning (0.00 sec)
```
4. Конфиг в измененном виде:
```
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

innodb_flush_method = O_DSYNC
innodb_file_per_table = 1
innodb_log_buffer_size = 1M
innodb_buffer_pool_size= 1G
innodb_log_file_size = 100M

# Custom config should go here
!includedir /etc/mysql/conf.d/
```
