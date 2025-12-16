# 🧩 SQL JOIN 

- In SQL, the JOIN clause is used to `combine rows from two or more tables based on a related column between them`. There are several types of joins, each serving a different purpose.

### 1️⃣ INNER JOIN

- ✅ Returns only the rows where there is a match in both tables.
- If we use 'JOIN' in query by default it consider as `INNER JOIN`. `JOIN` is a shorthand for `INNER JOIN`.
  ``` sql
    // syntax
  
      SELECT table1.column1, table2.column2
      FROM table1
      INNER JOIN table2
      ON table1.common_column = table2.common_column;

    // example
  
    SELECT employees.name, departments.department_name
    FROM employees
    INNER JOIN departments
    ON employees.department_id = departments.id;


  ```

### 2️⃣ LEFT JOIN (or LEFT OUTER JOIN)

- ⬅️ Returns all rows from the left table, and the matched rows from the right table.  
- ❌ If there’s no match, the result is NULL for the right table.
   ``` sql
    // syntax
  
      SELECT table1.column1, table2.column2
      FROM table1
      LEFT JOIN table2
      ON table1.common_column = table2.common_column;

    // example
  
    SELECT employees.name, departments.department_name
    FROM employees
    LEFT JOIN departments
    ON employees.department_id = departments.id;


  ```

### 3️⃣ RIGHT JOIN (or RIGHT OUTER JOIN)

- ➡️ Returns all rows from the right table, and the matched rows from the left table.  
- ❌ If there’s no match, the result is NULL for the left table.
   ``` sql
    // syntax
  
      SELECT table1.column1, table2.column2
      FROM table1
      RIGHT JOIN table2
      ON table1.common_column = table2.common_column;

    // example
  
    SELECT employees.name, departments.department_name
    FROM employees
    RIGHT JOIN departments
    ON employees.department_id = departments.id;


  ```

### 4️⃣ FULL OUTER JOIN

- 🔄 Returns rows when there is a match in one of the tables.  
- ⚪ Non-matching rows from both tables appear with NULL for missing values.
   ``` sql
    // syntax
  
      SELECT table1.column1, table2.column2
      FROM table1
      FULL OUTER JOIN table2
      ON table1.common_column = table2.common_column;

    // example
  
    SELECT employees.name, departments.department_name
    FROM employees
    FULL OUTER  JOIN departments
    ON employees.department_id = departments.id;


  ```

### 5️⃣ CROSS JOIN

- ✖️ Returns the Cartesian product of the two tables (every row from the first table paired with every row from the second table).
  ``` sql
    // syntax
  
      SELECT table1.column1, table2.column2
      FROM table1
      CROSS JOIN table2;

  ```

### 6️⃣ SELF JOIN

- 🔁 A table joins with itself.
  ``` sql
   // syntax
  
    SELECT a.column1, b.column2
    FROM table_name a
    JOIN table_name b
    ON a.common_column = b.common_column;


  ```

---

# 📈 Aggregate Functions
- In SQL, aggregate functions are `used to perform calculations on multiple rows of a table` and return `a single summarized value`. 
- They are very useful for reports, analytics, and summarizing data.

## 📊 Common Aggregate Functions

### 🧮 COUNT()

- 🔢 Counts the number of rows (or non-NULL values)  
- ℹ️ Notes: COUNT(*) counts all rows, including those with NULL. COUNT(column) counts only non-NULL values in that column.
  ``` sql
  
    SELECT COUNT(*) FROM employees;
    SELECT COUNT(salary) FROM employees;

  ```

### ➕ SUM()

- 💰 Adds up all values in a numeric column.
  ``` sql
  
    SELECT SUM(salary) FROM employees;
  ```

### ➗ AVG()

- 📈 Calculates the average of a numeric column.
  ``` sql

   SELECT AVG(salary) FROM employees;

  ```

### 🔽 MIN()

- 📉 Finds the minimum value in a column.
  ``` sql

  SELECT MIN(salary) FROM employees;

  ```

### 🔼 MAX()

- 🏔 Finds the maximum value in a column.
  ``` sql
  
    SELECT MAX(salary) FROM employees;

  ```

---

## 🗂 Using Aggregate Functions with GROUP BY

- 📑 Aggregate functions are often used with GROUP BY to summarize data for each group.
  ``` sql
   SELECT department_id, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department_id;

  // This calculates the average salary for each department.

  ```

## 🔍 Using Aggregate Functions with HAVING

- 🛡 HAVING is like WHERE but for aggregated data.
  ``` sql

  SELECT department_id, COUNT(*) AS employee_count
  FROM employees
  GROUP BY department_id
  HAVING COUNT(*) > 5;

  // Shows departments with more than 5 employees.

  ```

---

## 💡 Rule of thumb

- 📌 Use COUNT(*) when you want to count all rows regardless of NULLs.  
- 🔍 Use COUNT(column) if you want to count only rows where that column has a value.
