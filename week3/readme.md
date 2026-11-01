# Basic psql Commands

I opened the Terminal and connected to PostgreSQL with `psql -U postgres -d mydb -h localhost -p 5432`

Inside, I tried the meta-commands: `\l` to list databases, `\c <dbname>` to connect to one, `\dt` to list tables, and `\d <table>` to see a table's structure. `\h` and `\?` showed me the help.

Then I ran basic SQL: I created a `students` table, inserted two rows, selected them, and dropped the table.

```sql
CREATE TABLE students (id SERIAL PRIMARY KEY, name VARCHAR(100), age INT);
INSERT INTO students (name, age) VALUES ('Alice', 21), ('Bob', 23);
SELECT * FROM students;
DROP TABLE students;
```

Finally, I exited with `\q`.
