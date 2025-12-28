# SQL Syntax Reference (Cross-Engine)

## Overview

SQL syntax differences between database engines. Use this reference when writing database-agnostic code or migrating between databases.

## Metadata

| Property            | Value                                                                   |
| ------------------- | ----------------------------------------------------------------------- |
| **Example Version** | 1.0.0                                                                   |
| **Last Updated**    | 2025-01                                                                 |
| **Engines**         | MySQL 8.x, MariaDB 10.11, PostgreSQL 15/16, SQLite 3.x, SQL Server 2022 |
| **Stacks**          | All                                                                     |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [Auto-Increment Syntax](#auto-increment-syntax)
  - [MySQL / MariaDB](#mysql--mariadb)
  - [PostgreSQL](#postgresql)
  - [SQLite](#sqlite)
  - [SQL Server](#sql-server)
- [Data Type Mappings](#data-type-mappings)
  - [Boolean Storage Examples](#boolean-storage-examples)
- [String Concatenation](#string-concatenation)
  - [MySQL / MariaDB](#mysql--mariadb-1)
  - [PostgreSQL](#postgresql-1)
  - [SQLite](#sqlite-1)
  - [SQL Server](#sql-server-1)
- [Date/Time Functions](#datetime-functions)
  - [Current Timestamp](#current-timestamp)
  - [Date Formatting](#date-formatting)
  - [Date Arithmetic](#date-arithmetic)
- [Pagination Syntax](#pagination-syntax)
  - [MySQL / MariaDB / PostgreSQL / SQLite](#mysql--mariadb--postgresql--sqlite)
  - [SQL Server](#sql-server-2)
- [JSON Support](#json-support)
  - [MySQL 5.7+ / MariaDB 10.2+](#mysql-57--mariadb-102)
  - [PostgreSQL](#postgresql-2)
  - [SQL Server 2016+](#sql-server-2016)
- [Upsert Syntax](#upsert-syntax)
  - [MySQL / MariaDB](#mysql--mariadb-2)
  - [PostgreSQL](#postgresql-3)
  - [SQLite](#sqlite-2)
  - [SQL Server](#sql-server-3)



## Auto-Increment Syntax

### MySQL / MariaDB

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```

### PostgreSQL

```sql
-- Using SERIAL (older method)
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);

-- Using IDENTITY (PostgreSQL 10+, preferred)
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```

### SQLite

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL
);
```

### SQL Server

```sql
CREATE TABLE users (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(255) NOT NULL
);
```

---

## Data Type Mappings

| Concept     | MySQL/MariaDB   | PostgreSQL      | SQLite    | SQL Server         |
| ----------- | --------------- | --------------- | --------- | ------------------ |
| Boolean     | `TINYINT(1)`    | `BOOLEAN`       | `INTEGER` | `BIT`              |
| UUID        | `CHAR(36)`      | `UUID`          | `TEXT`    | `UNIQUEIDENTIFIER` |
| JSON        | `JSON`          | `JSONB`         | `TEXT`    | `NVARCHAR(MAX)`    |
| Large text  | `LONGTEXT`      | `TEXT`          | `TEXT`    | `NVARCHAR(MAX)`    |
| Binary      | `BLOB`          | `BYTEA`         | `BLOB`    | `VARBINARY(MAX)`   |
| Timestamp   | `TIMESTAMP`     | `TIMESTAMPTZ`   | `TEXT`    | `DATETIME2`        |
| Decimal     | `DECIMAL(10,2)` | `NUMERIC(10,2)` | `REAL`    | `DECIMAL(10,2)`    |
| Big Integer | `BIGINT`        | `BIGINT`        | `INTEGER` | `BIGINT`           |

### Boolean Storage Examples

```sql
-- MySQL/MariaDB
ALTER TABLE users ADD COLUMN is_active TINYINT(1) DEFAULT 1;
INSERT INTO users (is_active) VALUES (1);  -- true
INSERT INTO users (is_active) VALUES (0);  -- false

-- PostgreSQL
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT TRUE;
INSERT INTO users (is_active) VALUES (TRUE);
INSERT INTO users (is_active) VALUES (FALSE);

-- SQL Server
ALTER TABLE users ADD is_active BIT DEFAULT 1;
INSERT INTO users (is_active) VALUES (1);  -- true
INSERT INTO users (is_active) VALUES (0);  -- false
```

---

## String Concatenation

### MySQL / MariaDB

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
```

### PostgreSQL

```sql
-- Using operator
SELECT first_name || ' ' || last_name AS full_name FROM users;

-- Using CONCAT function
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
```

### SQLite

```sql
SELECT first_name || ' ' || last_name AS full_name FROM users;
```

### SQL Server

```sql
-- Using operator
SELECT first_name + ' ' + last_name AS full_name FROM users;

-- Using CONCAT function (SQL Server 2012+)
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
```

---

## Date/Time Functions

### Current Timestamp

| Engine        | Function                               |
| ------------- | -------------------------------------- |
| MySQL/MariaDB | `NOW()`, `CURRENT_TIMESTAMP`           |
| PostgreSQL    | `NOW()`, `CURRENT_TIMESTAMP`           |
| SQLite        | `datetime('now')`, `CURRENT_TIMESTAMP` |
| SQL Server    | `GETDATE()`, `CURRENT_TIMESTAMP`       |

### Date Formatting

```sql
-- MySQL/MariaDB
SELECT DATE_FORMAT(created_at, '%d/%m/%Y') FROM users;

-- PostgreSQL
SELECT TO_CHAR(created_at, 'DD/MM/YYYY') FROM users;

-- SQLite
SELECT strftime('%d/%m/%Y', created_at) FROM users;

-- SQL Server
SELECT FORMAT(created_at, 'dd/MM/yyyy') FROM users;
```

### Date Arithmetic

```sql
-- MySQL/MariaDB
SELECT DATE_ADD(created_at, INTERVAL 30 DAY) FROM users;
SELECT DATE_SUB(created_at, INTERVAL 1 MONTH) FROM users;

-- PostgreSQL
SELECT created_at + INTERVAL '30 days' FROM users;
SELECT created_at - INTERVAL '1 month' FROM users;

-- SQLite
SELECT datetime(created_at, '+30 days') FROM users;
SELECT datetime(created_at, '-1 month') FROM users;

-- SQL Server
SELECT DATEADD(day, 30, created_at) FROM users;
SELECT DATEADD(month, -1, created_at) FROM users;
```

---

## Pagination Syntax

### MySQL / MariaDB / PostgreSQL / SQLite

```sql
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 10 OFFSET 20;
```

### SQL Server

```sql
-- SQL Server 2012+ (OFFSET-FETCH)
SELECT * FROM users
ORDER BY created_at DESC
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;

-- Legacy (TOP with subquery)
SELECT TOP 10 * FROM users
WHERE id NOT IN (
    SELECT TOP 20 id FROM users ORDER BY created_at DESC
)
ORDER BY created_at DESC;
```

---

## JSON Support

### MySQL 5.7+ / MariaDB 10.2+

```sql
-- Insert JSON
INSERT INTO settings (user_id, preferences)
VALUES (1, '{"theme": "dark", "language": "en"}');

-- Query JSON
SELECT preferences->>'$.theme' AS theme FROM settings;
SELECT JSON_EXTRACT(preferences, '$.theme') AS theme FROM settings;

-- Update JSON
UPDATE settings
SET preferences = JSON_SET(preferences, '$.theme', 'light')
WHERE user_id = 1;
```

### PostgreSQL

```sql
-- Insert JSON (JSONB recommended)
INSERT INTO settings (user_id, preferences)
VALUES (1, '{"theme": "dark", "language": "en"}'::jsonb);

-- Query JSON
SELECT preferences->>'theme' AS theme FROM settings;
SELECT preferences->'nested'->>'key' AS nested_value FROM settings;

-- Update JSON
UPDATE settings
SET preferences = preferences || '{"theme": "light"}'::jsonb
WHERE user_id = 1;

-- Query with JSON conditions
SELECT * FROM settings
WHERE preferences @> '{"theme": "dark"}'::jsonb;
```

### SQL Server 2016+

```sql
-- Insert JSON
INSERT INTO settings (user_id, preferences)
VALUES (1, N'{"theme": "dark", "language": "en"}');

-- Query JSON
SELECT JSON_VALUE(preferences, '$.theme') AS theme FROM settings;

-- Update JSON (requires full replace)
UPDATE settings
SET preferences = JSON_MODIFY(preferences, '$.theme', 'light')
WHERE user_id = 1;
```

---

## Upsert Syntax

### MySQL / MariaDB

```sql
-- ON DUPLICATE KEY UPDATE
INSERT INTO users (email, name, updated_at)
VALUES ('user@example.com', 'John', NOW())
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    updated_at = NOW();

-- REPLACE (deletes and re-inserts)
REPLACE INTO users (email, name)
VALUES ('user@example.com', 'John');
```

### PostgreSQL

```sql
-- ON CONFLICT (PostgreSQL 9.5+)
INSERT INTO users (email, name, updated_at)
VALUES ('user@example.com', 'John', NOW())
ON CONFLICT (email)
DO UPDATE SET
    name = EXCLUDED.name,
    updated_at = NOW();

-- ON CONFLICT DO NOTHING
INSERT INTO users (email, name)
VALUES ('user@example.com', 'John')
ON CONFLICT (email) DO NOTHING;
```

### SQLite

```sql
-- ON CONFLICT (SQLite 3.24+)
INSERT INTO users (email, name, updated_at)
VALUES ('user@example.com', 'John', datetime('now'))
ON CONFLICT (email)
DO UPDATE SET
    name = excluded.name,
    updated_at = datetime('now');

-- INSERT OR REPLACE
INSERT OR REPLACE INTO users (email, name)
VALUES ('user@example.com', 'John');
```

### SQL Server

```sql
-- MERGE statement
MERGE INTO users AS target
USING (VALUES ('user@example.com', 'John')) AS source (email, name)
ON target.email = source.email
WHEN MATCHED THEN
    UPDATE SET name = source.name, updated_at = GETDATE()
WHEN NOT MATCHED THEN
    INSERT (email, name, created_at) VALUES (source.email, source.name, GETDATE());
```
