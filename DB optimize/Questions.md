# 1. 📚 Difference Between JPA and Hibernate ⚙️

- 📜 **JPA (Java Persistence API):**
  - 🧩 JPA (Java Persistence API) is a specification that `defines a set of rules for object–relational mapping (ORM)` and `for managing data between Java objects and relational databases 🗄️`.
  - 🔌 It does not handle database connections by itself and does not eliminate queries entirely.
  - 📦 It maps Java classes to database tables using entities, but does **not** provide an implementation.

- 🛠️ **Hibernate:**  
  Hibernate is a **tool** (a library) that follows those rules.  
  It is an implementation of the JPA specification and provides additional features for database operations.
  - 🧠 Hibernate is a framework and a JPA implementation.
  - 💾 It provides the `actual functionality` to persist, retrieve, update, and delete data in the database.
  - 🧾 It supports `JPQL`, `Criteria API`, and `native SQL queries`, and it reduces the need to write SQL in many cases.


---

# 2. 🔄 JPA Entity Lifecycle States

## 🆕 New (Transient)

- 🛠️ The entity is just created but **not saved** in the database yet.
- 👀 JPA doesn’t know about it.
  
## 📦 Managed (Persistent)

- 💾 The entity is **saved** in the database or loaded from it.
- 🔍 JPA is **tracking changes** to it.

## 🔌 Detached

- 🚪 The entity was once managed but now is **disconnected** from JPA (for example, after closing the session).
- ⚠️ Changes won’t be saved unless reattached.

## 🗑️ Removed

- 🏷️ The entity is **marked for deletion** from the database but **not deleted yet**.
- ✔️ Once the transaction commits, it will be deleted.

``` java
    User user = new User();  // New/Transient - not saved yet
    entityManager.persist(user);  // Managed - saved in DB, JPA tracks it

    entityManager.detach(user);  // Detached - no longer tracked by JPA
    entityManager.remove(user);  // Removed - marked for deletion in DB
```

---

# 3. 🟢 EntityManager: `persist()` vs `merge()`

This document outlines the core differences between `EntityManager.persist()` and `EntityManager.merge()` in JPA (Java Persistence API).

| 🧠 **Method**     | 🎯 **Purpose**                         | 🧩 **Works On**                     | 🔁 **Returns**           |
|------------------|----------------------------------------|-------------------------------------|-----------------------------|
| `persist()`      | ➕ Insert a **new** entity into the DB | 🆕 Only **new (transient)** objects | 🚫 `void` (no return)      |
| `merge()`        | 🔄 Update existing or insert if not exist | 🔌 **Detached** or **new** objects | ✅ Managed entity (copy) |


## 📌 Summary

- **`persist()`**:
  - Used to **add** a new entity to the persistence context.
  - Throws an exception if the entity already exists.
  - Does **not** return the entity or any result.

- **`merge()`**:
  - Used to **update** an entity or insert it if it doesn't exist.
  - Can work with **detached** entities (previously persisted but not currently managed).
  - Returns a **new managed** instance (copy of the passed entity).

### 🧪 Example
``` java
import jakarta.persistence.*;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // Constructors
    public User() {}

    public User(String name) {
        this.name = name;
    }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

```

### ✅ Using `persist()` – Only for New Entities  
`persist()` should be used **only for new entities** that do **not** have an existing ID.
``` java
      EntityManager em = entityManagerFactory.createEntityManager();
      EntityTransaction tx = em.getTransaction();
      tx.begin();
      
      User user = new User("Alice"); // New (transient) object
      
      em.persist(user); // Managed and inserted into DB
      
      tx.commit();
      em.close();
```
🔹 If you try to use `persist()` on a detached or already existing entity (i.e., it has an existing ID),  
it will throw an exception:

> ❌ `EntityExistsException`

## ✅ What is a "Detached" Entity in JPA?

A **detached object** is an entity that:

🔹 Was once managed by the `EntityManager` (i.e., loaded or persisted in a previous session/transaction)

🔹 But now is `outside` the persistence context (e.g., after `em.close()` or between transactions)

🔹 It has an ID (primary key), but is no longer tracked by JPA.

### 📘 Term | 🧠 Meaning

| Term       | Meaning                                                                     |
|------------|-----------------------------------------------------------------------------|
| 🆕 Transient | New object, not in DB, no ID                                              |
| ✅ Managed   | Currently tracked by `EntityManager`                                      |
| 🔄 Detached  | Previously managed, now not tracked (usually after `em.close()`)          |

### ✅ Using `merge()` – For New or Detached Entities
``` java
      // 1. Load entity in one EntityManager
      EntityManager em1 = emf.createEntityManager();
      User managedUser = em1.find(User.class, 1L); // now managed
      em1.close(); // now managedUser becomes detached
      
      // 2. Modify detached object
      managedUser.setName("Updated Name");
      
      // 3. Merge in a new EntityManager
      EntityManager em2 = emf.createEntityManager();
      em2.getTransaction().begin();
      
      User mergedUser = em2.merge(managedUser); // managed copy returned
      
      // mergedUser is MANAGED, `user` is still DETACHED
      
      mergedUser.setName("Another Update"); // This will be saved
      managedUser.setName("Yet Another Update");   // This will NOT be saved 
      
      em2.getTransaction().commit();
      em2.close();
```
🔄 `merge()` always returns a ✅ `managed` instance, while the original object stays 🔄 `detached`.

### 📝 Notes

- ✅ `persist()` should only be used with **new (transient)** entities. makes the object **managed** and schedules an **INSERT**.
- ⚠️ Using `persist()` on a **detached** entity will throw an exception.

- 🔁 `merge()` can handle both **new** and **detached** entities, but it returns a **new managed instance**.
- 🛡️ `merge()` is safer if you're not sure whether the entity is new or detached.
- 🧼 Always remember: `merge()` **does not update the passed object**—it returns a new managed copy.


---

# 🆚 JPQL vs Native SQL Queries

## 📝 JPQL (Java Persistence Query Language)

- 🔹 JPQL is an **object-oriented query language** defined by JPA.  
- 🔹 It operates on **entity objects** rather than directly on database tables.  
- 🔹 When you execute a JPQL query (e.g. `SELECT e FROM Employee e`), **JPA automatically maps the result into entity objects**.  
- ✅ Hence, you don’t need to manually convert the table data into entity fields — the **persistence provider (like Hibernate) handles that**.

``` java
   List<Employee> employees = entityManager
    .createQuery("SELECT e FROM Employee e", Employee.class)
    .getResultList();

```
- Here, `each row from the Employee table` is automatically converted into an `Employee` entity.

### ✅ Advantages
- 🌐 Database independent (portable across DBs)  
- 🧩 Object-oriented (uses entities & fields, not tables & columns)  
- 🔄 Automatic mapping to entity objects  
- 🛡️ Safer and easier to maintain  
- ⚡ Works well with JPA features (caching, lazy loading, relationships)  

### ❌ Disadvantages
- 🚫 Limited access to database-specific features  
- 🐢 Can be less efficient for complex or highly optimized queries  
- ❌ Not suitable for vendor-specific SQL functions  
- 🔍 Debugging can be harder (JPQL → SQL translation is hidden)

  
## 🗄️ Native SQL Query

- 🔹 A **native query** directly uses SQL and works on **database tables** rather than entity objects.  
- 🔹 When you execute a native query, it returns **raw database rows** (typically as `Object[]` or scalar values).  
- ⚠️ Therefore, you need to manually **map those results to entities or DTOs (Data Transfer Objects)**, either:  
  - 🧩 **Raw `Object[]` mapping** → explicit conversion to entity/DTO.
      ``` java
       @Query(
         value = "SELECT name, salary FROM employee", 
         nativeQuery = true
       )
    List<Object[]> fetchEmployeeData(); // raw result, will map manually to DTO
    
    List<EmployeeDTO> employees = results.stream()
        .map(r -> new EmployeeDTO((Long) r[0], (String) r[1], (Double) r[2]))
        .collect(Collectors.toList());

     ```
  - ⚡ **Spring Data JPA DTO projection** → automatically mapped to DTO (interface or class-based).
      ``` java
         @Query(value = "SELECT name as name, salary as salary FROM employee", nativeQuery = true)
         List<EmployeeDTO> fetchEmployeeDTOs();
      ```
  - 🏗️ **`@SqlResultSetMapping` + `@NamedNativeQuery`** → maps native query results to DTO using constructor.
     ``` java
      inside Entity Employee Class
      @SqlResultSetMapping(
        name = "EmployeeDTOMapping",
        classes = @ConstructorResult(
            targetClass = EmployeeDTO.class,
            columns = {
                @ColumnResult(name = "name", type = String.class),
                @ColumnResult(name = "salary", type = Double.class)
            }
        )
     )
     @NamedNativeQuery(
        name = "Employee.fetchEmployeeDTOs",
        query = "SELECT name, salary FROM employee",
        resultSetMapping = "EmployeeDTOMapping"
     )

     in Respository Layer
     @Query(name = "Employee.fetchEmployeeDTOs", nativeQuery = true)
     List<EmployeeDTO> fetchEmployeeDTOs();

     ``` 


 

### ✅ Advantages
- 🎯 Full control over SQL  
- 🛠️ Can use database-specific features, hints, procedures  
- ⚡ Often better performance for complex queries  
- 🏗️ Useful for legacy databases or complex joins  

### ❌ Disadvantages
- 🌐 Database dependent (not portable)  
- 📝 Requires explicit result mapping in many cases  
- ⚠️ More error-prone and harder to maintain  
- ⛔ Bypasses some JPA features (caching, change tracking)  

## 📝 Summary
- Use **JPQL** for portable, maintainable, entity-based queries  
- Use **Native SQL** when you need `performance tuning` or `DB-specific features` 

