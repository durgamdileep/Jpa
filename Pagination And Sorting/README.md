# 📝 Pagination and 🔃 Sorting 

### 📄 What is `Pagination`?

Imagine you have a huge list of items (like products, users, or posts) in your database.  
Instead of loading `all items at once` (which is slow and inefficient),  
you load them `page by page`, for example, **10 items at a time**.

-  `🧮 Page number:` which page you want to see (page 0, 1, 2, ...).
-   `📏 Page size:` how many items you want on each page (10, 20, 50, ...).

> Pagination helps improve performance and user experience by breaking large data into manageable chunks.


### 🔃 What is `Sorting`?

Sorting means arranging the data in a particular order, such as:

- `🔤 Alphabetically by name:` A → Z or Z → A
- `📅 By date:` newest first or oldest first
- `💰 By price:` low to high or high to low

> Sorting helps users find and analyze data more easily by presenting it in a meaningful order.

---

### Spring Data `JPA Basics`
  #### 📌 What is it?
   - Spring Data JPA is a part of Spring Boot that helps you interact with a database easily without writing complex SQL.
   - And Spring Boot automatically gives you methods like:
        - `findAll()`
        - `save()`
        - `deleteById()`
        - `findById()`  
     
**✅ Main idea**: You don’t need to write SQL for basic operations.

---

### `JpaRepository` and `PagingAndSortingRepository`
   #### 📌 What are they?
   - These are `interfaces` provided by `Spring Data JPA`. When you use them,  
   - Spring gives you `ready-made` methods for working with the database.
        - `CrudRepository` → basic operations 
        - `PagingAndSortingRepository` → adds pagination + sorting 
        - `JpaRepository` → includes all the above + more JPA features

---

### `Pageable` and `Sort` Interfaces
   - These are helper tools in Spring Data JPA to make pagination and sorting work

📌 `Pageable Interfaces`  

  Used to define:
   - `page number` (which page to retrieve). 
   - `page size` (how many items per page). 
   - `sorting criteria` (How to sort the results)

``` java
   // Create a Pageable object: page 0, 10 items per page, sorted by lastName ascending
   Pageable pageable = PageRequest.of(0, 10, Sort.by("lastName").ascending());
```
PageRequest.of(0, 10, Sort.by("lastName").ascending()) means:
   - Get page number 0 (first page), 
   - With 10 users, 
   - Sorted by lastName in ascending order.

`📌 Sort`
   Used to define sorting rules.
   1. Sort by `multiple fields`
      - You can sort by `more than one field`. 
      ``` java
          //general way
          Sort sort = Sort.by("lastName").ascending()
                           .and(Sort.by("firstName").descending())
                           .and(Sort.by("age").ascending())
                           .and(Sort.by("city").descending());


         // Cleaner Alternative and compact(java 8+)
          Sort sort = Sort.by(
                                Sort.Order.asc("lastName"),
                                Sort.Order.desc("firstName"),
                                Sort.Order.asc("age"),
                                Sort.Order.desc("city")
                            );
      ```
   2.  Sort with `multiple orders at once`
       - multiple orders at once
       ``` java
         Sort sort = Sort.by(
                     new Sort.Order(Sort.Direction.ASC, "lastName"),
                     new Sort.Order(Sort.Direction.DESC, "firstName")
         );
       ```
   3. Sort `without chaining`
      - You can just create a sort for one field:
      ``` java
          // defaults to ascending
          Sort sort = Sort.by("age"); 
          // ascending() -> for ascending order
          // descending() -> for descending order
      ```
---

### `Page`, `Slice`, and `List` Differences

#### 📘 List<T>
   - Just a plain list of data. 
   - No pagination info. 
   - You’ve probably used it already.  
   
 `List<User> users = userRepository.findAll();`

**❌ You won’t know**:
   - Total pages
   - Current page 
   - Total items

#### 📘 Page<T>
   - This is a fully featured page.
   - A `Page` extends `Slice` and includes full pagination information.
   - Contains:
       - The list of items 
       - Total elements (Total number of elements in the `entire dataset`)
       - Total pages 
       - Page number 
       - Page size 
       - current page
       - no. of  items in a current page
       -  first /last page status
       - Whether it’s the last page
   ``` java
     Page<User> usersPage = userRepository.findAll(PageRequest.of(0, 5,Sort.by("lastname"));
     List<User> users = usersPage.getContent();
   ```
#### 📘 Slice<T>
   - Like Page, but cheaper.
   - A Slice is a  `lighter version` of Page — it’s used when you don’t need the total count, which can be expensive to compute in large datasets (e.g., in databases).
   - Doesn’t count total elements or pages. 
   - Good for "load more" functionality (like infinite scrolling).
   - 🟡 Why? Because Slice is usually used when you `only fetch enough data` to know whether another page exists, `without counting the total`.

`Slice<User> userSlice = userRepository.findAll(PageRequest.of(0, 5));`

---

### Using `@RequestParam` for `Page`, `Size`, and `Sort`
   - This is about how to accept pagination and sorting parameters from a REST API request.

``` java

    @RestController
    @RequestMapping("/users")
    public class UserController {

       @Autowired
       private UserRepository userRepository;

       @GetMapping
       public Page<User> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "5") int size,
            @RequestParam(defaultValue = "id,asc") String[] sort
       ) {
          Sort.Direction direction = Sort.Direction.fromString(sort[1]);
          Pageable pageable = PageRequest.of(page, size, Sort.by(direction, sort[0]));

          return userRepository.findAll(pageable);
       }
    }
```
✅ Sample Requests:
   - `GET /users` → Default page 0, size 5, sort by id asc 
   - `GET /users?page=2&size=3&sort=name,desc` → Page 2, 3 per page, sort by name desc

--- 

## Handling Edge Cases in Pagination
   - When building pagination, it’s important to handle situations where things don’t go as expected. 

### 🧨 Edge Case 1: Page number `out of range`
Example:
 - User requests page=1000 but there are only 5 pages. 
 - ✅ Spring will return:
      - An empty list 
      - totalPages = 5 
      - content = []

#### 🧠 You can check:
  ``` java
     if (usersPage.isEmpty()) {
      // return a message or empty response
    }
  ```

### 🧨 Edge Case 2: `Negative page or size`

Example:
 - `/users?page=-1&size=0`
 - ✅ Spring throws a BadRequest (400).

#### 💡 Solution: Add validation:
  ``` java
      @RequestParam(defaultValue = "0") @Min(0) int page,
      @RequestParam(defaultValue = "5") @Min(1) int size
  ```

### 🧨 Edge Case 3: Sorting by `non-existent field`

Example:
   - `/users?sort=unknownField,asc`
   - ✅ Spring throws an error like:
        - "No property unknownField found for type User"

#### 💡 Solution:
   - Use a whitelist for allowed sort fields 
   - Or handle exceptions using @ControllerAdvice
