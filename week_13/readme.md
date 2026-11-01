# LAB 13: Aggregate Functions
 
## What I did  (theory is below)
 
In the `intro` database, I ran the aggregate queries on my `employees_sales` and `sales` tables.
 
### 1. COUNT
 
I counted all employees, then the unique departments:
 
```sql
SELECT COUNT(*) AS total_employees FROM employees_sales;
SELECT COUNT(DISTINCT department) AS unique_departments FROM employees_sales;
```
 
| total_employees | unique_departments |
|---|---|
| 6 | 4 |
 
### 2. SUM
 
I calculated the total salary expenditure:
 
```sql
SELECT SUM(salary) AS total_salaries FROM employees_sales;
```
 
Result: `448000.00`
 
### 3. SUM with GROUP BY and JOIN
 
`department` is stored in `employees_sales`, not in `sales`, so I joined the two tables to get the total sales per department:
 
```sql
SELECT e.department, SUM(s.amount) AS total_sales
FROM sales s
JOIN employees_sales e ON s.employee_id = e.id
GROUP BY e.department;
```
 
| department | total_sales |
|---|---|
| Marketing | 1200.00 |
| Engineering | 800.00 |
| Sales | 4000.00 |
| HR | 300.00 |
 
### 4. AVG
 
```sql
SELECT AVG(salary) AS average_salary FROM employees_sales;
```
 
Result: `74666.666666666667`
 
### 5. MAX and MIN
 
```sql
SELECT MAX(salary) AS highest_salary, MIN(salary) AS lowest_salary
FROM employees_sales;
```
 
| highest_salary | lowest_salary |
|---|---|
| 120000.00 | 48000.00 |
 
### 6. STRING_AGG
 
I listed the employees of each department as one comma-separated string:
 
```sql
SELECT department, STRING_AGG(name, ', ') AS employee_names
FROM employees_sales
GROUP BY department;
```
 
| department | employee_names |
|---|---|
| Marketing | David |
| Engineering | Bob, Carol |
| Sales | Alice, Anna |
| HR | Brian |
 
### 7. ARRAY_AGG
 
I collected the salaries of each department into an array:
 
```sql
SELECT department, ARRAY_AGG(salary) AS salary_array
FROM employees_sales
GROUP BY department;
```
 
| department | salary_array |
|---|---|
| Marketing | {70000.00} |
| Engineering | {90000.00,120000.00} |
| Sales | {55000.00,65000.00} |
| HR | {48000.00} |




---
<br>




---
# THEORY

Aggregate functions take a set of rows and return **one value** (a total, an average, a count, etc.). They are usually combined with `GROUP BY`, which splits rows into groups so the calculation is done for each group.

## Common aggregate functions

| Function | What it does |
|---|---|
| `COUNT(*)` | Counts all rows (including `NULL`s) |
| `COUNT(column)` | Counts non-`NULL` values in a column |
| `COUNT(DISTINCT column)` | Counts unique values |
| `SUM(column)` | Total of numeric values |
| `AVG(column)` | Average (arithmetic mean) |
| `MAX(column)` / `MIN(column)` | Largest / smallest value |

```sql
SELECT COUNT(*) AS total_employees FROM employees;
SELECT COUNT(DISTINCT department) AS unique_departments FROM employees;

SELECT SUM(salary) AS total_salaries FROM employees;
SELECT AVG(salary) AS average_salary FROM employees;

SELECT MAX(salary) AS highest_salary, MIN(salary) AS lowest_salary
FROM employees;

-- with GROUP BY: one result row per department
SELECT department, SUM(sales_amount) AS total_sales
FROM sales_data
GROUP BY department;
```

## Advanced aggregate functions

| Function | What it does |
|---|---|
| `STRING_AGG(col, ', ')` | Joins values from many rows into one string |
| `ARRAY_AGG(col)` | Collects values from many rows into an array |
| `STDDEV(col)`, `VARIANCE(col)` | Standard deviation and variance |
| `CORR(col1, col2)` | Correlation between two numeric columns |

```sql
SELECT department, STRING_AGG(first_name, ', ') AS employee_names
FROM employees
GROUP BY department;

SELECT department, ARRAY_AGG(salary) AS salary_array
FROM employees
GROUP BY department;

SELECT department, STDDEV(salary) AS salary_std_dev
FROM employees
GROUP BY department;
```

Both `STRING_AGG` and `ARRAY_AGG` accept `ORDER BY` inside the parentheses, for example `STRING_AGG(skill_name, ', ' ORDER BY skill_name)`.

## GROUP BY

`GROUP BY` groups rows that have the same values in the given columns.

```sql
-- one column
SELECT category, COUNT(*) AS number_of_products, AVG(price) AS average_price
FROM products
GROUP BY category;

-- several columns / expressions
SELECT EXTRACT(YEAR FROM sale_date) AS year,
       EXTRACT(QUARTER FROM sale_date) AS quarter,
       SUM(amount) AS total_revenue
FROM sales
GROUP BY EXTRACT(YEAR FROM sale_date), EXTRACT(QUARTER FROM sale_date)
ORDER BY year, quarter;

-- group by a CASE expression
SELECT CASE WHEN age < 30 THEN 'Under 30'
            WHEN age BETWEEN 30 AND 50 THEN '30-50'
            ELSE 'Over 50' END AS age_group,
       COUNT(*) AS employee_count
FROM employees
GROUP BY CASE WHEN age < 30 THEN 'Under 30'
              WHEN age BETWEEN 30 AND 50 THEN '30-50'
              ELSE 'Over 50' END;
```

## HAVING: filtering groups

| Clause | Filters | When |
|---|---|---|
| `WHERE` | Individual rows | **Before** grouping |
| `HAVING` | Groups | **After** grouping |

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```


## NULL values

- `COUNT(*)` counts `NULL`s; `COUNT(column)` does not.
- `SUM`, `AVG`, `MAX` and `MIN` **ignore** `NULL`s.

```sql
SELECT COUNT(*) AS total_rows,
       COUNT(email) AS non_null_emails,
       COUNT(*) - COUNT(email) AS null_emails
FROM employees;
```


**Percentage of total:**

```sql
SELECT department,
       COUNT(*)::FLOAT / (SELECT COUNT(*) FROM employees) * 100 AS percentage
FROM employees
GROUP BY department
ORDER BY percentage DESC;
```
