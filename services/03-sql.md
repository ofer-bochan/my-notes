# SQL Basics

> **What it is:** SQL (Structured Query Language) is the standard language for managing and querying relational databases like MySQL, MariaDB, PostgreSQL, and SQLite.

## Database Concepts

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Relational Database Concepts                         │
│                                                                              │
│   Database                                                                   │
│   └── Tables                                                                 │
│       ├── Columns (fields/attributes)                                        │
│       ├── Rows (records/tuples)                                             │
│       ├── Primary Key (unique identifier)                                   │
│       └── Foreign Key (reference to another table)                          │
│                                                                              │
│   ┌─────────────────────────────────────────┐                               │
│   │  users table                            │                               │
│   │  ┌────┬──────────┬─────────────┬─────┐  │                               │
│   │  │ id │ username │ email       │ age │  │  ◄── Columns                  │
│   │  ├────┼──────────┼─────────────┼─────┤  │                               │
│   │  │ 1  │ john     │ j@email.com │ 25  │  │  ◄── Rows                     │
│   │  │ 2  │ jane     │ jane@x.com  │ 30  │  │                               │
│   │  │ 3  │ bob      │ bob@y.org   │ 28  │  │                               │
│   │  └────┴──────────┴─────────────┴─────┘  │                               │
│   │    ▲                                    │                               │
│   │    └── Primary Key                      │                               │
│   └─────────────────────────────────────────┘                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Connecting to Databases

```bash
# MySQL/MariaDB
mysql -u username -p
mysql -u username -p database_name
mysql -h hostname -u username -p

# PostgreSQL
psql -U username -d database_name
psql -h hostname -U username -d database_name

# SQLite
sqlite3 database.db
```

## Database Operations

```sql
-- Show all databases
SHOW DATABASES;                    -- MySQL
\l                                  -- PostgreSQL

-- Create database
CREATE DATABASE myapp;

-- Select database
USE myapp;                         -- MySQL
\c myapp                           -- PostgreSQL

-- Drop database (CAREFUL!)
DROP DATABASE myapp;

-- Show current database
SELECT DATABASE();                 -- MySQL
SELECT current_database();         -- PostgreSQL
```

## Table Operations

### Create Table

```sql
-- Basic table
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,    -- MySQL
    -- id SERIAL PRIMARY KEY,             -- PostgreSQL
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    age INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);

-- Table with foreign key
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    product_name VARCHAR(100),
    quantity INT DEFAULT 1,
    price DECIMAL(10, 2),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### Modify Table

```sql
-- Add column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Modify column
ALTER TABLE users MODIFY COLUMN phone VARCHAR(30);    -- MySQL
ALTER TABLE users ALTER COLUMN phone TYPE VARCHAR(30); -- PostgreSQL

-- Rename column
ALTER TABLE users RENAME COLUMN phone TO phone_number;

-- Drop column
ALTER TABLE users DROP COLUMN phone_number;

-- Add index
CREATE INDEX idx_username ON users(username);

-- Show table structure
DESCRIBE users;                    -- MySQL
\d users                           -- PostgreSQL
```

## CRUD Operations

### INSERT - Create

```sql
-- Insert single row
INSERT INTO users (username, email, password_hash, age)
VALUES ('john_doe', 'john@example.com', 'hash123', 25);

-- Insert multiple rows
INSERT INTO users (username, email, password_hash, age) VALUES
    ('jane_doe', 'jane@example.com', 'hash456', 30),
    ('bob_smith', 'bob@example.com', 'hash789', 28),
    ('alice_jones', 'alice@example.com', 'hash012', 35);

-- Insert from SELECT
INSERT INTO archived_users (username, email)
SELECT username, email FROM users WHERE is_active = FALSE;
```

### SELECT - Read

```sql
-- Select all columns
SELECT * FROM users;

-- Select specific columns
SELECT username, email FROM users;

-- Select with alias
SELECT username AS name, email AS contact FROM users;

-- Select distinct values
SELECT DISTINCT age FROM users;
```

### WHERE - Filtering

```sql
-- Comparison operators
SELECT * FROM users WHERE age > 25;
SELECT * FROM users WHERE age >= 25;
SELECT * FROM users WHERE age < 30;
SELECT * FROM users WHERE age = 25;
SELECT * FROM users WHERE age != 25;

-- Multiple conditions
SELECT * FROM users WHERE age > 25 AND is_active = TRUE;
SELECT * FROM users WHERE age < 20 OR age > 40;
SELECT * FROM users WHERE NOT is_active;

-- BETWEEN
SELECT * FROM users WHERE age BETWEEN 25 AND 35;

-- IN
SELECT * FROM users WHERE age IN (25, 30, 35);
SELECT * FROM users WHERE username IN ('john', 'jane', 'bob');

-- LIKE (pattern matching)
SELECT * FROM users WHERE email LIKE '%@gmail.com';     -- Ends with
SELECT * FROM users WHERE username LIKE 'john%';        -- Starts with
SELECT * FROM users WHERE username LIKE '%doe%';        -- Contains
SELECT * FROM users WHERE username LIKE 'j___';         -- j + 3 chars

-- NULL checks
SELECT * FROM users WHERE phone IS NULL;
SELECT * FROM users WHERE phone IS NOT NULL;
```

### ORDER BY - Sorting

```sql
-- Ascending (default)
SELECT * FROM users ORDER BY age;
SELECT * FROM users ORDER BY age ASC;

-- Descending
SELECT * FROM users ORDER BY age DESC;

-- Multiple columns
SELECT * FROM users ORDER BY age DESC, username ASC;
```

### LIMIT - Pagination

```sql
-- First 10 rows
SELECT * FROM users LIMIT 10;

-- Skip 10, get next 10 (page 2)
SELECT * FROM users LIMIT 10 OFFSET 10;

-- MySQL shorthand
SELECT * FROM users LIMIT 10, 10;  -- offset, count
```

### UPDATE - Modify

```sql
-- Update single column
UPDATE users SET age = 26 WHERE username = 'john_doe';

-- Update multiple columns
UPDATE users 
SET age = 26, email = 'newemail@example.com' 
WHERE username = 'john_doe';

-- Update with calculation
UPDATE products SET price = price * 1.1;  -- 10% increase

-- ⚠️ ALWAYS use WHERE unless updating ALL rows!
```

### DELETE - Remove

```sql
-- Delete specific rows
DELETE FROM users WHERE username = 'john_doe';

-- Delete with multiple conditions
DELETE FROM users WHERE age < 18 AND is_active = FALSE;

-- Delete all rows (keeps table structure)
DELETE FROM users;

-- Faster way to delete all rows
TRUNCATE TABLE users;

-- ⚠️ ALWAYS use WHERE unless deleting ALL rows!
```

## Aggregate Functions

```sql
-- COUNT
SELECT COUNT(*) FROM users;                        -- All rows
SELECT COUNT(phone) FROM users;                    -- Non-null only
SELECT COUNT(DISTINCT age) FROM users;             -- Unique values

-- SUM
SELECT SUM(price) FROM orders;

-- AVG
SELECT AVG(age) FROM users;

-- MIN / MAX
SELECT MIN(age), MAX(age) FROM users;

-- Combined
SELECT 
    COUNT(*) as total_users,
    AVG(age) as average_age,
    MIN(age) as youngest,
    MAX(age) as oldest
FROM users;
```

## GROUP BY

```sql
-- Count users by age
SELECT age, COUNT(*) as count 
FROM users 
GROUP BY age;

-- Total orders per user
SELECT user_id, COUNT(*) as order_count, SUM(price) as total_spent
FROM orders
GROUP BY user_id;

-- HAVING - filter groups
SELECT user_id, COUNT(*) as order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;

-- WHERE vs HAVING
SELECT user_id, SUM(price) as total
FROM orders
WHERE order_date > '2024-01-01'    -- Filters ROWS first
GROUP BY user_id
HAVING SUM(price) > 100;           -- Then filters GROUPS
```

## JOIN - Combining Tables

```
users                    orders
┌────┬──────────┐       ┌────┬─────────┬─────────────┐
│ id │ username │       │ id │ user_id │ product     │
├────┼──────────┤       ├────┼─────────┼─────────────┤
│ 1  │ john     │◄──────│ 1  │ 1       │ Laptop      │
│ 2  │ jane     │◄──────│ 2  │ 1       │ Mouse       │
│ 3  │ bob      │       │ 3  │ 2       │ Keyboard    │
└────┴──────────┘       └────┴─────────┴─────────────┘
```

```sql
-- INNER JOIN (only matching rows)
SELECT users.username, orders.product_name
FROM users
INNER JOIN orders ON users.id = orders.user_id;

-- LEFT JOIN (all from left, matching from right)
SELECT users.username, orders.product_name
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
-- Shows bob with NULL (has no orders)

-- RIGHT JOIN (all from right, matching from left)
SELECT users.username, orders.product_name
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;

-- Table aliases for readability
SELECT u.username, o.product_name, o.price
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.price > 50;

-- Multiple JOINs
SELECT u.username, o.product_name, c.category_name
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN categories c ON o.category_id = c.id;
```

## Subqueries

```sql
-- Subquery in WHERE
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE price > 100);

-- Subquery in SELECT
SELECT username,
    (SELECT COUNT(*) FROM orders WHERE user_id = users.id) as order_count
FROM users;

-- Subquery in FROM (derived table)
SELECT avg_by_age.age, avg_by_age.avg_orders
FROM (
    SELECT u.age, AVG(o.price) as avg_orders
    FROM users u
    JOIN orders o ON u.id = o.user_id
    GROUP BY u.age
) as avg_by_age
WHERE avg_by_age.avg_orders > 50;
```

## Useful Patterns

```sql
-- Check if exists before insert
INSERT INTO users (username, email, password_hash)
SELECT 'newuser', 'new@example.com', 'hash'
WHERE NOT EXISTS (SELECT 1 FROM users WHERE username = 'newuser');

-- Upsert (Insert or Update) - MySQL
INSERT INTO users (username, email) 
VALUES ('john', 'john@new.com')
ON DUPLICATE KEY UPDATE email = VALUES(email);

-- Upsert - PostgreSQL
INSERT INTO users (username, email)
VALUES ('john', 'john@new.com')
ON CONFLICT (username) DO UPDATE SET email = EXCLUDED.email;

-- Get second highest value
SELECT MAX(price) FROM products 
WHERE price < (SELECT MAX(price) FROM products);

-- Running total (window function)
SELECT id, price,
    SUM(price) OVER (ORDER BY id) as running_total
FROM orders;
```

## Database Administration

### MySQL/MariaDB

```bash
# Connect
mysql -u root -p

# Common commands
SHOW DATABASES;
USE database_name;
SHOW TABLES;
DESCRIBE table_name;
SHOW CREATE TABLE table_name;

# Export database
mysqldump -u username -p database_name > backup.sql

# Import database
mysql -u username -p database_name < backup.sql

# Create user
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON database.* TO 'username'@'localhost';
FLUSH PRIVILEGES;
```

### PostgreSQL

```bash
# Connect
psql -U postgres

# Common commands
\l                    # List databases
\c database_name      # Connect to database
\dt                   # List tables
\d table_name         # Describe table
\du                   # List users
\q                    # Quit

# Export database
pg_dump -U username database_name > backup.sql

# Import database
psql -U username -d database_name -f backup.sql

# Create user
CREATE USER username WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE dbname TO username;
```

## Views

> **What it is:** A view is a virtual table based on a SELECT query. Simplifies complex queries and controls data access.

```sql
-- Create view
CREATE VIEW active_customers AS
SELECT id, name, email FROM customers WHERE status = 'active';

-- Use view
SELECT * FROM active_customers;

-- Create view with join
CREATE VIEW order_details AS
SELECT o.id, c.name AS customer, p.name AS product, oi.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;

-- Drop view
DROP VIEW IF EXISTS active_customers;
```

## Indexes

> **What it is:** Indexes speed up data retrieval by creating a structure for quick row lookup.

```sql
-- Create index
CREATE INDEX idx_email ON customers(email);

-- Composite index
CREATE INDEX idx_order_date ON orders(customer_id, order_date);

-- Unique index
CREATE UNIQUE INDEX idx_unique_email ON customers(email);

-- Drop index
DROP INDEX idx_email ON customers;

-- Show indexes
SHOW INDEX FROM customers;         -- MySQL
\di                                -- PostgreSQL
```

## Triggers

> **What it is:** A trigger is a stored procedure that automatically executes on INSERT, UPDATE, or DELETE events.

### MySQL Triggers

```sql
-- Update timestamp on change
DELIMITER //
CREATE TRIGGER update_timestamp
BEFORE UPDATE ON customers
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END//
DELIMITER ;

-- Audit log trigger
DELIMITER //
CREATE TRIGGER audit_changes
AFTER UPDATE ON customers
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, record_id, old_email, new_email, changed_at)
    VALUES ('customers', OLD.id, OLD.email, NEW.email, NOW());
END//
DELIMITER ;

-- Prevent delete
DELIMITER //
CREATE TRIGGER prevent_delete_active
BEFORE DELETE ON customers
FOR EACH ROW
BEGIN
    IF OLD.status = 'active' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot delete active customers';
    END IF;
END//
DELIMITER ;

-- Show triggers
SHOW TRIGGERS;

-- Drop trigger
DROP TRIGGER IF EXISTS update_timestamp;
```

### PostgreSQL Triggers

```sql
-- Create function first
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER update_timestamp
BEFORE UPDATE ON customers
FOR EACH ROW
EXECUTE FUNCTION update_timestamp();

-- Drop trigger
DROP TRIGGER IF EXISTS update_timestamp ON customers;
```

## Stored Procedures

> **What it is:** A stored procedure is a reusable set of SQL statements stored in the database.

### MySQL

```sql
-- Simple procedure
DELIMITER //
CREATE PROCEDURE get_active_customers()
BEGIN
    SELECT * FROM customers WHERE status = 'active';
END//
DELIMITER ;

-- Call procedure
CALL get_active_customers();

-- Procedure with parameters
DELIMITER //
CREATE PROCEDURE get_customer_orders(IN cust_id INT)
BEGIN
    SELECT * FROM orders WHERE customer_id = cust_id;
END//
DELIMITER ;

CALL get_customer_orders(123);

-- Procedure with output
DELIMITER //
CREATE PROCEDURE count_orders(IN cust_id INT, OUT total INT)
BEGIN
    SELECT COUNT(*) INTO total FROM orders WHERE customer_id = cust_id;
END//
DELIMITER ;

CALL count_orders(123, @count);
SELECT @count;

-- Drop procedure
DROP PROCEDURE IF EXISTS get_active_customers;
```

### PostgreSQL

```sql
CREATE OR REPLACE PROCEDURE update_status(p_id INT, p_status VARCHAR)
LANGUAGE plpgsql AS $$
BEGIN
    UPDATE customers SET status = p_status WHERE id = p_id;
END;
$$;

CALL update_status(123, 'inactive');
```

## Transactions

> **What it is:** A transaction is a sequence of operations performed as a single unit - all succeed or all fail.

```sql
-- Basic transaction
START TRANSACTION;

INSERT INTO orders (customer_id, total) VALUES (1, 100.00);
UPDATE customers SET last_order = NOW() WHERE id = 1;

COMMIT;  -- Save changes

-- Rollback on error
START TRANSACTION;
INSERT INTO orders (customer_id, total) VALUES (1, 100.00);
ROLLBACK;  -- Undo all changes

-- Savepoints
START TRANSACTION;
INSERT INTO orders (customer_id, total) VALUES (1, 100.00);
SAVEPOINT order_created;

INSERT INTO order_items (order_id, product_id) VALUES (1, 5);
ROLLBACK TO SAVEPOINT order_created;  -- Undo only items

COMMIT;
```

## Quick Reference

| Operation | SQL |
|-----------|-----|
| Create | `INSERT INTO t (cols) VALUES (vals)` |
| Read | `SELECT cols FROM t WHERE condition` |
| Update | `UPDATE t SET col=val WHERE condition` |
| Delete | `DELETE FROM t WHERE condition` |
| Join | `SELECT * FROM t1 JOIN t2 ON t1.id=t2.fk` |
| Group | `SELECT col, COUNT(*) FROM t GROUP BY col` |
| View | `CREATE VIEW v AS SELECT * FROM t` |
| Index | `CREATE INDEX idx ON t(col)` |
| Trigger | `CREATE TRIGGER name BEFORE UPDATE ON t ...` |
| Procedure | `CREATE PROCEDURE name() BEGIN ... END` |

---

*Part of the [Services Documentation](README.md)*

