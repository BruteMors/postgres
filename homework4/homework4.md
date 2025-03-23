# ДЗ 4
### 1

Создал таблицу accounts(id integer, amount numeric)
```sql
CREATE TABLE accounts(id integer, amount numeric);
```

### 2

Добавил несколько записей и подключившись через 2 терминала добился ситуации взаимоблокировки (deadlock).
```sql
INSERT INTO accounts VALUES (1,10.00), (2,10.00);

-- 1 terminal
BEGIN;
UPDATE accounts SET amount = amount + 1 WHERE id = 1;
-- 2 terminal
BEGIN;
UPDATE accounts SET amount = amount + 1 WHERE id = 2;
-- 1
UPDATE accounts SET amount = amount + 1 WHERE id = 2;
-- 2
UPDATE accounts SET amount = amount + 1 WHERE id = 1;

[2025-03-23 17:20:34] [40P01] ERROR: deadlock detected
[2025-03-23 17:20:34] Подробности: Process 40 waits for ShareLock on transaction 3522; blocked by process 34.
[2025-03-23 17:20:34] Process 34 waits for ShareLock on transaction 3523; blocked by process 40.
[2025-03-23 17:20:34] Подсказка: See server log for query details.
[2025-03-23 17:20:34] Где: while updating tuple (0,1) in relation "accounts"
```

### 3 
Посмотрел логи и убедился, что информация о дедлоке туда попала
```sql
2025-03-23 17:20:34 2025-03-23 14:20:34.205 UTC [40] ERROR:  deadlock detected
2025-03-23 17:20:34 2025-03-23 14:20:34.205 UTC [40] DETAIL:  Process 40 waits for ShareLock on transaction 3522; blocked by process 34.
2025-03-23 17:20:34     Process 34 waits for ShareLock on transaction 3523; blocked by process 40.
2025-03-23 17:20:34     Process 40: UPDATE accounts SET amount = amount + 1 WHERE id = 1
2025-03-23 17:20:34     Process 34: UPDATE accounts SET amount = amount + 1 WHERE id = 2
2025-03-23 17:20:34 2025-03-23 14:20:34.205 UTC [40] HINT:  See server log for query details.
2025-03-23 17:20:34 2025-03-23 14:20:34.205 UTC [40] CONTEXT:  while updating tuple (0,1) in relation "accounts"
2025-03-23 17:20:34 2025-03-23 14:20:34.205 UTC [40] STATEMENT:  UPDATE accounts SET amount = amount + 1 WHERE id = 1
2025-03-23 17:20:34 2025-03-23 14:20:34.234 UTC [40] ERROR:  current transaction is aborted, commands ignored until end of transaction block
2025-03-23 17:20:34 2025-03-23 14:20:34.234 UTC [40] STATEMENT:  select current_database() as a, current_schemas(false) as b
2025-03-23 17:20:34 2025-03-23 14:20:34.240 UTC [40] ERROR:  current transaction is aborted, commands ignored until end of transaction block
2025-03-23 17:20:34 2025-03-23 14:20:34.240 UTC [40] STATEMENT:  SHOW TRANSACTION ISOLATION LEVEL
```
