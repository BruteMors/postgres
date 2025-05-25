# ДЗ 6
### 1
`\timing`

`SELECT payment_type, round(sum(tips)/sum(tips+fare)*100) AS tips_percent, count(*)`
`FROM taxi_trips`
`GROUP BY payment_type`
`ORDER BY tips_percent DESC;`

Время выполнения:  
27.401 секунд

Создадим индекс по колонке payment_type
CREATE INDEX idx_payment_type ON taxi_trips(payment_type);

`SELECT payment_type,
ound(sum(tips)/sum(tips+fare)*100) AS tips_percent,
count(*)
FROM taxi_trips
GROUP BY payment_type
ORDER BY tips_percent DESC;`


Время выполнения:
Time: 3829.121 ms

### 2
Вывод:Создание индекса по колонке payment_type значительно повысило производительность запроса.
Время выполнения запроса уменьшилось примерно в 7 раз.
