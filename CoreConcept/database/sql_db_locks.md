Locks are one of the fundamental mechanisms databases use to maintain **data consistency** when multiple transactions access the same data concurrently.

---

# What is a Lock?

A **lock** is a temporary restriction placed by the database on a resource (row, table, page, etc.) to coordinate concurrent transactions.

The purpose is to prevent problems like:

* Dirty Read
* Lost Update
* Non-repeatable Read
* Data corruption

Think of a lock like a hotel room.

* **Shared lock** = Many people can enter and look around.
* **Exclusive lock** = Only one person can enter and nobody else can even come in.

---

# Two Main Types of Locks

## 1. Shared Lock (S Lock)

A Shared Lock is used when a transaction **reads** data.

### Properties

* Multiple transactions can hold Shared Locks on the same row.
* Other transactions can also read.
* Nobody can modify the row until all shared locks are released.

Example:

```
Row:
Balance = 1000
```

Transaction T1

```sql
BEGIN;
SELECT * FROM account WHERE id = 1;
```

Database places

```
S Lock
```

Now T2 executes

```sql
SELECT * FROM account WHERE id = 1;
```

T2 also gets

```
S Lock
```

Both transactions can read simultaneously.

```
T1 ---- S Lock ---- Row
T2 ---- S Lock ---- Row
```

No problem.

---

## Can someone update?

Suppose T3 executes

```sql
UPDATE account
SET balance = 1200
WHERE id = 1;
```

Database needs an Exclusive Lock.

But:

```
S Lock exists
```

So T3 must wait.

```
T1 ---- S
T2 ---- S
T3 ---- waits for X
```

---

# 2. Exclusive Lock (X Lock)

Exclusive Lock is used when a transaction modifies data.

Example

```
Balance = 1000
```

Transaction T1

```sql
BEGIN;

UPDATE account
SET balance = 1200
WHERE id = 1;
```

Database automatically acquires

```
Exclusive Lock
```

Now the row looks like

```
T1 ---- X Lock ---- Row
```

---

## What happens if another transaction reads?

T2

```sql
SELECT * FROM account WHERE id = 1;
```

This depends on the database's concurrency control:

### PostgreSQL (MVCC)

The `SELECT` usually **does not block**. Instead, it reads the last committed version of the row using MVCC (Multi-Version Concurrency Control).

### Databases that rely purely on locking

The read would typically wait until the Exclusive Lock is released.

---

## What happens if another transaction updates?

T2

```sql
UPDATE account
SET balance = 1500
WHERE id = 1;
```

Needs another Exclusive Lock.

Not possible.

T2 waits.

```
T1 ---- X Lock ---- Row
T2 ---- waiting
```

---

# Compatibility Table

| Existing Lock     | Requested S                                                                    | Requested X |
| ----------------- | ------------------------------------------------------------------------------ | ----------- |
| **Shared (S)**    | ✅ Allowed                                                                      | ❌ Wait      |
| **Exclusive (X)** | ❌ (in lock-based systems; in PostgreSQL plain `SELECT` usually reads via MVCC) | ❌ Wait      |

This is the most important table to remember.

---

# Timeline Example

Suppose

```
Balance = 1000
```

### Step 1

T1

```sql
SELECT balance
FROM account
WHERE id = 1;
```

Database

```
S Lock
```

---

### Step 2

T2

```sql
SELECT balance
FROM account
WHERE id = 1;
```

Database

```
Another S Lock
```

Allowed.

---

### Step 3

T3

```sql
UPDATE account
SET balance = 2000
WHERE id = 1;
```

Needs

```
X Lock
```

Cannot acquire it.

Must wait.

---

### Step 4

T1 commits

```
S Lock released
```

---

### Step 5

T2 still reading.

Still one Shared Lock exists.

T3 still waits.

---

### Step 6

T2 commits.

All Shared Locks gone.

Now

```
T3 acquires X Lock
```

Update happens.

---

# Who Owns the Lock?

The **transaction** owns the lock.

```
Transaction
     |
     | acquires
     v
Database Lock Manager
     |
     v
Locked Row
```

When the transaction ends (`COMMIT` or `ROLLBACK`), the database automatically releases its locks.

---

# Does PostgreSQL Automatically Acquire Locks?

Yes.

You usually don't need to acquire row locks manually for normal operations.

For example:

```sql
UPDATE account
SET balance = 100
WHERE id = 1;
```

PostgreSQL automatically acquires the necessary row-level lock.

Similarly:

```sql
DELETE ...
```

also acquires appropriate row locks.

For explicit locking, PostgreSQL also supports statements like:

```sql
SELECT * FROM account
WHERE id = 1
FOR UPDATE;
```

or

```sql
SELECT * FROM account
FOR SHARE;
```

These intentionally lock rows beyond what a plain `SELECT` does.

---

# Interview Summary (HLD Notes)

> **Database locks are used to coordinate concurrent transactions and ensure data consistency.**
>
> * **Shared Lock (S):** Used for reading. Multiple transactions can hold shared locks on the same row simultaneously. Updates must wait until all shared locks are released.
> * **Exclusive Lock (X):** Used for writing. Only one transaction can hold an exclusive lock on a row. Other writes must wait, and in purely lock-based databases reads also wait. In PostgreSQL, ordinary `SELECT` statements usually read a previous committed version via MVCC instead of waiting on an exclusive row lock.
>
> The transaction owns the lock, and the database automatically releases it on `COMMIT` or `ROLLBACK`.

### One important PostgreSQL note

Because you've been asking specifically about PostgreSQL: **don't think of every `SELECT` as taking a shared row lock.** PostgreSQL's MVCC allows plain reads and writes to proceed concurrently in many cases. Explicit shared/exclusive row locks are primarily acquired through commands like `SELECT ... FOR SHARE` and `SELECT ... FOR UPDATE`, while normal `UPDATE`/`DELETE` operations automatically acquire the necessary write locks on the affected rows. This distinction is frequently tested in backend and system design interviews.
