# 🔄 What is a Transaction?

A transaction is a group of operations that are treated as a single unit of work — either all succeed or all fail to keep data consistent.

---

## 🧪 ACID Properties

| Property | Meaning |
|----------|---------|
| 🅰️ A - Atomicity | All steps in a transaction must succeed or none are applied. |
| 🧮 C - Consistency | Data must remain valid and follow rules before and after the transaction. |
| 🔐 I - Isolation | Transactions run independently without interfering with each other. |
| 💽 D - Durability | Once a transaction is committed, changes are saved even if the system crashes. |

---

### ⚛️ 1. Atomicity

✅ **Definition** :

Atomicity means that a transaction is all or nothing — either everything succeeds, or everything is rolled back.

  🏦 **Example**:  
     Transfer ₹1000 from Account A to Account B  
  **Steps**:
  1. Debit ₹1000 from Account A ✅ 
  2. Credit ₹1000 to Account B ❌ (fails)

  Since the second step fails, Atomicity ensures that the first step is also undone.  
  💥 So, ₹1000 is not deducted from Account A — the transaction never happened.

✔️ **No partial updates are allowed.**

---

### 🔍 2. Consistency

✅ **Definition**

Consistency means that data must always follow the rules (constraints, relationships, validations) defined in the database — before and after a transaction.

If a transaction violates those rules, it should fail and not affect the database.

🏦 **Example**:

Let’s say you have a bank database with a rule:  
🔒 Account balance must never be negative  
Transfer ₹1000 from Account A to Account B

If Account A has only ₹500, and the transaction tries to move ₹1000:

That violates the consistency rule (because Account A would go negative).

So the database rejects the transaction.

The data stays unchanged and valid.

✔️ **Consistency is protected.**

---

### 🛡️ 3. Isolation

✅ **Definition** :

Isolation means that transactions run independently, even if they happen at the same time.

🏦 **Example**:

Two users are trying to buy the last movie ticket at the same time:

- User A starts a transaction to book the ticket.
- User B also starts a transaction before A finishes.

Without Isolation, both might get the same ticket — ❌ inconsistent data.

With Isolation, the database ensures:

- One transaction waits or uses a snapshot of the data.
- Only one succeeds in booking the ticket.

✔️ **Data stays safe and accurate, even with concurrency.**

---

### 💾 4. Durability

✅ **Definition** :

Durability means that once a transaction is committed, its changes are permanent, even if the system crashes.

🏦 **Example**:

You transfer money and see a “Transaction Successful” message.

Suddenly, the server crashes! 😱

With Durability:

- The database has already saved the changes to disk.
- After the system restarts, the money is still transferred — nothing is lost.

✔️ **Committed = Saved Forever.**


---

## ✅ What Happens When a `@Transactional` Method is Called

### 🔁 1. Spring AOP Intercepts the Call

When you call a method annotated with `@Transactional`, **Spring AOP** intercepts the call using a proxy (JDK dynamic proxy or CGLIB, depending on the bean type).  
📌 The interception is based on a pointcut expression that matches methods annotated with `@Transactional`.


### ⚙️ 2. `TransactionInterceptor` is Invoked

The `TransactionInterceptor` is the AOP advice responsible for handling transactional behavior.  
🧰 It wraps your method call inside a logic block that manages the transaction before and after the method execution.


   ##### 🧬 Internal Flow in `TransactionInterceptor`
   ```
       invoke() in TransactionInterceptor
       └──     invokeWithinTransaction()
       ├── 🔹 [Before Advice] Begin or Join Transaction
       ├── 🧠 Execute Target Method (your business logic)
       └── 🔸 [After Advice] Commit or Rollback Transaction
   ```

#### 🔹 Before Advice – Begin or Join Transaction
  - 🔄 Uses `PlatformTransactionManager.getTransaction(...)`
  - Based on the `@Transactional` **propagation type** (e.g., `REQUIRED`, `REQUIRES_NEW`), it either:
    - 🔗 Joins an existing transaction 
    - 🆕 Or starts a new transaction


#### 🧠 Your Method Executes

   - The actual business logic in the annotated method is executed.
     - ✅ If **no exceptions** are thrown, it proceeds to **commit**.
     - ❌ If a **RuntimeException** (or declared rollback exception) is thrown, it proceeds to **rollback**.


#### 🔸 After Advice – Commit or Rollback

   ##### ✅ If method executes successfully:
   - Calls `commit()` on `PlatformTransactionManager`. 
     - 🛠️ Internally:
       ``` java
          EntityManager.flush()  
          Transaction.commit()
          EntityManager.clear()
       ```

   ##### ❌ If method throws an exception (that matches rollback rules):
   - Calls `rollback()` on `PlatformTransactionManager`. 
     - 🛠️ Internally:
      ``` java
         Transaction.rollback()  
         EntityManager.clear()
      ```
--- 


## 🧾 Transaction Propagation

Transaction propagation defines how transactions behave when one service calls another. It answers:
> **“Should the called service run in the same transaction, start a new one, or not be part of a transaction at all?”**

In frameworks like Spring, there are different **Propagation** types, like:

| 🔁 Propagation Type | 💡 What it Means |
|---------------------|------------------|
| `REQUIRED`          | Join the existing transaction if present, else start a new one. |
| `REQUIRES_NEW`      | Always start a new transaction. Suspend the old one if it exists. |
| `SUPPORTS`          | Join if there’s a transaction; else run non-transactionally. |
| `NOT_SUPPORTED`     | Always run non-transactionally, suspend if there's an existing transaction. |
| `MANDATORY`         | Must run inside an existing transaction; else throw an exception. |
| `NEVER`             | Must not run inside a transaction; else throw an exception. |
| `NESTED`            | Run inside a nested transaction within the main one (like a sub-task). |

---

## 🧩 Imagine we have different Services like:

- 🛒 **Order Service**
- 💳 **Payment Service**
- 📦 **Inventory Service**
- 📬 **Notification Service**

---

## 1. ✅ `REQUIRED` (Default)

### 💡 What it means:
Join existing transaction if one exists, else start a new one.

### 🛒 Example:
**Order Service** calls **Payment Service**

- If Order Service already started a transaction, Payment Service joins it.
- If anything fails, both roll back.

### 🔁 Flow:
  ``` java
     Order Service (Start TX)
     └─> Payment Service (joins TX)
     └─> Success or Failure
     (If failure, rollback whole TX)
  ```


### 🕒 When to Use:
When you want all services to commit or rollback together.

**Typical for order + payment flows.**

---

## 2. 🆕 `REQUIRES_NEW`

### 💡 What it means:
Always starts a new transaction, suspending any existing one.

### 🛒 Example:
**Order Service** starts a transaction.  
Calls **Notification Service** with `REQUIRES_NEW`.

- Even if Order or Payment fails and rolls back, Notification is still committed.

### 🔁 Flow:
  ``` java
    Order Service (Start TX)
    └─> Payment Service (joins TX)
    └─> Notification Service (new TX)
    └─> Commits even if Order TX fails
  ```

### 🕒 When to Use:
When a service must succeed independently,  
e.g., sending confirmation emails even if payment fails.

---

## 3. ❗ `MANDATORY`

### 💡 What it means:
Must be called within an existing transaction. Else, it fails.

### 🛒 Example:
**Inventory Service** is marked `MANDATORY`.

- If Order Service didn’t start a transaction and calls Inventory, it throws an error.

### 🔁 Flow:
   ``` java
      Order Service (No TX) → Inventory Service (MANDATORY) → ❌ Error
   ```


### 🕒 When to Use:
When service **must participate** in an existing transaction.

Use to **enforce consistency**, not allow standalone execution.

---

## 4. 🚫 `NEVER`

### 💡 What it means:
Must be called **without a transaction**.  
If called inside a transaction, it throws an error.

### 🛒 Example:
**Logging Service** should not be in a transaction (maybe due to performance).

- If Order Service has a transaction and calls it, it's an error.

### 🔁 Flow:
  ``` java
    Order Service (TX) → Logging Service (NEVER) → ❌ Error
  ```


### 🕒 When to Use:
For services that **must run outside a transaction**,  
like logging, metrics.

---

## 5. 🔁 `NESTED`

### 💡 What it means:
Starts a nested (sub) transaction within the main one.

- If nested fails, it can be rolled back independently, but parent can continue.

### 🛒 Example:
**Order Service** adds order and inventory in a single transaction.

- Inventory update is nested. If it fails, it rolls back only that part, order still proceeds.

### 🔁 Flow:
   ``` java
   Order Service (TX)
    └─> Inventory Service (Nested TX)
    └─> Inventory fails → Rollback nested TX only
    └─> Continue Order
   ```

### 🕒 When to Use:
When **partial rollback** is needed.  
Rare in microservices (better in monolith or single DB systems).

---

## 6. 🆗 `SUPPORTS`

### 💡 What it means:
- If there’s a transaction, join it.
- If not, run without transaction.

### 🛒 Example:
**Search Indexing Service** can run with or without a transaction.

- If called within order flow, it joins. Otherwise, runs standalone.

### 🔁 Flow:
   ``` java
    Order Service (TX) → Indexing Service (joins TX)
     or
    Standalone call → Indexing Service (no TX)
   ```

### 🕒 When to Use:
For services that are **flexible**, no strict need for TX.

---

## 7. 🔇 `NOT_SUPPORTED`

### 💡 What it means:
Suspend any existing transaction until the method completes execution.  
Run without a transaction.

### 🛒 Example:
**Audit Service** shouldn't be part of order transaction.

- If Order is in a transaction, it’s suspended during audit logging.

### 🔁 Flow:
   ``` java
    Order Service (TX) → Audit Service (TX suspended) → Executes outside TX
   ```


### 🕒 When to Use:
For services where **transaction should not affect outcome**,  
like logging or auditing.


📌 **Tip**: Choosing the right propagation strategy is key for maintaining data integrity and ensuring system reliability in distributed or monolithic systems.

---

## 🔒 Types of Database Locks

### 1. 🔄 Shared Lock (S Lock)

**Purpose:**  
📖 Used when a transaction wants to **read** (`SELECT`) data.

**Behavior:**
- 🔓 Multiple transactions can acquire a shared lock on the same data at the same time.
- 🚫 No transaction can modify (`UPDATE`/`DELETE`) the data while a shared lock is held.
- ✅ It ensures **read consistency** and prevents **dirty reads**.


### 2. ✋ Exclusive Lock (X Lock)

**Purpose:**  
✍️ Used when a transaction wants to **modify** (`UPDATE`/`DELETE`/`INSERT`) data.

**Behavior:**
- 🔒 Only **one** transaction can hold an exclusive lock on a data item at a time.
- 🚫 While a transaction holds an exclusive lock, **no other transaction** (not even for reading) can access the locked data.
- ✅ It ensures **data consistency** during modifications.

---
# 📘 Database Isolation Problem: Dirty Read

## 🔍 What is `Dirty Read`?

A **dirty read** occurs when a transaction reads data that has been modified by another transaction but **not yet committed**. If the modifying transaction is rolled back, the data read by the first transaction becomes **invalid or inconsistent**.

### 📋 Example:

1. **Transaction A (TXA)** updates a row in the database but does **not commit** the change.
2. **Transaction B (TXB)** reads the updated data from TXA.
3. Later, **TXA rolls back** the transaction.

  > 🚨 TXB has read data that was never permanently saved — this is **dirty data**.

## 🛠️ Solution: 

To prevent dirty reads, use an isolation level that does **not allow uncommitted reads**.

### ✅ Recommended: `READ COMMITTED`

- Ensures a transaction only reads data that has been **committed**.
- Prevents reading temporary or uncommitted changes from other transactions.

 ``` java
 @Transactional(isolation = Isolation.READ_COMMITTED)
```

---

## 📘 2. What is `Non-Repeatable Read`?

A **Non-Repeatable Read** occurs when a transaction reads the **same row twice** and gets **different results** because another transaction **updated or deleted** that row in between.

### ⚠️ Happens When:

- 🔁 A row is read **twice** during the same transaction.
- 🔧 Another transaction **updates or deletes** that same row between the two reads.
- ✏️ It typically involves **`UPDATE`** or **`DELETE`** on existing rows.


### 📋 Example Scenario:

1. 🧾 **TXA** starts and reads a row ➡️ gets **initial value**.
2. 🔧 **TXB** updates or deletes that row ➡️ and **commits**.
3. 🧾 **TXA** reads the same row again ➡️ sees **new value** or finds it **missing**.


## 🛠️ Solution: 

To prevent non-repeatable reads, you need to **control the transaction isolation level**.

### ✅ Recommended Isolation Level:

- 🔐 **`REPEATABLE_READ`** or higher  
  Ensures the row data **doesn’t change** during the transaction by using locks mechanism.

- 🧱 **`SERIALIZABLE`**  
  Strongest level: prevents **all concurrency issues** including **phantom reads**.

``` java
   @Transactional(isolation = Isolation.REPEATABLE_READ) // prevents dirty reads and non-repeatable reads but still occurs phantom reads
  or `Isolation.SERIALIZABLE` // prevents dirty reads , non-repeatable reads and phantom reads.
```

---


## 📘 3. What is `Phantom Read`?

Running the same query with a condition twice and getting a different number of rows because another transaction `inserted, deleted, or updated rows` that now match/don’t match the condition.

### ✅ Phantom Reads happen when:

- A transaction reads a set of rows matching a condition (e.g., `WHERE department = 'HR'`).
- Another transaction inserts, deletes, or updates other rows so that they appear or disappear from the result set of the same query when it’s run again.
- This is about **new rows appearing or disappearing** — not the same row changing.

### 🔁 Phantom reads can be caused by:

- 📥 **INSERT** – A new row is added that now matches the query condition.
- ❌ **DELETE** – A row that previously matched is removed.
- ♻️ **UPDATE** – A row is changed so that it starts to match or no longer matches the query condition.

## 📋 Example:

1. **TXA** runs:  
   `SELECT * FROM employee WHERE department = 'HR';`  
   → gets 2 rows

2. **TXB** inserts or updates a row to make it `department = 'HR'` and commits

3. **TXA** runs the same SELECT again  
   → now gets 3 rows


## 🧩 Solution:

To prevent Phantom Reads in **Spring**, use the highest isolation level:

- `Isolation.SERIALIZABLE` (prevents phantom reads) , SERIALIZABLE  will `apply lock on range` which satisfies `WHERE` clause

You can set this in Spring's `@Transactional` annotation:

```java
   @Transactional(isolation = Isolation.SERIALIZABLE)
```

---


# Database Phenomena Overview

| 🔑 Concept           | 🎯 Focuses On               | ⚠️ Affects                                               |
|----------------------|----------------------------|----------------------------------------------------------|
| 🔄 **Non-Repeatable Read** | 🧩 A specific row             | 🔄 Reading the same row gives different values            |
| 👻 **Phantom Read**         | 📊 A set of rows (result set)  | 👻 Running the same query (with a condition) gives more/fewer rows |

---
