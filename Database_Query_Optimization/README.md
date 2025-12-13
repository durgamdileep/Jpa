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


# 🗄️ Optimize Database Schema

## 1. 📌 What is Database Schema Optimization?

- The database schema is how your tables, columns, and relationships are organized.
- Optimizing the schema means structuring your data efficiently so queries run faster and use less resources.  

**Analogy:**  

📚 Think of your database like a library.  

- ❌ Bad schema → books scattered randomly, takes forever to find anything.  
- ✅ Good schema → books organized into sections and shelves, fast to locate.  

## 2. ⚡ Why Optimize Schema?

- ⏱️ Reduce query execution time.  
- 🔗 Avoid unnecessary joins or data duplication.  
- 💾 Improve storage efficiency.  
- 🔍 Makes indexing and searching faster.  

## 3. 🧠 Key Concepts in Schema Optimization

### A. 🗃️ Normalization

- 🎯 Goal: Remove duplicate data and organize into logical tables.  
- 📏 Rules: 1NF, 2NF, 3NF (basic normalization rules).  
**Example**
- ❌ Bad design: 
  ``` java
       Orders table:
    | order_id | customer_name | customer_email | product | amount |

  ```
  Problem: Customer info repeats in every order → waste of space and hard to update.  

- ✅ Normalized design:
  ``` sql
          In Database
             Customers table:
          | customer_id | name | email |
          
          Orders table:
          | order_id | customer_id | product | amount |
      
  ```
  ``` java
    
         - In Springboot ( use GeneratedValue to prevent redundancy and normalize the entity in two separate entity and make relationship among them
         
                @Entity
          public class Customer {
              @Id
              @GeneratedValue(strategy = GenerationType.IDENTITY)
              private Long id;
          
              private String name;
              private String email;
          }
          
          @Entity
          @Table(name = "orders", indexes = {
                  @Index(name = "idx_customer_id", columnList = "customer_id")
                })
          public class Order {
              @Id
              @GeneratedValue(strategy = GenerationType.IDENTITY)
              private Long id;
          
              private String product;
              private Double amount;
          
              @ManyToOne
              @JoinColumn(name = "customer_id")  // foreign key to Customer
              private Customer customer;
          }
    
          /**
          - ✅ Benefits:
             - No duplicate customer info.
             - Easy to update customer details.
             - Queries can use indexes on customer_id.
             - Spring Boot/JPA will create the index automatically when generating the schema.
             - here in Order class we are using join cloumn consider as Foreign key
           **/  
    
  ```
   - Benefits: No duplicate customer info, easier updates, smaller storage. 

  ### B. 🔄 Denormalization

- 🎯 Goal: Combine tables or add redundant columns for faster reads, especially when joins are slow.  
- ⏱️ When to use: High read frequency and performance-critical queries.  

**Example:**  

- If you frequently need order + customer name:
    ``` java
          Orders table:
      | order_id | customer_id | customer_name | product | amount |
  
    ```
  - Now you `don’t need to join Customers table` `every time` → faster query.
  - ⚖️ Trade-off: Takes `more storage` and can `make updates slower`.  

### C. 📐 Proper Data Types

- 💾 Use the smallest suitable data type to save storage and improve query speed.  

**Example:**  

- Use `INT` instead of `BIGINT` if values are small.  
- Use `VARCHAR(50)` instead of `TEXT` for short strings.

``` java
   @Column(length = 50, nullable = false)
   private String product
```

### D. 🔍 Indexing

- 🏎️ Proper indexes on frequently searched columns speed up queries.  

**Example:**  

- Add index on `customer_id` in Orders if filtering orders by customer often.  
- ⚠️ Caution: Too many indexes → slower inserts/updates.  

### E. 🔑 Primary and Foreign Keys

- 🗝️ Primary key: Uniquely identifies each row. Helps fast lookup.  
- 🔗 Foreign key: Defines relationship between tables. Helps joins and ensures data integrity.  

### F. 🚀 Partitioning and Sharding (Advanced)

- 🧩 Partitioning: Split large tables into smaller parts (by date, region, etc.) → faster queries.  
- 🌐 Sharding: Split data across multiple servers → scale horizontally for huge datasets.  

### G. ⚙️ Schema Generation Options

- 🛠️ Spring Boot can automatically create or update schema via `spring.jpa.hibernate.ddl-auto`.
   ``` java
     spring.jpa.hibernate.ddl-auto=update
   ``` 
- Options: `validate | update | create | create-drop`.  
- ✅ Useful during development to reflect optimized schema changes automatically. 

##  📝 Summary for Optimizing Schema

- 🔄 Normalize to remove redundancy.  
- ⚡ Denormalize for read-heavy queries.  
- 📐 Use correct data types.  
- 🔍 Add indexes on frequently queried columns.  
- 🔑 Use primary and foreign keys to maintain integrity.  
- 📊 Analyze queries with EXPLAIN after schema changes.
