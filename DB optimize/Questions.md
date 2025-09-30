# 📚 Difference Between JPA and Hibernate ⚙️

- 📜 **JPA (Java Persistence API):**  
  JPA is like a **set of rules** (a standard) for working with databases in Java.  
  It defines how Java objects should be mapped to database tables, but does **not** provide an implementation.

- 🛠️ **Hibernate:**  
  Hibernate is a **tool** (a library) that follows those rules.  
  It is an implementation of the JPA specification and provides additional features for database operations.

---

# 🟢 2. EntityManager: `persist()` vs `merge()`

This document outlines the core differences between `EntityManager.persist()` and `EntityManager.merge()` in JPA (Java Persistence API).

| 🧠 **Method**     | 🎯 **Purpose**                         | 🧩 **Works On**                     | 🔁 **Returns**         |
|------------------|----------------------------------------|-------------------------------------|------------------------|
| `persist()`      | ➕ Insert a **new** entity into the DB | 🆕 Only **new (transient)** objects | 🚫 `void` (no return)  |
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



