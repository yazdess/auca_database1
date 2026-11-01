# LAB 9: Database Design Basics

Database design is the process of creating a data model that defines how data is stored, organized and accessed.

## ER Modeling

An ER (Entity-Relationship) model shows the structure of a database graphically before it is built.

| Component | Becomes | Example |
|---|---|---|
| **Entity** | Table | Student, Course, Professor |
| **Attribute** | Column | StudentID, Name, Email |
| **Relationship** | Foreign key / junction table | Students enroll in Courses |

**Attribute types:** simple (`FirstName`), composite (`Address` → Street, City), derived (`Age` from `DateOfBirth`), key (`StudentID`).

## Normalization

Normalization splits data into smaller related tables to reduce redundancy and keep data consistent. Three normal forms cover about 99% of cases.

| Form | Rule | Example fix |
|---|---|---|
| **1NF** | Atomic values, no repeating groups, unique rows | Store one course per row instead of `Math, Physics` in one cell |
| **2NF** | 1NF + no partial dependencies (depend on the whole key) | Move `course_name` out of enrollments into `courses` |
| **3NF** | 2NF + no transitive dependencies | Move `dept_name` out of `employees` into `departments` |

## Example of WHAT I DID: Library System

- **Entities:** Book, Author, Member, Loan
- **Relationships:** Author WRITES Book (many-to-many, needs a junction table); Member BORROWS Book through Loan (one-to-many)

```sql
CREATE TABLE authors (author_id SERIAL PRIMARY KEY, name VARCHAR(100) NOT NULL);
CREATE TABLE books (book_id SERIAL PRIMARY KEY, title VARCHAR(200) NOT NULL);
CREATE TABLE book_authors (
    book_id INTEGER REFERENCES books (book_id),
    author_id INTEGER REFERENCES authors (author_id),
    PRIMARY KEY (book_id, author_id)
);
CREATE TABLE members (member_id SERIAL PRIMARY KEY, name VARCHAR(100) NOT NULL);
CREATE TABLE loans (
    loan_id SERIAL PRIMARY KEY,
    member_id INTEGER REFERENCES members (member_id),
    book_id INTEGER REFERENCES books (book_id),
    due_date DATE NOT NULL,
    late_fee DECIMAL(6,2) DEFAULT 0
);
```

### INSERT sample data

```sql
INSERT INTO authors (name) VALUES
('George Orwell'), ('Neil Gaiman'), ('Terry Pratchett');

INSERT INTO books (title) VALUES
('1984'), ('Animal Farm'), ('Good Omens');

INSERT INTO book_authors (book_id, author_id) VALUES
(1, 1), (2, 1), (3, 2), (3, 3);

INSERT INTO members (name) VALUES
('Alice Johnson'), ('Bob Smith');

INSERT INTO loans (member_id, book_id, due_date, late_fee) VALUES
(1, 1, '2026-10-15', 0),
(1, 3, '2026-10-19', 3.00),
(2, 2, '2026-10-24', 0);
```

### SELECT queries

**Books with their authors** (joins through the junction table):

```sql
SELECT b.title, a.name AS author
FROM books b
JOIN book_authors ba ON b.book_id = ba.book_id
JOIN authors a ON a.author_id = ba.author_id
ORDER BY b.title;
```

**Who borrowed which book:**

```sql
SELECT m.name AS member, b.title, l.due_date
FROM loans l
JOIN members m ON m.member_id = l.member_id
JOIN books b ON b.book_id = l.book_id;
```

**Books that are currently borrowed(not returned):**
```sql
SELECT b.title, m.name AS borrowed_by, l.due_date
FROM loans l
JOIN books b ON b.book_id = l.book_id
JOIN members m ON m.member_id = l.member_id
WHERE l.return_date IS NULL;
```
