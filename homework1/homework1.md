# ДЗ 1
### 1
Развернул ВМ в сервисе WB Cloud виртуальную машину с БД Postgres.
Подключился к ней по ssh.

### 2
Залил тайские перевозки

wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql
--2025-03-09 21:38:19--  https://storage.googleapis.com/thaibus/thai_small.tar.gz
Распознаётся storage.googleapis.com (storage.googleapis.com)… 108.177.14.207, 142.251.1.207, 74.125.131.207, ...
Подключение к storage.googleapis.com (storage.googleapis.com)|108.177.14.207|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 84252589 (80M) [application/x-gzip]
Сохранение в: «thai_small.tar.gz.1»

thai_small.tar.gz.1                            100%[==================================================================================================>]  80,35M  25,7MB/s    за 3,6s 


psql -U postgres -f thai.sql
SET
SET
SET
SET
SET
set_config
------------

(1 строка)

SET
SET
SET
SET
CREATE DATABASE
ALTER DATABASE
Вы подключены к базе данных "thai" как пользователь "postgres".
SET
SET
SET
SET
SET
set_config
------------

(1 строка)

SET
SET
SET
SET
CREATE SCHEMA
ALTER SCHEMA
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
COPY 5
COPY 60
COPY 10
COPY 111
COPY 109
COPY 144000
COPY 1440
COPY 200
COPY 3
COPY 5185505
setval
      1
(1 строка)

setval
     60
(1 строка)

setval
      1
(1 строка)

setval
144000
(1 строка)

setval
1440
(1 строка)

setval
    200
(1 строка)

setval
      1
(1 строка)

setval
5185505
(1 строка)

ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE


### 3

Количество поездок

from book.tickets;

count

---------
5185505
(1 строка)

### 4

ВМ выключил