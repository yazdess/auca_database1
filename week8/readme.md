# LAB 8: Foreign Keys and Relationships

## Concept

A **foreign key** is a column (or columns) in one table that refers to the primary key of another table. It links the tables and enforces **referential integrity**: a foreign key value must exist in the referenced table.

## Ways to create a foreign key

```sql
-- 1. Inline, during table creation
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    dept_id INTEGER REFERENCES departments (dept_id)
);

-- 2. Table-level, during creation
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    dept_id INTEGER,
    FOREIGN KEY (dept_id) REFERENCES departments (dept_id)
);

-- 3. Added to an existing table
ALTER TABLE employees
ADD FOREIGN KEY (dept_id) REFERENCES departments (dept_id);

-- 4. Named constraint, during creation
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    dept_id INTEGER,
    CONSTRAINT fk_employee_dept FOREIGN KEY (dept_id) REFERENCES departments (dept_id)
);
```

## Referential integrity rules

| Rule | Meaning |
|---|---|
| Insert | Cannot insert a foreign key value that doesn't exist in the parent table |
| Update | Cannot update a foreign key to a value that doesn't exist in the parent table |
| Delete | Deleting a referenced parent row is handled by the `ON DELETE` option |

## CASCADE options

| Option | What happens when the parent changes |
|---|---|
| `ON DELETE CASCADE` | Child rows are deleted too |
| `ON DELETE SET NULL` | Foreign key is set to `NULL` |
| `ON DELETE SET DEFAULT` | Foreign key is set to its default value |
| `ON DELETE RESTRICT` | Deleting the parent is blocked if child rows exist |
| `ON UPDATE CASCADE` | Foreign key values are updated when the parent key changes |

They can be combined:

```sql
FOREIGN KEY (dept_id) REFERENCES departments (dept_id)
    ON DELETE SET NULL ON UPDATE CASCADE
```

---

## What I did

### 1. One-to-many: departments and employees

In the `intro` database, I created two tables. The `employees` table refers to `departments` through an inline foreign key:

```sql
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL,
    location VARCHAR(100)
);

CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    dept_id INTEGER REFERENCES departments (dept_id)
);
```

Then I inserted 3 departments and 3 employees (`INSERT 0 3` both times) and checked the data with `SELECT * FROM employees;`.

### 2. Testing ON UPDATE CASCADE

I dropped the automatically named constraint and added it again with `ON UPDATE CASCADE`:

```sql
ALTER TABLE employees
DROP CONSTRAINT employees_dept_id_fkey;

ALTER TABLE employees
ADD CONSTRAINT employees_dept_id_fkey
FOREIGN KEY (dept_id)
REFERENCES departments (dept_id)
ON UPDATE CASCADE;
```

Then I changed the ID of a department:

```sql
UPDATE departments SET dept_id = 10 WHERE dept_id = 1;
```

Result: `UPDATE 1`. In `employees`, John's `dept_id` changed from 1 to 10 automatically:

| emp_id | first_name | last_name | dept_id |
|---|---|---|---|
| 2 | Laura | Johnson | 2 |
| 3 | Chris | Brown | 3 |
| 1 | John | Smith | 10 |

### 3. One-to-one: users and user_profiles

A one-to-one relationship uses a foreign key with a `UNIQUE` constraint. First, I created `users`:

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Creating `user_profiles`:

```sql
CREATE TABLE user_profiles (
    profile_id SERIAL PRIMARY KEY,
    user_id INTEGER UNIQUE NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    bio TEXT,
    profile_picture_url VARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE
);
```

I inserted a user (`INSERT 0 1`):

```sql
INSERT INTO users (username, email) VALUES ('johndoe', 'john@example.com');

INSERT INTO user_profiles (user_id, first_name, last_name, bio)
VALUES (1, 'John', 'Doe', 'Software Developer passionate');
```

Result: `INSERT 0 1`. `SELECT * FROM user_profiles;` showed one row with `profile_id` 1 for John Doe.

---

## Relationship types

| Type | How it's implemented |
|---|---|
| One-to-one | Foreign key + `UNIQUE` (or sharing the same primary key) |
| One-to-many | Foreign key in the "many" table (most common) |
| Many-to-many | Junction table with foreign keys to both tables |

```sql
-- Many-to-many: students and courses
CREATE TABLE enrollments (
    student_id INTEGER REFERENCES students (student_id),
    course_id INTEGER REFERENCES courses (course_id),
    PRIMARY KEY (student_id, course_id)
);
```
