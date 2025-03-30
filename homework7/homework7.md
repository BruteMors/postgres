# ДЗ 7
### 1
Выполнил запрос (Проверил скорость выполнения сложного запроса)

```sql
explain analyze
WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
     order_place AS (
         SELECT count(t.id) as order_place, t.fkride
         FROM book.tickets t
         group by t.fkride
     )
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
       t.order_place, st.all_place
FROM book.ride r
         JOIN book.schedule as s
              on r.fkschedule = s.id
         JOIN book.busroute br
              on s.fkroute = br.id
         JOIN book.busstation bs
              on br.fkbusstationfrom = bs.id
         JOIN order_place t
              on t.fkride = r.id
         JOIN all_place st
              on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;

QUERY PLAN                                                                                            
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=324873.64..324873.67 rows=10 width=56) (actual time=3486.250..3486.342 rows=10 loops=1)
   ->  Sort  (cost=324873.64..325214.13 rows=136197 width=56) (actual time=3467.858..3467.949 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 25kB
         ->  Group  (cost=319547.02..321930.47 rows=136197 width=56) (actual time=3425.560..3455.193 rows=144000 loops=1)
               Group Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
               ->  Sort  (cost=319547.02..319887.52 rows=136197 width=56) (actual time=3425.537..3434.034 rows=144000 loops=1)
                     Sort Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
                     Sort Method: external merge  Disk: 7864kB
                     ->  Hash Join  (cost=258936.93..303274.10 rows=136197 width=56) (actual time=3069.047..3245.298 rows=144000 loops=1)
                           Hash Cond: (r.fkbus = s_1.fkbus)
                           ->  Hash Join  (cost=258931.82..301927.45 rows=136197 width=36) (actual time=3068.933..3220.745 rows=144000 loops=1)
                                 Hash Cond: (r.fkschedule = s.id)
                                 ->  Merge Join  (cost=258875.41..299998.34 rows=136197 width=24) (actual time=3068.308..3198.827 rows=144000 loops=1)
                                       Merge Cond: (r.id = t.fkride)
                                       ->  Index Scan using ride_pkey on ride r  (cost=0.42..4555.42 rows=144000 width=16) (actual time=0.029..13.420 rows=144000 loops=1)
                                       ->  Finalize GroupAggregate  (cost=258874.99..293380.45 rows=136197 width=12) (actual time=3068.245..3159.543 rows=144000 loops=1)
                                             Group Key: t.fkride
                                             ->  Gather Merge  (cost=258874.99..290656.51 rows=272394 width=12) (actual time=3068.209..3114.800 rows=432000 loops=1)
                                                   Workers Planned: 2
                                                   Workers Launched: 2
                                                   ->  Sort  (cost=257874.97..258215.46 rows=136197 width=12) (actual time=3016.486..3024.751 rows=144000 loops=3)
                                                         Sort Key: t.fkride
                                                         Sort Method: external merge  Disk: 3672kB
                                                         Worker 0:  Sort Method: external merge  Disk: 3672kB
                                                         Worker 1:  Sort Method: external merge  Disk: 3672kB
                                                         ->  Partial HashAggregate  (cost=221248.77..243929.54 rows=136197 width=12) (actual time=2690.389..2798.859 rows=144000 loops=3)
                                                               Group Key: t.fkride
                                                               Planned Partitions: 4  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27456kB
                                                               Worker 0:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27416kB
                                                               Worker 1:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27416kB
                                                               ->  Parallel Seq Scan on tickets t  (cost=0.00..81397.45 rows=2183045 width=12) (actual time=1.761..313.537 rows=1746561 loops=3)
                                 ->  Hash  (cost=38.40..38.40 rows=1440 width=20) (actual time=0.617..0.620 rows=1440 loops=1)
                                       Buckets: 2048  Batches: 1  Memory Usage: 91kB
                                       ->  Hash Join  (cost=3.58..38.40 rows=1440 width=20) (actual time=0.052..0.481 rows=1440 loops=1)
                                             Hash Cond: (br.fkbusstationfrom = bs.id)
                                             ->  Hash Join  (cost=2.35..31.80 rows=1440 width=8) (actual time=0.030..0.278 rows=1440 loops=1)
                                                   Hash Cond: (s.fkroute = br.id)
                                                   ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.007..0.069 rows=1440 loops=1)
                                                   ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.017..0.017 rows=60 loops=1)
                                                         Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                                         ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.007..0.010 rows=60 loops=1)
                                             ->  Hash  (cost=1.10..1.10 rows=10 width=20) (actual time=0.009..0.009 rows=10 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                                   ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=20) (actual time=0.005..0.006 rows=10 loops=1)
                           ->  Hash  (cost=5.05..5.05 rows=5 width=12) (actual time=0.072..0.073 rows=5 loops=1)
                                 Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                 ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.068..0.070 rows=5 loops=1)
                                       Group Key: s_1.fkbus
                                       Batches: 1  Memory Usage: 24kB                                       ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.012..0.033 rows=200 loops=1)
 Planning Time: 35.877 ms
 JIT:
   Functions: 79
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 3.588 ms (Deform 1.079 ms), Inlining 0.000 ms, Optimization 3.560 ms, Emission 38.092 ms, Total 45.239 ms
 Execution Time: 3499.261 ms
(57 rows)
```

### 2
Навесил индекс на внешние ключи

```sql
CREATE INDEX idx_ride_fkschedule ON book.ride(fkschedule);
CREATE INDEX idx_ride_fkbus ON book.ride(fkbus);
CREATE INDEX idx_schedule_fkroute ON book.schedule(fkroute);
CREATE INDEX idx_busroute_fkstation ON book.busroute(fkbusstationfrom);
CREATE INDEX idx_tickets_fkride ON book.tickets(fkride);
CREATE INDEX idx_seat_fkbus ON book.seat(fkbus);
```

### 3
Выполнил ANALYZE

```sql
ANALYZE
```

### 4
Выполнил еще раз запрос (Проверил скорость выполнения сложного запроса)

```sql
explain analyze
WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
     order_place AS (
         SELECT count(t.id) as order_place, t.fkride
         FROM book.tickets t
         group by t.fkride
     )
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
       t.order_place, st.all_place
FROM book.ride r
         JOIN book.schedule as s
              on r.fkschedule = s.id
         JOIN book.busroute br
              on s.fkroute = br.id
         JOIN book.busstation bs
              on br.fkbusstationfrom = bs.id
         JOIN order_place t
              on t.fkride = r.id
         JOIN all_place st
              on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;


QUERY PLAN                                                                                            
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=324864.77..324864.79 rows=10 width=56) (actual time=3421.023..3421.121 rows=10 loops=1)
   ->  Sort  (cost=324864.77..325205.14 rows=136147 width=56) (actual time=3408.374..3408.471 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 25kB
         ->  Group  (cost=319540.11..321922.68 rows=136147 width=56) (actual time=3366.424..3395.877 rows=144000 loops=1)
               Group Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
               ->  Sort  (cost=319540.11..319880.48 rows=136147 width=56) (actual time=3366.400..3374.928 rows=144000 loops=1)
                     Sort Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
                     Sort Method: external merge  Disk: 7864kB
                     ->  Hash Join  (cost=258952.61..303275.31 rows=136147 width=56) (actual time=3032.974..3207.227 rows=144000 loops=1)
                           Hash Cond: (r.fkbus = s_1.fkbus)
                           ->  Hash Join  (cost=258947.50..301929.15 rows=136147 width=36) (actual time=3032.752..3182.565 rows=144000 loops=1)
                                 Hash Cond: (r.fkschedule = s.id)
                                 ->  Merge Join  (cost=258891.10..300000.73 rows=136147 width=24) (actual time=3032.132..3160.964 rows=144000 loops=1)
                                       Merge Cond: (r.id = t.fkride)
                                       ->  Index Scan using ride_pkey on ride r  (cost=0.42..4555.42 rows=144000 width=16) (actual time=0.014..11.159 rows=144000 loops=1)
                                       ->  Finalize GroupAggregate  (cost=258890.68..293383.47 rows=136147 width=12) (actual time=3032.109..3123.595 rows=144000 loops=1)
                                             Group Key: t.fkride
                                             ->  Gather Merge  (cost=258890.68..290660.53 rows=272294 width=12) (actual time=3032.077..3077.963 rows=432000 loops=1)
                                                   Workers Planned: 2
                                                   Workers Launched: 2
                                                   ->  Sort  (cost=257890.65..258231.02 rows=136147 width=12) (actual time=2949.473..2958.105 rows=144000 loops=3)
                                                         Sort Key: t.fkride
                                                         Sort Method: external merge  Disk: 3672kB
                                                         Worker 0:  Sort Method: external merge  Disk: 3672kB
                                                         Worker 1:  Sort Method: external merge  Disk: 3672kB
                                                         ->  Partial HashAggregate  (cost=221270.25..243953.35 rows=136147 width=12) (actual time=2649.594..2763.828 rows=144000 loops=3)
                                                               Group Key: t.fkride
                                                               Planned Partitions: 4  Batches: 5  Memory Usage: 8241kB  Disk Usage: 26464kB
                                                               Worker 0:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27416kB
                                                               Worker 1:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27408kB
                                                               ->  Parallel Seq Scan on tickets t  (cost=0.00..81400.35 rows=2183335 width=12) (actual time=25.552..295.601 rows=1746561 loops=3)
                                 ->  Hash  (cost=38.40..38.40 rows=1440 width=20) (actual time=0.608..0.611 rows=1440 loops=1)
                                       Buckets: 2048  Batches: 1  Memory Usage: 91kB
                                       ->  Hash Join  (cost=3.58..38.40 rows=1440 width=20) (actual time=0.046..0.469 rows=1440 loops=1)
                                             Hash Cond: (br.fkbusstationfrom = bs.id)
                                             ->  Hash Join  (cost=2.35..31.80 rows=1440 width=8) (actual time=0.025..0.273 rows=1440 loops=1)
                                                   Hash Cond: (s.fkroute = br.id)
                                                   ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.004..0.064 rows=1440 loops=1)
                                                   ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.015..0.016 rows=60 loops=1)
                                                         Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                                         ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.005..0.008 rows=60 loops=1)
                                             ->  Hash  (cost=1.10..1.10 rows=10 width=20) (actual time=0.016..0.016 rows=10 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                                   ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=20) (actual time=0.012..0.013 rows=10 loops=1)
                           ->  Hash  (cost=5.05..5.05 rows=5 width=12) (actual time=0.212..0.213 rows=5 loops=1)
                                 Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                 ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.207..0.208 rows=5 loops=1)
                                       Group Key: s_1.fkbus
                                       Batches: 1  Memory Usage: 24kB                                       ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.021..0.035 rows=200 loops=1)
 Planning Time: 0.581 ms
 JIT:
   Functions: 79
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 2.054 ms (Deform 0.659 ms), Inlining 0.000 ms, Optimization 2.752 ms, Emission 30.520 ms, Total 35.326 ms
 Execution Time: 3434.182 ms
(57 rows)
```
Как можно увидеть, индексы сократили Planning Time запроса. 35.877 ms vs 0.581 ms