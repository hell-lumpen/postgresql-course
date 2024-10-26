### Лекция по PostgreSQL: Сложные запросы, подзапросы, JOIN, CTE и оптимизация

#### Введение

Сегодняшняя тема — сложные SQL-запросы в PostgreSQL. Мы рассмотрим такие важные аспекты, как подзапросы, объединения таблиц (JOIN), общие табличные выражения (CTE) и способы оптимизации запросов. Особое внимание будет уделено тонкостям использования этих инструментов, их влиянию на производительность и практическим рекомендациям по написанию высокоэффективных запросов. В конце лекции мы также разберем использование `EXPLAIN ANALYZE` для анализа и оптимизации запросов.

---

### 1. Подзапросы (Subqueries)

#### Теория

**Подзапрос** — это запрос, который выполняется внутри другого запроса и возвращает результат, используемый в основном запросе. Подзапросы могут быть скалярными (возвращают одно значение), или множественными (возвращают набор данных). PostgreSQL поддерживает вложенные подзапросы практически в любом месте SQL-запроса: в `SELECT`, `FROM`, `WHERE`, `JOIN` и других частях.

Подзапросы особенно полезны, когда данные из одного запроса должны быть использованы в качестве фильтра или аргумента для другого запроса.

#### Виды подзапросов

1. **Скалярные подзапросы**: возвращают одно значение.
   Пример: вычисление количества заказов для каждого клиента.

   ```sql
   SELECT first_name, last_name,
          (SELECT COUNT(*) FROM orders WHERE customers.id = orders.customer_id) AS order_count
   FROM customers;
   ```

   Здесь подзапрос возвращает количество заказов для каждого клиента.

2. **Множественные подзапросы**: возвращают несколько строк.
   Пример: выборка клиентов, которые сделали заказы на сумму более 1000.

   ```sql
   SELECT first_name, last_name
   FROM customers
   WHERE id IN (SELECT customer_id FROM orders WHERE amount > 1000);
   ```

   В данном примере подзапрос возвращает список `customer_id` из таблицы `orders`, которые затем используются для фильтрации в основном запросе.

3. **Подзапросы в секции `FROM` (Derived Tables)**: временные таблицы, созданные подзапросом, используются в основном запросе.
   Пример: получение статистики по заказам.

   ```sql
   SELECT *
   FROM (SELECT customer_id, COUNT(*) AS order_count FROM orders GROUP BY customer_id) AS order_stats
   WHERE order_count > 10;
   ```

   В этом примере подзапрос создает временную таблицу с результатами агрегирования по количеству заказов на каждого клиента.

#### Практические тонкости и советы

1. **Когда использовать подзапросы:**
   - Используйте подзапросы, когда вычисления или фильтрации невозможно или неудобно сделать на одном уровне запроса.
   - Хороший пример — когда вам нужно выполнить промежуточные вычисления для дальнейшей фильтрации или группировки.

2. **Избегайте избыточных подзапросов:** Если подзапрос повторяется несколько раз, это может привести к его многократному выполнению, что замедлит работу. Рассмотрите использование `JOIN` или CTE для избежания повторяющихся подзапросов.

3. **Скалярные подзапросы в `SELECT`:** могут замедлять выполнение запросов, если они зависят от больших таблиц. В таких случаях лучше использовать `JOIN` для получения тех же данных, но более эффективно.

4. **Подзапросы в `WHERE`:** когда вы используете подзапросы в `WHERE` для фильтрации, убедитесь, что данные подзапроса могут быть эффективно получены с помощью индексов. В противном случае может выполняться полный скан таблицы (Seq Scan).

#### Антипаттерны

1. **Использование подзапросов вместо `JOIN`:** Когда возможно, лучше использовать `JOIN` для объединения данных из нескольких таблиц, а не подзапросы в `WHERE`. Это снизит количество выполнений подзапроса.

2. **Избегайте коррелированных подзапросов:** Это подзапросы, которые выполняются для каждой строки основного запроса. Они могут быть очень медленными при больших объемах данных.
   Пример коррелированного подзапроса:

   ```sql
   SELECT customer_id, (SELECT SUM(amount) FROM orders WHERE orders.customer_id = customers.id) AS total_amount
   FROM customers;
   ```

   В этом примере подзапрос выполняется для каждой строки основной таблицы.

#### Оптимизация

- Рассмотрите возможность использования индексов для столбцов, участвующих в подзапросах, чтобы улучшить производительность.
- Подумайте о преобразовании сложных подзапросов в `JOIN` или использование CTE для повышения читаемости и производительности.

---

### 2. JOIN

#### Теория

**JOIN** — это операция объединения данных из двух или более таблиц на основе условия. В PostgreSQL поддерживаются различные типы `JOIN`, что позволяет решать разнообразные задачи, такие как фильтрация данных, агрегация и работа с разнородными данными.

#### Виды `JOIN`

1. **INNER JOIN**: Возвращает только те строки, которые соответствуют условию соединения.

   Пример:

   ```sql
   SELECT customers.first_name, customers.last_name, orders.order_id
   FROM customers
   INNER JOIN orders ON customers.id = orders.customer_id;
   ```

2. **LEFT JOIN**: Возвращает все строки из левой таблицы и совпадающие строки из правой таблицы. Если совпадений нет, возвращаются `NULL`.

   Пример:

   ```sql
   SELECT customers.first_name, orders.order_id
   FROM customers
   LEFT JOIN orders ON customers.id = orders.customer_id;
   ```

3. **RIGHT JOIN**: Аналогично `LEFT JOIN`, но возвращает все строки из правой таблицы.
   
   Пример:

   ```sql
   SELECT orders.order_id, customers.first_name
   FROM orders
   RIGHT JOIN customers ON orders.customer_id = customers.id;
   ```

4. **FULL OUTER JOIN**: Возвращает все строки, которые есть хотя бы в одной из таблиц. Если нет совпадений, возвращаются `NULL`.

   Пример:

   ```sql
   SELECT customers.first_name, orders.order_id
   FROM customers
   FULL OUTER JOIN orders ON customers.id = orders.customer_id;
   ```

5. **CROSS JOIN**: Выполняет декартово произведение, т.е. соединяет каждую строку из первой таблицы с каждой строкой из второй.

   Пример:

   ```sql
   SELECT customers.first_name, orders.order_id
   FROM customers
   CROSS JOIN orders;
   ```

#### Практические тонкости и советы

1. **Используйте правильный тип JOIN для вашей задачи:**
   - `INNER JOIN` полезен, когда вам нужны только совпадающие строки.
   - `LEFT JOIN` полезен, когда вам нужны все строки из левой таблицы, даже если нет совпадений в правой таблице.

2. **Оптимизация с использованием индексов:**
   - Убедитесь, что столбцы, по которым выполняется соединение, индексированы. Это может значительно ускорить выполнение запроса.
   - Пример создания индекса для улучшения производительности `JOIN`:
     ```sql
     CREATE INDEX idx_customer_id ON orders(customer_id);
     ```

3. **Избегайте использования `JOIN` с большим количеством таблиц:** При большом количестве соединений может возникнуть проблема производительности, так как каждый `JOIN` увеличивает объем данных для обработки.

#### Антипаттерны

1. **Избыточное использование `LEFT JOIN`:** Если результат запроса вам требуется только при наличии совпадений в обеих таблицах, не используйте `LEFT JOIN` — это приведет к неоправданным затратам ресурсов.
   
2. **Избегайте `CROSS JOIN`, если это не специально задумано:** `CROSS JOIN` создает полное декартово произведение таблиц, что может привести к огромному количеству строк и замедлить выполнение запроса.

3. **Неиспользование индексов:** Если столбцы, участвующие в соединении, не индексированы, то выполнение запроса будет замедлено из-за необходимости последовательного сканирования таблиц.

#### Оптимизация

- Используйте индексы на столбцах, участвующих в `JOIN`.
- В случае сложных запросов рассмотрите возможность использования CTE для упрощения логики.
- Проверяйте планы выполнения запросов с помощью `EXPLAIN ANALYZE`, чтобы убедиться, что запросы выполняются оптимально.

---

### 3. Общие табличные выражения (CTE)

#### Теория

**Общие табличные выражения (CTE)** — это временные именованные результаты, определяемые в запросах с помощью ключевого слова `WITH`. CTE позволяют разбивать сложные запросы на части, делая их более понятными и поддерживаемыми. В отличие

 от подзапросов в `FROM`, CTE могут ссылаться друг на друга и, что особенно важно, могут быть рекурсивными.

#### Пример CTE

```sql
WITH order_totals AS (
  SELECT customer_id, SUM(amount) AS total_amount
  FROM orders
  GROUP BY customer_id
)
SELECT customers.first_name, order_totals.total_amount
FROM customers
JOIN order_totals ON customers.id = order_totals.customer_id
WHERE order_totals.total_amount > 500;
```

#### Рекурсивный CTE

Рекурсивные CTE особенно полезны при работе с иерархическими данными, такими как структура подчиненности в компаниях.

Пример:

```sql
WITH RECURSIVE subordinates AS (
  SELECT employee_id, manager_id, full_name
  FROM employees
  WHERE manager_id IS NULL
  UNION ALL
  SELECT e.employee_id, e.manager_id, e.full_name
  FROM employees e
  JOIN subordinates s ON s.employee_id = e.manager_id
)
SELECT * FROM subordinates;
```

#### Практические тонкости и советы

1. **Использование CTE для разбиения сложных запросов:** CTE позволяют разделить сложные запросы на более простые шаги. Это улучшает читаемость и поддержку запросов.

2. **Рекурсивные CTE:** могут быть полезны для обработки иерархических данных, таких как структура сотрудников или папок.

3. **Ограничение итераций в рекурсивных CTE:** Будьте осторожны при использовании рекурсивных CTE, так как они могут бесконечно зациклиться. Всегда добавляйте ограничение на количество итераций.

#### Антипаттерны

1. **Избыточное использование CTE:** В некоторых случаях использование CTE может замедлять выполнение запроса, так как CTE выполняются как временные таблицы. Если CTE используется несколько раз в одном запросе, это может привести к его многократному выполнению.
   
2. **Рекурсия без ограничений:** Рекурсивные CTE могут стать причиной больших накладных расходов, если не ограничить количество итераций.

---

### 4. Оптимизация запросов с помощью EXPLAIN ANALYZE

#### Теория

`EXPLAIN` — это команда, которая показывает, как PostgreSQL планирует выполнить запрос. Она помогает выявить, какие шаги выполняются, какие индексы используются и как обрабатываются данные. 

`EXPLAIN ANALYZE` выполняет запрос и выводит реальный план выполнения с метриками времени.

#### Пример использования EXPLAIN

```sql
EXPLAIN SELECT * FROM customers WHERE id = 1;
```

#### Пример использования EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE SELECT * FROM customers WHERE id = 1;
```

---

#### Заключение

Сложные запросы требуют внимания к деталям, особенно когда речь идет о производительности. Подзапросы, `JOIN`, CTE и другие инструменты PostgreSQL должны использоваться с умом, чтобы избежать узких мест и перегрузки базы данных. Оптимизация с помощью инструментов, таких как `EXPLAIN ANALYZE`, позволяет точно определить, где запрос можно улучшить, что делает PostgreSQL мощным инструментом для анализа и обработки данных.

### Задача 1. Суммарная стоимость заказов для каждого клиента

**Задача:** Для каждого клиента вывести общее количество заказов и суммарную стоимость всех заказов.

#### Решение с использованием `JOIN`:

```sql
SELECT c.id, c.first_name, c.last_name, COUNT(o.id) AS order_count, SUM(o.total_amount) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.first_name, c.last_name;
```

#### Решение с использованием подзапросов:

```sql
SELECT c.id, c.first_name, c.last_name,
       (SELECT COUNT(*) FROM orders WHERE orders.customer_id = c.id) AS order_count,
       (SELECT SUM(total_amount) FROM orders WHERE orders.customer_id = c.id) AS total_spent
FROM customers c;
```

**Вывод и наилучшее решение:**

Использование **`JOIN`** более предпочтительно в данном случае:

- **Преимущества**:
  - **Производительность**: В этом решении выполняется одно объединение с таблицей `orders`, и данные агрегируются сразу. При подзапросах же каждый раз выполняется отдельный запрос для подсчета заказов и суммарной стоимости для каждого клиента, что накладывает дополнительную нагрузку на базу данных.
  - **Читаемость**: Запрос с `JOIN` проще понять, так как агрегации видны сразу и логика более линейна.

- **Недостатки подзапросов**:
  - **Повторное выполнение**: Один и тот же подзапрос (к таблице `orders`) выполняется дважды, что снижает производительность.
  - **Проблемы с масштабируемостью**: В больших базах данных подзапросы могут значительно замедлять выполнение запросов из-за повторной выборки данных для каждого клиента.

---

### Задача 2. Список товаров, которые были заказаны более 10 раз

**Задача:** Найти все товары, которые были заказаны более 10 раз.

#### Решение с использованием подзапроса:

```sql
SELECT p.id, p.name
FROM products p
WHERE (SELECT SUM(oi.quantity) FROM order_items oi WHERE oi.product_id = p.id) > 10;
```

#### Решение с использованием `JOIN` и группировки:

```sql
SELECT p.id, p.name, SUM(oi.quantity) AS total_quantity
FROM products p
JOIN order_items oi ON p.id = oi.product_id
GROUP BY p.id, p.name
HAVING SUM(oi.quantity) > 10;
```

**Вывод и наилучшее решение:**

Использование **`JOIN` и группировки** является оптимальным решением:

- **Преимущества**:
  - **Производительность**: `JOIN` с последующей группировкой по продуктам позволяет один раз собрать данные и сразу агрегировать их по каждому продукту, в то время как подзапрос для каждого товара выполняется отдельно.
  - **Читаемость**: Запрос с использованием группировки лучше отражает бизнес-логику задачи — суммирование количества по каждому продукту. В случае подзапроса логика разбивается на части, что затрудняет понимание запроса.

- **Недостатки подзапросов**:
  - **Плохая производительность**: Подзапрос для каждого товара создает значительную нагрузку на базу данных, особенно при большом количестве записей.
  - **Повторение логики**: Подзапрос сложно масштабировать или расширить для дополнительных условий, например, если нужно учитывать еще и временной диапазон.

---

### Задача 3. Найти категории товаров, чья суммарная стоимость заказов превышает $5000

**Задача:** Найти категории товаров, общая стоимость заказанных товаров которых превышает $5000.

#### Решение с использованием CTE:

```sql
WITH category_totals AS (
    SELECT pc.category_id, SUM(oi.quantity * oi.price) AS total_sales
    FROM order_items oi
    JOIN products p ON oi.product_id = p.id
    JOIN product_categories pc ON p.id = pc.product_id
    GROUP BY pc.category_id
)
SELECT c.id, c.name, ct.total_sales
FROM categories c
JOIN category_totals ct ON c.id = ct.category_id
WHERE ct.total_sales > 5000;
```

#### Решение с использованием подзапроса:

```sql
SELECT c.id, c.name,
       (SELECT SUM(oi.quantity * oi.price)
        FROM order_items oi
        JOIN products p ON oi.product_id = p.id
        JOIN product_categories pc ON p.id = pc.product_id
        WHERE pc.category_id = c.id) AS total_sales
FROM categories c
WHERE (SELECT SUM(oi.quantity * oi.price)
       FROM order_items oi
       JOIN products p ON oi.product_id = p.id
       JOIN product_categories pc ON p.id = pc.product_id
       WHERE pc.category_id = c.id) > 5000;
```

**Вывод и наилучшее решение:**

Использование **CTE** является более предпочтительным:

- **Преимущества**:
  - **Производительность**: CTE сначала рассчитывает суммы для всех категорий в одном запросе, а затем можно выбирать или фильтровать данные по этим результатам. Это особенно полезно при сложных или больших наборах данных, где подзапросы могут многократно пересчитываться.
  - **Читаемость и поддержка**: CTE позволяет отделить логику подсчета суммы продаж от основного запроса. Это упрощает читаемость и позволяет легко модифицировать или расширить запрос.

- **Недостатки подзапросов**:
  - **Медленная работа**: Подзапросы выполняются для каждой категории, что приводит к значительной задержке при большом количестве данных.
  - **Сложность в управлении**: Подзапросы более сложно поддерживать и изменять, особенно если требуется внести дополнительные условия или изменения.

---

### Задача 4. Иерархия подчиненности клиентов (реферальная структура)

**Задача:** Построить иерархию клиентов, показывая всех клиентов, приглашенных одним человеком и их подчиненных.

#### Решение с использованием рекурсивного CTE:

```sql
WITH RECURSIVE referral_tree AS (
    SELECT id, first_name, last_name, NULL::INT AS referrer_id, 0 AS level
    FROM customers
    WHERE referrer_id IS NULL  -- корневые клиенты
    UNION ALL
    SELECT c.id, c.first_name, c.last_name, c.referrer_id, rt.level + 1
    FROM customers c
    JOIN referral_tree rt ON c.referrer_id = rt.id
)
SELECT * FROM referral_tree;
```

**Вывод и наилучшее решение:**

Использование **рекурсивного CTE** — единственное подходящее решение:

- **Преимущества**:
  - **Естественное представление иерархий**: Рекурсивный CTE идеально подходит для работы с иерархическими данными, такими как реферальные структуры, и позволяет легко строить сложные древовидные структуры.
  - **Читаемость и поддержка**: CTE позволяет писать запросы, которые логически разделены на рекурсивные шаги. Это делает код более понятным, особенно при анализе иерархий.

- **Недостатков** почти нет, однако:
  - **Проблемы с производительностью**: При слишком глубокой или большой иерархии рекурсивные запросы могут стать медленными. Чтобы избежать бесконечной рекурсии, рекомендуется использовать ограничение на глубину иерархии.

---

### Задача 5. Найти самых активных клиентов, сделавших заказы за последние 30 дней

**Задача:** Найти клиентов, которые сделали наибольшее количество заказов за последние 30 дней.

#### Решение с использованием `JOIN` и подзапроса:

```sql
SELECT c.id, c.first_name, c.last_name, order_count
FROM customers c
JOIN (
    SELECT customer_id, COUNT(*) AS order_count
    FROM orders
    WHERE order_date > NOW() - INTERVAL '30 days'
    GROUP BY customer_id
    ORDER BY order_count DESC
    LIMIT 10
) AS top_customers ON c.id = top_customers.customer_id;
```

#### Решение с использованием CTE:

```sql
WITH recent_orders AS (
    SELECT customer_id, COUNT(*) AS order_count
    FROM orders
    WHERE order_date > NOW() - INTERVAL '30 days'
    GROUP BY customer_id
)
SELECT c.id, c.first_name, c.last_name, ro.order_count
FROM customers c
JOIN recent_orders ro ON c.id = ro.customer_id
ORDER BY ro.order_count DESC
LIMIT 10;
```

**Вывод и наилучшее решение:**

Оба решения **похожи по эффективности**, но использование CTE предпочтительнее:

- **Преимущества CTE**:
  - **Читаемость**: Запрос с CTE проще понять и поддерживать, так как CTE выделяет отдельный логический шаг — подсчет заказов за последние 30 дней.
  -

 **Расширяемость**: Если потребуется усложнение логики, например, добавление дополнительных условий к подсчету заказов, CTE позволяет легко это сделать, не перегружая основной запрос.

- **Недостатки подзапроса**:
  - **Сложность изменения**: Подзапросы более громоздкие и сложные для изменений или добавления новых условий.
 
### Сравнительная таблица по подзапросам, JOIN и CTE (включая рекурсивные)

| Инструмент        | Определение | Советы по использованию | Тонкости использования | Примеры задач, где лучше использовать |
|-------------------|-------------|-------------------------|------------------------|---------------------------------------|
| **Подзапросы**    | Подзапрос — это вложенный запрос, который выполняется отдельно и передает результат в основной запрос. Подзапрос может возвращать одно или несколько значений. | - Используйте подзапросы, если требуется вычисление, которое нельзя просто включить в основной запрос.<br>- Подзапросы подходят для фильтрации или вычисления значений, которые используются в других частях запроса.<br>- Подходят, когда нужно изолировать логику. | - Может сильно замедлить запрос, так как каждый подзапрос выполняется независимо для каждой строки.<br>- Можно использовать в `SELECT`, `FROM`, и `WHERE` частях запроса.<br>- Подзапросы бывают зависимыми и независимыми: зависимые подзапросы зависят от внешнего запроса и выполняются многократно, что может быть неэффективно.<br>- Читаемость запроса усложняется, если в нем много вложенных подзапросов. | - Найти клиентов, которые совершили заказ больше 5 раз, используя подзапрос в `WHERE`.<br>- Найти товары, которых было продано более 10 штук, с подзапросом в `SELECT` для подсчета общего количества заказанных единиц. |
| **JOIN**          | `JOIN` объединяет строки из двух или более таблиц на основе связанного между ними поля (например, по первичному и внешнему ключу). Это позволяет работать с несколькими таблицами в одном запросе. | - Используйте `JOIN` для работы с несколькими таблицами одновременно.<br>- Подходит для агрегации данных и соединения данных по схожим полям.<br>- Рекомендуется использовать индексы на полях, по которым производится `JOIN`, чтобы избежать проблем с производительностью.<br>- Оптимальный вариант для выборок и агрегирования данных из нескольких таблиц. | - `INNER JOIN` выбирает только те строки, которые имеют совпадения в обеих таблицах.<br>- `LEFT JOIN` позволяет сохранить все строки из левой таблицы, даже если нет совпадений в правой таблице.<br>- Большие таблицы без индексов на полях для `JOIN` могут значительно замедлить запрос.<br>- При работе с большим числом таблиц могут возникнуть сложности с пониманием структуры запроса. | - Вывести список клиентов и их заказов.<br>- Найти сумму продаж товаров в каждой категории.<br>- Определить, сколько товаров каждой категории продано за последний месяц, используя `JOIN` между таблицами продуктов, категорий и заказов. |
| **CTE**           | CTE (Common Table Expression) — это временный набор данных, который определен в начале запроса с помощью `WITH`. Используется для разделения сложных запросов на более простые части. | - Используйте CTE, если нужно улучшить читаемость сложных запросов.<br>- Подходит для разделения сложных логических шагов.<br>- Можно использовать несколько CTE в одном запросе, что улучшает модульность.<br>- Подходит для случаев, когда нужно многократно использовать одни и те же данные в разных частях запроса. | - CTE — это временная таблица, которая существует только в рамках одного запроса.<br>- Выполняется один раз за запрос, даже если используется несколько раз (в отличие от подзапросов, которые могут выполняться многократно).<br>- Упрощает управление сложной логикой запросов, улучшая читаемость.<br>- Может быть менее производительным, если запросы требуют высокой частоты обновлений данных (из-за временной природы CTE). | - Построение списка товаров и категорий с общей суммой заказов для каждой категории.<br>- Поиск суммарных продаж за последний месяц для каждого клиента с делением логики на несколько шагов.<br>- Подходит, когда в запросе требуется использование одной и той же логики в разных частях, например, для подсчета суммы заказов и общего количества заказов одновременно. |
| **Рекурсивный CTE**| Рекурсивный CTE — это CTE, который сам вызывает себя до тех пор, пока не выполнится определенное условие остановки. Обычно используется для работы с иерархическими данными. | - Используйте рекурсивные CTE для работы с иерархиями или деревьями (например, структура реферальных клиентов, иерархия категорий).<br>- Подходит для задач, где необходима рекурсивная обработка данных (например, вычисление пути в графе). | - Рекурсивные CTE используют два выражения: начальное и рекурсивное, которое выполняется до тех пор, пока не выполнится условие остановки.<br>- Рекомендуется ограничить глубину рекурсии для предотвращения бесконечных циклов.<br>- На больших объемах данных может быть медленным, особенно при работе с глубокими иерархиями. | - Построение структуры подчиненных клиентов в реферальной программе.<br>- Вычисление иерархии категорий товаров и их подкатегорий.<br>- Поиск кратчайшего пути в графе (например, в логистических задачах). |

### Подробные объяснения:

#### **Подзапросы:**
Подзапросы — это инструмент для изоляции логики внутри более крупного запроса. Они особенно полезны, когда требуется предварительно вычислить определенные значения или условия для основной выборки данных. Например, подзапросы могут использоваться в `SELECT` для добавления вычисляемых полей, в `WHERE` для фильтрации или даже в `FROM` для создания временной таблицы.

- **Плюсы:**
  - Легко изолировать сложные логики.
  - Позволяют использовать результаты вычислений в других частях запроса.
  
- **Минусы:**
  - Низкая производительность при повторном выполнении подзапросов.
  - Усложнение запроса при большой вложенности.

#### **JOIN:**
`JOIN` позволяет объединять таблицы по связанным полям. Это основной инструмент для работы с реляционными базами данных, так как именно в этом и заключается их суть — связывать различные сущности.

- **Плюсы:**
  - Высокая производительность при наличии индексов.
  - Четкая структура данных с возможностью группировки и фильтрации.
  
- **Минусы:**
  - Без индексов `JOIN` может значительно замедлить работу.
  - Усложненные запросы при большом количестве таблиц.

#### **CTE (Common Table Expression):**
CTE позволяет улучшить читаемость и структурировать сложные запросы. Они не влияют на производительность напрямую, но делают код более модульным и понятным.

- **Плюсы:**
  - Легкость понимания и поддержки сложных запросов.
  - Выполняется один раз за запрос, что улучшает производительность по сравнению с подзапросами.
  
- **Минусы:**
  - Может быть менее производительным при работе с очень большими данными или частыми обновлениями.

#### **Рекурсивный CTE:**
Рекурсивный CTE используется для решения задач, требующих рекурсивной обработки данных, таких как иерархии или графы. Это мощный инструмент, когда требуется вычисление на нескольких уровнях иерархии, например, в задачах с реферальными структурами или древовидными данными.

- **Плюсы:**
  - Эффективен для работы с иерархическими данными.
  - Удобен для рекурсивных вычислений.
  
- **Минусы:**
  - Потенциально медленная работа с глубокими иерархиями.
  - Риск бесконечных циклов без ограничения глубины рекурсии.

### Вывод:
Каждый из этих инструментов имеет свое назначение и должен применяться в зависимости от задачи:

- **Подзапросы** хороши для простых изолированных логик, но следует избегать их при работе с большими объемами данных.
- **JOIN** является стандартным инструментом для объединения таблиц, особенно полезен при агрегации данных.
- **CTE** отлично подходит для сложных многошаговых запросов, улучшая их читаемость и поддержку.
- **Рекурсивный CTE** — лучший выбор для работы с иерархиями и рекурсивными структурами данных.