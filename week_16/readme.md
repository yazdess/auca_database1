# LAB 16: Transactions and ACID Properties

## What I did

### 1. Setting up a separate schema and the accounts table

To keep my practice tables apart from the earlier labs, I worked in the `intro` database but created a new schema, `tx_lab`, and made it the default. The first command printed `NOTICE: schema "tx_lab" does not exist, skipping`, because the schema was new.

```sql
DROP SCHEMA IF EXISTS tx_lab CASCADE;
CREATE SCHEMA tx_lab;
SET search_path = tx_lab, pg_catalog;

CREATE TABLE accounts (
    account_id INTEGER PRIMARY KEY,
    owner TEXT,
    balance NUMERIC(12,2) CHECK (balance >= 0)
);

INSERT INTO accounts VALUES (1, 'Misha', 1000), (2, 'Vasya', 500);
```

The `CHECK` constraint does not allow a negative balance. This table is the base for all the transaction examples.

| account_id | owner | balance |
|---|---|---|
| 1 | Misha | 1000.00 |
| 2 | Vasya | 500.00 |

### 2. COMMIT: transferring 200

I moved 200 from Misha to Vasya and saved the changes with `COMMIT`:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 200 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 200 WHERE account_id = 2;
COMMIT;
```

| account_id | owner | balance |
|---|---|---|
| 1 | Misha | 800.00 |
| 2 | Vasya | 700.00 |

### 3. ROLLBACK: undoing a transfer

I started the same kind of transfer with 100. Inside the transaction (the prompt changed to `intro=*#`), the balances were already changed to 700 and 800. Then I ran `ROLLBACK`:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
SELECT * FROM accounts ORDER BY account_id;   -- Misha 700, Vasya 800
ROLLBACK;
SELECT * FROM accounts ORDER BY account_id;   -- back to Misha 800, Vasya 700
```

After the rollback, the balances were back to 800 and 700, as if the transfer do not happen.

### 4. Atomicity: a transfer that fails

I tried to move 2000 from Misha, who only has 800. The first `UPDATE` (adding 2000 to Vasya) worked. The second failed, because Misha's balance would become negative:

```sql
BEGIN;
UPDATE accounts SET balance = balance + 2000 WHERE account_id = 2;
UPDATE accounts SET balance = balance - 2000 WHERE account_id = 1;
```

```
ERROR:  new row for relation "accounts" violates check constraint "accounts_balance_check"
DETAIL:  Failing row contains (1, Misha, -1200.00).
```

After the error, the prompt changed to `intro=!#`, which means the transaction is aborted and only `ROLLBACK` is accepted. I ran `ROLLBACK`, and the balances were still 800 and 700. Even Vasya's successful +2000 was undone. This is the all-or-nothing behavior.

### 5. Consistency: foreign keys

I created `customers` and `orders` (with a foreign key) and inserted a customer and an order for him in one transaction. `LASTVAL()` returns the ID of the customer I had just inserted:

```sql
BEGIN;
INSERT INTO customers (name, email) VALUES ('John Doe', 'john@email.com');
INSERT INTO orders (customer_id, total) VALUES (LASTVAL(), 100.00);
COMMIT;
```

```sql
SELECT c.name, o.order_id, o.total
FROM customers c JOIN orders o ON c.customer_id = o.customer_id;
```

| name | order_id | total |
|---|---|---|
| John Doe | 1 | 100.00 |

### 6. Isolation levels

First I checked the default level:

```sql
SHOW transaction_isolation;   -- read committed
```

Then I ran a query in transactions with the `READ COMMITTED` and `REPEATABLE READ` levels:

```sql
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM accounts WHERE balance > 1000;
COMMIT;
```

The result was `(0 rows)`, because no account has more than 1000.

With `SERIALIZABLE`, I applied 5% interest and looked at the result inside the transaction, then used `ROLLBACK` so the real data stayed unchanged:

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
UPDATE accounts SET balance = balance * 1.05;
SELECT * FROM accounts ORDER BY account_id;   -- Misha 840.00, Vasya 735.00
ROLLBACK;
```

Finally, I set the level in two ways. For one transaction (it must come right after `BEGIN`), and for the whole session. Then I switched the session back to the default:

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SHOW transaction_isolation;   -- repeatable read
COMMIT;

SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SHOW transaction_isolation;   -- repeatable read

SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### 7. Savepoints

I created a `products` table (with `CHECK (price >= 0)`). Then I inserted a customer, set a savepoint, inserted an order of 500, and rolled back to the savepoint. The order was undone, but the customer stayed. I inserted an order of 300 instead and committed:

```sql
BEGIN;
INSERT INTO customers (name, email) VALUES ('Alice', 'alice@email.com');
SAVEPOINT after_customer_insert;

INSERT INTO orders (customer_id, total)
VALUES ((SELECT customer_id FROM customers WHERE email = 'alice@email.com'), 500.00);

ROLLBACK TO SAVEPOINT after_customer_insert;

INSERT INTO orders (customer_id, total)
VALUES ((SELECT customer_id FROM customers WHERE email = 'alice@email.com'), 300.00);
COMMIT;
```

| name | total |
|---|---|
| John Doe | 100.00 |
| Alice | 300.00 |

The order of 500 is missing, as expected.

### 8. Error handling in a DO block

I ran a block that checks Misha's balance and raises an error if it is too low:

```sql
DO $$
DECLARE
    current_balance NUMERIC;
BEGIN
    SELECT balance INTO current_balance FROM accounts WHERE account_id = 1;
    IF current_balance < 100000 THEN
        RAISE EXCEPTION 'Transaction failed: insufficient funds';
    END IF;
    UPDATE accounts SET balance = balance - 100000 WHERE account_id = 1;
END $$;
```

```
ERROR:  Transaction failed: insufficient funds
CONTEXT:  PL/pgSQL function inline_code_block line 7 at RAISE
```

The balance was below 100000, so the block stopped at `RAISE EXCEPTION` (line 7) and the `UPDATE` never ran. A `DO` block is one transaction, so the error cancels everything it did, and no `ROLLBACK` is needed inside.

---
