# Домашнее задание к занятию «Индексы» - Дьяков Владимир

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

**Решение:**

```sql
SELECT ROUND(SUM(INDEX_LENGTH) / (SUM(INDEX_LENGTH + SUM(DATA_LENGTH)) * 100, 2) AS proc_index_data
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'sakila';
```

---

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

**Решение:**

Узкие места:

- последовательное сканирование таблиц payment, rental, customer, inventory;
- функция date() в условии фильтрации payment_date препятствует использованию индекса;
- множественные хеш-соединения увеличивают нагрузку на память и процессор;
- оконная функция sum() over() может быть неэффективной без правильной индексации;
- отсутствие явных JOIN может привести к неправильному планированию запроса.

Индексы:


```sql
CREATE INDEX idx_payment_date ON payment(payment_date);
CREATE INDEX idx_rental_date ON rental(rental_date);
CREATE INDEX idx_rental_customer ON rental(customer_id, inventory_id);
CREATE INDEX idx_inventory_film ON inventory(film_id);
```

Оптимизированный запрос:

```sql
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name), SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE p.payment_date >= '2005-07-30 00:00:00' AND p.payment_date < '2005-07-31 00:00:00';
```

---

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*

**Решение:**

Уникальные для PostgreSQL типы индексов:
- Hash индексы — оптимизированы исключительно для операций сравнения на равенство;
- GiST индексы — поддерживают многомерные и сложные типы данных;
- SP-GiST индексы — специализируются на иерархических структурах данных;
- GIN индексы — эффективны для составных значений и полнотекстового поиска;
- BRIN индексы — оптимизированы для больших наборов данных с корреляцией.
