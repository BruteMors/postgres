# ДЗ 3
### 1

Создал таблицу с текстовым полем и заполнил сгенерированными данным в размере 1 млн строк
```sql
CREATE TABLE text_data (
text TEXT
);

INSERT INTO text_data (text) SELECT md5(random()::text) FROM generate_series(1, 1000000);

CREATE TABLE
INSERT 0 1000000
```

### 2
Посмотрел размер файла с таблицей
```sql
SELECT
n.nspname || '.' || c.relname AS table_name,
pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size,
pg_size_pretty(pg_total_relation_size(c.reltoastrelid)) AS toast_size
FROM pg_class c
JOIN pg_namespace n
ON c.relnamespace = n.oid
WHERE
relname = 'text_data';

    table_name    | total_size | toast_size 
------------------+------------+------------
 public.text_data | 65 MB      | 8192 bytes
(1 строка)
```

### 3

5 раз обновил все строчки и добавил к каждой строчке по символу
```sql
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
```

### 4
Посмотрел количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```sql
postgres=# SELECT relname, n_live_tup, n_dead_tup,
trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'text_data';
relname  | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
-----------+------------+------------+--------+-------------------------------
text_data |     999677 |    2231751 |    223 | 2025-03-16 20:32:57.990006+03
(1 строка)
```

### 5
Подождал некоторое время, проверяя, пришел ли автовакуум
```sql
SELECT relname, n_live_tup, n_dead_tup,
trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'text_data';
relname  | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
-----------+------------+------------+--------+-------------------------------
text_data |    1464278 |          0 |      0 | 2025-03-16 20:33:50.698766+03
```

### 6
5 раз обновил все строчки и добавил к каждой строчке по символу
```sql
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
```


### 7
Посмотрел размер файла с таблицей
```sql
SELECT
    n.nspname || '.' || c.relname AS table_name,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size,
    pg_size_pretty(pg_total_relation_size(c.reltoastrelid)) AS toast_size
FROM pg_class c
         JOIN pg_namespace n
              ON c.relnamespace = n.oid
WHERE
    relname = 'text_data';

table_name    | total_size | toast_size 
------------------+------------+------------
 public.text_data | 391 MB     | 8192 bytes
(1 строка)
```

### 8
Отключил Автовакуум на таблице
```sql
ALTER TABLE text_data SET (autovacuum_enabled = off);
```

### 9
10 раз обновил все строчки и добавил к каждой строчке по символу
```sql
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
UPDATE text_data SET text = text || 'a';
```


### 10
Посмотреть размер файла с таблицей
```sql
SELECT
    n.nspname || '.' || c.relname AS table_name,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size,
    pg_size_pretty(pg_total_relation_size(c.reltoastrelid)) AS toast_size
FROM pg_class c
         JOIN pg_namespace n
              ON c.relnamespace = n.oid
WHERE
    relname = 'text_data';

table_name    | total_size | toast_size 
------------------+------------+------------
 public.text_data | 1102 MB    | 8192 bytes
(1 строка)
```

### 11
Объясните полученный результат

При выполнении UPDATE каждая строка физически копируется, а старая версия становится "мёртвой".
Без VACUUM мертвые строки остаются в таблице, увеличивая её размер.

### 12
Включил автовакуум
```sql
ALTER TABLE text_data SET (autovacuum_enabled = on);
```

### Задача со звездочкой - 10 раз обновляем все строчки в таблице text_data
```sql
DO $$
DECLARE
i INT := 1;
BEGIN
WHILE i <= 10 LOOP
UPDATE text_data
SET text = md5(random()::text);
i := i + 1;
END LOOP;
END $$;
```