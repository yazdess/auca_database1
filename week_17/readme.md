
# LAB 17: Data Import_Export and Backup

## What I did

### 1. Creating the schema, employees table and sample data

I connected to the `intro` database, created a separate schema called `io_lab`, and set it as the default schema for the session. Then I created the `employees` table with a `SERIAL` primary key and columns for employee names, email, department, hire date and salary.

```sql
CREATE SCHEMA io_lab;
SET search_path = io_lab, pg_catalog;

CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name TEXT,
    last_name TEXT,
    email TEXT,
    department TEXT,
    hire_date DATE,
    salary NUMERIC(10,2)
);
```

I inserted six employee records. One employee (Adil) has a `NULL` email value, which let me see how PostgreSQL exports missing values.

```sql
INSERT INTO employees
    (first_name, last_name, email, department, hire_date, salary)
VALUES
    ('Samir', 'Sabirov', 'samir@auca.kg', 'AMI', '2024-05-12', 70000),
    ('Isa', 'Satimbaev', 'isa@example.com', 'AMI', '2021-09-23', 85000),
    ('Adil', 'Akhmetov', NULL, 'IT', '2023-01-15', 48000),
    ('Azalea', 'Turarbekova', 'azalea@example.com', 'Sales', '2020-07-30', 60000),
    ('Beka', 'Mamatov', 'beka@example.com', 'IT', '2024-04-10', 55000),
    ('Meerim', 'Zhumanazarova', 'meerim@example.com', 'Marketing', '2019-06-12', 66000);
```

The successful insert returned `INSERT 0 6`.

### 2. Exporting data with COPY TO

I exported all six rows to a CSV file with a header:

```sql
COPY employees TO '/tmp/employees.csv' WITH CSV HEADER;
```

PostgreSQL returned `COPY 6`. I viewed the file from inside `psql` by using `\!` to run a Terminal command:

```bash
\! cat /tmp/employees.csv
```

I also exported the table using a custom delimiter (`|`) and a custom representation for `NULL` values:

```sql
COPY employees TO '/tmp/employees.txt'
WITH DELIMITER '|' NULL 'N/A' CSV HEADER;
```

```bash
\! cat /tmp/employees.txt
```

In this file, Adil's missing email appeared as `N/A`.

Next, I exported only employees from the IT department:

```sql
COPY (
    SELECT * FROM employees
    WHERE department = 'IT'
)
TO '/tmp/it_employees.csv' WITH CSV HEADER;
```

```bash
\! cat /tmp/it_employees.csv
```

The query export contained two employees: Adil and Beka.

I also tested a semicolon delimiter and forced PostgreSQL to quote the `hire_date` column:

```sql
COPY employees TO '/tmp/employees_semicolon.csv'
WITH (
    FORMAT csv,
    HEADER,
    DELIMITER ';',
    QUOTE '"',
    FORCE_QUOTE (hire_date)
);
```

```bash
\! cat /tmp/employees_semicolon.csv
```

### 3. Importing data with COPY FROM

I created a copy of the original table structure and imported the CSV file into it:

```sql
CREATE TABLE employees_copy (LIKE employees INCLUDING ALL);

COPY employees_copy
FROM '/tmp/employees.csv' WITH CSV HEADER;

SELECT * FROM employees_copy ORDER BY employee_id;
```

The import returned `COPY 6`, and the query showed all six employee records in `employees_copy`.

I then practiced importing only selected columns. First, I exported the names and email addresses:

```sql
COPY (
    SELECT first_name, last_name, email
    FROM employees
)
TO '/tmp/new_employees.csv' WITH CSV HEADER;
```

Then I created another table and imported those three columns into it:

```sql
CREATE TABLE employees_short (LIKE employees INCLUDING ALL);

COPY employees_short (first_name, last_name, email)
FROM '/tmp/new_employees.csv' WITH CSV HEADER;

SELECT employee_id, first_name, last_name, email, department
FROM employees_short;
```

The import returned `COPY 6`. PostgreSQL generated new `employee_id` values (7–12) through the `SERIAL` sequence. The columns that were not included in the import, such as `department`, remained `NULL`.

### 4. Checking WAL settings and functions

For this part, I connected to the `postgres` database and inspected the current WAL-related settings. I did not change the server configuration or set up archiving, PITR or replication.

```sql
SHOW wal_level;
SHOW archive_mode;
SHOW data_directory;
```

The results showed:

- `wal_level`: `replica`
- `archive_mode`: `off`
- `data_directory`: `/opt/homebrew/var/postgresql@18` (the Homebrew PostgreSQL data directory shown by my installation)

I also ran the WAL functions:

```sql
SELECT pg_current_wal_lsn();
SELECT pg_switch_wal();
SELECT pg_create_restore_point('before migration');
```

All three statements returned WAL positions successfully

### 5. Migrating the practice schema to another database

I practiced a dump-and-restore pipeline from the Terminal. First, I created a new database called `migration_test`, then piped a dump of the `io_lab` schema from `intro` directly into `psql` connected to the new database:

```bash
createdb migration_test
pg_dump -d intro -n io_lab | psql -d migration_test
```

The restore output showed that PostgreSQL recreated the schema and its objects and copied the table data.

Finally, I checked the number of employees in both databases:

```bash
psql migration_test -c "SELECT COUNT(*) FROM io_lab.employees;"
psql intro -c "SELECT COUNT(*) FROM io_lab.employees;"
```

Both queries returned `6`, confirming that the six employee records were transferred successfully.

---
