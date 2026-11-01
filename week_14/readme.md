 #  LAB 14: Joining Tables

## What I did  (theory is below)
 
### 1. Setting up a separate schema
 
To keep my practice tables apart from the earlier labs, I worked in the `intro` database but created a new schema called `joins_lab` and made it the default:
 
```sql
DROP SCHEMA IF EXISTS joins_lab CASCADE;
CREATE SCHEMA joins_lab;
SET search_path = joins_lab, pg_catalog;
```
 
The first command printed `NOTICE: schema "joins_lab" does not exist, skipping`, because the schema was new.
 
### 2. Creating the tables and inserting data
 
I created `customers` and `orders`. Each order refers to a customer through a foreign key:
 
```sql
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    name TEXT,
    email TEXT
);
 
CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER REFERENCES customers,
    order_date DATE,
    total_amount NUMERIC(10,2)
);
```

```sql
INSERT INTO customers VALUES
(1, 'Alymbek', 'alymbek@example.com'),
(2, 'Timur', 'timur@example.com'),
(3, 'Beka', 'beka@example.com');
 
INSERT INTO orders VALUES
(1, 1, '2026-10-01', 150),
(2, 1, '2026-10-02', 50),
(3, 2, '2026-10-03', 200),
(4, NULL, '2026-10-04', 25);
```
 
### 3. INNER JOIN
 
Only customers who placed orders:
 
```sql
SELECT c.name, o.order_id, o.total_amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
ORDER BY o.order_id;
```
 
| name | order_id | total_amount |
|---|---|---|
| Alymbek | 1 | 150.00 |
| Alymbek | 2 | 50.00 |
| Timur | 3 | 200.00 |
 
Beka and the guest order are missing, because neither has a match.
 
### 4. LEFT JOIN
 
All customers, including Beka, who has no orders:
 
```sql
SELECT c.name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_id;
```
 
| name | order_id |
|---|---|
| Alymbek | 1 |
| Alymbek | 2 |
| Timur | 3 |
| Beka | NULL |
 
### 5. RIGHT JOIN
 
All orders, including the guest order that has no customer:
 
```sql
SELECT c.name, o.order_id
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY o.order_id;
```
 
| name | order_id |
|---|---|
| Alymbek | 1 |
| Alymbek | 2 |
| Timur | 3 |
| NULL | 4 |
 
### 6. FULL OUTER JOIN
 
Everything from both tables:
 
```sql
SELECT c.name, o.order_id
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_id;
```
 
| name | order_id |
|---|---|
| Alymbek | 1 |
| Alymbek | 2 |
| Timur | 3 |
| Beka | NULL |
| NULL | 4 |
 
### 7. Join with extra conditions
 
I added a `WHERE` to keep only orders after 2024-01-01 with an amount over 100:
 
```sql
SELECT c.name, o.order_date, o.total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= DATE '2024-01-01'
  AND o.total_amount > 100
ORDER BY o.order_id;
```
 
| name | order_date | total_amount |
|---|---|---|
| Alymbek | 2026-10-01 | 150.00 |
| Timur | 2026-10-03 | 200.00 |
 
### 8. Cleaning up
 
When I finished, I deleted the practice schema with all its tables:
 
```sql
DROP SCHEMA joins_lab CASCADE;
```




---
<br>




---
# THEORY
 
A `JOIN` combines rows from several tables using a related column, usually a foreign key matched to a primary key. It lets me "rebuild" information that was split across tables by normalization, for example customers next to their orders.
 
For extra practice, the course recommends interactive SQL problems on DataLemur.
 
## Types of joins
 
| Join | Returns |
|---|---|
| `INNER JOIN` | Only rows that match in **both** tables |
| `LEFT JOIN` | **All** rows from the left table + matches from the right (`NULL` if no match) |
| `RIGHT JOIN` | **All** rows from the right table + matches from the left (`NULL` if no match) |
| `FULL OUTER JOIN` | All rows from **both** tables, matched where possible (`NULL` otherwise) |
| `CROSS JOIN` | Every row of the first table combined with every row of the second (Cartesian product) |
 
The examples use two tables:
 
```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);
 
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers (customer_id),
    order_date DATE,
    total_amount DECIMAL(10,2)
);
```
 
### INNER JOIN
 
Only customers who placed orders:
 
```sql
SELECT c.name, c.email, o.order_date, o.total_amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```
 
### LEFT JOIN
 
All customers, including those without orders (their order columns are `NULL`):
 
```sql
SELECT c.name, c.email, o.order_date, o.total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```
 
### RIGHT JOIN
 
All orders, even if the customer is missing (customer columns are `NULL`):
 
```sql
SELECT c.name, c.email, o.order_date, o.total_amount
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```
 
### FULL OUTER JOIN
 
All customers and all orders, with `NULL` where there is no match:
 
```sql
SELECT c.name, c.email, o.order_date, o.total_amount
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;
```
 
### CROSS JOIN
 
Every customer paired with every product. It has no `ON` condition and can produce very large results, so use it carefully:
 
```sql
SELECT c.name, p.product_name
FROM customers c
CROSS JOIN products p;
```
 
## Advanced techniques
 
**Multiple tables** (chain the joins):
 
```sql
SELECT c.name, o.order_date, oi.quantity, p.product_name, p.price
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id;
```
 
**Self join** (a table joined with itself, e.g. employees and their managers):
 
```sql
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.employee_id;
```
 
**Join with extra conditions:**
 
```sql
SELECT c.name, o.order_date, o.total_amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2024-01-01' AND o.total_amount > 100;
```
 
## Joins and relationship types
 
| Relationship | How to join |
|---|---|
| One-to-one | `INNER JOIN` or `LEFT JOIN` (e.g. `users` and `user_profiles`) |
| One-to-many | `INNER JOIN`; the "one" side repeats for each match (e.g. `authors` and `books`) |
| Many-to-many | Join **through the junction table** (e.g. `students` → `enrollments` → `courses`) |
 
```sql
SELECT s.student_name, c.course_name, e.enrollment_date, e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE e.grade IS NOT NULL
ORDER BY s.student_name, c.course_name;
```
---
 
