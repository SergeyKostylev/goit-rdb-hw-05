## 1. Відображення таблиці `order_details` з `customer_id` з таблиці `orders`
Цей запит використовує вкладений запит у секції `SELECT`, щоб отримати `customer_id` з таблиці `orders` для кожного запису з таблиці `order_details`.

```sql
SELECT
  od.*,
  (SELECT o.customer_id FROM orders o WHERE o.id = od.order_id) AS customer_id
FROM order_details od;
```

## 2. Фільтрація `order_details` за `shipper_id=3`
Цей запит фільтрує записи з таблиці `order_details`, де відповідний запис у таблиці `orders` має `shipper_id=3`, використовуючи вкладений запит у секції `WHERE`.

```sql
SELECT * FROM order_details
WHERE order_id IN (
            SELECT id FROM orders WHERE shipper_id = 3
);
```

## 3. Середнє значення `quantity` для записів з `quantity > 10`
Цей запит використовує вкладений запит у секції `FROM`, щоб обрати записи з таблиці `order_details`, де `quantity > 10`, та обчислює середнє значення `quantity` для кожного `order_id`.

```sql
SELECT
  sub.order_id,
  AVG(sub.quantity) AS avg_quantity
FROM (
  SELECT * FROM order_details WHERE quantity > 10
) AS sub
GROUP BY sub.order_id;
```

## 4. Використання `WITH` для створення тимчасової таблиці
Цей запит виконує те саме завдання, що й попереднє, але використовує конструкцію `WITH` для створення тимчасової таблиці `temp`.

```sql
WITH temp AS (
  SELECT * FROM order_details WHERE quantity > 10
)
SELECT
  order_id,
  AVG(quantity) AS avg_quantity
FROM temp
GROUP BY order_id;
```

## 5. Функція для ділення чисел
Цей код створює функцію `divide_floats`, яка ділить перший параметр на другий, обидва з типом `FLOAT`. Функція включає перевірку ділення на нуль та застосовується до атрибута `quantity` таблиці `order_details`.

```sql
SET GLOBAL log_bin_trust_function_creators = 1;

DROP FUNCTION IF EXISTS divide_floats;

DELIMITER //
CREATE FUNCTION divide_floats(a FLOAT, b FLOAT)
RETURNS FLOAT
DETERMINISTIC
BEGIN
  IF b = 0 THEN
    RETURN NULL;
  END IF;
  RETURN a / b;
END;
//
DELIMITER ;

SELECT
  id,
  quantity,
  divide_floats(quantity, 2) AS divided_quantity
FROM order_details;
```