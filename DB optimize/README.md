# ⚠️ N+1 Problem in JPA

### 🧩 What is the N+1 Problem?

In **JPA (Java Persistence API)**, the **N+1 problem** occurs when you fetch a list of parent entities (e.g., `Post`) and each entity has a lazily-loaded child collection (e.g., `List<Comment>`).  
Accessing those lazy-loaded associations one by one causes **N additional queries**, resulting in performance degradation. 

  - 1 query to fetch all Posts. 
  - N queries to fetch Comments for each Post.

This results in performance issues and unnecessary DB load.

### 🛠 Why is it bad?

  - More network traffic to the database. 
  - Slower performance as the number of comments increases. 
  - Hard to detect unless you're monitoring SQL logs.

#### 🔍 Example Scenario

Suppose you want to fetch all blog posts and then print their comments:
##### 🚫 Problem: N+1 with LAZY fetch
``` java
    List<Post> posts = postRepository.findAll(); // 1 query internally calls SELECT * FROM POSTS;
    
    for (Post post : posts) {
      List<Comment> comments = post.getComments(); // N queries
      /*
         SELECT * FROM employees WHERE post_id = 1;
         SELECT * FROM employees WHERE post_id = 2;
         ...
         SELECT * FROM employees WHERE post_id = N;
      */
    }
    
    // So if there are 10 Posts, this will run:
    // 1 query for posts
    // 10 queries for Comments
    👉 Total: 11 queries (hence called "N+1" problem)
```

### ✅ Solutions:

  - `@EntityGraph` 
  - `JOIN FETCH`

#### ✅ Example: JOIN FETCH
  - Now, all `Posts` and their `Comments` are fetched in `1 query`.
  - ``` java
       Example 1: 
    
             @Query("SELECT p FROM Post p JOIN FETCH p.comments")
             List<Post> findAllWithComments();
    
       Example 2:
    
            @Query("""
                     SELECT p FROM Post p
                     JOIN FETCH p.comments
                     JOIN FETCH p.author
                     JOIN FETCH p.category
            """)
            List<Post> findAllWithDetails();
       // You must handle duplicate rows and pagination issues manually.
       // It gets messier with deeper nested relations.
    
    ```
#### ✅ Example: `@EntityGraph`
  - Declarative way to avoid N+1. 
  - Cleaner and easier to manage. 
  - Works well with Spring Data JPA
  - Much `cleaner` for `multiple relationships`.
  - ``` java
        Example 1:
    
            @EntityGraph(attributePaths = {"comments"})
            List<Post> findAll();
    
        Example 2: 
    
           @EntityGraph(attributePaths = {"comments", "author", "category"})
           List<Post> findAll();
    ```

## 🆚 Why not just use JOIN instead of JOIN FETCH?

### ✅ JOIN (without FETCH)

- It joins the tables at the `SQL level`.
- But it does not fetch the association eagerly. JPA still follows the original fetch type (e.g., LAZY).
- So you may `still trigger N+1 queries` because the associated entities (e.g., comments) aren’t loaded into the persistence context — they're still lazy.
- It is typically `used for filtering` (`WHERE c.status = 'active'`), `not for fetching`.

### ✅ JOIN FETCH

- It tells `JPA to eagerly load the association` in the `same query`.
- So `no extra queries are executed` later when accessing the association.
- It avoids the N+1 problem and populates the associated entities in the persistence context.

---

# 📉 Missing or Improper Indexes

### 🔍 What is an Index?

An index in a database is like the index in a 📖 book —  
it helps you `find information faster` `without` `reading every single page` (row).

### 💣 What goes wrong without indexes?

When you run a query on an un-indexed column, the `database has to scan every row` — called a full table scan  
which is `🐌 slow for large tables`.

#### 💻 Example (Spring Boot + JPA)
- If there's `no index on the email column`, the database must `check every user row`. On a large table, this is slow.
  ``` java
        @Query("SELECT u FROM User u WHERE u.email = :email")
        User findByEmail(String email);

  ```
### ✅ How to fix it: Add an index by using `@Index`
- Now `searches on email are very fast` because the database can `jump directly` to the right row.
  ``` java
     @Entity
     @Table(name = "users", indexes = @Index(name = "idx_email", columnList = "email"))
     public class User {
      ...
   }
  ```
  ## ❌ Problems of Using Indexes
1. Slower Write Performance

- Every time you INSERT, UPDATE, or DELETE a row, the database also has to update the associated index(es).
- More indexes = more overhead during write operations.
Example:
 - If you update the email column, and you have an index on it, the DB must update the index as well. This adds latency.

---

# 🔍 Over-Fetching Data

### What is Over-Fetching?

Over-fetching happens when your application retrieves more data than it actually needs — either:
  - 📄 Too many rows, or
  - 📋 Too many columns (entire entities with all fields)

This can slow down your app, especially over the network.

### 💻 Bad Example (Spring Boot + JPA)

   - we're `fetching every field` (address, profile, photo, etc.) but `only using the emai`l. That’s over-fetching.
   ``` java
         List<User> users = userRepository.findAll(); // fetches all columns
      - Then you just use:
         for (User u : users) {
           System.out.println(u.getEmail());
         }
   ```
### ✅ Solution :
  - Use `DTOs Constructor Expression` (class/record) / `Spring Data Projection`

### ✅ Option 1: Using DTO  Constructor Expression (`Class`)
   1. Create your DTO class with required fields along with setters and getters (by using lombok or manually)
   2. Write a `custom query` in your `UserRepository`:
       - ``` java
              public interface UserRepository extends JpaRepository<User, Long> {

                   @Query("SELECT new com.example.dto.UserEmailDto(u.name, u.email) FROM User u")
                   List<UserEmailDto> findAllUserEmails();
              }
         ```
         - `Key point`: The `@Query must match` the `constructor of UserEmailDto` class.
       
   3. **📋 Summary**
       - A normal Java class with fields, constructor, getters
       #### ⭐️ Pros:
         - Familiar to all Java versions. 
         - Can add methods and logic.

       #### ❌ Cons:
         - More boilerplate code. 
         - Mutable unless you make fields final.

## What is a Java Record? 📚
   - Introduced in Java 14 (preview) and finalized in Java 16. 🚀
   - A record is a special kind of class designed to be a `simple, immutable data carrier`. 🎯
   - It `automatically creates`:
       - 🔒 private final fields,
       - 🏗️ a constructor, 
       - 🎛️ getters, 
       - ✔️ equals(), hashCode(), and 
       - 📝 toString() methods.

## Why use record for DTOs? 🤔

DTOs (Data Transfer Objects) `often just hold data` and `don't have behavior or mutable state`. Using a record makes your code:

   - ✨ **Cleaner**: Less boilerplate code. 
   - 🔐 **Immutable**: Once created, the data can't change. 
   - 🧹 **Concise**: One line to declare a data holder class.

### No Annotations Needed for Getters or Setters 🚫✨

 Unlike traditional Java classes where you might use libraries like Lombok (`@Getter`, `@Setter`, `@Builder`), **Java Records automatically provide all the getters for you** without needing any annotations.

   - **No setters:** Records are immutable, so no setters are generated or needed.
   - **No Lombok:** No external dependencies are required for boilerplate code.
   - **Built-in concise syntax:** Everything you need is included in the record declaration itself.

This makes records a clean and minimalistic choice for simple data carriers!

### ✅ Option 2: Using DTO Constructor Expression (`Record`)
   1. Create your DTO Record: 
      ``` java
            package com.example.dto;

            public record UserEmailDTO(String name, String email) {}
      ```
   2. Use a JPQL query with the record constructor
      ``` java
          import com.example.dto.UserEmailDTO;
          public interface UserRepository extends JpaRepository<User, Long> {

              @Query("SELECT new com.example.dto.UserEmailDTO(u.name, u.email) FROM User u")
              List<UserEmailDTO> findAllUserEmails();
          }
      ```
   
   3. **📋 Summary**
      - A concise, immutable data holder.
      #### ⭐️ Pros:
        - Very concise. 
        - Immutable by default. 
        - Auto-generated equals, hashCode, toString.

      #### ❌ Cons:
        - Requires Java 16+. 
        - No custom setters or additional mutable logic.
      
### ✅ Option 3: Using Spring Data Projection (`Interface-Based`)
   - This is simpler for `read-only use cases`.
   1. Create a projection interface:
      ``` java
          public interface UserEmailProjection {
             String getName();
             String getEmail();
          }
      ```
   2. Use it in your repository:
      -  ✅ Spring will `automatically create instances` of UserEmailProjection for `each row returned`.
      ``` java
          public interface UserRepository extends JpaRepository<User, Long> {

            List<UserEmailProjection> findAllBy();
          }
      ```
   3. **📋 Summary**
      - An interface with getter methods matching fields you want.
      #### ⭐️ Pros:

      - Very lightweight. 
      - No need to write constructors. 
      - Spring Data automatically creates implementation at runtime.

      #### ❌ Cons:
      - Read-only, no behavior or extra methods. 
      - Can be less flexible for complex data transformations.

---

# 🐌 Poor Pagination Strategy (e.g., using `OFFSET` on large datasets)

### 🧠 What is OFFSET?
`OFFSET` is a keyword in SQL that tells the database:

> “Skip the first N rows, and then return the next few rows.”

### ❌ What's the Problem with OFFSET in Pagination?

   #### 🚨 Problem:
   - The **higher the offset**, the **slower the query** becomes.
   - **Why?**  
       - Because the database **still reads all the previous rows** `internally before skipping` them.
     
   #### 🔍 Real-world Analogy:
   - Imagine you’re in line at a bank with **10,000 people**.
   - You say:
      > “I want to serve customer number 9,001.”
   - The clerk still has to **count and skip the first 9,000 people**, even though they don’t help them. That wastes a lot of time.
   - ☝️ This is what `OFFSET` does.

### 🔧  How It Looks in Spring Boot + JPA

   #### ❌ Default JPA Pagination (Bad for large data):

   ``` java
   Pageable pageable = PageRequest.of(900, 10); // Page 901, 10 items Offset use the Zero-based indexing
   Page<User> page = userRepository.findAll(pageable);
   ```
   - Behind the scenes, it runs:
   ``` java
   SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 9000;
   ```
   - It skips 9,000 users to give you just 10. `That’s slow`.

### ✅ Solution: `keyset pagination`
   - Use keyset pagination (also known as "`seek pagination`"),   
   - Instead of skipping, we can say:
      > “Give me the next 10 users after the last user I saw.”
   - This is called keyset pagination.
   ``` java
   // Controller
   @GetMapping("/users/next")
   public List<User> getUsers(@RequestParam Long lastId,
                           @RequestParam(defaultValue = "10") int size) {
      return userService.getNextUsers(lastId, size);
   }
   
   // Service
   public List<User> getNextUsers(Long lastId, int size) {
    Pageable pageable = PageRequest.of(0, size); // always page 0
    return userRepository.findNextPage(lastId, pageable);
   }

   // Repository
   public interface UserRepository extends JpaRepository<User, Long> {

      @Query("SELECT u FROM User u WHERE u.id > :lastId ORDER BY u.id ASC")
      List<User> findNextPage(@Param("lastId") Long lastId, Pageable pageable);
   }
   
   // if we pass lastId value as 9000 then 
      u.id > 9000 here instead of skiping it will directly goes to the 9001 user and print users from 9001 to 9010
      
   ```

---

## Persistence Context (JPA Workspace) 📦

   - The persistence context is the **in-memory workspace or environment** managed by the **EntityManager**. It keeps track of all entity instances you work with during a transaction, including their changes, so it knows what to write to the database when the transaction ends.
   - It’s a **runtime container** that keeps track of entity instances within a transaction. 
   - Manages entities you’re working with: **newly created, updated, or deleted**. 
   - Ensures changes are synchronized (**flushed**) to the database at the right time. 
   - Exists only during a **transaction** or **EntityManager lifecycle**. 
   - Prevents multiple database calls for the same entity within the transaction by returning the **same Java object**.

   ### Important Persistence Context Methods ⚙️

   #### `flush()` method 💧

   - Think of `flush()` as **"pushing" all your changes right now** from memory to the database. 
   - It saves new, updated, or deleted entities immediately to the database instead of waiting for the transaction to finish. 
   - Typically called automatically before transaction commit, but can be invoked manually to control when changes are sent to the database.

   #### `clear()` method 🧹

   - Think of `clear()` as **"cleaning up" the workspace**. 
   - It removes all entities from the persistence context, so the EntityManager forgets about them. 
   - After calling `clear()`, if you change those entities, those changes won’t be saved unless you reattach them. 
   - Useful to release memory or reset the EntityManager state during `long transactions` or `batch processing`.

## Caching 🗄️

Caching refers to storing entities or query results to avoid unnecessary database hits. It comes in two main types:

   ### First-Level Cache (L1 Cache) 🔄
    
   - This is actually the **persistence context itself**.
   - Automatically enabled per **EntityManager**.
   - Only lasts for the duration of the **EntityManager** or **transaction**.
   - No separate config needed.

   ### Second-Level Cache (L2 Cache) ⚡

   - A **shared cache** across multiple sessions or transactions. 
   - Configured explicitly (e.g., with Hibernate’s cache providers like **Ehcache**). 
   - Stores entities between transactions, improving performance by avoiding DB calls for frequently accessed data. 
   - Works across multiple users and requests.


## Where Do We Use Persistence Context and Caching? 🔍

- **Persistence Context** is used during the lifecycle of a transaction or EntityManager. It helps manage the state of entities, track changes, and ensure data integrity before committing to the database.
- It is essential in `transactional workflows`, where you want to `batch operations and flush all changes` atomically.
- **Caching** improves performance by reducing the number of database calls:
    - **First-Level Cache** avoids repeated fetching of the same entity within a transaction.
    - **Second-Level Cache** is used in applications with high read frequency or shared data across sessions, improving scalability and reducing DB load.


# In short: 📊

| Aspect             | Persistence Context                 | Cache                            |
|--------------------|-----------------------------------|---------------------------------|
| **Scope**          | One transaction / EntityManager   | Multiple transactions / app-wide|
| **Purpose**        | Track changes, manage entity states| Improve performance by reusing data|
| **Lifetime**       | Short-lived                       | Longer-lived (L2 cache)          |
| **Configuration**  | No (built-in)                     | Yes (L2 cache setup required)    |


---

# 🚀 Unbatched Inserts/Updates in Spring Boot + JPA

### 🧠 What does it mean?
   - If you're inserting or updating many records **one by one**, JPA will send a **separate SQL query for each record** — and that’s **very slow**. 
   - This is called **unbatched operations**.

### 💻 In Spring Boot + JPA:
    
   #### ❌ Example (unbatched insert):
   ``` java
   for (User user : userList) {
       userRepository.save(user); // Sends 1 insert per user
   }
   ```
   - This results in **N separate `INSERT` statements**.

   #### ❌ Example Without batch configuration:

   > 🗨️ *"If I use `userRepository.saveAll(userList)`, doesn't that mean it's doing batch inserts automatically?"*

### 🧨 The reality: No, not really (by default)

Even though `saveAll(...)` **looks like a batch insert**, Hibernate will still send **individual insert statements**  
**unless you explicitly configure batching**.
``` java
userRepository.saveAll(userList);

// under hood (one SQL per row):
INSERT INTO user (...) VALUES (...);
INSERT INTO user (...) VALUES (...);
INSERT INTO user (...) VALUES (...);
...
```

⚠️ Hibernate **doesn’t batch unless you tell it to explicitly**.

### ✅ Solution 1 : Enable Batch Inserts/Updates

Step 1: Add these `configuration` in application.properties or application.yml
   ``` java
   spring.jpa.properties.hibernate.jdbc.batch_size=50
   spring.jpa.properties.hibernate.order_inserts=true
   spring.jpa.properties.hibernate.order_updates=true
   ```
   - This tells Hibernate:
        - "Group INSERTs/UPDATEs together into batches of N size"
        - "Try to order them for better performance"

Step 2: Save in batches
  ``` java
   List<User> users = ...; // some list of new users
   userRepository.saveAll(users); // Hibernate will batch them , now list of users will saves together by one query
   // we are not manually flushing the data into db , jpa will behalf of use after the transaction is completed
   // no recommended try to use batching + transaction
  ```

### ✅ Solution 2 : Enable Batch Inserts/Updates with transaction annotation (`Recommended one`)
``` java
   spring.jpa.properties.hibernate.jdbc.batch_size=50
   spring.jpa.properties.hibernate.order_inserts=true
   spring.jpa.properties.hibernate.order_updates=true

```

``` java
  
  @Transactional
  public void saveUsers(List<User> users) {
      int batchSize = 50;
      for (int i = 0; i < users.size(); i++) {
         entityManager.persist(users.get(i));  // save each user to persistence context

        if (i % batchSize == 0 && i > 0) {
          entityManager.flush();  // push all changes to the database
          entityManager.clear();  // clear persistence context to free memory
        }
     }
    // Flush and clear remaining entities after loop finishes
    entityManager.flush();
    entityManager.clear();
  }
```


### ⚠️ Important Notes:

- 📏 **Batch size** depends on the **DB** and **JDBC driver** support.
- 🆔 Entities should **not have auto-generated IDs** like `GenerationType.IDENTITY`, which may **break batching**  
  👉 Use `SEQUENCE` or `TABLE` instead.
- ⚡ Works best when doing **many inserts/updates in one transaction**.

---

## 🛑 Not Utilizing Connection Pool Optimally (Slow Due to Waiting)

## ✅ Why an application becomes slow due to database connection issues

Here are the main reasons:

### 1. ⛔ Limited Connection Pool Size

If the connection pool size is too small (e.g. 10 connections):

- Only 10 DB operations can run at the same time.
- Any extra requests will be forced to wait.
- If waiting exceeds the timeout, errors will occur.

🔁 **Result:** Requests are delayed, app becomes slow under load.

### 2. 🐢 Long-running or Slow Queries

If your queries are slow or inefficient, they hold connections longer.  
This means:
- Even if only a few users are active, they block connections for others.

🔁 **Result:** Connection pool gets exhausted even with low traffic.

### 3. 🔄 Multiple Queries per Request (N+1 Problem)

If you're using lazy loading with JPA without JOIN FETCH, one request can make dozens of DB queries.  
Each query needs a connection.

### 4. 🔒 Poor Transaction Handling

Holding a transaction open for too long (e.g., doing file I/O or API calls inside a `@Transactional` method).  
Keeps a DB connection busy even though the DB is idle.

🔁 **Result:** Connections are tied up unnecessarily, causing other requests to wait.

### 5. 💤 Connections Not Released Quickly

If you’re not properly returning connections to the pool (e.g., due to coding mistakes or poor resource management), they stay "checked out".  
This reduces the available connections for other users.

🔁 **Result:** Pool looks full even though no real DB activity is happening.

---

## 🔧 Solution: Tune the Connection Pool Size

### 🛑 Problem:
Default is just **10 connections**.

### ✅ Solution:
Increase it based on traffic and database capacity.

### 🔍 How:
In your `application.properties`:

```properties
spring.datasource.hikari.maximum-pool-size=30      # 🔢 Max number of connections in pool (Active + Idle)
spring.datasource.hikari.minimum-idle=10           # 💤 Minimum number of idle connections
spring.datasource.hikari.idle-timeout=30000        # ⏱️ 30 seconds , Time in ms before an idle connection is removed
spring.datasource.hikari.max-lifetime=1800000      # ⌛ 30 minutes , Max lifetime of a connection in ms
spring.datasource.hikari.connection-timeout=30000  # ⌚ 30 seconds , Max time to wait for a connection
spring.datasource.pool-name = MyHikariCP           # 🏷️ Optional: naming the pool for logs
```
### ⚠️ Important Notes:
  - Don’t go too high — it can overwhelm your DB. 
  - 📈 Monitor and tune carefully based on performance metrics.

---

#### 🔢 `maximum-pool-size = 30`
This is the **maximum number of database connections** that can exist at the same time in the pool.

This includes:
- 🟢 **Active** (in-use) connections
- 🔵 **Idle** (available but not in-use) connections

## 🔧 HikariCP Connection Pool Behavior with 40 Users

#### 🧮 Assumptions:

- `maximum-pool-size = 30`
- All **40 users** send requests at the **same time**
- Each user requires a **DB connection** to process the request

#### 🧱 What Happens Step by Step:

##### ✅ First 30 Users:

- 🟢 Each gets a **connection from the pool** (active or idle)
- 🛠️ DB handles them **simultaneously** (assuming DB supports 30 connections)

##### ⏳ Remaining 10 Users:

- 🔒 **No available connection** in the pool
- 🕒 These users must **wait** until a connection is **released** by one of the 30 active users

##### 🔄 Waiting Behavior: `connection-timeout`

HikariCP will **queue the remaining 10 requests**, waiting for a connection to become available.

##### ⏱️ How long do they wait?

That depends on:
- `spring.datasource.hikari.connection-timeout`  
  _(e.g., 30000 ms → 30 seconds)_

##### ⚙️ Outcome:

- ✅ If a **connection becomes available** within the timeout → the request **proceeds**
- ❌ If **not**, the request **fails** with the exception: `SQLTransientConnectionException` or `PoolAccessTimeoutException`.

#### ❓ So What is `idle-timeout` For?
- 🔄 idle-timeout =  Hikari starts closing those extra idle connections
  ##### 💡 Purpose of `idle-timeout`:
   - 🧹 Frees up **resources** (DB memory, CPU) by **closing unnecessary idle connections**
   - 📈 Especially useful in **spiky traffic patterns** — when many connections are needed **briefly**, but not **long-term**
  ##### 🔒 Important Safeguard:
    - 🛡️ HikariCP will **never reduce the number of idle connections below the `minimum-idle` value** unless demand requires it.
    - 🔄 HikariCP always tries to keep at least the number of idle connections you set with `minimum-idle`. 
    - ⚠️ But if **all connections in the pool are busy** (no idle ones left), it can’t keep that minimum idle count because every connection is in use. 
    
---

### ❓ Question:

> "If we have 10 users, then 10 users can work together. Will it use idle connections or active connections?"

#### ✅ Answer:
  - 🛠️ When a user needs a connection, HikariCP will **first use an idle connection**, if available. 
  - 🔄 Once the connection is assigned to a user, it becomes an **active connection** (no longer idle). 
  - ⚙️ So technically, connections **start as idle**, and when a user/thread uses one, it **switches to active**.
