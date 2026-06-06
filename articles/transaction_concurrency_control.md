# Database Transactions, Concurrency Control, and Scheduling in DBMS

Database Management Systems (DBMS) form the backbone of data handling in modern applications, ensuring that data is stored, retrieved, and manipulated reliably. Among the foundational concepts within DBMS are **transactions**, **concurrency control**, and **scheduling**, which play crucial roles in maintaining data integrity and performance.

---

# 1. What is a Database Transaction?

A **database transaction** is a sequence of operations performed as a single logical unit of work. A transaction must satisfy the **ACID** properties to ensure accurate and reliable data handling.

## ACID Properties

### Atomicity

Atomicity guarantees that all operations within a transaction are completed successfully. If any operation fails, the entire transaction is aborted, and the database remains unchanged.

**Example:** Preventing partial updates during a money transfer.

### Consistency

A transaction must bring the database from one valid state to another while adhering to all predefined rules and constraints.

### Isolation

Isolation ensures that concurrently executing transactions do not interfere with one another. Each transaction behaves as if it is the only transaction running.

### Durability

Once a transaction is committed, its changes become permanent, even in the event of system crashes or failures.

## Example of a Transaction

Consider a banking application where a user transfers money from one account to another.

The transaction involves:

1. Deducting money from Account A.
2. Adding money to Account B.

If either operation fails, the entire transaction must be rolled back to preserve data integrity.

---

# 2. Transaction Control Language (TCL)

Transaction Control Language (TCL) commands are used to manage transactions in SQL.

## COMMIT

Saves all changes made during the current transaction permanently.

```sql
COMMIT;
```

### Use Case

After successfully transferring money between accounts.

---

## ROLLBACK

Reverts all changes made during the current transaction.

```sql
ROLLBACK;
```

### Use Case

If an error occurs during the transaction process.

---

## SAVEPOINT

Creates a checkpoint within a transaction that can be rolled back to later.

```sql
SAVEPOINT savepoint_name;
```

### Use Case

Managing complex transactions with multiple stages.

---

# 3. Understanding Atomicity and Durability

## What is Atomicity?

Atomicity treats all actions within a transaction as a single indivisible unit.

If any operation fails:

* The entire transaction fails.
* All completed operations are undone.
* The database remains unchanged.

### Example

Money Transfer:

```text
Account A -> Deduct ₹1000
Account B -> Add ₹1000
```

If the second operation fails, the first operation must also be reversed.

---

## What is Durability?

Durability guarantees that once a transaction is committed, it remains permanent.

Even if:

* Power fails
* The server crashes
* The database restarts

The committed changes must persist.

---

# Challenges in Implementing ACID Properties

Traditional implementations rely heavily on:

* Transaction logs
* Undo logs
* Recovery mechanisms

A crash occurring immediately after commit but before disk writes can lead to inconsistencies if not handled properly.

---

# 4. Shadow Copy Technique

## What is Shadow Copy?

The **Shadow Copy Technique** is a mechanism used to implement Atomicity and Durability.

A copy of the database is created before transaction execution begins.

This copy acts as a backup version of the database.

---

## How It Works

### Step 1: Create a Snapshot

A shadow copy of the database is generated.

```text
Original Database
       |
       v
 Shadow Copy Created
```

---

### Step 2: Execute Transaction

All modifications are performed on the shadow copy rather than the original database.

---

### Step 3: Commit Changes

If successful:

```text
Shadow Copy -> Original Database
```

If failure occurs:

```text
Discard Shadow Copy
```

The original database remains intact.

---

## Advantages of Shadow Copy

### Enhanced Reliability

Maintains a consistent state before transaction execution.

### Simplified Rollback

No need for complicated undo logs.

### Performance Optimization

Can simplify recovery procedures.

### Reduced Data Loss

Incomplete transactions never affect the primary database.

---

## Challenges of Shadow Copy

### Storage Requirements

Requires additional storage space.

### Performance Overhead

Creating and managing copies consumes resources.

### Implementation Complexity

Version management becomes more challenging.

### Concurrency Management

Multiple simultaneous transactions become harder to coordinate.

---

# 5. Understanding Concurrency in DBMS

## What is Concurrency?

Concurrency refers to the ability of a database system to execute multiple transactions simultaneously.

Benefits include:

* Improved throughput
* Better resource utilization
* Faster response times

---

## Importance of Concurrency Control

Concurrency control ensures:

* Isolation
* Consistency
* Correct transaction execution

Without proper control, concurrent transactions can lead to serious anomalies.

---

# Common Concurrency Problems

## 1. Dirty Read

Occurs when a transaction reads uncommitted data from another transaction.

### Example

```text
Transaction A updates salary
Transaction B reads updated salary
Transaction A rolls back
```

Transaction B has now read invalid data.

---

## 2. Non-Repeatable Read

Occurs when a transaction reads the same row twice and gets different values.

### Example

```text
Transaction A reads balance = 5000
Transaction B updates balance = 7000
Transaction A reads again
```

Result:

```text
5000 -> 7000
```

---

## 3. Phantom Read

Occurs when rows satisfying a query condition change during transaction execution.

### Example

Transaction A executes:

```sql
SELECT * FROM Employees
WHERE Salary > 50000;
```

Transaction B inserts:

```text
Salary = 60000
```

Transaction A executes the query again and sees a new row.

---

## 4. Lost Update

Occurs when two transactions overwrite each other's changes.

### Example

```text
Initial Balance = 1000

Transaction A reads 1000
Transaction B reads 1000

Transaction A writes 1200
Transaction B writes 1100
```

Final balance:

```text
1100
```

Transaction A's update is lost.

---

# Techniques to Manage Concurrency Problems

## 1. Locking Mechanisms

### Shared Lock (S-Lock)

* Multiple transactions can read.
* No transaction can write.

### Exclusive Lock (X-Lock)

* One transaction can read and write.
* Others cannot access the resource.

---

## 2. Timestamp Ordering

Each transaction receives a unique timestamp.

Transactions execute according to timestamp order, preventing conflicts.

---

## 3. Optimistic Concurrency Control

Assumes conflicts are rare.

Transactions execute freely.

Before committing:

* Conflict detection occurs.
* Conflicting transactions are rolled back.

---

## 4. Serialization

Ensures concurrent execution produces the same result as serial execution.

Goal:

```text
Concurrent Execution ≡ Serial Execution
```

---

# 6. Understanding Schedules in DBMS

## What is a Schedule?

A schedule is the chronological order in which operations from multiple transactions are executed.

Operations typically include:

* Read
* Write
* Commit
* Rollback

---

## Importance of Schedules

Schedules help maintain:

* Data consistency
* Isolation
* Reliability

They prevent:

* Dirty reads
* Lost updates
* Non-repeatable reads
* Phantom reads

---

# Types of Schedules

## 1. Serial Schedule

Transactions execute one after another without overlap.

### Example

```text
T1: Read(A)
T1: Write(A)

T2: Read(B)
T2: Write(B)
```

### Advantages

* Simple
* No concurrency issues

### Disadvantages

* Low performance

---

## 2. Non-Serial Schedule

Transactions execute concurrently and may interleave operations.

### Advantages

* Better throughput
* Improved performance

### Disadvantages

* Requires concurrency control mechanisms

---

## 3. Conflict Serializable Schedule

A non-serial schedule that can be transformed into a serial schedule without changing the outcome.

### Conditions for Conflict

Two operations conflict when:

1. They belong to different transactions.
2. They access the same data item.
3. At least one operation is a write.

### Example

```text
T1: Read(A)
T1: Write(A)

T2: Read(A)
T2: Write(A)
```

If rearrangement produces a valid serial schedule, it is conflict serializable.

---

## 4. View Serializable Schedule

Two schedules are view equivalent if:

1. Transactions read the same values.
2. Final writes are performed by the same transactions.

View serializability is more general than conflict serializability.

---

## 5. Recoverable Schedule

If transaction T1 reads data written by T2:

```text
T2 must commit before T1 commits.
```

This ensures safe recovery after failures.

---

## 6. Cascadeless Schedule

Transactions only read committed data.

### Benefit

Prevents cascading rollbacks.

```text
Read only committed values
```

---

## 7. Strict Schedule

The most restrictive schedule type.

A transaction cannot:

* Read data modified by another uncommitted transaction.
* Write data modified by another uncommitted transaction.

### Benefits

* Maximum isolation
* Simplified recovery
* Strong data integrity guarantees

---

# Conclusion

Transactions, concurrency control, and scheduling are fundamental concepts in DBMS that ensure databases remain reliable, consistent, and efficient in multi-user environments.

Key takeaways:

* Transactions follow ACID properties.
* TCL commands manage transaction execution.
* Shadow Copy helps implement Atomicity and Durability.
* Concurrency introduces issues such as Dirty Reads and Lost Updates.
* Locking, Timestamp Ordering, Optimistic Control, and Serialization help manage concurrency.
* Different schedule types determine how transactions execute while maintaining consistency.

A solid understanding of these concepts is essential for designing robust and scalable database systems.
