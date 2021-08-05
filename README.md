## Домашнее задание к занятию "6.2. SQL"

### Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.
Приведите получившуюся команду или docker-compose манифест.

---
***Ответ:*** 

Команды:
``` 
docker pull postgres:12

docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
postgres     12        0f698a6badfa   2 weeks ago   314MB

docker volume create vol1
vol1
docker volume create vol2
vol2
```
```
docker run --rm --name pg-docker -e POSTGRES_PASSWORD=12345678 -ti -p 5432:5432 -v vol1:/var/lib/postgresql/data -v  vol2:/var/lib/postgresql postgres:12
```
```
docker run --rm --name pg-docker2 -e POSTGRES_PASSWORD=12345678 -ti -p 5433:5433 -v vol1:/var/lib/postgresql/data -v vol2:/var/lib/postgresql postgres:12
```
```
docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
4adb8cca9d5f   postgres:12   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker


```
```
docker exec -i -t 4adb8cca9d5f bash

root@4adb8cca9d5f:/#
root@4adb8cca9d5f:/#
root@3af289ce1404:/# su - postgres
postgres@3af289ce1404:~$ pwd
/var/lib/postgresql

postgres@3af289ce1404:~$ psql
psql (12.7 (Debian 12.7-1.pgdg100+1))
Type "help" for help.

postgres=#

```
![альт](https://i.ibb.co/BjXFqGF/Screenshot-4.jpg)

---
### Задание 2
В БД из задачи 1:
* создайте пользователя `test-admin-user` и БД `test_db`
* в БД `test_db` создайте таблицу `orders` и `clients` (спeцификация таблиц ниже)
* предоставьте привилегии на все операции пользователю `test-admin-user` на таблицы БД `test_db`
* создайте пользователя `test-simple-user` предоставьте пользователю `test-simple-user` права на `SELECT/INSERT/UPDATE/DELETE` данных таблиц БД `test_db`

Таблица orders:
* id (serial primary key)
* Наименование (string)
* Цена (integer)
* Таблица clients:

id (serial primary key)
* Фамилия (string)
* Страна проживания (string, index)
* Заказ (foreign key orders)

Приведите:
* Итоговый список БД после выполнения пунктов выше,
* Описание таблиц (describe)
* SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
* Список пользователей с правами над таблицами test_db

---
***Ответ:***
```
postgres@3af289ce1404:~$ psql
postgres=# create database "test_db";
CREATE DATABASE

postgres=# CREATE USER "test-admin-user";
CREATE ROLE

postgres=# ALTER USER "test-admin-user" SUPERUSER CREATEROLE CREATEDB;
ALTER ROLE

postgres=#
postgres=# ALTER DATABASE "test_db" OWNER to "test-admin-user";
ALTER DATABASE
postgres=#
```
```
postgres=# CREATE ROLE "test-simple-user" NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT LOGIN;
CREATE ROLE
```
```
test_db=# CREATE TABLE orders
test_db-# (id integer,
test_db(# name text,
test_db(# price integer,
test_db(# PRIMARY KEY (id)
test_db(# );
CREATE TABLE
test_db=#

test_db=# CREATE TABLE clients
test_db-# (id integer PRIMARY KEY,
test_db(# lastname text,
test_db(# country text,
test_db(# booking integer,
test_db(# FOREIGN KEY (booking) REFERENCES orders (Id)
test_db(# );
CREATE TABLE
```
Базы данных:
![альт](https://i.ibb.co/QK5qWGc/Screenshot-1.jpg)

Пользователи и созданные таблицы:
![альт](https://i.ibb.co/jyyjs5p/Screenshot-2.jpg)

Предоставим пользователю `test-simple-user` права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД `test_db`:

![альт](https://i.ibb.co/mHKM1cW/Screenshot-10.jpg)

![альт](https://i.ibb.co/2PRjKj5/Screenshot-11.jpg)


---
### Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

Таблица `orders`

Наименование  | цена
------------- | -------------
Шоколад       | 10
Принтер       | 3000
Книга         | 500
Монитор       | 7000
Гитара        | 4000

Таблица `clients`
ФИО  | Страна проживания
------------- | -------------
Иванов Иван Иванович	|USA
Петров Петр Петрович	|Canada
Иоганн Себастьян Бах	|Japan
Ронни Джеймс Дио	|Russia
Ritchie Blackmore	|Russia

Используя SQL синтаксис:
* Вычислите количество записей для каждой таблицы
* Приведите в ответе:
* Запросы
* Результаты их выполнения.

---
***Ответ:***
```
test_db=# insert into orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
INSERT 0 5
```
```
test_db=# insert into clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), (4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');
INSERT 0 5
```
Скриншот (количество записей для каждой таблицы)

![альт](https://i.ibb.co/bz8XQnS/Screenshot-3.jpg)

---
### Задача 4

Часть пользователей из таблицы `clients` решили оформить заказы из таблицы `orders`.
Используя `foreign keys` свяжите записи из таблиц, согласно таблице:

ФИО  | Заказ
------------- | -------------
Иванов Иван Иванович	|Книга
Петров Петр Петрович	|Монитор
Иоганн Себастьян Бах	|Гитара

Приведите SQL-запросы для выполнения данных операций.
Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
Подсказка - используйте директиву UPDATE.

---
***Ответ***:
```
test_db=# update  clients set booking = 3 where id = 1;
UPDATE 1
test_db=# update  clients set booking = 4 where id = 2;
UPDATE 1
test_db=# update  clients set booking = 5 where id = 3;
UPDATE 1
test_db=#
```
Просмотр у каких клиентов не нулевые значения по "заказам":

![альт](https://i.ibb.co/N2ZMWZ9/Screenshot-6.jpg)


---
### Задача 5
Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 (используя директиву `EXPLAIN`).
Приведите получившийся результат и объясните что значат полученные значения.

---
***Ответ:***
```
test_db=# explain select * from clients as c where exists (select id from orders as o where c.booking = o.id);
```
![альт](https://i.ibb.co/GHx9Nk4/Screenshot-7.jpg)

***Из дополнительных материалов к модулю***:
"Наибольший интерес в выводимой информации представляет ожидаемая стоимость выполнения оператора, которая показывает, сколько, по мнению планировщика, будет выполняться этот оператор.  Выводятся два числа: стоимость запуска до выдачи первой строки и общая стоимость выдачи всех строк". 

---

### Задача 6
Создайте бэкап БД `test_db` и поместите его в `volume`, предназначенный для бэкапов (см. Задачу 1).
Остановите контейнер с PostgreSQL (но не удаляйте volumes).
Поднимите новый пустой контейнер с PostgreSQL.
Восстановите БД `test_db` в новом контейнере.
Приведите список операций, который вы применяли для бэкапа данных и восстановления.

---
***Ответ***:
```
user@ubuntu:~/netology/6.2$ docker exec -t pg-docker pg_dump -U postgres test_db -f /var/lib/postgresql/data/dump_test_DB.sql
```
```
user@ubuntu:~/netology/6.2$ docker exec -i pg-docker2 psql -U postgres -d test_db -f /var/lib/postgresql/data/dump_test_DB.sql
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
CREATE TABLE
ALTER TABLE
COPY 5
COPY 5
ALTER TABLE
ALTER TABLE
ALTER TABLE
GRANT
GRANT
user@ubuntu:~/netology/6.2$
```