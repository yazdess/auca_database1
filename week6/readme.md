# LAB 6: Temporary Tables

## What they are

Temporary tables are dropped automatically at the end of a database session or the current transaction. They are isolated from other sessions, which makes them useful for storing intermediate results during complex data manipulation.

## Syntax

```sql
CREATE TEMP TABLE temp_table_name (
    column1 datatype,
    ...
);
```

## What I did

In pgAdmin, I ran a `DROP TABLE` with `IF EXISTS` on a table that did not exist:

```sql
DROP TABLE IF EXISTS test_table;
```

Instead of an error, pgAdmin returned a notice and finished successfully:

```
NOTICE: table "test_table" does not exist, skipping
DROP TABLE
Query returned successfully in 56 msec.
```

This shows what `IF EXISTS` does: if the table is missing, PostgreSQL skips the command with a notice rather than raising an error.
