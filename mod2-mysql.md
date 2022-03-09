### Задача 1
Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

```
root@ubuntukurs:/opt# docker pull mysql:8.0.28
```

далее запускаеи с подключенным volume для хранения БД

```
root@ubuntukurs:/opt# docker run --name mysql8 -v /opt/mysqldb_1:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Netology -d mysql:8.0.28
055838e8629799dd3306099f1a533984277731fd0ae5601818974f19ceeb0841
root@ubuntukurs:/opt# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                 NAMES
055838e86297   mysql:8.0.28   "docker-entrypoint.s…"   15 seconds ago   Up 11 seconds   3306/tcp, 33060/tcp   mysql8

```

устанавливаем клиент
```
apt-get install mysql-client
```

Изучите бэкап БД и восстановитесь из него.

```
root@055838e86297:/var/lib/mysql# mysql -uroot -pNetology -h localhost
```
```
mysql> create database test_db;
Query OK, 1 row affected (0.09 sec)

```
```
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
5 rows in set (0.13 sec)
```

восстановление и проверка, что восстановилось

```
root@055838e86297:/var/lib/mysql# mysql -uroot -pNetology test_db < /var/lib/mysql/test_dump.sql 
mysql: [Warning] Using a password on the command line interface can be insecure.
root@055838e86297:/var/lib/mysql# mysql -uroot -pNetology -h localhost
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 8.0.28 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

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
5 rows in set (0.03 sec)

mysql> use test_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.01 sec)

```


Перейдите в управляющую консоль mysql внутри контейнера.


Используя команду \h получите список управляющих команд.

```
mysql> \h

For information about MySQL products and services, visit:
   http://www.mysql.com/
For developer information, including the MySQL Reference Manual, visit:
   http://dev.mysql.com/
To buy MySQL Enterprise support, training, or other products, visit:
   https://shop.mysql.com/

List of all MySQL commands:
Note that all text commands must be first on line and end with ';'
?         (\?) Synonym for `help'.
clear     (\c) Clear the current input statement.
connect   (\r) Reconnect to the server. Optional arguments are db and host.
delimiter (\d) Set statement delimiter.
edit      (\e) Edit command with $EDITOR.
ego       (\G) Send command to mysql server, display result vertically.
exit      (\q) Exit mysql. Same as quit.
go        (\g) Send command to mysql server.
help      (\h) Display this help.
nopager   (\n) Disable pager, print to stdout.
notee     (\t) Don't write into outfile.
pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
print     (\p) Print current command.
prompt    (\R) Change your mysql prompt.
quit      (\q) Quit mysql.
rehash    (\#) Rebuild completion hash.
source    (\.) Execute an SQL script file. Takes a file name as an argument.
status    (\s) Get status information from the server.
system    (\!) Execute a system shell command.
tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
use       (\u) Use another database. Takes database name as argument.
charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
warnings  (\W) Show warnings after every statement.
nowarning (\w) Don't show warnings after every statement.
resetconnection(\x) Clean session context.
query_attributes Sets string parameters (name1 value1 name2 value2 ...) for the next query to pick up.

For server side help, type 'help contents'

```

Найдите команду для выдачи статуса БД и приведите в ответе из ее вывода версию сервера БД.

```
mysql> status

...
Server version:		8.0.28 MySQL Community Server - GPL
...

```

Подключитесь к восстановленной БД и получите список таблиц из этой БД.
```
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
5 rows in set (0.03 sec)

mysql> use test_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.01 sec)

```

Приведите в ответе количество записей с price > 300.

```
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
5 rows in set (0.03 sec)

mysql> select * from orders where price>300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)

```

В следующих заданиях мы будем продолжать работу с данным контейнером.

### Задача 2
Создайте пользователя test в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней
- количество попыток авторизации - 3
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
   - Фамилия "Pretty"
   - Имя "James"
   
```
create user 'test'@'localhost' \
identified with mysql_native_password by 'test-pass' \
WITH MAX_QUERIES_PER_HOUR 100 \
password expire interval 180 day \
failed_login_attempts 3 \
Attribute '{"fname":"Pretty", "lname":"James"}';


```
- Предоставьте привелегии пользователю test на операции SELECT базы test_db.
```
mysql> grant select on test_db.* to 'test'@'localhost';
Query OK, 0 rows affected, 1 warning (0.07 sec)
```
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю test и приведите в ответе к задаче.

```
mysql> select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user=test;
ERROR 1054 (42S22): Unknown column 'test' in 'where clause'
mysql> select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user='test';
+------+-----------+---------------------------------------+
| USER | HOST      | ATTRIBUTE                             |
+------+-----------+---------------------------------------+
| test | localhost | {"fname": "Pretty", "lname": "James"} |
+------+-----------+---------------------------------------+
1 row in set (0.00 sec)


```

```
mysql> show grants for 'test'@'localhost';
+---------------------------------------------------+
| Grants for test@localhost                         |
+---------------------------------------------------+
| GRANT USAGE ON *.* TO `test`@`localhost`          |
| GRANT SELECT ON `test_db`.* TO `test`@`localhost` |
+---------------------------------------------------+
2 rows in set (0.00 sec)

```

### Задача 3  
Установите профилирование SET profiling = 1. Изучите вывод профилирования команд SHOW PROFILES;.
```
mysql> show profiles;
+----------+------------+--------------+
| Query_ID | Duration   | Query        |
+----------+------------+--------------+
|        1 | 0.00019575 | .            |
|        2 | 0.00072350 | show engines |
+----------+------------+--------------+
2 rows in set, 1 warning (0.00 sec)

```

Исследуйте, какой engine используется в таблице БД test_db и приведите в ответе.

```
mysql> show table status\G;
*************************** 1. row ***************************
           Name: orders
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 5
 Avg_row_length: 3276
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 6
    Create_time: 2022-03-09 20:21:17
    Update_time: 2022-03-09 20:21:17
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.00 sec)

```

Измените engine и приведите время выполнения и запрос на изменения из профайлера в ответе:

на MyISAM
на InnoDB


поменял на MyISAM 
```
mysql> alter table orders engine=myisam;
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> show table status\G;
*************************** 1. row ***************************
           Name: orders
         Engine: MyISAM
        Version: 10
     Row_format: Dynamic
           Rows: 5
 Avg_row_length: 3276
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 6
    Create_time: 2022-03-09 21:12:32
    Update_time: 2022-03-09 20:21:17
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.00 sec)

ERROR: 
No query specified

```

время, которое заняло:  

8 | 0.02959525 | alter table orders engine=myisam 

```
mysql> show profiles;
+----------+------------+----------------------------------+
| Query_ID | Duration   | Query                            |
+----------+------------+----------------------------------+
|        1 | 0.00019575 | .                                |
|        2 | 0.00072350 | show engines                     |
|        3 | 0.00083750 | SELECT DATABASE()                |
|        4 | 0.00441250 | show databases                   |
|        5 | 0.00260100 | show tables                      |
|        6 | 0.00502550 | show table status                |
|        7 | 0.00177650 | show table status                |
|        8 | 0.02959525 | alter table orders engine=myisam |
|        9 | 0.00281800 | show table status                |
+----------+------------+----------------------------------+
9 rows in set, 1 warning (0.01 sec)

```

поменял на InnoDB

```
mysql> alter table orders engine=innodb;
Query OK, 5 rows affected (0.05 sec)
Records: 5  Duplicates: 0  Warnings: 0

```

время, которое заняло  

|       10 | 0.04688550 | alter table orders engine=innodb

```
mysql> show profiles;
+----------+------------+----------------------------------+
| Query_ID | Duration   | Query                            |
+----------+------------+----------------------------------+
|        1 | 0.00019575 | .                                |
|        2 | 0.00072350 | show engines                     |
|        3 | 0.00083750 | SELECT DATABASE()                |
|        4 | 0.00441250 | show databases                   |
|        5 | 0.00260100 | show tables                      |
|        6 | 0.00502550 | show table status                |
|        7 | 0.00177650 | show table status                |
|        8 | 0.02959525 | alter table orders engine=myisam |
|        9 | 0.00281800 | show table status                |
|       10 | 0.04688550 | alter table orders engine=innodb |
+----------+------------+----------------------------------+

```

### Задача 4
Изучите файл my.cnf в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- Скорость IO важнее сохранности данных
innodb_flush_log_at_trx_commit = 2

- Нужна компрессия таблиц для экономии места на диске
innodb_file_per_table =1

- Размер буффера с незакомиченными транзакциями 1 Мб
innodb_log_buffer_size=1M

- Буффер кеширования 30% от ОЗУ
innodb_buffer_pool_size = 1200M

- Размер файла логов операций 100 Мб
innodb_log_file_size = 100M

Приведите в ответе измененный файл my.cnf.
```
root@anton-v-m:/opt/mysqldb_1# vim my.cnf

# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
# 
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

#Скорость IO важнее сохранности данных
innodb_flush_log_at_trx_commit = 2

#Нужна компрессия таблиц для экономии места на диске
innodb_file_per_table =1

#Размер буффера с незакомиченными транзакциями 1 Мб
innodb_log_buffer_size=1M

#Буффер кеширования 30% от ОЗУ (RAM = 4 GB)
innodb_buffer_pool_size = 1200M

#Размер файла логов операций 100 Мб
innodb_log_file_size = 100M


```
