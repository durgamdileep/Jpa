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
