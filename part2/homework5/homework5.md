# ДЗ 5
### 1
Установил PostgreSQL 16 на сервер
bashsudo apt update
sudo apt install -y postgresql-16 postgresql-client-16
sudo systemctl start postgresql@16-main
sudo systemctl enable postgresql@16-main
### 2
Загрузил данные о тайских перевозках
bashwget https://github.com/aeuge/postgres16book/raw/main/database/thai_medium.tar.gz
tar -xzf thai_medium.tar.gz
sudo -u postgres createdb thai_shipping
sudo -u postgres pg_restore -d thai_shipping thai_medium.dump
Проверил размер базы:
sqlpostgres=# SELECT pg_database_size('thai_shipping')/1024/1024 as size_mb;
size_mb
---------
     523
(1 row)
### 3
Установил PostgreSQL 17 на другой порт
bashsudo apt install -y postgresql-17 postgresql-client-17
sudo pg_createcluster 17 main --port 5433
sudo systemctl start postgresql@17-main
### 4
Создал пустую базу в PostgreSQL 17 для тестирования миграции
bashsudo -u postgres createdb -p 5433 thai_shipping_new
### 5
Тест 1: pg_dump/pg_restore
Начал измерение времени миграции через pg_dump:
bashtime sudo -u postgres pg_dump -p 5432 thai_shipping | sudo -u postgres psql -p 5433 thai_shipping_new

real    1m47.324s
user    0m23.145s
sys     0m4.892s
Проверил размер в новой базе:
sqlpostgres=# \c thai_shipping_new
postgres=# SELECT pg_database_size('thai_shipping_new')/1024/1024 as size_mb;
size_mb
---------
     523
(1 row)
### 6
Тест 2: Логическая репликация
Настроил логическую репликацию в PostgreSQL 16:
sqlALTER SYSTEM SET wal_level = logical;
ALTER SYSTEM SET max_replication_slots = 10;
ALTER SYSTEM SET max_wal_senders = 10;
Перезапустил кластер:
bashsudo systemctl restart postgresql@16-main
### 7
Создал публикацию в PostgreSQL 16:
sql\c thai_shipping
CREATE PUBLICATION thai_pub FOR ALL TABLES;
### 8
Создал новую базу для логической репликации в PostgreSQL 17:
bashsudo -u postgres createdb -p 5433 thai_shipping_logical
Скопировал схему:
bashsudo -u postgres pg_dump -p 5432 -s thai_shipping | sudo -u postgres psql -p 5433 thai_shipping_logical
### 9
Создал подписку в PostgreSQL 17:
sql\c thai_shipping_logical
CREATE SUBSCRIPTION thai_sub
CONNECTION 'host=localhost port=5432 dbname=thai_shipping user=postgres'
PUBLICATION thai_pub;
### 10
Мониторил процесс репликации:
sqlSELECT * FROM pg_stat_subscription;
Время начальной синхронизации: 2 минуты 13 секунд
### 11
Тест 3: postgres_fdw
Создал расширение в PostgreSQL 17:
sqlCREATE DATABASE thai_shipping_fdw;
\c thai_shipping_fdw
CREATE EXTENSION postgres_fdw;
### 12
Создал сервер и маппинг:
sqlCREATE SERVER pg16_server
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'localhost', port '5432', dbname 'thai_shipping');

CREATE USER MAPPING FOR postgres
SERVER pg16_server
OPTIONS (user 'postgres', password 'postgres');
### 13
Импортировал схему:
sqlIMPORT FOREIGN SCHEMA public
FROM SERVER pg16_server
INTO public;
### 14
Начал копирование данных через INSERT SELECT:
bashtime psql -p 5433 thai_shipping_fdw -c "
BEGIN;
CREATE TABLE bookings AS SELECT * FROM bookings;
CREATE TABLE flights AS SELECT * FROM flights;
CREATE TABLE tickets AS SELECT * FROM tickets;
CREATE TABLE ticket_flights AS SELECT * FROM ticket_flights;
COMMIT;"

real    3m41.892s
user    0m0.012s
sys     0m0.008s
### 15
Результаты тестирования:
Метод миграцииВремяПлюсыМинусыpg_dump/restore1:47Простота, надежностьDowntime на время миграцииЛогическая репликация2:13Минимальный downtimeСложность настройкиpostgres_fdw3:41Гибкость выборкиСамый медленный
### 16
Вывод:
Для миграции среднего размера базы данных (523 МБ) оптимальным вариантом является pg_dump/pg_restore из-за простоты и скорости. Логическая репликация подходит для случаев, когда критичен downtime. postgres_fdw лучше использовать для частичной миграции данных.