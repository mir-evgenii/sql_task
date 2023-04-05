## Задачи по SQL
---

## Задача 1

Есть таблица с некой историей изменений .
Необходимо собрать периоды "C" "ПО" в которых одно из полей field1 или field2  менялось. Группировка по полю name.

**Есть записи:**

| raisetime           | name       | field1 | field2 | field3 | field4 | field5 |
|---------------------|------------|--------|--------|--------|--------|--------|
| 2023-01-22 19:00:00 | test_name2 | 1      | 1      | 1      | 0      | 0      |
| 2023-01-22 20:00:00 | test_name2 | 1      | 0      | 1      | 0      | 0      |
| 2023-01-22 22:00:00 | test_name2 | 1      | 0      | 0      | 0      | 0      |
| 2023-01-23 00:00:00 | test_name2 | 1      | 0      | 0      | 0      | 0      |
| 2023-01-23 08:00:00 | test_name2 | 1      | 0      | 1      | 1      | 0      |
| 2023-01-23 09:00:00 | test_name2 | 1      | 1      | 1      | 1      | 0      |
| 2023-01-23 10:36:45 | test_name2 | 1      | 1      | 1      | 1      | 0      |
| 2023-01-23 10:38:00 | test_name2 | 0      | 1      | 1      | 1      | 0      |

**Необходимо получить:**

| name       | begin               | end                 | field1 | field2 |
|------------|---------------------|---------------------|--------|--------|
| test_name2 | 2023-01-22 19:00:00 | 2023-01-22 20:00:00 | 1      | 1      |
| test_name2 | 2023-01-22 20:00:00 | 2023-01-23 09:00:00 | 1      | 0      |
| test_name2 | 2023-01-23 09:00:00 | 2023-01-23 10:38:00 | 1      | 1      |
| test_name2 | 2023-01-23 10:38:00 | null                | 0      | 1      |

**Решение:**

```
SELECT name,
       raisetime as start,
       LEAD(raisetime) OVER (ORDER BY raisetime) as end,
       field1,
       field2
FROM (
      SELECT *,
             LAG(field1) OVER (PARTITION BY name ORDER BY raisetime) prev_field1,
             LAG(field2) OVER (PARTITION BY name ORDER BY raisetime) prev_field2
      FROM logs
     )
WHERE prev_field1 != field1 OR prev_field2 != field2 OR prev_field1 IS NULL OR prev_field2 IS NULL
ORDER BY raisetime;
```

---
## Задача 2

Спроектировать БД, хранящую данные о людях и их паспортах.
(в произвольном формате описать структуру таблиц, поля, индексы).
Написать SQL запросы, выводящие след. результаты:

1. Вывести список (ФИО) людей, имеющих более 1 ребенка в паспорте.
2. Средний возраст и кол-во людей, сменивших паспорт в заданный период времени.

**Решение:**

Схема БД:

![Схема БД](https://github.com/mir-evgenii/sql_task/blob/main/db.drawio.png)

1. Запрос:

```
SELECT CONCAT(p.first_name, ' ', p.middle_name, ' ', p.last_name)
FROM people AS p 
JOIN 
(SELECT p_id, count(c_id)
FROM children
GROUP BY p_id) AS c ON p.id = c.p_id
WHERE c.count > 1
GROUP BY p.first_name, p.middle_name, p.last_name;
```

2. Запрос:
```
SELECT AVG(date_part('year', age(p.b_date))), COUNT(p.id)
FROM people AS p JOIN passport AS ps ON p.id = ps.p_id
WHERE g_date BETWEEN '2000-01-01' AND '2020-12-31';
```
