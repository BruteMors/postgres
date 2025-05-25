# ДЗ 1
### 1
Создал таблицу с продажами:
   CREATE TABLE sales (
   sale_id SERIAL PRIMARY KEY,
   sale_date DATE NOT NULL,
   amount DECIMAL(10, 2) NOT NULL
   );

### 2
Функция для определения трети года:
```postgresql


CREATE OR REPLACE FUNCTION GetYearPart(sale_date DATE)
RETURNS INT
AS $$
DECLARE
month INT;
BEGIN
IF sale_date IS NULL THEN
RETURN NULL;
END IF;

    month := EXTRACT(MONTH FROM sale_date);

    CASE
        WHEN month BETWEEN 1 AND 4 THEN
            RETURN 1;
        WHEN month BETWEEN 5 AND 8 THEN
            RETURN 2;
        WHEN month BETWEEN 9 AND 12 THEN
            RETURN 3;
        ELSE
            RETURN 0;
    END CASE;
END;
$$ LANGUAGE plpgsql;
```