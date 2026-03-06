---
layer: 03_software_engineering
type: engineering
tool: SQL/PostgreSQL
status: growing
tags: [sql, postgresql, databases, query-optimization, indexing, transactions]
created: 2026-03-05
---

# SQL Patterns

## Purpose

Comprehensive reference for SQL query patterns, indexing strategies, and query optimization
techniques in relational databases, with a focus on PostgreSQL. Covers patterns applicable
from data engineering to backend development to ML feature pipelines.

## Architecture

### Join Types

```sql
-- INNER JOIN: only matching rows in both tables
SELECT o.id, c.name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id;

-- LEFT JOIN: all rows from left, matched rows from right (NULLs where no match)
SELECT c.name, COUNT(o.id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;

-- RIGHT JOIN: mirror of LEFT; prefer LEFT JOIN for readability
-- FULL OUTER JOIN: all rows from both tables
SELECT a.id AS a_id, b.id AS b_id
FROM table_a a
FULL OUTER JOIN table_b b ON a.key = b.key;

-- CROSS JOIN: cartesian product — every row × every row
SELECT d.name AS department, r.name AS role
FROM departments d
CROSS JOIN roles r;

-- SELF JOIN: join a table to itself; useful for hierarchies and comparisons
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### Aggregations, GROUP BY, HAVING

```sql
-- Basic aggregation
SELECT
    department_id,
    COUNT(*)          AS headcount,
    AVG(salary)       AS avg_salary,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary,
    MAX(salary)       AS max_salary
FROM employees
GROUP BY department_id;

-- HAVING filters aggregated groups (WHERE filters rows before aggregation)
SELECT department_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 70000;

-- GROUPING SETS — multiple GROUP BY levels in one query
SELECT department_id, job_title, COUNT(*) AS cnt
FROM employees
GROUP BY GROUPING SETS ((department_id), (job_title), ());

-- ROLLUP — hierarchical subtotals
SELECT year, quarter, SUM(revenue)
FROM sales
GROUP BY ROLLUP(year, quarter);

-- CUBE — all combinations
SELECT region, product, SUM(revenue)
FROM sales
GROUP BY CUBE(region, product);
```

### Subqueries and CTEs

```sql
-- Correlated subquery (executes once per outer row — can be slow)
SELECT name
FROM employees e
WHERE salary > (
    SELECT AVG(salary) FROM employees WHERE department_id = e.department_id
);

-- Non-correlated subquery in WHERE
SELECT * FROM orders
WHERE customer_id IN (
    SELECT id FROM customers WHERE country = 'SE'
);

-- CTE: readability and reuse within a query
WITH regional_sales AS (
    SELECT region, SUM(amount) AS total
    FROM orders
    GROUP BY region
),
top_regions AS (
    SELECT region FROM regional_sales
    WHERE total > (SELECT AVG(total) FROM regional_sales)
)
SELECT o.id, o.region, o.amount
FROM orders o
JOIN top_regions t ON o.region = t.region;

-- Recursive CTE: traversing hierarchies (org charts, BOM, graphs)
WITH RECURSIVE subordinates AS (
    SELECT id, name, manager_id, 0 AS depth
    FROM employees
    WHERE id = 1  -- root

    UNION ALL

    SELECT e.id, e.name, e.manager_id, s.depth + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates ORDER BY depth;
```

### Window Functions

```sql
-- ROW_NUMBER: unique row rank within partition
SELECT
    id,
    department_id,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
FROM employees;

-- RANK vs DENSE_RANK: ties get same rank; RANK skips next numbers, DENSE_RANK doesn't
SELECT name, salary,
    RANK()       OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- LAG / LEAD: access previous / next row value
SELECT
    date,
    revenue,
    LAG(revenue)  OVER (ORDER BY date) AS prev_revenue,
    LEAD(revenue) OVER (ORDER BY date) AS next_revenue,
    revenue - LAG(revenue) OVER (ORDER BY date) AS delta
FROM daily_revenue;

-- Running totals and moving averages
SELECT
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date ROWS UNBOUNDED PRECEDING) AS cumulative,
    AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS ma7
FROM daily_revenue;

-- FIRST_VALUE / LAST_VALUE / NTH_VALUE
SELECT
    id,
    salary,
    FIRST_VALUE(salary) OVER (PARTITION BY dept ORDER BY salary DESC) AS dept_max
FROM employees;

-- NTILE: bucket rows into N groups
SELECT name, salary, NTILE(4) OVER (ORDER BY salary) AS quartile
FROM employees;
```

## Implementation Notes

### Indexing

```sql
-- B-tree: default; supports =, <, >, BETWEEN, LIKE 'prefix%'
CREATE INDEX idx_orders_customer ON orders(customer_id);

-- Composite index: column order matters; leftmost prefix rule
CREATE INDEX idx_orders_status_date ON orders(status, created_at);
-- Useful for: WHERE status = 'pending' ORDER BY created_at
-- NOT useful for: WHERE created_at > '2026-01-01' alone (no status filter)

-- Partial index: index only a subset of rows — smaller, faster
CREATE INDEX idx_orders_pending ON orders(created_at)
WHERE status = 'pending';

-- Hash index: faster equality lookups, no range queries
CREATE INDEX idx_sessions_token ON sessions USING hash(token);

-- Covering index: include extra columns to avoid heap fetch
CREATE INDEX idx_orders_cover ON orders(customer_id) INCLUDE (status, total);

-- Expression index: index on a function result
CREATE INDEX idx_email_lower ON users(lower(email));
-- Requires: WHERE lower(email) = lower($1)

-- GIN index: full-text search, JSONB containment, arrays
CREATE INDEX idx_docs_body ON documents USING gin(to_tsvector('english', body));
CREATE INDEX idx_meta ON products USING gin(metadata jsonb_path_ops);
```

### Query Optimization

```sql
-- EXPLAIN ANALYZE: actual execution plan with timing
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders WHERE customer_id = 42;

-- Key plan nodes to understand:
-- Seq Scan: full table scan — fine for small tables, bad for large ones
-- Index Scan: uses index, fetches heap rows
-- Index Only Scan: covers query from index alone (no heap fetch)
-- Hash Join: build hash table on smaller relation, probe with larger
-- Nested Loop: for small outer + indexed inner; bad when outer is large
-- Merge Join: both inputs sorted; efficient for large sorted sets

-- N+1 problem: loading N child records with N separate queries
-- Bad pattern (ORM default for lazy loading):
--   SELECT * FROM posts             → 1 query
--   SELECT * FROM comments WHERE post_id = 1  → N queries
-- Fix: eager load with JOIN or subquery; use SELECT IN (ids)

-- Statistics: keep them fresh
ANALYZE orders;
-- Or set autovacuum/autoanalyze — check pg_stat_user_tables

-- Query planner hints (PostgreSQL has no direct hints; use these levers):
SET enable_seqscan = off;           -- force index usage (session-scoped)
SET enable_hashjoin = off;          -- force merge or nested loop
SET work_mem = '256MB';             -- more memory for sorts/hashes
-- pg_hint_plan extension provides Oracle-style hints
```

### Transactions and ACID

```sql
-- Explicit transaction block
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- or ROLLBACK on error

-- Savepoints: partial rollback within a transaction
BEGIN;
INSERT INTO log(msg) VALUES ('start');
SAVEPOINT sp1;
UPDATE risky_table SET val = 0;  -- may fail
ROLLBACK TO SAVEPOINT sp1;       -- undo just this step
INSERT INTO log(msg) VALUES ('recovered');
COMMIT;
```

**Isolation levels and their anomalies:**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| READ UNCOMMITTED | ✓ possible | ✓ | ✓ |
| READ COMMITTED (PG default) | ✗ | ✓ | ✓ |
| REPEATABLE READ | ✗ | ✗ | ✗ (PG: snapshot) |
| SERIALIZABLE | ✗ | ✗ | ✗ |

```sql
-- Set per-transaction isolation level
BEGIN ISOLATION LEVEL REPEATABLE READ;
-- ...
COMMIT;
```

- **PostgreSQL uses MVCC** — readers never block writers; each transaction sees a snapshot.
- **SERIALIZABLE** in PostgreSQL uses SSI (Serializable Snapshot Isolation) — detects
  dangerous read-write cycles without locking.

### PostgreSQL-Specific Patterns

```sql
-- UPSERT with ON CONFLICT
INSERT INTO counters(key, value)
VALUES ('page_views', 1)
ON CONFLICT (key) DO UPDATE SET value = counters.value + EXCLUDED.value;

-- RETURNING clause — avoid extra round-trip
INSERT INTO events(type, payload) VALUES ('click', '{}')
RETURNING id, created_at;

-- JSONB operations
SELECT metadata->>'name' AS name           -- extract as text
FROM products
WHERE metadata @> '{"active": true}'       -- containment
  AND metadata->>'price' IS NOT NULL;

UPDATE products
SET metadata = metadata || '{"discounted": true}'::jsonb
WHERE id = 42;

-- Array operations
SELECT * FROM users WHERE 'admin' = ANY(roles);
SELECT array_agg(name ORDER BY name) FROM tags WHERE active;

-- Lateral join: correlated subquery in FROM — can reference outer table
SELECT c.name, recent.total
FROM customers c
LEFT JOIN LATERAL (
    SELECT SUM(amount) AS total
    FROM orders
    WHERE customer_id = c.id
    ORDER BY created_at DESC
    LIMIT 5
) recent ON true;

-- pg_trgm for fuzzy text search
CREATE EXTENSION pg_trgm;
CREATE INDEX idx_name_trgm ON products USING gin(name gin_trgm_ops);
SELECT * FROM products WHERE name % 'posgres';  -- trigram similarity
```

## Trade-offs

| Decision | When to choose |
|---|---|
| CTE vs subquery | CTEs for readability and recursion; subqueries can be inlined by planner |
| Normalized schema | OLTP: reduces write anomalies, saves space |
| Denormalized / wide tables | OLAP / read-heavy: fewer joins, better columnar compression |
| Many small indexes | Fast reads; slow writes, high storage overhead |
| Partial indexes | Low-cardinality filtered queries (status, active flags) |
| SERIALIZABLE isolation | Correctness-critical; higher abort rate under contention |
| PostgreSQL JSONB column | Schema flexibility; loses some query planner statistics |

- **CTEs in PostgreSQL 12+** are inlined by default (no longer optimization fences unless
  `WITH ... AS MATERIALIZED`).
- **Composite index column order**: put the equality-filtered column first, range-filtered last.
- **Connection pooling** (PgBouncer) is almost always needed in production — PostgreSQL
  processes are heavyweight (~5 MB each).

## References

- [PostgreSQL documentation — Query Planning](https://www.postgresql.org/docs/current/query-planning.html)
- [Use The Index, Luke](https://use-the-index-luke.com/) — SQL indexing guide
- [PostgreSQL EXPLAIN visualizer](https://explain.dalibo.com/)
- *SQL Performance Explained* — Markus Winand
- [Window Functions cheat sheet](https://www.postgresql.org/docs/current/functions-window.html)

## Links
- [[nosql_patterns|NoSQL Patterns]]
- [[caching_strategies|Caching Strategies]]
- [[04_ml_engineering/01_data_engineering/feature_stores|Feature Stores]]
