# LAB 12: Querying Data
## What I did. (theory is below)

In the `intro` database, I created all the tables for this lab and inserted the sample data. The employee data is in a table named `employees_sales`. Then I ran the queries one by one.

### 1. SELECT with a calculation

```sql
SELECT name, salary, salary * 0.10 AS potential_bonus FROM employees_sales;
```

| name | salary | potential_bonus |
|---|---|---|
| Alice | 55000.00 | 5500.0000 |
| Bob | 90000.00 | 9000.0000 |
| Carol | 120000.00 | 12000.0000 |
| Anna | 65000.00 | 6500.0000 |
| Brian | 48000.00 | 4800.0000 |
| David | 70000.00 | 7000.0000 |

### 2. WHERE with AND

```sql
SELECT name FROM employees_sales
WHERE salary > 75000 AND department = 'Engineering';
```

Result: **Bob**, **Carol** (2 rows)

### 3. WHERE with BETWEEN

```sql
SELECT product_name, price FROM products WHERE price BETWEEN 20 AND 50;
```

| product_name | price |
|---|---|
| Strawberry | 25.00 |
| Blueberry | 30.00 |
| Headphones | 45.00 |

### 4. LIKE

```sql
SELECT customer_name FROM customers WHERE customer_name LIKE 'Bra%';
```

Result: **Brad Miller**, **Brandon Lee** (2 rows)

### 5. Regular expression

```sql
SELECT name FROM employees_sales WHERE name ~ '^[AB]';
```

Result: **Alice**, **Bob**, **Anna**, **Brian** (4 rows). These are all the names that start with A or B.

### 6. CASE

```sql
SELECT name, salary,
  CASE
    WHEN salary > 100000 THEN 'Senior'
    WHEN salary BETWEEN 60000 AND 100000 THEN 'Mid-Level'
    ELSE 'Junior'
  END AS employee_level
FROM employees_sales;
```

| name | salary | employee_level |
|---|---|---|
| Alice | 55000.00 | Junior |
| Bob | 90000.00 | Mid-Level |
| Carol | 120000.00 | Senior |
| Anna | 65000.00 | Mid-Level |
| Brian | 48000.00 | Junior |
| David | 70000.00 | Mid-Level |

### 7. CTE

```sql
WITH department_averages AS (
  SELECT department, AVG(salary) AS avg_salary
  FROM employees_sales
  GROUP BY department
)
SELECT e.name, e.salary, e.department, da.avg_salary
FROM employees_sales e
JOIN department_averages da ON e.department = da.department
WHERE e.salary > da.avg_salary;
```

Only employees who earn more than their department's average were returned:

| name | salary | department | avg_salary |
|---|---|---|---|
| Carol | 120000.00 | Engineering | 105000.00 |
| Anna | 65000.00 | Sales | 60000.00 |


---
<br>




---
# THEORY

## SELECT

`SELECT` chooses which columns to retrieve from a table.

```sql
SELECT column1, column2 FROM table_name;
```

```sql
SELECT name, salary, salary * 0.10 AS potential_bonus
FROM employees;                           
```

`AS` gives a column or calculation a new name in the output.

## WHERE

`WHERE` filters rows and comes after `FROM`.

| Type | Operators |
|---|---|
| Comparison | `=`, `<>` or `!=`, `>`, `<`, `>=`, `<=` |
| Logical | `AND`, `OR`, `NOT` |
| Range | `BETWEEN` (inclusive) |
| Null check | `IS NULL`, `IS NOT NULL` |

```sql
SELECT name, salary FROM employees WHERE department = 'Sales';

SELECT name FROM employees
WHERE salary > 75000 AND department = 'Engineering';

SELECT product_name, price FROM products
WHERE price BETWEEN 20 AND 50;
```

>  to check for empty values use `IS NULL`, never `= NULL`.

## Pattern matching: LIKE and ILIKE

| Symbol | Meaning |
|---|---|
| `%` | Zero, one or many characters |
| `_` | Exactly one character |
| `ILIKE` | Case-insensitive version of `LIKE` (PostgreSQL only) |

```sql
SELECT customer_name FROM customers WHERE customer_name LIKE 'Bra%';     -- starts with Bra
SELECT product_name FROM products WHERE product_name LIKE '%berry';      -- ends with berry
SELECT email FROM users WHERE email ILIKE '%@gmail.com';                 -- Gmail, any case
```

## Regular expressions

For more complex patterns than `LIKE`:

| Operator | Meaning |
|---|---|
| `~` | Matches (case-sensitive) |
| `~*` | Matches (case-insensitive) |
| `!~` | Does not match (case-sensitive) |
| `!~*` | Does not match (case-insensitive) |

```sql
SELECT name FROM employees WHERE name ~ '^[AB]';    -- starts with A or B

SELECT email FROM users                              -- simple email check
WHERE email ~* '^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$';
```

## IN and EXISTS

Both check whether a value appears in a set of results.

| Operator | Use | Notes |
|---|---|---|
| `IN` | Value matches any item in a list or subquery | Best for static lists or small subqueries |
| `EXISTS` | Subquery returns at least one row | Often faster on large data, stops at the first match |

```sql
SELECT name, department FROM employees
WHERE department_id IN (3, 5, 7);

SELECT name FROM employees
WHERE id IN (SELECT employee_id FROM sales WHERE amount > 1000);

SELECT name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM sales s
    WHERE s.employee_id = e.id AND s.amount > 1000
);
```

## CASE

`CASE` is the SQL version of IF-THEN-ELSE. It creates conditional output.

```sql
SELECT name, salary,
  CASE
    WHEN salary > 100000 THEN 'Senior'
    WHEN salary BETWEEN 60000 AND 100000 THEN 'Mid-Level'
    ELSE 'Junior'
  END AS employee_level
FROM employees;
```

## CTEs (WITH clause)

A CTE (Common Table Expression) is a temporary named result you can use in the main query. It breaks complex queries into simple parts.

```sql
WITH cte_name AS (
    SELECT ... FROM ... WHERE ...
)
SELECT * FROM cte_name WHERE ...;
```

Example: employees who earn more than their department's average:

```sql
WITH department_averages AS (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT e.name, e.salary, e.department, da.avg_salary
FROM employees e
JOIN department_averages da ON e.department = da.department
WHERE e.salary > da.avg_salary;
```

---
