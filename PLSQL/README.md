# Introduction to PL/SQL

## 📝 What is PL/SQL?

- PL/SQL (Procedural Language / SQL) is Oracle’s extension of SQL that `allows you to write programs inside the database`.

### 🛠️ SQL alone: 
- ⚙️ Works on data
- 🚫 Cannot use loops, conditions, or variables

### ➕ PL/SQL adds:
- 🔑 Variables
- ⚖️ Conditions (IF, CASE)
- 🔁 Loops
- 🛡️ Error handling

## 🚀 Why PL/SQL?

PL/SQL is used when:
- 💼 You need business logic
- 🔄 You want to process data row-by-row
- ⚡ You want faster execution inside Oracle DB

## 📍 Where is PL/SQL used?

- 🗃️ Stored Procedures
- 🧮 Functions
- 🕹️ Triggers
- 📦 Packages

---

## 2️⃣ PL/SQL Block Structure

Every PL/SQL program is written in a block.

``` sql

    DECLARE
       -- Variable declarations
    BEGIN
       -- Executable statements
    EXCEPTION
       -- Error handling
    END;
    /

```

### 📋 1. DECLARE (Optional)

- Used to define:
  - 🔤 Variables
  - 🔒 Constants
  - 🔎 Cursors

  ``` sql
  
        DECLARE
         name VARCHAR2(20);
         pi CONSTANT NUMBER(1,2) := 3.14;
         CURSOR <cursor-name> IS 
             DML query;

  ```

### ▶️ 2. BEGIN (Mandatory)

- Contains executable code.
  ``` sql
  
     BEGIN
     DBMS_OUTPUT.PUT_LINE('Hello PL/SQL');

  ```

### ⚠️ 3. EXCEPTION (Optional)

- Handles runtime errors.
  ``` sql
  
     EXCEPTION
     WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error occurred');

  ```

### ⏹️ 4. END (Mandatory)
- Ends the block.
  ``` sql
  
        END;
        /

  ```

---

## 🔑 Variables and Constants

### 🏷️ Variables

- A variable stores a value temporarily.

### 🔒 Constants

- A constant stores a fixed value (cannot be changed).
- If you try to change a constant → error

---

## 📊 Data Types in PL/SQL

### Common PL/SQL Data Types

| Data Type   | Description              | Example      |
|-------------|--------------------------|--------------|
| 🔢 NUMBER   | Numeric values           | 100, 25.5    |
| 🔤 VARCHAR2 | String                   | 'Hello'      |
| 🅰️ CHAR     | Fixed-length string      | 'A'          |
| 📅 DATE     | Date and time            | SYSDATE      |
| ✅ BOOLEAN  | TRUE / FALSE             | TRUE         |

``` sql

    DECLARE
       v_name VARCHAR2(20);
       v_salary NUMBER;
       v_join_date DATE;
    BEGIN
       v_name := 'John';
       v_salary := 50000;
       v_join_date := SYSDATE;
    
       DBMS_OUTPUT.PUT_LINE(v_name);
       DBMS_OUTPUT.PUT_LINE(v_salary);
       DBMS_OUTPUT.PUT_LINE(v_join_date);
    END;
    /

```

---


# 🧭 What is a Cursor in PL/SQL?

- A cursor is  `a pointer to a result set` `returned by a SQL query`.
- `PL/SQL processes one row at a time`, so `cursors are needed for multi-row result sets`.
- A cursor `does not store data`; it `only references and iterates` over the `result set`.

- When you run a SELECT statement in PL/SQL:
  - 💾 Oracle stores the result in memory
  - 👉 A cursor points to those rows
  - 🔁 You fetch rows one by one

- 👉 Cursors are needed when a query returns more than one row

---

## 🔷 Why Do We Need Cursors?

- 📊 SQL works on sets of rows  
- 🧩 PL/SQL works on one row at a time  

So we need a bridge between SQL and PL/SQL.

### 🚫 Without Cursor

- ✔ Works only if one row is returned  
- ❌ Error if multiple rows  

### ✅ With Cursor

You can:

- 📖 Read multiple rows  
- 🔍 Process each row individually  
- ⚙️ Perform logic on every row  

---

## 🔷 When Do We Use Cursors?

Use a cursor when:

- ✔ A query returns multiple rows  
- ✔ You want to process rows one by one  
- ✔ You want to apply logic per record  
- ✔ You want to update or delete rows individually  

---

## 🗂️ Types of Cursors

| Cursor Type | Description |
|------------|-------------|
| 🟢 Implicit Cursor | Created automatically by Oracle |
| 🔵 Explicit Cursor | Created by programmer |
| 🔁 Cursor FOR Loop | Simplified explicit cursor |
| 🎯 Parameterized Cursor | Cursor with parameters |
| 🧷 REF Cursor | Pointer cursor (advanced) |

---

## 🟢 Implicit Cursor

Oracle automatically creates this cursor for:

- ➕ INSERT  
- ✏️ UPDATE  
- 🗑️ DELETE  
- 📥 SELECT INTO  

## 🧾 Implicit Cursor Attributes

| Attribute | Meaning |
|---------|---------|
| SQL%FOUND | TRUE if at least one row affected |
| SQL%NOTFOUND | TRUE if no row affected |
| SQL%ROWCOUNT | Number of rows affected |
| SQL%ISOPEN | Always FALSE |

### 📌 Example

``` sql

    BEGIN
     UPDATE employees
     SET salary = salary + 1000
     WHERE department_id = 10;
  
     DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ' rows updated');
  END;
  /

```
---

## ❓ Why do we need explicit cursors when Oracle already creates implicit cursors?

- 👉 🚫 Implicit cursors `cannot handle multiple-row` SELECT queries.  
- 👉 ✅ Explicit cursors are needed when `a SELECT query returns more than one row` and we want to `process each row one by one`.

---

## 🔵 Explicit Cursor (Most Important)

Used when:

- 📄 SELECT returns multiple rows  
- 🎛️ You want full control  

### 🔹 Steps of Explicit Cursor

- 1️⃣ Declare cursor  
- 2️⃣ Open cursor  
- 3️⃣ Fetch data  
- 4️⃣ Close cursor

  ``` sql

  // syntax
    DECLARE
       CURSOR cursor_name IS
          SELECT column1, column2 FROM table;
    BEGIN
       OPEN cursor_name;
       FETCH cursor_name INTO variable1, variable2;
       CLOSE cursor_name;
    END;
    /

  // example

  DECLARE
       CURSOR emp_cur IS
          SELECT emp_name, salary FROM employees;
    
       v_name employees.emp_name%TYPE;
       v_salary employees.salary%TYPE;
    BEGIN
       OPEN emp_cur;
    
       LOOP
          FETCH emp_cur INTO v_name, v_salary;
          EXIT WHEN emp_cur%NOTFOUND;
    
          DBMS_OUTPUT.PUT_LINE(v_name || ' earns ' || v_salary);
       END LOOP;
    
       CLOSE emp_cur;
    END;
    /

    %NOTFOUND is used when are using infinit LOOP  and  also we need to  open cursor and close cursor
    %FOUND is used when are using WHILE LOOP  and  also we need to  open cursor and close cursor

    IN FOR LOOP we donot need any condition based we can directly use cursor in condition , no need for open cursor and close cursor (Recommended)


  ``` 

---

## 🧾 Explicit Cursor Attributes

| Attribute | Meaning |
|---------|---------|
| %FOUND | Row fetched successfully |
| %NOTFOUND | No row fetched |
| %ROWCOUNT | Rows fetched so far |
| %ISOPEN | Cursor open or not |

---

## 🔁 Cursor FOR Loop (Recommended)

Oracle automatically:

- ▶️ Opens cursor  
- 🔄 Fetches rows  
- ⏹️ Closes cursor

  ``` sql
  \\ syntax

    FOR record_name IN cursor_name LOOP
       statements;
    END LOOP;


  \\ exmaple
  
    DECLARE
       CURSOR emp_cur IS
          SELECT emp_name, salary FROM employees;
    BEGIN
       FOR emp_rec IN emp_cur LOOP
          DBMS_OUTPUT.PUT_LINE(emp_rec.emp_name || ' - ' || emp_rec.salary);
       END LOOP;
    END;
    /

  ```
- ✔ Clean
- ✔ Safe
- ✔ Less code
- ✔ Most commonly used
---

## 🎯 Parameterized Cursor

- Used when:
  - 🔧 Cursor query needs dynamic values

``` sql
  \\ syntax

      CURSOR cursor_name (parameter datatype) IS
            SELECT columns FROM table WHERE column = parameter;


  \\ example

    DECLARE
       CURSOR emp_cur (dept_id NUMBER) IS
          SELECT emp_name FROM employees WHERE department_id = dept_id;
    BEGIN
       FOR emp_rec IN emp_cur(10) LOOP
          DBMS_OUTPUT.PUT_LINE(emp_rec.emp_name);
       END LOOP;
    END;
    /

```


---

# 🧾 What is a Stored Procedure?

- 🧩 A Stored Procedure is a named PL/SQL block that is stored in the Oracle database and can be executed repeatedly.
- 🗂️ Think of it as a function or program stored in the database.
- 🔁 Once created, you can call it anytime without rewriting the code.
- 📥📤 Can accept parameters and return results.

---

## ✅ Advantages of Stored Procedures

| Advantage | Explanation |
|---------|-------------|
| ♻️ Reusability | Write once, call anywhere |
| 🧩 Modularity | Break programs into smaller, manageable pieces |
| ⚡ Performance | Stored in database, execution is faster |
| 🔐 Security | Control access to data through procedures |
| 🛠️ Maintenance | Update logic in one place instead of multiple applications |

---

## ❓ Why Do We Need Stored Procedures?

- 🔁 When you have repeated business logic  
- 🧠 When you want centralized code for multiple applications  
- 🧮 When complex SQL and PL/SQL logic is needed  
- 🚀 When you want better performance by avoiding sending multiple queries from client  
- 🔄 When you need transaction control (commit/rollback) in one block  

---

##  3️⃣ When to Use Stored Procedures

Use stored procedures when:

- 📦 You want to encapsulate business logic  
- 📊 You need to process data in batches  
- 🙈 You want to hide complex queries from users  
- 🔒 You want to improve security (users execute procedure, not direct SQL)  
- 🌐 You want to reduce network traffic (processing happens on server)  

---

##  4️⃣ Structure of a Stored Procedure

``` sql

        CREATE [OR REPLACE] PROCEDURE procedure_name
           [ (parameter1 IN datatype, parameter2 OUT datatype) ]
        IS  -- or AS
           -- Declarations (variables, cursors, constants)
        BEGIN
           -- Executable statements
        EXCEPTION
           -- Error handling (optional)
        END procedure_name;
        /

```

### 📂 Sections

- 🧾 Procedure Header – Name + Parameters  
- 📋 Declaration Section (Optional) – Variables, cursors  
- ▶️ Executable Section (Mandatory) – SQL/PLSQL logic  
- ⚠️ Exception Section (Optional) – Error handling  

---

## 🔢 5️⃣ Parameters in Stored Procedures

| Mode | Description |
|------|------------|
| 📥 IN | Pass value to procedure (read-only) |
| 📤 OUT | Return value from procedure |
| 🔄 IN OUT | Pass value in and get updated value back |

---

## ▶️ Ways to Execute a Stored Procedure

A stored procedure can be executed in two main ways:

### 1️⃣ Using EXEC (or EXECUTE) command

- 🧪 Works in SQL*Plus, SQL Developer, or tools that support anonymous execution.
- ⚡ Convenient for quick testing.
  ``` sql

  // syntax

    EXEC procedure_name;
      -- or
    EXEC procedure_name(param1, param2);

  ```

### 2️⃣ Using an anonymous PL/SQL block

- 🧩 Useful when calling the procedure as part of a larger PL/SQL block.
- ⚠️ Required if you want to combine multiple procedure calls or handle exceptions.
  ``` sql
    // syntax
  
        BEGIN
          procedure_name;
            -- or with parameters
          procedure_name(param1, param2);
        END;
        /

  ```
---

## 🌟 Benefits of Using Stored Procedures

- ⚡ Performance – SQL runs inside DB, reduces network traffic  
- ♻️ Reusability – Write once, call many times  
- 🔐 Security – Control access, hide table structure  
- 🛠️ Maintainability – Easy to update and manage  
- 🎁 Encapsulation – Hides complexity of SQL queries  


---


# 1️⃣ 🔧 What is a Stored Function?

- 🧾 A Stored Function is a named PL/SQL block stored in the database that:
- 📥 Accepts parameters (optional)
- ⚙️ Performs a specific task
- 🔙 Returns a single value using the RETURN statement

### ✅ Key difference from a procedure:

- 🧩 Procedure: Does not return a value (can use OUT parameters)
- 🧮 Function: Must return a value

📌 Stored functions are often used in SQL statements, PL/SQL blocks, or other functions/procedures.

---

## 2️⃣ ❓ Why Do We Need Stored Functions?

- ♻️ To reuse logic that returns a single value
- 🧮 To simplify complex SQL or calculations
- 🛠️ To improve maintainability (centralized logic)
- 📊 To use functions inside SQL queries
- 🔁 To ensure consistency (same calculation everywhere)

---

## 3️⃣ ⏰ When to Use Stored Functions

- 📌 Use functions when:
   - 🔢 You need a value computed based on input parameters
   - ♻️ You want to reuse logic in multiple places
   - 🔄 You want calculations or data transformations
   - 📊 You want to embed logic inside SQL queries
   - ✅ You need deterministic results for the same input

---

##  4️⃣ 🧱 Structure of a Stored Function

``` sql

        CREATE [OR REPLACE] FUNCTION function_name
                          (parameter1 datatype, parameter2 datatype)
        RETURN return_datatype -- specify the what type of value to be return
        IS  -- or AS
           -- Variable declarations (optional)
        BEGIN
           -- Logic
           RETURN value;  -- Mandatory
        EXCEPTION
           -- Error handling (optional)
        END function_name;
        /

```

### 📂 Sections:

- 🧾 Function Header: Name, parameters, return type
- 📋 Declaration Section (Optional): Variables, cursors
- ▶️ Executable Section (Mandatory): Logic and RETURN statement
- ⚠️ Exception Section (Optional): Error handling

---

## 🔷 5️⃣ 🔢 Parameters in Functions

- 📥 Functions can have IN parameters only
- 🚫 They cannot have OUT parameters (use RETURN instead)

---

## ▶️ Stored functions can be executed in two main ways

### 1️⃣ 📊 Using SELECT or direct SQL (preferred)

- 🔙 Returns the value of the function.
- 📈 Can be used in SELECT, WHERE, ORDER BY, etc. 

  ``` sql
     // example
  
        CREATE OR REPLACE FUNCTION square_number(p_num IN NUMBER)
        RETURN NUMBER
        IS
        BEGIN
           RETURN p_num * p_num;
        END;
        /

        SELECT square_number(2) FROM DUAL;

  
  ```

### 2️⃣ 🧩 Using an anonymous PL/SQL block

- 📥 Must capture the return value in a variable.
- 🚫 Cannot just call the function alone like a procedure in a PL/SQL block without using the return value.
  ``` sql

    // example
  
        CREATE OR REPLACE FUNCTION square_number(p_num IN NUMBER)
        RETURN NUMBER
        IS
        BEGIN
           RETURN p_num * p_num;
        END;
        /

         DECLARE
         v_result NUMBER;
        BEGIN
           v_result := square_number(5);
           DBMS_OUTPUT.PUT_LINE('Square is: ' || v_result);
        END;
        /

  ```

---

## 🧠 What is DUAL?

- 📄 DUAL is a special one-row, one-column table provided by Oracle.
- 🔢 It has exactly one row and one column named DUMMY.
- 🧪 Used when you need to select a value, expression, or function without referencing a real table.


## ❓ Why do we use DUAL?

- 📊 In SQL, a SELECT statement requires a FROM clause.
- 🧮 If you just want to evaluate an expression or call a function without querying a real table, you use DUAL.

- 🔁 Oracle executes it once, because DUAL has only one row.
- 🚫 Without DUAL, you cannot just write SELECT 5*10; — Oracle needs a FROM clause.


## 📝 Notes

- 🌐 In other databases, DUAL may not exist (e.g., SQL Server, MySQL) — they allow SELECT 5*10; without a table.
- 🏛️ In Oracle, DUAL is standard for evaluating expressions, constants, or functions in SQL.

---

## 🔷 7️⃣ 📊 Using Functions Inside SQL

- ⭐ One of the main advantages of functions is that they can be used in SQL queries.

``` sql

   SELECT emp_id, salary, square_number(salary) AS salary_square
   FROM employees;

```

---

## 🌟 Benefits of Stored Functions

- ♻️ Reusability – Use function in multiple queries or programs
- 🔁 Consistency – Same calculation everywhere
- 🎁 Encapsulation – Hide complex logic
- 🛠️ Maintainability – Update logic in one place
- ⚡ Performance – Execution happens in DB, reduces network load

---

## 🔑 Key Points

- 🔙 Function must have RETURN statement
- 📥 Parameters are IN only
- 📊 Can be used inside SQL queries
- 🧮 Good for calculations, data transformations, validations
- ⚠️ Can include exception handling

---

## 🔄 Main Differences Between Stored Procedure and Stored Function

| Feature | Stored Procedure | Stored Function |
|-------|-----------------|----------------|
| 🔙 Returns Value | No direct return; can return values via OUT parameters | Returns a single value using RETURN statement |
| 📊 Can be used in SQL query | ❌ No, cannot be called directly in SQL | ✅ Yes, can be called directly in SQL |
| 🎯 Purpose | Perform an action or a series of actions (DML, business logic) | Perform computation and return a value |
| 🧾 Syntax | PROCEDURE proc_name | FUNCTION func_name RETURN datatype |
| 🔢 Can have IN/OUT/IN OUT parameters | ✅ Yes | ✅ Only IN parameters (OUT not allowed) |
| 🧩 Called from PL/SQL block | ✅ Yes | ✅ Yes |

