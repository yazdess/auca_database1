# LAB 5: Tables, Data Types and Constraints

In psql, I connected to my `intro` database and practiced with a **temporary table** (it is dropped automatically when the session ends).

## 1. Creating a table

My first attempt failed with `ERROR: syntax error at or near "first_name"`, so I retyped the command as a single line. The second attempt worked:

```sql
CREATE TEMP TABLE temp_table_stud (
    student_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    faculty VARCHAR(100)
);
```

Result: `CREATE TABLE`

## 2. Altering the table

I added a new column to the existing table:

```sql
ALTER TABLE temp_table_stud ADD COLUMN date_of_birth DATE;
```

Result: `ALTER TABLE`

## 3. Checking the result

```sql
SELECT * FROM temp_table_stud;
```

The output showed all six columns with no rows yet, since I hadn't inserted any data:

| student_id | first_name | last_name | email | faculty | date_of_birth |
|---|---|---|---|---|---|

## Data types I used

| Type | Description |
|---|---|
| `SERIAL` | Auto-incrementing integer |
| `VARCHAR(n)` | Text with a maximum length of n |
| `DATE` | Date value |

Other common types: `INTEGER`, `BIGINT`, `TEXT`, `BOOLEAN`, `TIMESTAMP`.

## Constraints I used

| Constraint | Description |
|---|---|
| `PRIMARY KEY` | Uniquely identifies each row |
| `NOT NULL` | The column cannot be empty |
| `UNIQUE` | All values in the column must differ |

Also available: `CHECK` and `FOREIGN KEY`.

## Other useful commands

```sql
ALTER TABLE t DROP COLUMN c;                 -- drop a column
ALTER TABLE t ALTER COLUMN c TYPE TEXT;      -- change a data type
ALTER TABLE t ADD CONSTRAINT name CHECK (c > 0);  -- add a constraint
ALTER TABLE t RENAME COLUMN c TO new_c;      -- rename a column
DROP TABLE IF EXISTS temp_table_stud;        -- remove a table
```
