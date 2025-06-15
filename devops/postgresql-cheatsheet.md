# PostgreSQL Cheatsheet

PostgreSQL is a powerful, open-source object-relational database system with over 35 years
of active development. It has earned a strong reputation for reliability, feature robustness,
and performance.

## Overview

### Key Features

- **ACID Compliance** - Full support for transactions, consistency, isolation, durability
- **Extensibility** - Custom data types, operators, functions, and extensions
- **SQL Standards** - Extensive SQL standard compliance with advanced features
- **Concurrency** - Multi-version concurrency control (MVCC)
- **Replication** - Built-in streaming replication and logical replication
- **Full-text Search** - Advanced text search capabilities
- **JSON Support** - Native JSON and JSONB data types
- **Partitioning** - Table partitioning for large datasets
- **Foreign Data Wrappers** - Access external data sources
- **Procedural Languages** - PL/pgSQL, Python, Perl, JavaScript support

### Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Apps  │    │  PostgreSQL     │    │   Storage       │
│                 │    │   Server        │    │                 │
│ • psql          │◄──►│ • Postmaster    │◄──►│ • Data Files    │
│ • Applications  │    │ • Backend       │    │ • WAL Files     │
│ • pgAdmin       │    │ • Shared Memory │    │ • Log Files     │
│ • Drivers       │    │ • Processes     │    │ • Config Files  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                       ┌─────────────────┐
                       │   Extensions    │
                       │                 │
                       │ • PostGIS       │
                       │ • pg_stat_*     │
                       │ • pgcrypto      │
                       │ • uuid-ossp     │
                       └─────────────────┘
```

## Installation

### macOS Installation

```bash
# Using Homebrew
brew install postgresql@15
brew services start postgresql@15

# Or install latest version
brew install postgresql
brew services start postgresql

# Using PostgreSQL.app (GUI)
# Download from https://postgresapp.com/

# Verify installation
psql --version

# Create initial database
initdb /usr/local/var/postgres

# Start PostgreSQL
pg_ctl -D /usr/local/var/postgres start

# Connect to default database
psql postgres
```

### Linux Installation

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql

# CentOS/RHEL
sudo yum install postgresql-server postgresql-contrib
sudo postgresql-setup initdb
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Arch Linux
sudo pacman -S postgresql
sudo -u postgres initdb -D /var/lib/postgres/data
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Switch to postgres user
sudo -i -u postgres
psql
```

### Docker Installation

```bash
# Run PostgreSQL container
docker run --name postgres-db \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -d postgres:15

# With persistent data
docker run --name postgres-db \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydb \
  -v postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  -d postgres:15

# Connect to container
docker exec -it postgres-db psql -U postgres -d mydb

# Using Docker Compose
cat > docker-compose.yml << EOF
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

volumes:
  postgres_data:
EOF

docker-compose up -d
```

### Configuration

```bash
# Find configuration files
psql -c "SHOW config_file;"
psql -c "SHOW data_directory;"
psql -c "SHOW hba_file;"

# Common locations
# macOS (Homebrew): /usr/local/var/postgres/
# Linux: /etc/postgresql/15/main/ or /var/lib/pgsql/15/data/
# Docker: /var/lib/postgresql/data/

# Edit postgresql.conf
sudo nano /etc/postgresql/15/main/postgresql.conf

# Edit pg_hba.conf for authentication
sudo nano /etc/postgresql/15/main/pg_hba.conf

# Reload configuration
sudo systemctl reload postgresql
# or
psql -c "SELECT pg_reload_conf();"
```

## Basic Commands

### psql Commands

```bash
# Connect to PostgreSQL
psql -h localhost -p 5432 -U username -d database

# Connect with connection string
psql "postgresql://username:password@localhost:5432/database"

# Connect to local database as current user
psql database_name

# Meta-commands in psql
\l                    # List databases
\c database_name      # Connect to database
\dt                   # List tables
\d table_name         # Describe table
\du                   # List users/roles
\dn                   # List schemas
\df                   # List functions
\dv                   # List views
\di                   # List indexes
\ds                   # List sequences
\dp                   # List permissions
\timing               # Enable/disable timing
\x                    # Toggle expanded output
\q                    # Quit
\h SQL_COMMAND        # Help on SQL command
\?                    # Help on psql commands

# Execute SQL file
\i /path/to/file.sql

# Output to file
\o /path/to/output.txt
\o                    # Return to stdout

# Copy results to CSV
\copy (SELECT * FROM table) TO '/path/to/file.csv' WITH CSV HEADER;
```

### Command Line Tools

```bash
# Create database
createdb mydb
createdb -U username -h hostname mydb

# Drop database
dropdb mydb

# Create user
createuser myuser
createuser --interactive myuser

# Drop user
dropuser myuser

# Backup database
pg_dump mydb > backup.sql
pg_dump -U username -h hostname mydb > backup.sql
pg_dump -d mydb -f backup.sql --clean --if-exists

# Backup in custom format
pg_dump -Fc mydb > backup.dump

# Backup all databases
pg_dumpall > all_databases.sql

# Restore database
psql mydb < backup.sql
pg_restore -d mydb backup.dump

# Restore with custom options
pg_restore -d mydb --clean --if-exists backup.dump

# Vacuum and analyze
vacuumdb mydb
vacuumdb --analyze mydb
vacuumdb --all --analyze

# Reindex
reindexdb mydb
reindexdb --all
```

## SQL Fundamentals

### Data Types

```sql
-- Numeric Types
SMALLINT                    -- 2 bytes, -32768 to 32767
INTEGER, INT                -- 4 bytes, -2147483648 to 2147483647
BIGINT                      -- 8 bytes, large range
DECIMAL(p,s), NUMERIC(p,s) -- Variable, exact decimal
REAL                        -- 4 bytes, 6 decimal digits precision
DOUBLE PRECISION            -- 8 bytes, 15 decimal digits precision
SERIAL                      -- Auto-incrementing integer
BIGSERIAL                   -- Auto-incrementing bigint

-- Character Types
CHAR(n), CHARACTER(n)      -- Fixed-length string
VARCHAR(n), CHARACTER VARYING(n) -- Variable-length string
TEXT                        -- Variable-length string, no limit

-- Date/Time Types
DATE                        -- Date (no time)
TIME                        -- Time (no date)
TIMESTAMP                   -- Date and time
TIMESTAMPTZ                 -- Date and time with timezone
INTERVAL                    -- Time interval

-- Boolean
BOOLEAN, BOOL              -- TRUE, FALSE, NULL

-- Binary Data
BYTEA                       -- Binary data

-- JSON Types
JSON                        -- JSON data, stored as text
JSONB                       -- JSON data, stored in binary format

-- Arrays
INTEGER[]                   -- Array of integers
TEXT[]                      -- Array of text

-- UUID
UUID                        -- Universally unique identifier

-- Geometric Types
POINT                       -- Point in 2D plane
LINE                        -- Infinite line
LSEG                        -- Line segment
BOX                         -- Rectangle
PATH                        -- Geometric path
POLYGON                     -- Polygon
CIRCLE                      -- Circle

-- Network Address Types
CIDR                        -- IPv4 or IPv6 network
INET                        -- IPv4 or IPv6 address
MACADDR                     -- MAC address
```

### Database Operations

```sql
-- Create database
CREATE DATABASE mydb
    WITH OWNER = myuser
    ENCODING = 'UTF8'
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8'
    TEMPLATE = template0;

-- Drop database
DROP DATABASE IF EXISTS mydb;

-- List databases
SELECT datname FROM pg_database;

-- Get database size
SELECT pg_size_pretty(pg_database_size('mydb'));

-- Connect to database
\c mydb
```

### Schema Operations

```sql
-- Create schema
CREATE SCHEMA myschema;
CREATE SCHEMA IF NOT EXISTS myschema;

-- Drop schema
DROP SCHEMA myschema;
DROP SCHEMA myschema CASCADE;  -- Drop with all objects

-- List schemas
SELECT schema_name FROM information_schema.schemata;

-- Set search path
SET search_path TO myschema, public;
SHOW search_path;
```

### Table Operations

```sql
-- Create table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    profile JSONB,
    tags TEXT[]
);

-- Create table with constraints
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    order_date DATE NOT NULL DEFAULT CURRENT_DATE,
    total_amount DECIMAL(10,2) CHECK (total_amount >= 0),
    status VARCHAR(20) DEFAULT 'pending'
        CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))
);

-- Create table from query
CREATE TABLE user_summary AS
SELECT 
    username,
    email,
    created_at
FROM users
WHERE is_active = TRUE;

-- Create temporary table
CREATE TEMP TABLE temp_data (
    id SERIAL,
    value TEXT
);

-- Alter table
ALTER TABLE users ADD COLUMN last_login TIMESTAMP;
ALTER TABLE users DROP COLUMN tags;
ALTER TABLE users ALTER COLUMN username TYPE VARCHAR(100);
ALTER TABLE users RENAME COLUMN username TO user_name;
ALTER TABLE users RENAME TO app_users;

-- Add constraints
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE (email);
ALTER TABLE users ADD CONSTRAINT check_email 
    CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Drop table
DROP TABLE IF EXISTS users CASCADE;

-- Truncate table
TRUNCATE TABLE users RESTART IDENTITY CASCADE;

-- Get table info
\d+ users
SELECT column_name, data_type, is_nullable 
FROM information_schema.columns 
WHERE table_name = 'users';
```

## CRUD Operations

### INSERT

```sql
-- Basic insert
INSERT INTO users (username, email, password_hash)
VALUES ('john_doe', 'john@example.com', 'hashed_password');

-- Insert multiple rows
INSERT INTO users (username, email, password_hash) VALUES
    ('alice', 'alice@example.com', 'hash1'),
    ('bob', 'bob@example.com', 'hash2'),
    ('charlie', 'charlie@example.com', 'hash3');

-- Insert with returning
INSERT INTO users (username, email, password_hash)
VALUES ('jane', 'jane@example.com', 'hash')
RETURNING id, created_at;

-- Insert from select
INSERT INTO user_backup (username, email)
SELECT username, email FROM users WHERE is_active = FALSE;

-- Insert with conflict handling (UPSERT)
INSERT INTO users (username, email, password_hash)
VALUES ('john_doe', 'john.doe@example.com', 'new_hash')
ON CONFLICT (username) 
DO UPDATE SET email = EXCLUDED.email, password_hash = EXCLUDED.password_hash;

-- Insert JSON data
INSERT INTO users (username, email, password_hash, profile)
VALUES ('user1', 'user1@example.com', 'hash', 
        '{"age": 30, "city": "New York", "interests": ["reading", "coding"]}');
```

### SELECT

```sql
-- Basic select
SELECT * FROM users;
SELECT username, email FROM users;

-- With conditions
SELECT * FROM users WHERE is_active = TRUE;
SELECT * FROM users WHERE created_at >= '2023-01-01';
SELECT * FROM users WHERE username LIKE 'john%';
SELECT * FROM users WHERE username ILIKE 'JOHN%';  -- Case insensitive

-- With ordering
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users ORDER BY username ASC, created_at DESC;

-- With limit and offset
SELECT * FROM users ORDER BY id LIMIT 10;
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

-- Aggregate functions
SELECT COUNT(*) FROM users;
SELECT COUNT(DISTINCT email) FROM users;
SELECT AVG(total_amount) FROM orders;
SELECT MIN(created_at), MAX(created_at) FROM users;

-- Group by
SELECT status, COUNT(*) 
FROM orders 
GROUP BY status;

SELECT DATE(created_at) as signup_date, COUNT(*) as signups
FROM users 
GROUP BY DATE(created_at)
ORDER BY signup_date;

-- Having clause
SELECT status, COUNT(*) 
FROM orders 
GROUP BY status 
HAVING COUNT(*) > 10;

-- Subqueries
SELECT username FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE total_amount > 100);

SELECT username, 
       (SELECT COUNT(*) FROM orders WHERE user_id = users.id) as order_count
FROM users;

-- Window functions
SELECT username, created_at,
       ROW_NUMBER() OVER (ORDER BY created_at) as signup_order,
       RANK() OVER (ORDER BY created_at) as signup_rank
FROM users;

SELECT username,
       LAG(created_at) OVER (ORDER BY created_at) as prev_signup,
       LEAD(created_at) OVER (ORDER BY created_at) as next_signup
FROM users;
```

### UPDATE

```sql
-- Basic update
UPDATE users SET last_login = CURRENT_TIMESTAMP WHERE id = 1;

-- Update multiple columns
UPDATE users 
SET email = 'newemail@example.com', 
    is_active = FALSE 
WHERE username = 'john_doe';

-- Update with conditions
UPDATE users 
SET is_active = FALSE 
WHERE last_login < CURRENT_DATE - INTERVAL '1 year';

-- Update with joins
UPDATE orders 
SET status = 'shipped' 
FROM users 
WHERE orders.user_id = users.id 
  AND users.username = 'john_doe';

-- Update with returning
UPDATE users 
SET last_login = CURRENT_TIMESTAMP 
WHERE id = 1 
RETURNING username, last_login;

-- Update JSON data
UPDATE users 
SET profile = profile || '{"last_updated": "2023-12-01"}'
WHERE id = 1;

UPDATE users 
SET profile = jsonb_set(profile, '{age}', '31')
WHERE id = 1;
```

### DELETE

```sql
-- Basic delete
DELETE FROM users WHERE id = 1;

-- Delete with conditions
DELETE FROM users WHERE is_active = FALSE;
DELETE FROM users WHERE created_at < '2022-01-01';

-- Delete with joins
DELETE FROM orders 
USING users 
WHERE orders.user_id = users.id 
  AND users.is_active = FALSE;

-- Delete with returning
DELETE FROM users 
WHERE is_active = FALSE 
RETURNING username, email;

-- Delete all rows
DELETE FROM temp_table;
```

## Joins and Relationships

### Join Types

```sql
-- Inner Join
SELECT u.username, o.total_amount, o.order_date
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- Left Join
SELECT u.username, o.total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- Right Join
SELECT u.username, o.total_amount
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- Full Outer Join
SELECT u.username, o.total_amount
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- Cross Join
SELECT u.username, p.product_name
FROM users u
CROSS JOIN products p;

-- Self Join
SELECT e1.name as employee, e2.name as manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;

-- Multiple Joins
SELECT u.username, o.total_amount, oi.quantity, p.product_name
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

### Common Table Expressions (CTEs)

```sql
-- Basic CTE
WITH active_users AS (
    SELECT id, username, email
    FROM users
    WHERE is_active = TRUE
)
SELECT au.username, COUNT(o.id) as order_count
FROM active_users au
LEFT JOIN orders o ON au.id = o.user_id
GROUP BY au.username;

-- Recursive CTE
WITH RECURSIVE employee_hierarchy AS (
    -- Base case
    SELECT id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT name, level FROM employee_hierarchy ORDER BY level, name;

-- Multiple CTEs
WITH 
user_stats AS (
    SELECT user_id, COUNT(*) as order_count, SUM(total_amount) as total_spent
    FROM orders
    GROUP BY user_id
),
high_value_users AS (
    SELECT user_id
    FROM user_stats
    WHERE total_spent > 1000
)
SELECT u.username, us.order_count, us.total_spent
FROM users u
JOIN user_stats us ON u.id = us.user_id
JOIN high_value_users hvu ON u.id = hvu.user_id;
```

## Indexes and Performance

### Index Types

```sql
-- B-tree index (default)
CREATE INDEX idx_users_email ON users (email);
CREATE UNIQUE INDEX idx_users_username ON users (username);

-- Composite index
CREATE INDEX idx_orders_user_date ON orders (user_id, order_date);

-- Partial index
CREATE INDEX idx_active_users ON users (username) WHERE is_active = TRUE;

-- Expression index
CREATE INDEX idx_users_lower_email ON users (LOWER(email));

-- Hash index
CREATE INDEX idx_users_id_hash ON users USING HASH (id);

-- GIN index for arrays and JSON
CREATE INDEX idx_users_tags ON users USING GIN (tags);
CREATE INDEX idx_users_profile ON users USING GIN (profile);

-- GiST index for geometric data
CREATE INDEX idx_locations_point ON locations USING GIST (coordinates);

-- Full-text search index
CREATE INDEX idx_posts_content ON posts USING GIN (to_tsvector('english', content));

-- List indexes
\di
SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'users';

-- Drop index
DROP INDEX idx_users_email;
DROP INDEX CONCURRENTLY idx_users_email;  -- Non-blocking

-- Reindex
REINDEX INDEX idx_users_email;
REINDEX TABLE users;
REINDEX DATABASE mydb;
```

### Query Performance

```sql
-- Explain query plan
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- Explain with costs and timing
EXPLAIN (ANALYZE, BUFFERS) 
SELECT u.username, COUNT(o.id) 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id 
GROUP BY u.username;

-- Analyze table statistics
ANALYZE users;
ANALYZE;  -- All tables

-- Vacuum table
VACUUM users;
VACUUM FULL users;  -- More thorough but locks table
VACUUM ANALYZE users;  -- Vacuum and analyze

-- Auto-vacuum settings
SHOW autovacuum;
SELECT * FROM pg_stat_user_tables WHERE relname = 'users';

-- Index usage statistics
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE tablename = 'users';

-- Table size information
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## Advanced SQL Features

### JSON Operations

```sql
-- JSON operators
SELECT profile->>'name' as name FROM users;  -- Get as text
SELECT profile->'address'->>'city' as city FROM users;  -- Nested access
SELECT profile #> '{address,city}' as city FROM users;  -- Path access

-- JSON functions
SELECT json_extract_path_text(profile, 'name') FROM users;
SELECT jsonb_array_elements_text(profile->'interests') FROM users;

-- JSON aggregation
SELECT jsonb_agg(profile->'name') as all_names FROM users;
SELECT jsonb_object_agg(username, profile->'name') FROM users;

-- JSON containment
SELECT * FROM users WHERE profile @> '{"age": 30}';
SELECT * FROM users WHERE profile ? 'age';  -- Key exists
SELECT * FROM users WHERE profile ?| array['age', 'city'];  -- Any key exists
SELECT * FROM users WHERE profile ?& array['age', 'city'];  -- All keys exist

-- Update JSON
UPDATE users SET profile = profile || '{"updated": true}' WHERE id = 1;
UPDATE users SET profile = profile - 'age' WHERE id = 1;  -- Remove key
```

### Array Operations

```sql
-- Array operations
SELECT * FROM users WHERE 'reading' = ANY(tags);
SELECT * FROM users WHERE tags @> ARRAY['reading', 'coding'];
SELECT * FROM users WHERE tags && ARRAY['sports', 'music'];  -- Overlap

-- Array functions
SELECT array_length(tags, 1) FROM users;
SELECT unnest(tags) as tag FROM users;
SELECT array_agg(username) FROM users;
SELECT array_cat(ARRAY['a','b'], ARRAY['c','d']);

-- Array aggregation
SELECT user_id, array_agg(product_name) as products
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
GROUP BY user_id;
```

### Full-Text Search

```sql
-- Basic text search
SELECT * FROM posts 
WHERE to_tsvector('english', title || ' ' || content) @@ to_tsquery('postgresql');

-- Ranked search results
SELECT title, 
       ts_rank(to_tsvector('english', title || ' ' || content), 
               to_tsquery('postgresql & database')) as rank
FROM posts
WHERE to_tsvector('english', title || ' ' || content) @@ to_tsquery('postgresql & database')
ORDER BY rank DESC;

-- Search with highlights
SELECT title,
       ts_headline('english', content, to_tsquery('postgresql')) as highlight
FROM posts
WHERE to_tsvector('english', content) @@ to_tsquery('postgresql');

-- Create text search configuration
CREATE TEXT SEARCH CONFIGURATION my_config (COPY = english);
ALTER TEXT SEARCH CONFIGURATION my_config
ALTER MAPPING FOR asciiword WITH english_stem, simple;
```

### Window Functions

```sql
-- Ranking functions
SELECT username, total_spent,
       ROW_NUMBER() OVER (ORDER BY total_spent DESC) as row_num,
       RANK() OVER (ORDER BY total_spent DESC) as rank,
       DENSE_RANK() OVER (ORDER BY total_spent DESC) as dense_rank,
       NTILE(4) OVER (ORDER BY total_spent DESC) as quartile
FROM user_spending;

-- Offset functions
SELECT username, order_date, total_amount,
       LAG(total_amount) OVER (PARTITION BY user_id ORDER BY order_date) as prev_amount,
       LEAD(total_amount) OVER (PARTITION BY user_id ORDER BY order_date) as next_amount,
       FIRST_VALUE(total_amount) OVER (PARTITION BY user_id ORDER BY order_date) as first_amount,
       LAST_VALUE(total_amount) OVER (PARTITION BY user_id ORDER BY order_date 
                                      ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as last_amount
FROM orders;

-- Aggregate window functions
SELECT username, order_date, total_amount,
       SUM(total_amount) OVER (PARTITION BY user_id ORDER BY order_date) as running_total,
       AVG(total_amount) OVER (PARTITION BY user_id ORDER BY order_date 
                               ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg
FROM orders;
```

## User Management and Security

### User and Role Management

```sql
-- Create users
CREATE USER myuser WITH PASSWORD 'mypassword';
CREATE USER readonly_user WITH PASSWORD 'password';
CREATE USER app_user WITH 
    PASSWORD 'secure_password' 
    CREATEDB 
    VALID UNTIL '2024-12-31';

-- Create roles
CREATE ROLE app_role;
CREATE ROLE readonly_role;
CREATE ROLE admin_role WITH LOGIN PASSWORD 'admin_pass' CREATEDB CREATEROLE;

-- Grant role to user
GRANT app_role TO app_user;
GRANT readonly_role TO readonly_user;

-- Alter user
ALTER USER myuser WITH PASSWORD 'newpassword';
ALTER USER myuser CREATEDB;
ALTER USER myuser VALID UNTIL '2025-01-01';

-- Drop user
DROP USER IF EXISTS myuser;
DROP ROLE IF EXISTS app_role;

-- List users and roles
\du
SELECT rolname, rolsuper, rolcreatedb, rolcreaterole FROM pg_roles;
```

### Permissions and Privileges

```sql
-- Grant database privileges
GRANT CONNECT ON DATABASE mydb TO app_user;
GRANT CREATE ON DATABASE mydb TO app_user;

-- Grant schema privileges
GRANT USAGE ON SCHEMA public TO app_user;
GRANT CREATE ON SCHEMA public TO app_user;
GRANT ALL ON SCHEMA myschema TO app_user;

-- Grant table privileges
GRANT SELECT ON users TO readonly_user;
GRANT SELECT, INSERT, UPDATE ON orders TO app_user;
GRANT ALL PRIVILEGES ON users TO app_user;

-- Grant privileges on all tables in schema
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO app_user;

-- Grant privileges on future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO readonly_user;

-- Grant sequence privileges
GRANT USAGE ON SEQUENCE users_id_seq TO app_user;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO app_user;

-- Grant function privileges
GRANT EXECUTE ON FUNCTION my_function() TO app_user;

-- Revoke privileges
REVOKE INSERT ON users FROM app_user;
REVOKE ALL PRIVILEGES ON users FROM app_user;

-- Check privileges
\dp users
SELECT grantee, privilege_type 
FROM information_schema.role_table_grants 
WHERE table_name = 'users';
```

### Row Level Security (RLS)

```sql
-- Enable RLS on table
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Create policy
CREATE POLICY user_orders ON orders
    FOR ALL TO app_role
    USING (user_id = current_setting('app.current_user_id')::int);

-- Create policy for different operations
CREATE POLICY orders_select ON orders
    FOR SELECT TO app_role
    USING (user_id = current_setting('app.current_user_id')::int);

CREATE POLICY orders_insert ON orders
    FOR INSERT TO app_role
    WITH CHECK (user_id = current_setting('app.current_user_id')::int);

-- Force RLS for table owner
ALTER TABLE orders FORCE ROW LEVEL SECURITY;

-- Drop policy
DROP POLICY user_orders ON orders;

-- Disable RLS
ALTER TABLE orders DISABLE ROW LEVEL SECURITY;

-- Set session variable for RLS
SET app.current_user_id = '123';
```

## Backup and Recovery

### Backup Strategies

```bash
# SQL dump backup
pg_dump mydb > mydb_backup.sql
pg_dump -U username -h hostname -p 5432 mydb > mydb_backup.sql

# Custom format backup (recommended)
pg_dump -Fc mydb > mydb_backup.dump

# Directory format backup
pg_dump -Fd mydb -f mydb_backup_dir

# Backup specific tables
pg_dump -t users -t orders mydb > tables_backup.sql

# Backup with compression
pg_dump mydb | gzip > mydb_backup.sql.gz

# Backup all databases
pg_dumpall > all_databases.sql

# Backup roles and global objects only
pg_dumpall --roles-only > roles_backup.sql
pg_dumpall --globals-only > globals_backup.sql

# Parallel backup (faster for large databases)
pg_dump -j 4 -Fd mydb -f mydb_backup_dir

# Backup with specific options
pg_dump --clean --if-exists --create --inserts mydb > mydb_backup.sql
```

### Restore Operations

```bash
# Restore from SQL dump
psql mydb < mydb_backup.sql

# Restore from custom format
pg_restore -d mydb mydb_backup.dump

# Restore with options
pg_restore --clean --if-exists -d mydb mydb_backup.dump

# Restore specific tables
pg_restore -t users -t orders -d mydb mydb_backup.dump

# Parallel restore
pg_restore -j 4 -d mydb mydb_backup_dir

# Restore to different database
creatdb newdb
pg_restore -d newdb mydb_backup.dump

# List contents of backup file
pg_restore --list mydb_backup.dump

# Restore specific items by OID
pg_restore --use-list=restore_list.txt -d mydb mydb_backup.dump
```

### Point-in-Time Recovery (PITR)

```bash
# Configure WAL archiving in postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /path/to/archive/%f'

# Take base backup
pg_basebackup -D /path/to/backup -Ft -z -P

# Restore from base backup
tar -xzf /path/to/backup/base.tar.gz -C /var/lib/postgresql/data

# Create recovery.conf for PITR
echo "restore_command = 'cp /path/to/archive/%f %p'" > recovery.conf
echo "recovery_target_time = '2023-12-01 12:00:00'" >> recovery.conf
```

## Transactions and Concurrency

### Transaction Control

```sql
-- Basic transaction
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Transaction with rollback
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Something goes wrong
ROLLBACK;

-- Savepoints
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT sp1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Problem with second update
ROLLBACK TO sp1;
-- First update is still there
COMMIT;

-- Transaction isolation levels
BEGIN ISOLATION LEVEL READ UNCOMMITTED;
BEGIN ISOLATION LEVEL READ COMMITTED;  -- Default
BEGIN ISOLATION LEVEL REPEATABLE READ;
BEGIN ISOLATION LEVEL SERIALIZABLE;

-- Set default isolation level
SET default_transaction_isolation = 'repeatable read';

-- Read-only transaction
BEGIN READ ONLY;
SELECT * FROM users;
COMMIT;
```

### Locking

```sql
-- Table-level locks
LOCK TABLE users IN ACCESS EXCLUSIVE MODE;
LOCK TABLE users IN SHARE MODE;

-- Row-level locks
SELECT * FROM users WHERE id = 1 FOR UPDATE;
SELECT * FROM users WHERE id = 1 FOR SHARE;
SELECT * FROM users WHERE id = 1 FOR UPDATE NOWAIT;
SELECT * FROM users WHERE id = 1 FOR UPDATE SKIP LOCKED;

-- Advisory locks
SELECT pg_advisory_lock(12345);
SELECT pg_advisory_unlock(12345);
SELECT pg_try_advisory_lock(12345);

-- Check locks
SELECT * FROM pg_locks;
SELECT 
    l.locktype,
    l.mode,
    l.granted,
    l.pid,
    a.query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid;
```

## Functions and Stored Procedures

### PL/pgSQL Functions

```sql
-- Simple function
CREATE OR REPLACE FUNCTION get_user_count()
RETURNS INTEGER AS $$
BEGIN
    RETURN (SELECT COUNT(*) FROM users);
END;
$$ LANGUAGE plpgsql;

-- Function with parameters
CREATE OR REPLACE FUNCTION get_user_orders(user_id_param INTEGER)
RETURNS TABLE(order_id INTEGER, order_date DATE, total_amount DECIMAL) AS $$
BEGIN
    RETURN QUERY
    SELECT id, order_date, total_amount
    FROM orders
    WHERE user_id = user_id_param;
END;
$$ LANGUAGE plpgsql;

-- Function with conditional logic
CREATE OR REPLACE FUNCTION calculate_discount(order_amount DECIMAL)
RETURNS DECIMAL AS $$
DECLARE
    discount_rate DECIMAL := 0;
BEGIN
    IF order_amount >= 1000 THEN
        discount_rate := 0.10;  -- 10% discount
    ELSIF order_amount >= 500 THEN
        discount_rate := 0.05;  -- 5% discount
    ELSE
        discount_rate := 0;     -- No discount
    END IF;
    
    RETURN order_amount * discount_rate;
END;
$$ LANGUAGE plpgsql;

-- Function with loops
CREATE OR REPLACE FUNCTION generate_series_custom(start_val INTEGER, end_val INTEGER)
RETURNS TABLE(value INTEGER) AS $$
DECLARE
    i INTEGER;
BEGIN
    FOR i IN start_val..end_val LOOP
        value := i;
        RETURN NEXT;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- Function with exception handling
CREATE OR REPLACE FUNCTION safe_divide(a NUMERIC, b NUMERIC)
RETURNS NUMERIC AS $$
BEGIN
    RETURN a / b;
EXCEPTION
    WHEN division_by_zero THEN
        RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Drop function
DROP FUNCTION IF EXISTS get_user_count();

-- List functions
\df
SELECT routine_name, routine_type 
FROM information_schema.routines 
WHERE routine_schema = 'public';
```

### Triggers

```sql
-- Create trigger function
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.modified_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER update_users_modified
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_column();

-- Audit trigger function
CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, operation, old_data, timestamp)
        VALUES (TG_TABLE_NAME, TG_OP, row_to_json(OLD), CURRENT_TIMESTAMP);
        RETURN OLD;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation, old_data, new_data, timestamp)
        VALUES (TG_TABLE_NAME, TG_OP, row_to_json(OLD), row_to_json(NEW), CURRENT_TIMESTAMP);
        RETURN NEW;
    ELSIF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, operation, new_data, timestamp)
        VALUES (TG_TABLE_NAME, TG_OP, row_to_json(NEW), CURRENT_TIMESTAMP);
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Create audit triggers
CREATE TRIGGER users_audit
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW
    EXECUTE FUNCTION audit_trigger();

-- Drop trigger
DROP TRIGGER IF EXISTS update_users_modified ON users;

-- List triggers
\dS users
SELECT trigger_name, event_manipulation, event_object_table
FROM information_schema.triggers
WHERE event_object_schema = 'public';
```

## Monitoring and Maintenance

### Database Statistics

```sql
-- Database size and statistics
SELECT 
    datname,
    pg_size_pretty(pg_database_size(datname)) as size,
    numbackends as connections
FROM pg_stat_database
WHERE datname NOT IN ('template0', 'template1', 'postgres');

-- Table statistics
SELECT 
    schemaname,
    tablename,
    n_tup_ins as inserts,
    n_tup_upd as updates,
    n_tup_del as deletes,
    n_live_tup as live_rows,
    n_dead_tup as dead_rows,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables;

-- Index usage statistics
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_tup_read,
    idx_tup_fetch,
    idx_scan
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Slow queries
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

### Connection Monitoring

```sql
-- Active connections
SELECT 
    pid,
    usename,
    datname,
    client_addr,
    state,
    query_start,
    state_change,
    query
FROM pg_stat_activity
WHERE state = 'active';

-- Connection counts by database
SELECT 
    datname,
    count(*) as connections
FROM pg_stat_activity
GROUP BY datname;

-- Long running queries
SELECT 
    pid,
    usename,
    datname,
    query_start,
    now() - query_start as duration,
    query
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '5 minutes';

-- Kill connection
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;

-- Cancel query
SELECT pg_cancel_backend(pid) FROM pg_stat_activity WHERE pid = 12345;
```

### Maintenance Tasks

```sql
-- Manual vacuum and analyze
VACUUM VERBOSE users;
VACUUM FULL users;  -- Reclaims more space but locks table
ANALYZE users;
VACUUM ANALYZE users;

-- Reindex
REINDEX INDEX idx_users_email;
REINDEX TABLE users;
REINDEX DATABASE mydb;

-- Check for bloat
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - 
                   pg_relation_size(schemaname||'.'||tablename)) as index_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Autovacuum settings
SHOW autovacuum;
SELECT name, setting FROM pg_settings WHERE name LIKE '%autovacuum%';
```

## Configuration and Tuning

### Key Configuration Parameters

```sql
-- View current settings
SELECT name, setting, unit, context FROM pg_settings 
WHERE name IN (
    'shared_buffers',
    'work_mem',
    'maintenance_work_mem',
    'effective_cache_size',
    'max_connections',
    'checkpoint_completion_target',
    'wal_buffers',
    'random_page_cost'
);

-- Memory settings
SET shared_buffers = '256MB';           -- Shared buffer cache
SET work_mem = '4MB';                   -- Memory for sorts/joins
SET maintenance_work_mem = '64MB';      -- Memory for vacuum/reindex
SET effective_cache_size = '1GB';       -- OS cache size estimate

-- Connection settings
SET max_connections = 100;              -- Maximum connections
SET superuser_reserved_connections = 3; -- Reserved for superusers

-- Checkpoint settings
SET checkpoint_completion_target = 0.9; -- Checkpoint spread
SET checkpoint_timeout = '5min';        -- Checkpoint frequency

-- WAL settings
SET wal_buffers = '16MB';               -- WAL buffer size
SET wal_level = 'replica';              -- WAL level for replication

-- Query planner settings
SET random_page_cost = 1.1;             -- SSD setting (default 4.0 for HDD)
SET seq_page_cost = 1.0;                -- Sequential page cost
SET cpu_tuple_cost = 0.01;              -- CPU cost per tuple

-- Logging settings
SET log_statement = 'all';              -- Log all statements
SET log_min_duration_statement = 1000;  -- Log slow queries (1 sec)
SET log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h ';

-- Auto vacuum settings
SET autovacuum = on;
SET autovacuum_vacuum_scale_factor = 0.1;
SET autovacuum_analyze_scale_factor = 0.05;
```

### Performance Tuning

```sql
-- Enable extensions for monitoring
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
CREATE EXTENSION IF NOT EXISTS pg_buffercache;
CREATE EXTENSION IF NOT EXISTS pgstattuple;

-- Buffer cache hit ratio (should be > 99%)
SELECT 
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) * 100 
    AS buffer_hit_ratio
FROM pg_statio_user_tables;

-- Index hit ratio (should be > 99%)
SELECT 
    sum(idx_blks_hit) / (sum(idx_blks_hit) + sum(idx_blks_read)) * 100 
    AS index_hit_ratio
FROM pg_statio_user_indexes;

-- Unused indexes
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY schemaname, tablename, indexname;

-- Table bloat estimation
SELECT 
    current_database(),
    schemaname,
    tablename,
    /*reltuples::bigint, relpages::bigint, otta,*/
    ROUND(CASE WHEN otta=0 OR sml.relpages=0 OR sml.relpages=otta THEN 0.0 
          ELSE sml.relpages/otta::numeric END,1) AS tbloat,
    CASE WHEN relpages < otta THEN 0 
         ELSE relpages::bigint - otta END AS wastedpages,
    CASE WHEN relpages < otta THEN 0 
         ELSE bs*(sml.relpages-otta)::bigint END AS wastedbytes,
    CASE WHEN relpages < otta THEN 0 
         ELSE (bs*(relpages-otta))::bigint END AS wastedsize,
    iname, /*ituples::bigint, ipages::bigint, iotta,*/
    ROUND(CASE WHEN iotta=0 OR ipages=0 OR ipages=iotta THEN 0.0 
          ELSE ipages/iotta::numeric END,1) AS ibloat,
    CASE WHEN ipages < iotta THEN 0 
         ELSE ipages::bigint - iotta END AS wastedipages,
    CASE WHEN ipages < iotta THEN 0 
         ELSE bs*(ipages-iotta) END AS wastedibytes,
    CASE WHEN ipages < iotta THEN 0 
         ELSE (bs*(ipages-iotta))::bigint END AS wastedisize
FROM (
    SELECT 
        schemaname, tablename, cc.reltuples, cc.relpages, bs,
        CEIL((cc.reltuples*((datahdr+ma-
            (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)) AS otta,
        COALESCE(c2.relname,'?') AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages,
        COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta
    FROM (
        SELECT 
            ma,bs,schemaname,tablename,
            (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr,
            (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2
        FROM (
            SELECT 
                schemaname, tablename, hdr, ma, bs,
                SUM((1-null_frac)*avg_width) AS datawidth,
                MAX(null_frac) AS maxfracsum,
                hdr+(
                    SELECT 1+count(*)/8
                    FROM pg_stats s2
                    WHERE null_frac<>0 AND s2.schemaname = s.schemaname AND s2.tablename = s.tablename
                ) AS nullhdr
            FROM pg_stats s, (
                SELECT 
                    (SELECT current_setting('block_size')::numeric) AS bs,
                    CASE WHEN substring(v,12,3) IN ('8.0','8.1','8.2') THEN 27 ELSE 23 END AS hdr,
                    CASE WHEN v ~ 'mingw32' THEN 8 ELSE 4 END AS ma
                FROM (SELECT version() AS v) AS foo
            ) AS constants
            GROUP BY 1,2,3,4,5
        ) AS foo
    ) AS rs
    JOIN pg_class cc ON cc.relname = rs.tablename
    JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname = rs.schemaname AND nn.nspname <> 'information_schema'
    LEFT JOIN pg_index i ON indrelid = cc.oid
    LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid
) AS sml
WHERE sml.relpages - otta > 0 OR ipages - iotta > 10
ORDER BY wastedbytes DESC, wastedibytes DESC;
```

## Common Patterns and Best Practices

### Naming Conventions

```sql
-- Table names: plural, lowercase, underscores
CREATE TABLE users (...);           -- Good
CREATE TABLE user_profiles (...);   -- Good
CREATE TABLE User (...);            -- Avoid
CREATE TABLE userProfile (...);     -- Avoid

-- Column names: lowercase, underscores
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,     -- Good
    first_name VARCHAR(50),         -- Good
    userId INTEGER,                 -- Avoid
    FirstName VARCHAR(50)           -- Avoid
);

-- Index names: descriptive
CREATE INDEX idx_users_email ON users (email);                    -- Good
CREATE INDEX idx_orders_user_id_status ON orders (user_id, status); -- Good
CREATE INDEX ix1 ON users (email);                                -- Avoid

-- Constraint names: descriptive
ALTER TABLE orders ADD CONSTRAINT fk_orders_user_id 
    FOREIGN KEY (user_id) REFERENCES users(id);
ALTER TABLE users ADD CONSTRAINT chk_users_email_format 
    CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
```

### Security Best Practices

```sql
-- Use parameterized queries (application level)
-- NEVER do this: "SELECT * FROM users WHERE id = " + user_input
-- DO this: "SELECT * FROM users WHERE id = $1" with parameter binding

-- Principle of least privilege
CREATE ROLE app_read_only;
GRANT CONNECT ON DATABASE myapp TO app_read_only;
GRANT USAGE ON SCHEMA public TO app_read_only;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_read_only;

-- Use different users for different purposes
CREATE USER app_reader WITH PASSWORD 'read_password';
CREATE USER app_writer WITH PASSWORD 'write_password';
GRANT app_read_only TO app_reader;
GRANT app_full_access TO app_writer;

-- Enable SSL
-- In postgresql.conf:
-- ssl = on
-- ssl_cert_file = 'server.crt'
-- ssl_key_file = 'server.key'

-- Audit important operations
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name VARCHAR(50),
    operation VARCHAR(10),
    user_name VARCHAR(50) DEFAULT current_user,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    old_data JSONB,
    new_data JSONB
);
```

### Performance Best Practices

```sql
-- Always define primary keys
CREATE TABLE users (
    id SERIAL PRIMARY KEY,  -- Always have a primary key
    -- other columns
);

-- Add appropriate indexes
CREATE INDEX idx_users_email ON users (email);           -- For lookups
CREATE INDEX idx_orders_created_at ON orders (created_at); -- For date ranges
CREATE INDEX idx_orders_status ON orders (status) WHERE status != 'completed'; -- Partial index

-- Use appropriate data types
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    price DECIMAL(10,2) NOT NULL,  -- Use DECIMAL for money
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,  -- Use TIMESTAMPTZ
    is_active BOOLEAN DEFAULT TRUE  -- Use BOOLEAN, not CHAR(1)
);

-- Normalize appropriately
-- Don't over-normalize (avoid too many joins)
-- Don't under-normalize (avoid data duplication)

-- Use LIMIT for large result sets
SELECT * FROM large_table ORDER BY created_at DESC LIMIT 100;

-- Use EXISTS instead of IN for subqueries when appropriate
SELECT * FROM users u 
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- Avoid SELECT * in production code
SELECT id, username, email FROM users;  -- Good
SELECT * FROM users;                    -- Avoid in production
```

---

*For more detailed information, visit the [official PostgreSQL documentation](https://www.postgresql.org/docs/)*
