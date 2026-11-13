# Расширенные возможности SQL — Константинова Надежда

Обязательные задания выполнены на MySQL 8.0.46 с официальной учебной базой Sakila. В базе 1000 фильмов, 599 покупателей и 16044 записи об аренде. SQL-запросы и фактический вывод сервера приложены к решению.

## Задание 1. Магазин с количеством покупателей более 300

```sql
SELECT s.store_id,
       st.last_name AS staff_last_name,
       st.first_name AS staff_first_name,
       c.city,
       counts.customer_count
FROM store AS s
JOIN staff AS st ON st.store_id = s.store_id
JOIN address AS a ON a.address_id = s.address_id
JOIN city AS c ON c.city_id = a.city_id
JOIN (
    SELECT store_id, COUNT(*) AS customer_count
    FROM customer
    GROUP BY store_id
    HAVING COUNT(*) > 300
) AS counts ON counts.store_id = s.store_id
ORDER BY s.store_id, st.staff_id;
```

Результат одного запроса:

| store_id | staff_last_name | staff_first_name | city | customer_count |
|---:|---|---|---|---:|
| 1 | Hillyer | Mike | Lethbridge | 326 |

В магазине № 2 закреплены 273 покупателя, поэтому он не входит в результат. Сначала покупатели подсчитываются по магазинам, затем присоединяются сведения о сотрудниках и городе. Это исключает умножение счётчика покупателей при наличии нескольких сотрудников в магазине.

[SQL-файл](sql/01_store_customers.sql) · [Вывод MySQL](results/01_store_customers.txt)

## Задание 2. Фильмы длиннее среднего

```sql
SELECT COUNT(*) AS films_above_average
FROM film
WHERE length > (SELECT AVG(length) FROM film);
```

Результат:

| films_above_average |
|---:|
| 489 |

Средняя продолжительность по всем фильмам составляет `115.2720` минуты. Условие строгое: учитываются фильмы с `length` больше среднего, а не равные ему.

[SQL-файл](sql/02_films_above_average.sql) · [Вывод MySQL](results/02_films_above_average.txt)

## Задание 3. Месяц с максимальной суммой платежей

```sql
WITH monthly_revenue AS (
    SELECT CAST(DATE_FORMAT(payment_date, '%Y-%m-01') AS DATE) AS month_start,
           SUM(amount) AS total_revenue
    FROM payment
    GROUP BY CAST(DATE_FORMAT(payment_date, '%Y-%m-01') AS DATE)
), top_month AS (
    SELECT month_start, total_revenue
    FROM monthly_revenue
    ORDER BY total_revenue DESC, month_start
    LIMIT 1
)
SELECT DATE_FORMAT(t.month_start, '%Y-%m') AS payment_month,
       t.total_revenue,
       COUNT(r.rental_id) AS rentals_in_month
FROM top_month AS t
LEFT JOIN rental AS r
       ON r.rental_date >= t.month_start
      AND r.rental_date < t.month_start + INTERVAL 1 MONTH
GROUP BY t.month_start, t.total_revenue;
```

Результат:

| payment_month | total_revenue | rentals_in_month |
|---|---:|---:|
| 2005-07 | 28368.91 | 6709 |

Наибольшая сумма платежей получена за **июль 2005 года**. Количество аренд подсчитано по `rental_date` за тот же календарный месяц. Группировка платежей учитывает одновременно месяц и год, поэтому июль разных лет не смешивается. Выручка предварительно суммируется отдельно от аренд: присоединение строк не увеличивает сумму платежей.

[SQL-файл](sql/03_highest_revenue_month.sql) · [Вывод MySQL](results/03_highest_revenue_month.txt)

Проверочные агрегаты по всем магазинам, фильмам, месяцам платежей и аренды: [verification.json](results/verification.json).

Источник учебной базы: [официальная инструкция Sakila](https://dev.mysql.com/doc/sakila/en/sakila-installation.html).
