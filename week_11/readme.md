# LAB 11: Basic Data Operations

As most of it I already know, I will focus on to what is new.

## DELETE: remove rows

Without `WHERE`, **all** rows are deleted.

```sql
DELETE FROM table_name
WHERE condition;
```

```sql
DELETE FROM Students
WHERE first_name = 'Gulzat' AND last_name = 'Sultanova';

DELETE FROM Students
WHERE birth_date < '2002-01-01';
```

## Bulk operations

Used for working with many rows at once.

| Operation | Example |
|---|---|
| Insert many rows | One `INSERT` with several `VALUES` (shown above) |
| Import from CSV | `COPY` (below) |
| Bulk update | `UPDATE Students SET last_name = 'Bekov' WHERE last_name IN ('Uulu', 'Isakov');` |
| Bulk delete | `DELETE FROM Students WHERE student_id IN (2, 3, 5);` |

### COPY: fast import from a file

```sql
COPY Students(first_name, last_name, birth_date)
FROM '/path/to/students.csv'
DELIMITER ','
CSV HEADER;
```
