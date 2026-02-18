# Database Patterns - Expert Reference

## Table of Contents

1. [Schema Design Principles](#schema-design-principles)
2. [Migration Strategies](#migration-strategies)
3. [Indexing Strategy](#indexing-strategy)
4. [Query Optimization](#query-optimization)
5. [Connection Pooling](#connection-pooling)
6. [ORM Patterns](#orm-patterns)
7. [Seed Data Strategy](#seed-data-strategy)
8. [Backup and Recovery](#backup-and-recovery)
9. [Database-Specific Patterns](#database-specific-patterns)

---

## Schema Design Principles

### Normalization vs Denormalization

**CRITICAL**: Start with 3NF, denormalize only when justified by metrics:

```sql
-- ✓ CORRECT: 3rd Normal Form (normalized)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  total DECIMAL(10, 2) NOT NULL,
  status VARCHAR(50) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER NOT NULL REFERENCES orders(id),
  product_id INTEGER NOT NULL,
  quantity INTEGER NOT NULL,
  unit_price DECIMAL(10, 2) NOT NULL,
  subtotal DECIMAL(10, 2) NOT NULL
);

-- Query: Get order total with items
SELECT o.id, o.total, COUNT(oi.id) as item_count
FROM orders o
LEFT JOIN order_items oi ON o.id = oi.order_id
WHERE o.user_id = $1
GROUP BY o.id;
```

**HIGH**: Denormalize ONLY when:
1. Query metrics show repeated joins causing slowness
2. Cache invalidation strategy is clear
3. Trade-off documented in schema comments

```sql
-- ✓ JUSTIFIED DENORMALIZATION
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  total DECIMAL(10, 2) NOT NULL,
  -- Denormalized for reporting performance (cache invalidated on item change)
  item_count INTEGER DEFAULT 0,
  average_item_price DECIMAL(10, 2),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Trigger to maintain denormalized data
CREATE FUNCTION update_order_denormalization()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE orders SET
    item_count = (SELECT COUNT(*) FROM order_items WHERE order_id = NEW.order_id),
    average_item_price = (SELECT AVG(unit_price) FROM order_items WHERE order_id = NEW.order_id)
  WHERE id = NEW.order_id;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER order_items_denormalize
AFTER INSERT OR UPDATE OR DELETE ON order_items
FOR EACH ROW EXECUTE FUNCTION update_order_denormalization();
```

### Surrogate vs Natural Keys

**CRITICAL**: Use surrogate (auto-increment) keys for flexibility:

```sql
-- ✓ CORRECT: Surrogate key with unique constraint
CREATE TABLE countries (
  id SERIAL PRIMARY KEY,
  code VARCHAR(2) UNIQUE NOT NULL,  -- ISO 3166-1 alpha-2
  name VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ✗ WRONG: Natural key (inflexible if requirements change)
CREATE TABLE countries (
  code VARCHAR(2) PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

-- Impact: If you later need to denormalize, surrogate key allows it
CREATE TABLE country_audit (
  id SERIAL PRIMARY KEY,
  country_id INTEGER NOT NULL REFERENCES countries(id),
  old_name VARCHAR(100),
  new_name VARCHAR(100),
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Temporal Data Patterns

**HIGH**: Use soft deletes or temporal tables for audit trails:

```sql
-- Soft delete approach (simpler, sufficient for most cases)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  deleted_at TIMESTAMP,  -- NULL = active
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Always filter for active records
-- SELECT * FROM users WHERE deleted_at IS NULL

-- PostgreSQL temporal table (full history)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) USING btree;

-- Enable temporal versioning
ALTER TABLE users
ADD COLUMN valid_from TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
ADD COLUMN valid_to TIMESTAMP DEFAULT NULL;

-- Retrieve user state at specific time
SELECT * FROM users
WHERE id = 123
  AND valid_from <= '2024-01-15'::TIMESTAMP
  AND (valid_to IS NULL OR valid_to > '2024-01-15'::TIMESTAMP);
```

### Constraint Design

**CRITICAL**: Enforce constraints at database level:

```sql
-- ✓ CORRECT: Comprehensive constraint strategy
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  sku VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 2) NOT NULL CHECK (price > 0),
  stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
  category_id INTEGER NOT NULL REFERENCES categories(id) ON DELETE RESTRICT,
  status VARCHAR(50) NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'archived', 'discontinued')),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  -- Multi-column constraint
  UNIQUE (category_id, sku)
);

-- Domain constraints with domain types
CREATE DOMAIN email_type AS VARCHAR(255) CHECK (VALUE ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$');

CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email email_type UNIQUE NOT NULL
);
```

---

## Migration Strategies

### Forward-Only Migrations

**CRITICAL**: Migrations must be idempotent and reversible:

```sql
-- migration_001_create_users_table.sql
-- Up
CREATE TABLE IF NOT EXISTS users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Down
DROP TABLE IF EXISTS users CASCADE;

-- ✓ CORRECT: Using migration tool (Flyway/Liquibase/Alembic)
-- File: V001__create_users_table.sql
-- Automatically tracked, cannot be re-run
```

### Zero-Downtime Migrations

**CRITICAL**: Deployed code must handle both old and new schema:

```sql
-- Step 1: Add new column with default (fast, no lock)
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- Step 2: Backfill data (in batches to avoid locking)
UPDATE users SET full_name = name WHERE full_name IS NULL
LIMIT 1000;  -- Batched in application code

-- Step 3: Add constraint only after backfill complete
ALTER TABLE users ALTER COLUMN full_name SET NOT NULL;

-- Step 4: Remove old column (only after code deployed)
ALTER TABLE users DROP COLUMN name;
```

**HIGH**: Use feature flags during schema transitions:

```python
# Application code during migration
class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)

    # Both columns exist during transition
    name = Column(String)           # Old column
    full_name = Column(String)      # New column

    @property
    def get_name(self):
        # Read from new column, fallback to old
        if feature_flag('use_full_name'):
            return self.full_name
        return self.name

    def set_name(self, value):
        self.full_name = value
        # Keep old column in sync during transition
        self.name = value
```

### Reversible Migrations

**CRITICAL**: Every migration must be reversible:

```javascript
// Migration tool: db/migrations/001_create_tables.js
module.exports = {
  up: async (db) => {
    await db.schema.createTable('users', table => {
      table.increments('id');
      table.string('email').unique();
      table.timestamp('created_at').defaultTo(db.fn.now());
    });
  },

  down: async (db) => {
    await db.schema.dropTableIfExists('users');
  }
};

// Safe rollback in case of error
// npm run migrate:down
```

### Index Migrations

**HIGH**: Create indexes with minimal locking:

```sql
-- PostgreSQL: Non-blocking index creation
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Then remove old index
DROP INDEX CONCURRENTLY idx_users_email_old;

-- MySQL: Algorithm hints for minimal locking
ALTER TABLE users ADD INDEX idx_email (email), ALGORITHM=INPLACE, LOCK=NONE;
```

---

## Indexing Strategy

### B-Tree Indexes (Default)

**CRITICAL**: Standard index for equality and range queries:

```sql
-- Single column B-tree (most common)
CREATE INDEX idx_users_email ON users(email);

-- Query plans that use this index
SELECT * FROM users WHERE email = 'user@test.com';           -- Seek + Scan
SELECT * FROM users WHERE email LIKE 'user%';                -- Index Scan
SELECT * FROM users WHERE created_at > '2024-01-01';         -- Index Scan
```

### Composite Indexes

**HIGH**: Optimize multi-column WHERE/ORDER BY clauses:

```sql
-- ✓ CORRECT: Composite index for specific query patterns
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at DESC);

-- This query uses index efficiently
SELECT * FROM orders
WHERE user_id = 123
ORDER BY created_at DESC
LIMIT 10;

-- ✗ WRONG: Generic composite index
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
-- Not helpful for: WHERE status = 'completed' (doesn't use index)

-- ✓ CORRECT: ESR rule - Equality, Sort, Range
CREATE INDEX idx_orders_esr ON orders(user_id, created_at DESC, total);
-- user_id = ? AND created_at ORDER BY total
```

### Partial Indexes

**CRITICAL**: Filter at index level for sparse data:

```sql
-- ✓ CORRECT: Only index active users (90% of queries)
CREATE INDEX idx_users_active_email ON users(email)
WHERE deleted_at IS NULL;

-- Query uses index
SELECT * FROM users WHERE email = 'test@test.com' AND deleted_at IS NULL;

-- For archived orders reporting (rare)
CREATE INDEX idx_orders_archived ON orders(id)
WHERE status = 'archived';
```

### GIN Indexes (PostgreSQL)

**HIGH**: For JSONB, arrays, and full-text search:

```sql
-- JSONB data indexing
CREATE TABLE user_metadata (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL,
  data JSONB,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_metadata_data ON user_metadata USING GIN (data);

-- Efficient JSONB queries
SELECT * FROM user_metadata WHERE data @> '{"role": "admin"}';
SELECT * FROM user_metadata WHERE data ? 'premium_feature';

-- Full-text search
CREATE INDEX idx_products_search ON products USING GIN (to_tsvector('english', name || ' ' || description));

SELECT * FROM products
WHERE to_tsvector('english', name || ' ' || description) @@ plainto_tsquery('english', 'wireless headphones');
```

### Hash Indexes

**MEDIUM**: For exact match only (rarely used):

```sql
-- ✓ CORRECT: Hash index for exact match performance
CREATE INDEX idx_users_id_hash ON users USING HASH (id);

-- Works for: WHERE id = 123
-- Does NOT work for: WHERE id > 123 or ORDER BY id

-- B-tree is almost always better due to range support
CREATE INDEX idx_users_id_btree ON users(id);  -- Preferred
```

### Index Maintenance

**HIGH**: Monitor and remove unused indexes:

```sql
-- Find unused indexes (PostgreSQL)
SELECT schemaname, tablename, indexname
FROM pg_indexes
WHERE indexdef NOT LIKE '%UNIQUE%'
  AND idx_scan = 0;

-- Index size analysis
SELECT
  schemaname,
  tablename,
  indexname,
  pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;

-- Rebuild fragmented indexes (PostgreSQL)
REINDEX INDEX idx_orders_user_created;
```

---

## Query Optimization

### EXPLAIN Analysis

**CRITICAL**: Analyze query plans before deployment:

```sql
-- PostgreSQL: Detailed analysis
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT o.*, COUNT(oi.id) as item_count
FROM orders o
LEFT JOIN order_items oi ON o.id = oi.order_id
WHERE o.user_id = 123
GROUP BY o.id;

/* Expected good plan:
  - Seq Scan on orders (filter by user_id)
  - Hash Join with order_items
  - Actual time: < 10ms
*/

-- ✗ BAD PLAN: High-cost operations
/*
  - Nested Loop (expensive for large datasets)
  - Actual time: 500ms+
  - HIGH cost indicates missing index
*/
```

**HIGH**: Read plan indicators:

```
Seq Scan       = Full table scan (bad for large tables)
Index Scan     = Using index (good)
Index Only Scan = Can satisfy query from index alone (best)
Hash Join      = Efficient for large datasets
Nested Loop    = Expensive, causes N+1 problems
```

### N+1 Query Detection

**CRITICAL**: Eager loading prevents N+1:

```python
# ✗ WRONG: N+1 queries
users = User.query.all()  # 1 query
for user in users:
    print(user.orders)    # N additional queries

# ✓ CORRECT: Eager loading with JOIN
users = User.query.joinedload('orders').all()  # 1 query with JOIN

# ✓ CORRECT: Explicit JOIN
users = db.session.query(User).join(Order).all()

# ✓ CORRECT: Batch loading (DataLoader pattern)
loader = DataLoader(batch_load_fn=load_orders_for_users)
for user in users:
    orders = await loader.load(user.id)
```

### Query Planning Strategy

**HIGH**: Index selection for common queries:

```sql
-- Query pattern analysis
-- Pattern 1: Find active user by email (frequent)
SELECT * FROM users WHERE deleted_at IS NULL AND email = ?
CREATE INDEX idx_users_active_email ON users(email) WHERE deleted_at IS NULL;

-- Pattern 2: List user orders by date
SELECT * FROM orders WHERE user_id = ? ORDER BY created_at DESC LIMIT 20
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);

-- Pattern 3: Count orders in date range
SELECT COUNT(*) FROM orders WHERE created_at BETWEEN ? AND ?
CREATE INDEX idx_orders_created ON orders(created_at);

-- Pattern 4: Complex filter
SELECT * FROM products WHERE status = 'active' AND category_id = ? AND price < ?
CREATE INDEX idx_products_filter ON products(category_id, price) WHERE status = 'active';
```

---

## Connection Pooling

### PgBouncer Configuration

**CRITICAL**: Prevent database connection exhaustion:

```ini
# pgbouncer.ini
[databases]
myapp_db = host=postgres.example.com port=5432 dbname=myapp

[pgbouncer]
# Connection limits
pool_mode = transaction      # Return connection after each transaction
max_client_conn = 1000       # Max client connections
default_pool_size = 25       # Connections per database
min_pool_size = 10
reserve_pool_size = 5        # Reserve for admin
reserve_pool_timeout = 3

# Timeouts
server_lifetime = 3600       # Recycle connections
server_idle_timeout = 600    # Close idle connections
query_timeout = 0            # Kill hanging queries

# Monitoring
stats_period = 60            # Update stats every 60s
```

**HIGH**: Connection pooling in application:

```javascript
// Node.js with pg pool
const { Pool } = require('pg');

const pool = new Pool({
  max: 20,                    // Maximum pool size
  min: 2,                     // Minimum connections to maintain
  idleTimeoutMillis: 30000,   // Close idle after 30s
  connectionTimeoutMillis: 2000,
  maxUses: 7200,              // Recycle connections after 7200 uses
});

// Query with auto-release
const result = await pool.query('SELECT * FROM users WHERE id = $1', [123]);

// Check pool state
console.log({
  total: pool.totalCount,
  idle: pool.idleCount,
  waiting: pool.waitingCount
});
```

### Connection Limits

**CRITICAL**: Respect database connection limits:

```
PostgreSQL default: 100 connections
MySQL default: 150 connections

Formula: (Connection pool size) × (Number of app instances) × (1.1 for headroom)

Example:
- 25 connections per pool × 4 app instances × 1.1 = 110 connections
- If PostgreSQL max is 100, reduce pool size to 20-22
```

### Pool Sizing Strategy

**HIGH**: Right-size based on workload:

```python
# Calculate optimal pool size
optimal_pool_size = (number_of_cores * 2) + effective_spindle_count

# Example: 8-core server, SSD storage
optimal_pool_size = (8 * 2) + 0 = 16 connections

# Conservative approach
min_size = 2
max_size = 20

# Configuration
pool = create_pool(
    min_size=2,
    max_size=20,
    max_overflow=10,  # Additional connections if needed
    pool_recycle=3600  # Recycle connections hourly
)
```

---

## ORM Patterns

### Repository Pattern

**CRITICAL**: Abstraction for data access:

```python
# ✓ CORRECT: Repository pattern
class UserRepository:
    def __init__(self, db_session):
        self.session = db_session

    def find_by_id(self, user_id):
        return self.session.query(User).filter(User.id == user_id).first()

    def find_by_email(self, email):
        return self.session.query(User).filter(User.email == email).first()

    def find_active(self):
        return self.session.query(User).filter(User.deleted_at == None).all()

    def create(self, email, name):
        user = User(email=email, name=name)
        self.session.add(user)
        self.session.flush()  # Get ID without commit
        return user

    def update(self, user_id, **kwargs):
        user = self.find_by_id(user_id)
        for key, value in kwargs.items():
            setattr(user, key, value)
        self.session.flush()
        return user

# Usage
repo = UserRepository(db.session)
user = repo.find_by_email('test@test.com')
repo.update(user.id, name='New Name')
```

### Unit of Work Pattern

**HIGH**: Group related operations:

```python
class UnitOfWork:
    def __init__(self, session):
        self.session = session
        self.users = UserRepository(session)
        self.orders = OrderRepository(session)

    def commit(self):
        self.session.commit()

    def rollback(self):
        self.session.rollback()

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self.rollback()
        else:
            self.commit()

# Usage: Transactional integrity
with UnitOfWork(db.session) as uow:
    user = uow.users.create(email='new@test.com', name='New User')
    order = uow.orders.create(user_id=user.id, total=100)
    # Both committed together, or rolled back together on error
```

### Query Builder Pattern

**HIGH**: Type-safe query construction:

```python
# SQLAlchemy query builder
query = (
    db.session.query(User, Order)
    .select_from(User)
    .join(Order, User.id == Order.user_id)
    .filter(User.deleted_at == None)
    .filter(Order.created_at > '2024-01-01')
    .order_by(Order.created_at.desc())
    .limit(10)
)

# Chainable, composable queries
def get_active_user_orders(user_id, days=30):
    cutoff = datetime.now() - timedelta(days=days)
    return (
        db.session.query(Order)
        .filter(Order.user_id == user_id)
        .filter(Order.created_at > cutoff)
        .order_by(Order.created_at.desc())
    )
```

---

## Seed Data Strategy

### Development Seeds

**CRITICAL**: Reproducible test data:

```python
# seeds/dev/users.py
def seed_users():
    users = [
        User(email='admin@test.com', name='Admin User', role='admin'),
        User(email='user@test.com', name='Regular User', role='user'),
        User(email='guest@test.com', name='Guest User', role='guest'),
    ]
    db.session.add_all(users)
    db.session.commit()
    return users

# seeds/dev/products.py
def seed_products():
    products = [
        Product(name='Widget', price=29.99, stock=100),
        Product(name='Gadget', price=49.99, stock=50),
    ]
    db.session.add_all(products)
    db.session.commit()
    return products

# seeds/__init__.py
def seed_all():
    # Order matters: users before orders
    seed_users()
    seed_products()
    seed_orders()

# Usage: python -c "from seeds import seed_all; seed_all()"
```

### Staging Seeds

**HIGH**: Realistic production-like data:

```python
# seeds/staging/realistic_data.py
from faker import Faker

faker = Faker()

def seed_staging_users(count=1000):
    users = [
        User(
            email=faker.email(),
            name=faker.name(),
            role=random.choice(['user', 'admin']),
            created_at=faker.date_time_this_year()
        )
        for _ in range(count)
    ]
    db.session.bulk_insert_mappings(User, users)
    db.session.commit()

def seed_staging_orders(users, count=5000):
    orders = []
    for _ in range(count):
        orders.append(Order(
            user_id=random.choice(users).id,
            total=random.uniform(10, 500),
            status=random.choice(['pending', 'completed', 'cancelled']),
            created_at=faker.date_time_this_year()
        ))
    db.session.bulk_insert_mappings(Order, orders)
    db.session.commit()
```

### Production Data Refresh

**CRITICAL**: Safe data management for sensitive environments:

```sql
-- DO NOT seed production with fake data
-- Instead, use anonymized copy of production data

-- 1. Anonymize sensitive data
UPDATE users SET
  email = CONCAT('user_', id, '@test.com'),
  phone = NULL
WHERE environment = 'staging';

-- 2. Truncate sensitive logs
DELETE FROM audit_logs WHERE created_at < NOW() - INTERVAL 90 DAY;

-- 3. Redact PII
UPDATE payments SET
  card_number = SUBSTR(card_number, -4),
  cvv = NULL;
```

---

## Backup and Recovery

### Point-in-Time Recovery (PITR)

**CRITICAL**: Enable WAL archiving for PITR:

```ini
# PostgreSQL postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backup/wal_archive/%f'
archive_timeout = 300

# Backup strategy
backup_full = daily         # Full backup daily
backup_incremental = hourly # Incremental hourly

# Retention
backup_retention = 30 days  # Keep 30 days of backups
```

**HIGH**: Test recovery procedures:

```bash
#!/bin/bash
# Test PITR recovery procedure

# 1. Create backup
pg_basebackup -h localhost -D /backup/base -X stream

# 2. Create archive for recovery
mkdir -p /backup/recovery_wal

# 3. Recover to specific point in time
RESTORE_COMMAND='cp /backup/wal_archive/%f %p'
TARGET_TIMELINE=latest
TARGET_TIME='2024-01-15 14:30:00'

# Recovery stops at specified time
recovery_target_time = '2024-01-15 14:30:00'
recovery_target_timeline = 'latest'
restore_command = 'cp /backup/wal_archive/%f %p'
```

### Logical Backups

**HIGH**: Full database dumps for migrations:

```bash
# PostgreSQL
pg_dump -U postgres -d myapp_db > backup.sql

# With compression
pg_dump -U postgres -d myapp_db | gzip > backup.sql.gz

# Restore
psql -U postgres -d myapp_db < backup.sql

# MySQL
mysqldump -u user -p database > backup.sql
mysql -u user -p database < backup.sql
```

### Physical Backups

**HIGH**: Block-level backups for large databases:

```bash
# MySQL with Percona Xtrabackup
xtrabackup --backup --target-dir=/backup/full

# Prepare for restore
xtrabackup --prepare --target-dir=/backup/full

# Restore
rsync -aP /backup/full/ /var/lib/mysql/
```

---

## Database-Specific Patterns

### PostgreSQL

**CRITICAL**: JSONB for semi-structured data:

```sql
CREATE TABLE user_preferences (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  settings JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Set nested value
UPDATE user_preferences SET settings = jsonb_set(
  settings,
  '{theme, dark_mode}',
  'true'
) WHERE user_id = 123;

-- Query nested values
SELECT * FROM user_preferences
WHERE settings @> '{"theme": {"dark_mode": true}}';
```

**HIGH**: Window functions for analytics:

```sql
SELECT
  user_id,
  order_date,
  total,
  SUM(total) OVER (PARTITION BY user_id ORDER BY order_date) as running_total,
  ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date DESC) as order_rank
FROM orders;
```

### MySQL

**CRITICAL**: Proper charset for Unicode:

```sql
-- Create table with UTF-8 support
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Convert existing table
ALTER TABLE users CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- JSON column (MySQL 5.7+)
CREATE TABLE metadata (
  id INT AUTO_INCREMENT PRIMARY KEY,
  data JSON NOT NULL,
  -- Extract JSON for indexing
  data_type VARCHAR(50) GENERATED ALWAYS AS (JSON_EXTRACT(data, '$.type')) STORED
);

CREATE INDEX idx_metadata_type ON metadata(data_type);
```

### MongoDB

**CRITICAL**: Schema validation:

```javascript
db.createCollection('users', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['email', 'name'],
      properties: {
        _id: { bsonType: 'objectId' },
        email: { bsonType: 'string' },
        name: { bsonType: 'string' },
        created_at: { bsonType: 'date' }
      }
    }
  }
});

// Compound index for efficient queries
db.users.createIndex({ email: 1, created_at: -1 });

// Aggregation pipeline (efficient alternative to joins)
db.orders.aggregate([
  { $match: { status: 'completed' } },
  { $lookup: {
      from: 'users',
      localField: 'user_id',
      foreignField: '_id',
      as: 'user'
    }
  },
  { $unwind: '$user' },
  { $group: {
      _id: '$user._id',
      total_spent: { $sum: '$total' }
    }
  }
]);
```

### Redis

**CRITICAL**: Data structure selection:

```
STRING: Basic key-value (user:123:name -> "John")
HASH: Object storage (user:123 -> {name: "John", email: "john@test.com"})
LIST: Queue/log (task_queue -> [task1, task2, task3])
SET: Unique items (user:123:tags -> {python, javascript})
ZSET: Sorted/ranked (leaderboard -> {user1: 1000, user2: 950})
```

```python
import redis

r = redis.Redis(host='localhost', port=6379)

# String: Cache user data
r.set('user:123', json.dumps({'name': 'John'}), ex=3600)  # Expire in 1 hour

# Hash: Store object
r.hset('user:123', mapping={'name': 'John', 'email': 'john@test.com'})

# List: Queue tasks
r.lpush('task_queue', json.dumps({'task': 'send_email', 'to': 'user@test.com'}))

# Set: Track unique tags
r.sadd('user:123:tags', 'python', 'javascript')

# ZSet: Leaderboard
r.zadd('leaderboard', {'user1': 1000, 'user2': 950})
```

### SQLite

**CRITICAL**: Suitable for small-to-medium apps:

```python
import sqlite3

conn = sqlite3.connect('app.db')
conn.execute('PRAGMA foreign_keys = ON')  # Enable constraints
conn.execute('PRAGMA journal_mode = WAL')  # Better concurrency

# WAL mode enables concurrent reads
cursor = conn.cursor()
cursor.execute('SELECT * FROM users WHERE id = ?', (123,))
```

---

## Summary: Database Checklist

- [ ] Schema normalized to 3NF (denormalization justified)
- [ ] Surrogate keys used for flexibility
- [ ] All constraints enforced at database level
- [ ] Migrations are forward-only and reversible
- [ ] Indexes match common query patterns
- [ ] Composite indexes follow ESR rule
- [ ] Query plans analyzed before deployment
- [ ] N+1 queries prevented with eager loading
- [ ] Connection pooling configured
- [ ] Test data factories created
- [ ] Backup and recovery tested
- [ ] Database-specific optimizations applied
