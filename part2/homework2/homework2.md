# ДЗ 2
### 1
```postgresql
CREATE MATERIALIZED VIEW book.bus_occupancy AS
WITH all_place AS (
SELECT count(s.id) as all_place, s.fkbus as fkbus
FROM book.seat s
GROUP BY s.fkbus
),
order_place AS (
SELECT count(t.id) as order_place, t.fkride
FROM book.tickets t
GROUP BY t.fkride
)
SELECT  
r.id as ride_id,
r.startdate as depart_date,
bs.city || ', ' || bs.name as busstation,  
st.all_place - coalesce(t.order_place, 0) as svobodno,  
coalesce(t.order_place, 0) as busy_places            
FROM book.ride r
JOIN book.schedule s ON r.fkschedule = s.id
JOIN book.busroute br ON s.fkroute = br.id
JOIN book.busstation bs ON br.fkbusstationfrom = bs.id
LEFT JOIN order_place t ON t.fkride = r.id
JOIN all_place st ON r.fkbus = st.fkbus
;


CREATE OR REPLACE FUNCTION book.refresh_bus_occupancy()
RETURNS trigger AS $$
BEGIN
REFRESH MATERIALIZED VIEW CONCURRENTLY book.bus_occupancy;
RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_refresh_bus_occupancy
AFTER INSERT OR UPDATE OR DELETE ON book.tickets
FOR EACH STATEMENT
EXECUTE FUNCTION book.refresh_bus_occupancy();
```

### 2
Оценить падение производительности по сравнению со вставкой без триггера
Падение Производительности - Execution Time - 0.004 ms vs 4.522 ms