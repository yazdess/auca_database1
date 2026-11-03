#  LAB 15: Advanced Querying

## What I did  (theory is below)

### 1. Setting up a separate schema

To keep my practice tables apart from the earlier labs, I worked in the `intro` database but created a new schema, `adv_lab`, and made it the default:

```sql
DROP SCHEMA IF EXISTS adv_lab CASCADE;
CREATE SCHEMA adv_lab;
SET search_path = adv_lab, pg_catalog;
```

The first command printed `NOTICE: schema "adv_lab" does not exist, skipping`, because the schema was new.

### 2. Subqueries

**Scalar subquery:** employees whose salary is above the company average:

```sql
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

| first_name | last_name | salary |
|---|---|---|
| Aigul | Sadykova | 150000.00 |
| Bek | Toktorov | 100000.00 |
| Carol | Smith | 120000.00 |
| David | Kim | 90000.00 |

**Table subquery with `IN`:** employees in departments where someone earns over 100000:

```sql
SELECT first_name, department
FROM employees
WHERE department IN (SELECT department FROM employees WHERE salary > 100000);
```

| first_name | department |
|---|---|
| Aigul | Management |
| Bek | Engineering |
| Carol | Engineering |
| David | Engineering |

### 3. CTE

```sql
WITH regional_sales AS (
    SELECT region_id, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region_id
)
SELECT region_id, total_sales
FROM regional_sales
WHERE total_sales > 1000;
```

| region_id | total_sales |
|---|---|
| 3 | 2000.00 |
| 1 | 1300.00 |

Region 2 (total 900) was filtered out.

### 4. Recursive CTE

I built the company hierarchy, starting from the top manager and adding the people who report to them, level by level:

```sql
WITH RECURSIVE org_chart AS (
    SELECT employee_id, first_name, last_name, manager_id
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.first_name, e.last_name, e.manager_id
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT * FROM org_chart;
```

| employee_id | first_name | last_name | manager_id |
|---|---|---|---|
| 1 | Aigul | Sadykova | NULL |
| 2 | Bek | Toktorov | 1 |
| 4 | Anna | Ivanova | 1 |
| 6 | Brian | Lee | 1 |
| 3 | Carol | Smith | 2 |
| 5 | Alice | Brown | 4 |
| 7 | David | Kim | 2 |

### 5. Set operations

**`UNION`:** all unique products from both warehouses (5 rows): Monitor, Mouse, Notebook, Book, Pen.

```sql
SELECT product_name FROM warehouse_1
UNION
SELECT product_name FROM warehouse_2;
```

**`UNION ALL`:** the same, but duplicates are kept (8 rows), so Book, Pen and Notebook appear twice.

**`INTERSECT`:** products out of stock in both warehouses:

```sql
SELECT product_id FROM warehouse_1 WHERE quantity = 0
INTERSECT
SELECT product_id FROM warehouse_2 WHERE quantity = 0;
```

Result: `1` (Book)

**`EXCEPT`:** products that are in `warehouse_1` but not in `warehouse_2`:

```sql
SELECT product_name FROM warehouse_1
EXCEPT
SELECT product_name FROM warehouse_2;
```

Result: `Monitor`

### 6. Window function: RANK

I ranked employees by salary inside each department. Every employee stays in the result:

```sql
SELECT first_name, last_name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_salary_rank
FROM employees;
```

| first_name | department | salary | dept_salary_rank |
|---|---|---|---|
| Carol | Engineering | 120000.00 | 1 |
| Bek | Engineering | 100000.00 | 2 |
| David | Engineering | 90000.00 | 3 |
| Brian | HR | 48000.00 | 1 |
| Aigul | Management | 150000.00 | 1 |
| Anna | Sales | 65000.00 | 1 |
| Alice | Sales | 55000.00 | 2 |

### 7. Pivot

I turned the years into columns with conditional aggregation:

```sql
SELECT product_name,
       SUM(CASE WHEN year = 2022 THEN sales_amount ELSE 0 END) AS year_2022,
       SUM(CASE WHEN year = 2023 THEN sales_amount ELSE 0 END) AS year_2023,
       SUM(CASE WHEN year = 2024 THEN sales_amount ELSE 0 END) AS year_2024
FROM sales
GROUP BY product_name;
```

| product_name | year_2022 | year_2023 | year_2024 |
|---|---|---|---|
| Pen | 50.00 | 60.00 | 80.00 |
| Book | 100.00 | 150.00 | 200.00 |

### 8. FILTER with window functions

```sql
SELECT employee_id,
       SUM(salary) OVER (PARTITION BY department) AS total_dept_salary,
       AVG(salary) FILTER (WHERE tenure > 5) OVER () AS avg_salary_senior
FROM employees;
```

The query returned all 7 employees. `total_dept_salary` is the same for everyone in a department, and `avg_salary_senior` is the same for all rows:

| Department | total_dept_salary | avg_salary_senior |
|---|---|---|
| Engineering | 310000.00 | 104500.00 |
| HR | 48000.00 | 104500.00 |
| Management | 150000.00 | 104500.00 |
| Sales | 120000.00 | 104500.00 |

`avg_salary_senior` is the average salary of employees with more than 5 years of tenure (Aigul, Bek, Carol and Brian).

### 9. Custom sorting with CASE

I forced my own order of the statuses instead of alphabetical:

```sql
SELECT product_name, status
FROM orders
ORDER BY
    CASE status
        WHEN 'High Priority' THEN 1
        WHEN 'Medium Priority' THEN 2
        WHEN 'Low Priority' THEN 3
        ELSE 4
    END;
```

| product_name | status |
|---|---|
| Pen | High Priority |
| Monitor | High Priority |
| Book | Medium Priority |
| Book | Low Priority |
| Notebook | Cancelled |





---
<br>




---
# THEORY
 
 
## Subqueries

A **subquery** is a query nested inside another query. The inner result is used by the outer query.

| Type | Returns | Used with |
|---|---|---|
| Scalar | One value (1 row, 1 column) | `=`, `>`, `<` |
| Row | One row, several columns | Row comparisons |
| Table | Several rows and columns | `IN`, `EXISTS`, `FROM` |

Employees whose salary is above the company average:

```sql
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);   -- scalar subquery
```

## CTEs (WITH clause)

A CTE names a subquery so the main query can use it, like a temporary view that lives for one query. It makes complex queries readable.

```sql
WITH regional_sales AS (
    SELECT region_id, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region_id
)
SELECT region_id, total_sales
FROM regional_sales
WHERE total_sales > 1000000;
```

## Recursive queries

A **recursive CTE** queries hierarchical (tree) data. It has two parts:

1. **Anchor:** the starting rows.
2. **Recursive member:** repeats using the previous result, until no new rows appear.

```sql
WITH RECURSIVE org_chart AS (
    SELECT employee_id, first_name, last_name, manager_id      -- anchor: top manager
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.first_name, e.last_name, e.manager_id   -- everyone who reports to them
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT * FROM org_chart;
```

## Set operations

They combine the results of several `SELECT` queries.

| Operation | Result |
|---|---|
| `UNION` | All rows, duplicates removed |
| `UNION ALL` | All rows, duplicates kept (faster) |
| `INTERSECT` | Only rows present in both results |
| `EXCEPT` | Rows from the first query not present in the second (`MINUS` in some other databases) |

```sql
SELECT product_name FROM warehouse_1
UNION
SELECT product_name FROM warehouse_2;

SELECT product_id FROM warehouse_1 WHERE quantity = 0
INTERSECT
SELECT product_id FROM warehouse_2 WHERE quantity = 0;
```

**Pivot** (rows → columns):

```sql
SELECT product_name,
       SUM(CASE WHEN year = 2022 THEN sales_amount ELSE 0 END) AS year_2022,
       SUM(CASE WHEN year = 2023 THEN sales_amount ELSE 0 END) AS year_2023,
       SUM(CASE WHEN year = 2024 THEN sales_amount ELSE 0 END) AS year_2024
FROM sales
GROUP BY product_name;
```

## Complex filtering and sorting

**`FILTER`:** applies an aggregate to a subset of rows, also usable with window functions.

```sql
SELECT employee_id,
       SUM(salary) OVER (PARTITION BY department) AS total_dept_salary,
       AVG(salary) FILTER (WHERE tenure > 5) OVER () AS avg_salary_senior
FROM employees;
```

**Custom sorting with `CASE`:** forces a non-alphabetical order.

```sql
SELECT product_name, status
FROM orders
ORDER BY CASE status
    WHEN 'High Priority' THEN 1
    WHEN 'Medium Priority' THEN 2
    WHEN 'Low Priority' THEN 3
    ELSE 4
END;
```
---
