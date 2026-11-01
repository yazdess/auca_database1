# LAB 4: First SQL Query

I started by learning the basics of SQL syntax. Keywords like `SELECT`, `FROM` and `WHERE` are written in uppercase, while table and column names are usually in snake_case.

To run my first query, I opened psql (or pgAdmin), connected to my database and executed a `SELECT` statement. First I picked specific columns, then I retrieved everything with `*`:

```sql
SELECT name, email FROM students;
SELECT * FROM students;
```

The result is a set of rows (records) and columns (fields), for example:

| name | email |
|---|---|
| Alymbek | alymbek@example.com |
| Timur | timur@example.com |
| Beka | beka@example.com |

Next, I filtered rows with `WHERE`, sorted them with `ORDER BY` (ascending by default, `DESC` for descending), and limited the output with `LIMIT`:

```sql
SELECT name, email FROM students WHERE name = 'Beka';
SELECT name, email FROM students LIMIT 2;
```

Finally, I added comments to make my code readable, using `--` for a single line and `/* ... */` for multiple lines:
