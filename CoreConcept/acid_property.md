**ACID** is a set of four properties that guarantee database transactions are processed **reliably and correctly**, even if multiple users access the database simultaneously or a system failure occurs.

A **transaction** is a group of SQL operations that are treated as a single unit of work.

For example:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

Money should either be transferred completely or not at all.

## A – Atomicity ("All or Nothing")

A transaction is either **fully completed** or **fully rolled back**.

If one statement fails, none of the changes are kept.

**Example:**

* Debit ₹1000 from Account A ✅
* Credit ₹1000 to Account B ❌ (fails)

Without atomicity:

* A loses ₹1000.
* B doesn't receive it.

With atomicity:

* The debit is rolled back.
* Both accounts remain unchanged.

---

## C – Consistency ("Database remains valid")

A transaction takes the database from **one valid state to another valid state**.

All constraints, rules, and relationships remain satisfied.

**Example:**

Suppose:

```text
balance >= 0
```

If a transaction tries:

```sql
UPDATE accounts
SET balance = -500;
```

The database rejects it (if such a constraint exists), keeping the data consistent.

Consistency also ensures:

* Primary key constraints
* Foreign key constraints
* Unique constraints
* Check constraints
* Triggers

are never violated after a committed transaction.

---

## I – Isolation ("Transactions don't interfere")

Multiple transactions running simultaneously should behave **as if they were executed one after another** (depending on the chosen isolation level).

**Example**

Suppose:

Current balance = ₹5000

Two transactions run at the same time.

**T1**

```sql
UPDATE accounts
SET balance = balance - 1000;
```

**T2**

```sql
UPDATE accounts
SET balance = balance - 500;
```

Without proper isolation:

* Both may read the same old balance.
* One update could overwrite the other (lost update).

With isolation:

* The database coordinates access using mechanisms such as locks or MVCC (used by PostgreSQL), preventing incorrect results.

This is why isolation levels (Read Committed, Repeatable Read, Serializable, etc.) exist.

---

## D – Durability ("Committed means permanent")

Once a transaction is committed, its changes are **permanently stored**, even if the database crashes immediately afterward.

**Example**

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000;

COMMIT;
```

Even if the server loses power one second after `COMMIT`, the change will still exist when the database restarts.

Databases achieve this using techniques like:

* Write-Ahead Logging (WAL)
* Transaction logs
* Checkpoints
* Crash recovery

---

## Summary Table

| Property        | Meaning                                             | Example                                            |
| --------------- | --------------------------------------------------- | -------------------------------------------------- |
| **Atomicity**   | All operations succeed or all are rolled back       | Money transfer completes entirely or not at all    |
| **Consistency** | Database remains valid after the transaction        | Constraints are never violated                     |
| **Isolation**   | Concurrent transactions don't interfere incorrectly | Two users booking the same seat don't corrupt data |
| **Durability**  | Committed data survives crashes                     | Data remains after power failure                   |

### Simple way to remember

* **Atomicity** → *All or Nothing*
* **Consistency** → *Always Valid*
* **Isolation** → *Transactions Don't Interfere*
* **Durability** → *Committed Forever*

Since you've recently been studying **database locking, isolation levels, and seat booking systems**, ACID is the foundation that ties those concepts together:

* **Isolation** is implemented using mechanisms such as **locks** and/or **MVCC**.
* **Atomicity** uses transaction rollback.
* **Consistency** relies on database constraints and transaction correctness.
* **Durability** is provided by transaction logs (e.g., PostgreSQL's WAL).
