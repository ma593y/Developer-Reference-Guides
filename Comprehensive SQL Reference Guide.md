# Comprehensive SQL Reference Guide

## Table of Contents
1. [Database Basics](#database-basics)
2. [Data Types](#data-types)
3. [DDL - Data Definition Language](#ddl-data-definition-language)
4. [DML - Data Manipulation Language](#dml-data-manipulation-language)
5. [DQL - Data Query Language](#dql-data-query-language)
6. [DCL - Data Control Language](#dcl-data-control-language)
7. [TCL - Transaction Control Language](#tcl-transaction-control-language)
8. [Constraints](#constraints)
9. [Joins](#joins)
10. [Subqueries](#subqueries)
11. [Aggregate Functions](#aggregate-functions)
12. [String Functions](#string-functions)
13. [Date and Time Functions](#date-and-time-functions)
14. [Numeric Functions](#numeric-functions)
15. [Window Functions](#window-functions)
16. [Indexes](#indexes)
17. [Views](#views)
18. [Stored Procedures](#stored-procedures)
19. [Triggers](#triggers)
20. [Advanced Concepts](#advanced-concepts)

---

## 1. Database Basics

### Database Creation and Management
```sql
-- Create database
CREATE DATABASE database_name;

-- Use database
USE database_name;

-- Drop database
DROP DATABASE database_name;

-- Show databases
SHOW DATABASES;

-- Show current database
SELECT DATABASE();
```

---

## 2. Data Types

### Numeric Types
- **INT / INTEGER** - Whole numbers (-2,147,483,648 to 2,147,483,647)
- **BIGINT** - Large whole numbers
- **SMALLINT** - Small whole numbers (-32,768 to 32,767)
- **TINYINT** - Very small numbers (0 to 255)
- **DECIMAL(p,s) / NUMERIC(p,s)** - Fixed precision (p=total digits, s=decimal places)
- **FLOAT** - Approximate floating point
- **DOUBLE** - Double precision floating point

### String Types
- **CHAR(n)** - Fixed length string (max 255)
- **VARCHAR(n)** - Variable length string (max 65,535)
- **TEXT** - Large text data (max 65,535)
- **MEDIUMTEXT** - Medium text (max 16,777,215)
- **LONGTEXT** - Very large text (max 4,294,967,295)
- **ENUM** - Enumeration of allowed values
- **SET** - Set of allowed values

### Date and Time Types
- **DATE** - Date (YYYY-MM-DD)
- **TIME** - Time (HH:MM:SS)
- **DATETIME** - Date and time (YYYY-MM-DD HH:MM:SS)
- **TIMESTAMP** - Timestamp (auto-updates)
- **YEAR** - Year in 2 or 4 digit format

### Binary Types
- **BLOB** - Binary large object
- **BINARY(n)** - Fixed length binary
- **VARBINARY(n)** - Variable length binary

### Other Types
- **BOOLEAN / BOOL** - True/False (stored as TINYINT)
- **JSON** - JSON formatted data

---

## 3. DDL - Data Definition Language

### CREATE TABLE
```sql
-- Basic table creation
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
);

-- Example
CREATE TABLE employees (
    emp_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    hire_date DATE DEFAULT CURRENT_DATE,
    salary DECIMAL(10,2) CHECK (salary > 0),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

-- Create table from another table
CREATE TABLE new_table AS
SELECT column1, column2 FROM existing_table WHERE condition;
```

### ALTER TABLE
```sql
-- Add column
ALTER TABLE table_name ADD column_name datatype;

-- Drop column
ALTER TABLE table_name DROP COLUMN column_name;

-- Modify column
ALTER TABLE table_name MODIFY COLUMN column_name new_datatype;

-- Rename column
ALTER TABLE table_name RENAME COLUMN old_name TO new_name;

-- Add constraint
ALTER TABLE table_name ADD CONSTRAINT constraint_name constraint_type;

-- Drop constraint
ALTER TABLE table_name DROP CONSTRAINT constraint_name;

-- Rename table
ALTER TABLE old_name RENAME TO new_name;
```

### DROP TABLE
```sql
-- Drop table
DROP TABLE table_name;

-- Drop if exists
DROP TABLE IF EXISTS table_name;

-- Drop multiple tables
DROP TABLE table1, table2, table3;
```

### TRUNCATE TABLE
```sql
-- Remove all records (faster than DELETE, resets auto-increment)
TRUNCATE TABLE table_name;
```

---

## 4. DML - Data Manipulation Language

### INSERT
```sql
-- Insert single row
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);

-- Insert multiple rows
INSERT INTO table_name (column1, column2)
VALUES 
    (value1, value2),
    (value3, value4),
    (value5, value6);

-- Insert from another table
INSERT INTO table_name (column1, column2)
SELECT column1, column2 FROM another_table WHERE condition;

-- Insert with default values
INSERT INTO table_name (column1) VALUES (value1);
```

### UPDATE
```sql
-- Basic update
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition;

-- Update with subquery
UPDATE table_name
SET column1 = (SELECT column2 FROM another_table WHERE condition)
WHERE condition;

-- Update multiple tables
UPDATE table1 t1
JOIN table2 t2 ON t1.id = t2.id
SET t1.column1 = value1, t2.column2 = value2
WHERE condition;
```

### DELETE
```sql
-- Delete specific rows
DELETE FROM table_name WHERE condition;

-- Delete all rows
DELETE FROM table_name;

-- Delete with subquery
DELETE FROM table_name
WHERE column1 IN (SELECT column1 FROM another_table WHERE condition);

-- Delete with JOIN
DELETE t1 FROM table1 t1
JOIN table2 t2 ON t1.id = t2.id
WHERE condition;
```

---

## 5. DQL - Data Query Language

### SELECT Basics
```sql
-- Select all columns
SELECT * FROM table_name;

-- Select specific columns
SELECT column1, column2 FROM table_name;

-- Select with alias
SELECT column1 AS alias1, column2 AS alias2 FROM table_name;

-- Select distinct values
SELECT DISTINCT column1 FROM table_name;

-- Select with calculations
SELECT column1, column2 * 1.1 AS new_value FROM table_name;
```

### WHERE Clause
```sql
-- Basic conditions
SELECT * FROM table_name WHERE column1 = value;

-- Multiple conditions
SELECT * FROM table_name 
WHERE column1 = value1 AND column2 > value2;

-- OR condition
SELECT * FROM table_name 
WHERE column1 = value1 OR column2 = value2;

-- IN operator
SELECT * FROM table_name WHERE column1 IN (value1, value2, value3);

-- NOT IN
SELECT * FROM table_name WHERE column1 NOT IN (value1, value2);

-- BETWEEN
SELECT * FROM table_name WHERE column1 BETWEEN value1 AND value2;

-- LIKE (pattern matching)
SELECT * FROM table_name WHERE column1 LIKE 'pattern%';
-- % = any characters, _ = single character

-- IS NULL / IS NOT NULL
SELECT * FROM table_name WHERE column1 IS NULL;
SELECT * FROM table_name WHERE column1 IS NOT NULL;

-- NOT operator
SELECT * FROM table_name WHERE NOT column1 = value;
```

### ORDER BY
```sql
-- Sort ascending (default)
SELECT * FROM table_name ORDER BY column1;

-- Sort descending
SELECT * FROM table_name ORDER BY column1 DESC;

-- Sort by multiple columns
SELECT * FROM table_name ORDER BY column1 ASC, column2 DESC;

-- Sort by position
SELECT column1, column2 FROM table_name ORDER BY 1, 2;
```

### LIMIT and OFFSET
```sql
-- Limit results
SELECT * FROM table_name LIMIT 10;

-- Limit with offset
SELECT * FROM table_name LIMIT 10 OFFSET 20;

-- Alternative syntax
SELECT * FROM table_name LIMIT 20, 10;  -- offset, count
```

### GROUP BY
```sql
-- Basic grouping
SELECT column1, COUNT(*) 
FROM table_name 
GROUP BY column1;

-- Group by multiple columns
SELECT column1, column2, SUM(column3)
FROM table_name
GROUP BY column1, column2;

-- Group with WHERE
SELECT column1, COUNT(*)
FROM table_name
WHERE column2 > value
GROUP BY column1;
```

### HAVING
```sql
-- Filter grouped results
SELECT column1, COUNT(*) as count
FROM table_name
GROUP BY column1
HAVING count > 5;

-- Having with aggregate functions
SELECT column1, AVG(column2) as avg_val
FROM table_name
GROUP BY column1
HAVING AVG(column2) > 100;
```

### CASE Statement
```sql
-- Simple CASE
SELECT column1,
    CASE column2
        WHEN value1 THEN 'Result1'
        WHEN value2 THEN 'Result2'
        ELSE 'Other'
    END AS category
FROM table_name;

-- Searched CASE
SELECT column1,
    CASE
        WHEN column2 > 100 THEN 'High'
        WHEN column2 > 50 THEN 'Medium'
        ELSE 'Low'
    END AS level
FROM table_name;
```

---

## 6. DCL - Data Control Language

### GRANT
```sql
-- Grant all privileges
GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'host';

-- Grant specific privileges
GRANT SELECT, INSERT, UPDATE ON database_name.table_name 
TO 'username'@'host';

-- Grant with grant option
GRANT SELECT ON database_name.* TO 'username'@'host' 
WITH GRANT OPTION;
```

### REVOKE
```sql
-- Revoke privileges
REVOKE SELECT, INSERT ON database_name.table_name 
FROM 'username'@'host';

-- Revoke all privileges
REVOKE ALL PRIVILEGES ON database_name.* FROM 'username'@'host';
```

---

## 7. TCL - Transaction Control Language

### Transactions
```sql
-- Start transaction
START TRANSACTION;
-- or
BEGIN;

-- Commit transaction
COMMIT;

-- Rollback transaction
ROLLBACK;

-- Savepoint
SAVEPOINT savepoint_name;

-- Rollback to savepoint
ROLLBACK TO savepoint_name;

-- Release savepoint
RELEASE SAVEPOINT savepoint_name;
```

### Transaction Example
```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- If everything is okay
COMMIT;

-- If there's an error
ROLLBACK;
```

---

## 8. Constraints

### PRIMARY KEY
```sql
-- Column level
CREATE TABLE table_name (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- Table level
CREATE TABLE table_name (
    id INT,
    name VARCHAR(50),
    PRIMARY KEY (id)
);

-- Composite primary key
CREATE TABLE table_name (
    id1 INT,
    id2 INT,
    name VARCHAR(50),
    PRIMARY KEY (id1, id2)
);
```

### FOREIGN KEY
```sql
-- Column level
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- With ON DELETE and ON UPDATE
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Options: CASCADE, SET NULL, SET DEFAULT, NO ACTION, RESTRICT
```

### UNIQUE
```sql
-- Column level
CREATE TABLE table_name (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE
);

-- Table level
CREATE TABLE table_name (
    id INT PRIMARY KEY,
    email VARCHAR(100),
    UNIQUE (email)
);

-- Composite unique
CREATE TABLE table_name (
    id INT PRIMARY KEY,
    col1 VARCHAR(50),
    col2 VARCHAR(50),
    UNIQUE (col1, col2)
);
```

### NOT NULL
```sql
CREATE TABLE table_name (
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL
);
```

### CHECK
```sql
-- Column level
CREATE TABLE employees (
    id INT PRIMARY KEY,
    age INT CHECK (age >= 18),
    salary DECIMAL(10,2) CHECK (salary > 0)
);

-- Table level
CREATE TABLE employees (
    id INT PRIMARY KEY,
    age INT,
    salary DECIMAL(10,2),
    CHECK (age >= 18 AND salary > 0)
);
```

### DEFAULT
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    hire_date DATE DEFAULT CURRENT_DATE,
    status VARCHAR(20) DEFAULT 'active',
    salary DECIMAL(10,2) DEFAULT 0.00
);
```

### AUTO_INCREMENT
```sql
CREATE TABLE table_name (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50)
);
```

---

## 9. Joins

### INNER JOIN
```sql
-- Returns matching rows from both tables
SELECT t1.column1, t2.column2
FROM table1 t1
INNER JOIN table2 t2 ON t1.common_column = t2.common_column;

-- Multiple joins
SELECT t1.col1, t2.col2, t3.col3
FROM table1 t1
INNER JOIN table2 t2 ON t1.id = t2.id
INNER JOIN table3 t3 ON t2.id = t3.id;
```

### LEFT JOIN (LEFT OUTER JOIN)
```sql
-- Returns all rows from left table, matching rows from right
SELECT t1.column1, t2.column2
FROM table1 t1
LEFT JOIN table2 t2 ON t1.common_column = t2.common_column;

-- Find non-matching rows
SELECT t1.*
FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.id
WHERE t2.id IS NULL;
```

### RIGHT JOIN (RIGHT OUTER JOIN)
```sql
-- Returns all rows from right table, matching rows from left
SELECT t1.column1, t2.column2
FROM table1 t1
RIGHT JOIN table2 t2 ON t1.common_column = t2.common_column;
```

### FULL OUTER JOIN
```sql
-- Returns all rows from both tables
-- MySQL doesn't support FULL OUTER JOIN, use UNION
SELECT t1.column1, t2.column2
FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.id
UNION
SELECT t1.column1, t2.column2
FROM table1 t1
RIGHT JOIN table2 t2 ON t1.id = t2.id;
```

### CROSS JOIN
```sql
-- Cartesian product of both tables
SELECT t1.column1, t2.column2
FROM table1 t1
CROSS JOIN table2 t2;

-- Alternative syntax
SELECT t1.column1, t2.column2
FROM table1 t1, table2 t2;
```

### SELF JOIN
```sql
-- Join table to itself
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.emp_id;
```

### NATURAL JOIN
```sql
-- Automatically joins on columns with same name
SELECT * FROM table1 NATURAL JOIN table2;
```

---

## 10. Subqueries

### Scalar Subquery
```sql
-- Returns single value
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Row Subquery
```sql
-- Returns single row
SELECT * FROM employees
WHERE (dept_id, salary) = (SELECT dept_id, MAX(salary) 
                           FROM employees 
                           GROUP BY dept_id);
```

### Column Subquery
```sql
-- Returns single column
SELECT name FROM employees
WHERE dept_id IN (SELECT dept_id FROM departments WHERE location = 'NY');

-- EXISTS
SELECT name FROM employees e
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.emp_id = e.emp_id);

-- NOT EXISTS
SELECT name FROM employees e
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.emp_id = e.emp_id);
```

### Table Subquery
```sql
-- Returns multiple rows and columns
SELECT dept_name, avg_salary
FROM (
    SELECT dept_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
) AS dept_stats
JOIN departments d ON dept_stats.dept_id = d.dept_id;
```

### Correlated Subquery
```sql
-- Subquery references outer query
SELECT e1.name, e1.salary
FROM employees e1
WHERE salary > (SELECT AVG(salary) 
                FROM employees e2 
                WHERE e2.dept_id = e1.dept_id);
```

### ANY / ALL
```sql
-- ANY: true if comparison is true for any value
SELECT name FROM employees
WHERE salary > ANY (SELECT salary FROM employees WHERE dept_id = 10);

-- ALL: true if comparison is true for all values
SELECT name FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE dept_id = 10);
```

---

## 11. Aggregate Functions

### COUNT
```sql
-- Count all rows
SELECT COUNT(*) FROM table_name;

-- Count non-null values
SELECT COUNT(column_name) FROM table_name;

-- Count distinct values
SELECT COUNT(DISTINCT column_name) FROM table_name;
```

### SUM
```sql
-- Sum of values
SELECT SUM(column_name) FROM table_name;

-- Sum with condition
SELECT SUM(CASE WHEN condition THEN column_name ELSE 0 END) 
FROM table_name;
```

### AVG
```sql
-- Average of values
SELECT AVG(column_name) FROM table_name;

-- Average with grouping
SELECT dept_id, AVG(salary) FROM employees GROUP BY dept_id;
```

### MIN / MAX
```sql
-- Minimum value
SELECT MIN(column_name) FROM table_name;

-- Maximum value
SELECT MAX(column_name) FROM table_name;

-- Min and Max together
SELECT MIN(salary), MAX(salary) FROM employees;
```

### GROUP_CONCAT (MySQL)
```sql
-- Concatenate values from group
SELECT dept_id, GROUP_CONCAT(name) 
FROM employees 
GROUP BY dept_id;

-- With separator
SELECT dept_id, GROUP_CONCAT(name SEPARATOR ', ') 
FROM employees 
GROUP BY dept_id;
```

---

## 12. String Functions

### CONCAT / CONCAT_WS
```sql
-- Concatenate strings
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

-- Concatenate with separator
SELECT CONCAT_WS(', ', last_name, first_name) AS name FROM employees;
```

### SUBSTRING / SUBSTR
```sql
-- Extract substring
SELECT SUBSTRING(column_name, start_position, length) FROM table_name;

-- Examples
SELECT SUBSTRING('Hello World', 1, 5);  -- 'Hello'
SELECT SUBSTRING('Hello World', 7);     -- 'World'
```

### LENGTH / CHAR_LENGTH
```sql
-- Length in bytes
SELECT LENGTH(column_name) FROM table_name;

-- Length in characters
SELECT CHAR_LENGTH(column_name) FROM table_name;
```

### UPPER / LOWER
```sql
-- Convert to uppercase
SELECT UPPER(column_name) FROM table_name;

-- Convert to lowercase
SELECT LOWER(column_name) FROM table_name;
```

### TRIM / LTRIM / RTRIM
```sql
-- Remove leading and trailing spaces
SELECT TRIM(column_name) FROM table_name;

-- Remove leading spaces
SELECT LTRIM(column_name) FROM table_name;

-- Remove trailing spaces
SELECT RTRIM(column_name) FROM table_name;

-- Remove specific characters
SELECT TRIM(BOTH 'x' FROM 'xxxHelloxxx');  -- 'Hello'
```

### REPLACE
```sql
-- Replace substring
SELECT REPLACE(column_name, 'old', 'new') FROM table_name;
```

### POSITION / LOCATE / INSTR
```sql
-- Find position of substring
SELECT POSITION('substring' IN column_name) FROM table_name;
SELECT LOCATE('substring', column_name) FROM table_name;
SELECT INSTR(column_name, 'substring') FROM table_name;
```

### LEFT / RIGHT
```sql
-- Get leftmost characters
SELECT LEFT(column_name, 5) FROM table_name;

-- Get rightmost characters
SELECT RIGHT(column_name, 5) FROM table_name;
```

### LPAD / RPAD
```sql
-- Pad left with characters
SELECT LPAD(column_name, 10, '0') FROM table_name;

-- Pad right with characters
SELECT RPAD(column_name, 10, '0') FROM table_name;
```

### REVERSE
```sql
-- Reverse string
SELECT REVERSE(column_name) FROM table_name;
```

---

## 13. Date and Time Functions

### Current Date/Time
```sql
-- Current date
SELECT CURRENT_DATE();
SELECT CURDATE();

-- Current time
SELECT CURRENT_TIME();
SELECT CURTIME();

-- Current datetime
SELECT NOW();
SELECT CURRENT_TIMESTAMP();
```

### Date Extraction
```sql
-- Extract year
SELECT YEAR(date_column) FROM table_name;

-- Extract month
SELECT MONTH(date_column) FROM table_name;

-- Extract day
SELECT DAY(date_column) FROM table_name;

-- Extract hour, minute, second
SELECT HOUR(datetime_column), MINUTE(datetime_column), SECOND(datetime_column)
FROM table_name;

-- Day of week (1=Sunday, 7=Saturday)
SELECT DAYOFWEEK(date_column) FROM table_name;

-- Day name
SELECT DAYNAME(date_column) FROM table_name;

-- Month name
SELECT MONTHNAME(date_column) FROM table_name;
```

### Date Arithmetic
```sql
-- Add interval
SELECT DATE_ADD(date_column, INTERVAL 1 DAY) FROM table_name;
SELECT DATE_ADD(date_column, INTERVAL 1 MONTH) FROM table_name;
SELECT DATE_ADD(date_column, INTERVAL 1 YEAR) FROM table_name;

-- Subtract interval
SELECT DATE_SUB(date_column, INTERVAL 1 DAY) FROM table_name;

-- Date difference
SELECT DATEDIFF(date1, date2) FROM table_name;

-- Time difference
SELECT TIMEDIFF(time1, time2) FROM table_name;
```

### Date Formatting
```sql
-- Format date
SELECT DATE_FORMAT(date_column, '%Y-%m-%d') FROM table_name;
SELECT DATE_FORMAT(date_column, '%W, %M %d, %Y') FROM table_name;

-- Common format specifiers:
-- %Y = 4-digit year, %y = 2-digit year
-- %M = month name, %m = month number
-- %D = day with suffix, %d = day number
-- %W = weekday name, %w = weekday number
-- %H = hour (00-23), %h = hour (01-12)
-- %i = minutes, %s = seconds
```

### String to Date
```sql
-- Convert string to date
SELECT STR_TO_DATE('2024-01-15', '%Y-%m-%d');
```

### LAST_DAY
```sql
-- Last day of month
SELECT LAST_DAY(date_column) FROM table_name;
```

---

## 14. Numeric Functions

### ABS
```sql
-- Absolute value
SELECT ABS(column_name) FROM table_name;
```

### ROUND / FLOOR / CEILING
```sql
-- Round to nearest integer
SELECT ROUND(column_name) FROM table_name;

-- Round to decimal places
SELECT ROUND(column_name, 2) FROM table_name;

-- Round down
SELECT FLOOR(column_name) FROM table_name;

-- Round up
SELECT CEILING(column_name) FROM table_name;
```

### POWER / SQRT
```sql
-- Power
SELECT POWER(column_name, 2) FROM table_name;

-- Square root
SELECT SQRT(column_name) FROM table_name;
```

### MOD
```sql
-- Modulo (remainder)
SELECT MOD(column_name, 10) FROM table_name;
SELECT column_name % 10 FROM table_name;
```

### RAND
```sql
-- Random number between 0 and 1
SELECT RAND();

-- Random row
SELECT * FROM table_name ORDER BY RAND() LIMIT 1;
```

### TRUNCATE
```sql
-- Truncate to decimal places
SELECT TRUNCATE(column_name, 2) FROM table_name;
```

### SIGN
```sql
-- Sign of number (-1, 0, or 1)
SELECT SIGN(column_name) FROM table_name;
```

---

## 15. Window Functions

### ROW_NUMBER
```sql
-- Assign unique sequential number
SELECT 
    column1,
    column2,
    ROW_NUMBER() OVER (ORDER BY column1) AS row_num
FROM table_name;

-- With partition
SELECT 
    dept_id,
    name,
    salary,
    ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rank_in_dept
FROM employees;
```

### RANK / DENSE_RANK
```sql
-- RANK: leaves gaps after ties
SELECT 
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- DENSE_RANK: no gaps after ties
SELECT 
    name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
```

### NTILE
```sql
-- Divide rows into n groups
SELECT 
    name,
    salary,
    NTILE(4) OVER (ORDER BY salary) AS quartile
FROM employees;
```

### LAG / LEAD
```sql
-- LAG: access previous row
SELECT 
    date,
    sales,
    LAG(sales, 1) OVER (ORDER BY date) AS previous_sales
FROM sales_data;

-- LEAD: access next row
SELECT 
    date,
    sales,
    LEAD(sales, 1) OVER (ORDER BY date) AS next_sales
FROM sales_data;
```

### FIRST_VALUE / LAST_VALUE
```sql
-- First value in window
SELECT 
    name,
    salary,
    FIRST_VALUE(salary) OVER (ORDER BY salary DESC) AS highest_salary
FROM employees;

-- Last value in window
SELECT 
    name,
    salary,
    LAST_VALUE(salary) OVER (ORDER BY salary DESC) AS lowest_salary
FROM employees;
```

### Aggregate Window Functions
```sql
-- Running total
SELECT 
    date,
    sales,
    SUM(sales) OVER (ORDER BY date) AS running_total
FROM sales_data;

-- Moving average
SELECT 
    date,
    sales,
    AVG(sales) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM sales_data;

-- Cumulative count
SELECT 
    name,
    hire_date,
    COUNT(*) OVER (ORDER BY hire_date) AS cumulative_hires
FROM employees;
```

### Window Frame Specification
```sql
-- ROWS vs RANGE
-- ROWS: physical rows
-- RANGE: logical range (handles ties)

-- Frame examples:
-- ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
-- ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
-- RANGE BETWEEN INTERVAL '1' DAY PRECEDING AND CURRENT ROW
```

---

## 16. Indexes

### Creating Indexes
```sql
-- Create index
CREATE INDEX index_name ON table_name (column_name);

-- Create unique index
CREATE UNIQUE INDEX index_name ON table_name (column_name);

-- Create composite index
CREATE INDEX index_name ON table_name (column1, column2);

-- Create index with specific length (for strings)
CREATE INDEX index_name ON table_name (column_name(10));
```

### Dropping Indexes
```sql
-- Drop index
DROP INDEX index_name ON table_name;

-- Alternative syntax
ALTER TABLE table_name DROP INDEX index_name;
```

### Showing Indexes
```sql
-- Show indexes for table
SHOW INDEX FROM table_name;

-- Show indexes from database
SHOW INDEX FROM table_name IN database_name;
```

### Index Types
```sql
-- B-Tree index (default)
CREATE INDEX index_name ON table_name (column_name) USING BTREE;

-- Hash index
CREATE INDEX index_name ON table_name (column_name) USING HASH;

-- Full-text index
CREATE FULLTEXT INDEX index_name ON table_name (column_name);

-- Spatial index
CREATE SPATIAL INDEX index_name ON table_name (geometry_column);
```

---

## 17. Views

### Creating Views
```sql
-- Simple view
CREATE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE condition;

-- View with joins
CREATE VIEW employee_dept AS
SELECT e.emp_id, e.name, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;

-- View with aggregation
CREATE VIEW dept_summary AS
SELECT dept_id, COUNT(*) as emp_count, AVG(salary) as avg_salary
FROM employees
GROUP BY dept_id;
```

### Using Views
```sql
-- Query view like a table
SELECT * FROM view_name;

-- Join views
SELECT v1.column1, v2.column2
FROM view1 v1
JOIN view2 v2 ON v1.id = v2.id;
```

### Modifying Views
```sql
-- Replace view
CREATE OR REPLACE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE new_condition;

-- Alter view
ALTER VIEW view_name AS
SELECT column1, column2, column3
FROM table_name;
```

### Dropping Views
```sql
-- Drop view
DROP VIEW view_name;

-- Drop if exists
DROP VIEW IF EXISTS view_name;
```

### Updatable Views
```sql
-- Insert through view (if updatable)
INSERT INTO view_name (column1, column2) VALUES (value1, value2);

-- Update through view
UPDATE view_name SET column1 = value WHERE condition;

-- Delete through view
DELETE FROM view_name WHERE condition;

-- View is updatable if it doesn't contain:
-- DISTINCT, GROUP BY, HAVING, UNION, aggregates, subqueries in SELECT
```

---

## 18. Stored Procedures

### Creating Procedures
```sql
-- Basic procedure
DELIMITER //
CREATE PROCEDURE procedure_name()
BEGIN
    SELECT * FROM table_name;
END //
DELIMITER ;
```

### Calling Procedures
```sql
-- Call simple procedure
CALL procedure_name();

-- Call with IN parameter
CALL get_employee(101);

-- Call with OUT parameter
CALL get_count(@count);
SELECT @count;

-- Call with INOUT parameter
SET @value = 5;
CALL increment_value(@value);
SELECT @value;
```

### Managing Procedures
```sql
-- Show procedures
SHOW PROCEDURE STATUS WHERE Db = 'database_name';

-- Show procedure code
SHOW CREATE PROCEDURE procedure_name;

-- Drop procedure
DROP PROCEDURE procedure_name;

-- Drop if exists
DROP PROCEDURE IF EXISTS procedure_name;
```

---

## 19. Triggers

### Creating Triggers
```sql
-- BEFORE INSERT trigger
DELIMITER //
CREATE TRIGGER trigger_name
BEFORE INSERT ON table_name
FOR EACH ROW
BEGIN
    SET NEW.created_at = NOW();
END //
DELIMITER ;

-- AFTER INSERT trigger
DELIMITER //
CREATE TRIGGER log_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (action, emp_id, timestamp)
    VALUES ('INSERT', NEW.emp_id, NOW());
END //
DELIMITER ;

-- BEFORE UPDATE trigger
DELIMITER //
CREATE TRIGGER check_salary_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary < OLD.salary THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Salary cannot be decreased';
    END IF;
END //
DELIMITER ;

-- AFTER UPDATE trigger
DELIMITER //
CREATE TRIGGER log_salary_change
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary != OLD.salary THEN
        INSERT INTO salary_history (emp_id, old_salary, new_salary, change_date)
        VALUES (NEW.emp_id, OLD.salary, NEW.salary, NOW());
    END IF;
END //
DELIMITER ;

-- BEFORE DELETE trigger
DELIMITER //
CREATE TRIGGER prevent_delete
BEFORE DELETE ON employees
FOR EACH ROW
BEGIN
    IF OLD.role = 'Admin' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot delete admin users';
    END IF;
END //
DELIMITER ;

-- AFTER DELETE trigger
DELIMITER //
CREATE TRIGGER log_delete
AFTER DELETE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO deleted_employees 
    SELECT *, NOW() FROM employees WHERE emp_id = OLD.emp_id;
END //
DELIMITER ;
```

### Managing Triggers
```sql
-- Show triggers
SHOW TRIGGERS;
SHOW TRIGGERS FROM database_name;
SHOW TRIGGERS LIKE 'pattern%';

-- Show trigger code
SHOW CREATE TRIGGER trigger_name;

-- Drop trigger
DROP TRIGGER trigger_name;
DROP TRIGGER IF EXISTS trigger_name;
```

### Trigger Limitations
```sql
-- Cannot use:
-- - CALL to stored procedures that return data
-- - Transactions (START TRANSACTION, COMMIT, ROLLBACK)
-- - Cannot trigger another trigger on same table

-- Can access:
-- - OLD: old row values (UPDATE, DELETE)
-- - NEW: new row values (INSERT, UPDATE)
```

---

## 20. Advanced Concepts

### Common Table Expressions (CTE)
```sql
-- Basic CTE
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT * FROM cte_name;

-- Multiple CTEs
WITH 
    cte1 AS (SELECT * FROM table1 WHERE condition1),
    cte2 AS (SELECT * FROM table2 WHERE condition2)
SELECT * FROM cte1 JOIN cte2 ON cte1.id = cte2.id;

-- Recursive CTE
WITH RECURSIVE cte AS (
    -- Anchor member
    SELECT emp_id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive member
    SELECT e.emp_id, e.name, e.manager_id, cte.level + 1
    FROM employees e
    INNER JOIN cte ON e.manager_id = cte.emp_id
)
SELECT * FROM cte;
```

### UNION / UNION ALL
```sql
-- UNION: removes duplicates
SELECT column1, column2 FROM table1
UNION
SELECT column1, column2 FROM table2;

-- UNION ALL: keeps duplicates (faster)
SELECT column1, column2 FROM table1
UNION ALL
SELECT column1, column2 FROM table2;
```

### INTERSECT / EXCEPT (MySQL 8.0+)
```sql
-- INTERSECT: common rows
SELECT column1 FROM table1
INTERSECT
SELECT column1 FROM table2;

-- EXCEPT: rows in first but not second
SELECT column1 FROM table1
EXCEPT
SELECT column1 FROM table2;
```

### Pivot / Unpivot
```sql
-- Pivot using CASE
SELECT 
    dept_id,
    SUM(CASE WHEN year = 2022 THEN sales ELSE 0 END) AS sales_2022,
    SUM(CASE WHEN year = 2023 THEN sales ELSE 0 END) AS sales_2023,
    SUM(CASE WHEN year = 2024 THEN sales ELSE 0 END) AS sales_2024
FROM sales
GROUP BY dept_id;

-- Unpivot using UNION
SELECT dept_id, 'sales_2022' AS year, sales_2022 AS sales FROM pivot_table
UNION ALL
SELECT dept_id, 'sales_2023' AS year, sales_2023 AS sales FROM pivot_table
UNION ALL
SELECT dept_id, 'sales_2024' AS year, sales_2024 AS sales FROM pivot_table;
```

### Temporary Tables
```sql
-- Create temporary table
CREATE TEMPORARY TABLE temp_table (
    id INT,
    name VARCHAR(50)
);

-- Insert into temp table
INSERT INTO temp_table VALUES (1, 'John'), (2, 'Jane');

-- Use temp table
SELECT * FROM temp_table;

-- Create temp table from query
CREATE TEMPORARY TABLE temp_results AS
SELECT * FROM employees WHERE salary > 50000;

-- Temp tables are automatically dropped when session ends
DROP TEMPORARY TABLE IF EXISTS temp_table;
```

### Table Variables (MySQL alternative)
```sql
-- MySQL doesn't have table variables, use temp tables instead
-- Or use session variables with JSON
SET @data = JSON_ARRAY(
    JSON_OBJECT('id', 1, 'name', 'John'),
    JSON_OBJECT('id', 2, 'name', 'Jane')
);
```

### Dynamic SQL
```sql
-- Prepare statement
PREPARE stmt FROM 'SELECT * FROM employees WHERE dept_id = ?';

-- Execute with parameter
SET @dept = 10;
EXECUTE stmt USING @dept;

-- Deallocate
DEALLOCATE PREPARE stmt;

-- Dynamic query construction
SET @table_name = 'employees';
SET @sql = CONCAT('SELECT * FROM ', @table_name);
PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

### JSON Functions
```sql
-- Create JSON
SELECT JSON_OBJECT('name', name, 'salary', salary) FROM employees;

-- Array
SELECT JSON_ARRAY(1, 2, 3, 'four');

-- Extract from JSON
SELECT JSON_EXTRACT(json_column, '$.name') FROM table_name;
SELECT json_column->'$.name' FROM table_name;  -- shorthand
SELECT json_column->>'$.name' FROM table_name; -- unquoted

-- Set JSON value
UPDATE table_name 
SET json_column = JSON_SET(json_column, '$.name', 'New Name');

-- Insert into JSON
UPDATE table_name 
SET json_column = JSON_INSERT(json_column, '$.age', 30);

-- Remove from JSON
UPDATE table_name 
SET json_column = JSON_REMOVE(json_column, '$.age');

-- Check if path exists
SELECT * FROM table_name 
WHERE JSON_CONTAINS_PATH(json_column, 'one', '$.name');

-- Array functions
SELECT JSON_LENGTH(json_array_column) FROM table_name;
SELECT JSON_CONTAINS(json_array_column, '3') FROM table_name;
```

### Full-Text Search
```sql
-- Create full-text index
CREATE FULLTEXT INDEX idx_name ON table_name(column_name);

-- Natural language search
SELECT * FROM articles
WHERE MATCH(title, content) AGAINST('search term');

-- Boolean search
SELECT * FROM articles
WHERE MATCH(title, content) AGAINST('+mysql -oracle' IN BOOLEAN MODE);

-- With relevance score
SELECT *, MATCH(title, content) AGAINST('search term') AS relevance
FROM articles
WHERE MATCH(title, content) AGAINST('search term')
ORDER BY relevance DESC;

-- Query expansion
SELECT * FROM articles
WHERE MATCH(title, content) 
AGAINST('database' WITH QUERY EXPANSION);
```

### Partitioning
```sql
-- Range partitioning
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10,2)
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- List partitioning
CREATE TABLE employees (
    id INT,
    name VARCHAR(50),
    region VARCHAR(20)
)
PARTITION BY LIST COLUMNS(region) (
    PARTITION p_north VALUES IN('NY', 'MA', 'CT'),
    PARTITION p_south VALUES IN('FL', 'GA', 'TX'),
    PARTITION p_west VALUES IN('CA', 'OR', 'WA')
);

-- Hash partitioning
CREATE TABLE users (
    id INT,
    name VARCHAR(50)
)
PARTITION BY HASH(id)
PARTITIONS 4;

-- Show partitions
SELECT * FROM INFORMATION_SCHEMA.PARTITIONS 
WHERE TABLE_NAME = 'table_name';
```

### Performance Optimization
```sql
-- EXPLAIN: analyze query execution
EXPLAIN SELECT * FROM employees WHERE dept_id = 10;
EXPLAIN ANALYZE SELECT * FROM employees WHERE dept_id = 10;

-- Force index usage
SELECT * FROM table_name FORCE INDEX (index_name) WHERE condition;

-- Ignore index
SELECT * FROM table_name IGNORE INDEX (index_name) WHERE condition;

-- Optimize table
OPTIMIZE TABLE table_name;

-- Analyze table (update statistics)
ANALYZE TABLE table_name;

-- Check table integrity
CHECK TABLE table_name;

-- Repair table
REPAIR TABLE table_name;
```

### Information Schema
```sql
-- List all databases
SELECT * FROM INFORMATION_SCHEMA.SCHEMATA;

-- List all tables in database
SELECT * FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_SCHEMA = 'database_name';

-- List all columns in table
SELECT * FROM INFORMATION_SCHEMA.COLUMNS 
WHERE TABLE_SCHEMA = 'database_name' AND TABLE_NAME = 'table_name';

-- List all indexes
SELECT * FROM INFORMATION_SCHEMA.STATISTICS 
WHERE TABLE_SCHEMA = 'database_name';

-- List all foreign keys
SELECT * FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE 
WHERE CONSTRAINT_SCHEMA = 'database_name' 
AND REFERENCED_TABLE_NAME IS NOT NULL;

-- List all triggers
SELECT * FROM INFORMATION_SCHEMA.TRIGGERS 
WHERE TRIGGER_SCHEMA = 'database_name';

-- List all procedures
SELECT * FROM INFORMATION_SCHEMA.ROUTINES 
WHERE ROUTINE_SCHEMA = 'database_name' AND ROUTINE_TYPE = 'PROCEDURE';

-- Table sizes
SELECT 
    TABLE_NAME,
    ROUND((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024, 2) AS size_mb
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'database_name'
ORDER BY (DATA_LENGTH + INDEX_LENGTH) DESC;
```

### User Management
```sql
-- Create user
CREATE USER 'username'@'host' IDENTIFIED BY 'password';

-- Change password
ALTER USER 'username'@'host' IDENTIFIED BY 'new_password';

-- Drop user
DROP USER 'username'@'host';

-- Show current user
SELECT USER(), CURRENT_USER();

-- Show all users
SELECT User, Host FROM mysql.user;

-- Show grants for user
SHOW GRANTS FOR 'username'@'host';
```

### Locking
```sql
-- Lock tables for read
LOCK TABLES table_name READ;

-- Lock tables for write
LOCK TABLES table_name WRITE;

-- Lock multiple tables
LOCK TABLES table1 READ, table2 WRITE;

-- Unlock tables
UNLOCK TABLES;

-- Row-level locking (InnoDB)
SELECT * FROM table_name WHERE id = 1 FOR UPDATE;  -- exclusive lock
SELECT * FROM table_name WHERE id = 1 LOCK IN SHARE MODE;  -- shared lock
```

### Import/Export
```sql
-- Export to CSV
SELECT * FROM table_name
INTO OUTFILE '/path/to/file.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';

-- Import from CSV
LOAD DATA INFILE '/path/to/file.csv'
INTO TABLE table_name
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;  -- skip header

-- Backup database (command line)
-- mysqldump -u username -p database_name > backup.sql

-- Restore database (command line)
-- mysql -u username -p database_name < backup.sql
```

---

## Quick Reference - SQL Order of Execution

1. **FROM** - Choose tables
2. **JOIN** - Combine tables
3. **WHERE** - Filter rows
4. **GROUP BY** - Group rows
5. **HAVING** - Filter groups
6. **SELECT** - Choose columns
7. **DISTINCT** - Remove duplicates
8. **ORDER BY** - Sort results
9. **LIMIT** - Limit rows returned

## Quick Reference - SQL Statement Order

```sql
SELECT [DISTINCT] column1, column2, aggregate_function(column3)
FROM table1
[JOIN table2 ON join_condition]
[WHERE condition]
[GROUP BY column1, column2]
[HAVING group_condition]
[ORDER BY column1 [ASC|DESC]]
[LIMIT number [OFFSET number]];
```

## Quick Reference - Common Patterns

### Find Duplicates
```sql
SELECT column1, COUNT(*) 
FROM table_name 
GROUP BY column1 
HAVING COUNT(*) > 1;
```

### Delete Duplicates (Keep One)
```sql
DELETE t1 FROM table_name t1
INNER JOIN table_name t2 
WHERE t1.id > t2.id AND t1.column = t2.column;
```

### Nth Highest Value
```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET n-1;
```

### Running Total
```sql
SELECT date, amount,
    SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;
```

### Top N per Group
```sql
SELECT * FROM (
    SELECT *, 
        ROW_NUMBER() OVER (PARTITION BY group_col ORDER BY value_col DESC) AS rn
    FROM table_name
) t
WHERE rn <= N;
```

---

## Best Practices

1. **Use explicit JOINs** instead of implicit joins in WHERE clause
2. **Always use table aliases** for readability
3. **Index frequently queried columns** (WHERE, JOIN, ORDER BY)
4. **Avoid SELECT *** - specify columns needed
5. **Use LIMIT** when testing queries
6. **Use transactions** for multiple related operations
7. **Normalize tables** to reduce redundancy
8. **Use appropriate data types** to save space
9. **Comment complex queries** for maintainability
10. **Use EXPLAIN** to analyze query performance

---

*This reference guide covers SQL based on MySQL syntax. Some features may vary slightly in other RDBMS like PostgreSQL, SQL Server, or Oracle.*

-- Procedure with parameters
DELIMITER //
CREATE PROCEDURE get_employee(IN emp_id INT)
BEGIN
    SELECT * FROM employees WHERE id = emp_id;
END //
DELIMITER ;

-- Procedure with OUT parameter
DELIMITER //
CREATE PROCEDURE get_count(OUT emp_count INT)
BEGIN
    SELECT COUNT(*) INTO emp_count FROM employees;
END //
DELIMITER ;

-- Procedure with INOUT parameter
DELIMITER //
CREATE PROCEDURE increment_value(INOUT value INT)
BEGIN
    SET value = value + 1;
END //
DELIMITER ;

-- Procedure with variables
DELIMITER //
CREATE PROCEDURE complex_procedure(IN dept_id INT)
BEGIN
    DECLARE avg_sal DECIMAL(10,2);
    DECLARE emp_count INT;
    
    SELECT AVG(salary), COUNT(*) 
    INTO avg_sal, emp_count
    FROM employees 
    WHERE department_id = dept_id;
    
    SELECT avg_sal, emp_count;
END //
DELIMITER ;

-- Procedure with IF statement
DELIMITER //
CREATE PROCEDURE check_salary(IN emp_id INT)
BEGIN
    DECLARE sal DECIMAL(10,2);
    SELECT salary INTO sal FROM employees WHERE id = emp_id;
    
    IF sal > 100000 THEN
        SELECT 'High salary';
    ELSEIF sal > 50000 THEN
        SELECT 'Medium salary';
    ELSE
        SELECT 'Low salary';
    END IF;
END //
DELIMITER ;

-- Procedure with CASE
DELIMITER //
CREATE PROCEDURE rate_employee(IN emp_id INT)
BEGIN
    DECLARE rating VARCHAR(20);
    DECLARE perf_score INT;
    
    SELECT performance_score INTO perf_score 
    FROM employees WHERE id = emp_id;
    
    CASE
        WHEN perf_score >= 90 THEN SET rating = 'Excellent';
        WHEN perf_score >= 70 THEN SET rating = 'Good';
        WHEN perf_score >= 50 THEN SET rating = 'Average';
        ELSE SET rating = 'Poor';
    END CASE;
    
    SELECT rating;
END //
DELIMITER ;

-- Procedure with LOOP
DELIMITER //
CREATE PROCEDURE loop_example()
BEGIN
    DECLARE counter INT DEFAULT 0;
    
    my_loop: LOOP
        SET counter = counter + 1;
        
        IF counter > 10 THEN
            LEAVE my_loop;
        END IF;
    END LOOP;
    
    SELECT counter;
END //
DELIMITER ;

-- Procedure with WHILE
DELIMITER //
CREATE PROCEDURE while_example()
BEGIN
    DECLARE counter INT DEFAULT 0;
    
    WHILE counter < 10 DO
        SET counter = counter + 1;
    END WHILE;
    
    SELECT counter;
END //
DELIMITER ;

-- Procedure with REPEAT
DELIMITER //
CREATE PROCEDURE repeat_example()
BEGIN
    DECLARE counter INT DEFAULT 0;
    
    REPEAT
        SET counter = counter + 1;
    UNTIL counter >= 10
    END REPEAT;
    
    SELECT counter;
END //
DELIMITER ;

-- Procedure with cursor
DELIMITER //
CREATE PROCEDURE cursor_example()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE emp_name VARCHAR(100);
    DECLARE emp_salary DECIMAL(10,2);
    
    DECLARE emp_cursor CURSOR FOR 
        SELECT name, salary FROM employees;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN emp_cursor;
    
    read_loop: LOOP
        FETCH emp_cursor INTO emp_name, emp_salary;
        
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        -- Process data here
        SELECT emp_name, emp_salary;
    END LOOP;
    
    CLOSE emp_cursor;
END //
DELIMITER ;

-- Procedure with error handling
DELIMITER //
CREATE PROCEDURE error_handling_example(IN emp_id INT)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT 'Error occurred, transaction rolled back';
    END;
    
    START TRANSACTION;
    
    UPDATE employees SET salary = salary * 1.1 WHERE id = emp_id;
    
    COMMIT;
    SELECT 'Transaction completed successfully';
END //
DELIMITER ;