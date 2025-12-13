# 🔍 Analyze Queries

## 1️⃣ What is `Analyze Queries`?

- Checking how a query runs inside the database to find why it might be slow.
- Databases can show:
  - 🟢 `How many rows are scanned`
  - 📑 `Which indexes are used`
  - 🔗 `How joins are executed`  

Analogy: Like checking a library map to find the fastest way to a book instead of searching randomly.

---

## 2️⃣ How to `Analyze a Query` 🛠️

- Use the SQL command `EXPLAIN` (works in MySQL, PostgreSQL, etc.)
- What it shows:
   - 📌 `Indexes used`
   - 📊 `Number of rows scanned`
   - 🔀 `Order of operations`
``` sql
  EXPLAIN SELECT * FROM orders WHERE customer_id = 101;

  // output
    type: ALL
    rows: 100000
    possible_keys: NULL
    key: NULL

    What this means:

   - type: ALL → Full table scan (it looks at every row).
   - rows: 100000 → It checks 100,000 rows.
   - key: NULL → No index is used.

  This shows the query is inefficient if the table is big.
```
### Optimize Based on Analysis
- Solution: Add an index on customer_id:
``` sql
CREATE INDEX idx_customer ON orders(customer_id);

EXPLAIN SELECT * FROM orders WHERE customer_id = 101;

The database might tell you:
 - type: ref
 - rows: 2
 - key: idx_customer


 ✅ Improvement:

- It only checks 2 rows instead of 100,000.
- Query runs much faster.
```

- Shows if the `query uses indexes` or `does a full table scan`.  
- If a `full table scan occurs`, `adding an index` can improve performance.

---

## 3️⃣ Spring Boot Approach ☕️

- Even though Spring Boot doesn’t analyze queries itself, you can:
- 📝 `Log generated SQL queries`  
- This shows the actual SQL JPA/Hibernate is sending to the database.
  ``` java
      spring.jpa.show-sql=true
      spring.jpa.properties.hibernate.format_sql=true
      logging.level.org.hibernate.SQL=DEBUG
      logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
  
  ```
- 📋 `Copy and run the SQL in the database using EXPLAIN`  
- Example: If Spring Boot generates:

  ``` sql
    SELECT * FROM orders WHERE customer_id = 101;
  
  
     Run in Sql tool
     EXPLAIN SELECT * FROM orders WHERE customer_id = 101;
  
  ```

- Analyze the query plan for optimization (e.g., add indexes).

## Optimize Spring Boot queries:

- 🎯 `Use projections (SELECT specific_columns) instead of fetching full entities`  
- 🔗 `Use JOIN FETCH to avoid N+1 queries`  
- 🗂️ `Add indexes in the database for frequently filtered columns`

## ✅ Key Point

- Spring Boot helps generate and log queries, but `EXPLAIN` and database tools are needed to analyze performance.  
- If the `EXPLAIN` output shows a full table scan and no index is used, the query can be slow.  
- Adding an index on `customer_id` can make the query much faster.


--- 
