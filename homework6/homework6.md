# ДЗ 6
### 1

Создал Docker compose
```sql
version: '3.8'

services:
  master:
    image: postgres:17
    container_name: pg-master
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: thai
    ports:
      - "5433:5432"
    volumes:
      - ./master-data:/var/lib/postgresql/data
      - ./master-init:/docker-entrypoint-initdb.d
    restart: unless-stopped

  replica:
    image: postgres:17
    container_name: pg-replica
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5434:5432"
    depends_on:
      - master
    volumes:
      - ./replica-data:/var/lib/postgresql/data
    restart: unless-stopped

```

master-init/01-replication.sql
```sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'secret$123';
```

master-init/02-config.sql
```sql
ALTER SYSTEM SET wal_level = replica;
ALTER SYSTEM SET max_wal_senders = 10;
ALTER SYSTEM SET wal_keep_size = '64MB';
ALTER SYSTEM SET hot_standby = on;
```

### 2
Скачал дамп и залил в мастер
```sql
wget https://storage.googleapis.com/thaibus/thai_small.tar.gz
tar -xf thai_small.tar.gz
psql -h 127.0.0.1 -p 5433 -U postgres -d thai < thai.sql
```

### 3
Создал слот
```sql
docker exec -it pg-master psql -U postgres -c "SELECT pg_create_physical_replication_slot('test');"

```

### 4
Остановил и очистил слейв
```sql
docker stop pg-replica
sudo rm -rf ./replica-data/*
```

### 5
Создал резервную копию с мастера
```sql
docker exec -u postgres -it pg-master \
pg_basebackup -h localhost -p 5432 -U replicator -D /tmp/backup -R -S test -Fp -Xs -P

docker cp pg-master:/tmp/backup/. ./replica-data/
```

### 6
Обновил конфиг подключения на реплике
```sql
primary_conninfo = 'host=master port=5432 user=replicator password=secret$123 application_name=replica'

```

### 7
Настроил pga_hba.conf на мастере
```sql
host replication replicator 172.25.0.0/16 scram-sha-256
```

### 8
Тест на запись (мастер)

```sql
cat > workload2.sql << EOL
INSERT INTO book.tickets (fkRide, fio, contact, fkSeat)
VALUES (
ceil(random()*100),
(array(SELECT fam FROM book.fam))[ceil(random()*110)]::text || ' ' ||
(array(SELECT nam FROM book.nam))[ceil(random()*110)]::text,
('{"phone":"+7' || (1000000000::bigint + floor(random()*9000000000)::bigint)::text || '"}')::jsonb,
ceil(random()*100));
EOL

pgbench -h 127.0.0.1 -p 5433 -U postgres -d thai -c 8 -j 4 -T 10 -f workload2.sql -n

transaction type: workload2.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
duration: 10 s
number of transactions actually processed: 24781
latency average = 3.209 ms
initial connection time = 66.503 ms
tps = 2492.681188 (without initial connection time)
```
### 9
Тест на чтение

```sql
cat > workload.sql << EOL
\set r random(1, 5000000)
SELECT id, fkRide, fio, contact, fkSeat FROM book.tickets WHERE id = :r;
EOL
```
Мастер:
```sql
pgbench -h 127.0.0.1 -p 5433 -U postgres -d thai -c 8 -j 4 -T 10 -f workload.sql -n

transaction type: workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
duration: 10 s
number of transactions actually processed: 58234
latency average = 1.365 ms
initial connection time = 73.819 ms
tps = 5859.374214 (without initial connection time)
```

Реплика:
```sql
pgbench -h 127.0.0.1 -p 5434 -U postgres -d thai -c 8 -j 4 -T 10 -f workload.sql -n

transaction type: workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
duration: 10 s
number of transactions actually processed: 62189
latency average = 1.278 ms
initial connection time = 70.699 ms
tps = 6260.123540 (without initial connection time)
```