# SQL cheat-sheet/guide

---

# Database (create / open / select / drop / settings)

### Step 1 - Create a database (MySQL / MariaDB) / create/open file (SQLite)

**What & why:** Databases are namespaces that contain tables. On server engines you `CREATE DATABASE` to make a new namespace; on SQLite the database is a file - “creating” is opening/creating that file.

**SQL**

```sql
-- MySQL / MariaDB
CREATE DATABASE app_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- SQLite: open (creates file) using client or attach another DB
-- command-line: sqlite3 app_db.sqlite
ATTACH DATABASE 'other.db' AS other_db;  -- attach in session
-- (when finished in the session) DETACH DATABASE other_db;  -- detach the attached database
```

**Example output**

* MySQL/MariaDB clients typically show success:

```
Query OK, 1 row affected (0.01 sec)
```

* SQLite: opening the file produces no rows; `ATTACH` returns success in the client.

**Details & pitfalls:**

* Choose `utf8mb4` for full Unicode in MySQL/MariaDB. Collation controls case-sensitivity and sort order.

* SQLite stores everything in a file and allows only **one writer at a time** (even in WAL mode); WAL mode lets readers keep reading while a write is in progress (see Concurrency section).

* **Permissions & privileges:** Creating a database usually requires `CREATE` or admin-level privileges. App accounts should have the minimum required rights (e.g., `INSERT/SELECT/UPDATE/DELETE` on specific schemas) rather than global privileges.

* **Storage engines (MySQL/MariaDB):** InnoDB (transactional, row-level locks, supports FKs, the default) vs MyISAM (non-transactional, table-level locking, no FKs). Prefer InnoDB for modern apps.

* **Collation effects:** Sorting, uniqueness checks, and `GROUP BY` behavior are affected by collation - a case-insensitive collation (`_ci`) treats `A` = `a`.

* **SQLite file-level considerations:** File locking, OS-level backups, and permissions matter. If multiple processes access the file concurrently, use WAL (`PRAGMA journal_mode = WAL`) so readers don't block the writer and the writer doesn't block readers (writes are still serialized; set a `busy_timeout` so writers wait instead of failing immediately).

---

### Step 2 - Select / use a database

**What & why:** Tell the server which database you want to run commands in (one session-level context).

**SQL**

```sql
-- MySQL / MariaDB
USE app_db;

-- SQLite: you connect directly to a file, or use the attached alias (other_db.table_name)
```

**Example output**

```
Database changed
```

**Details:** `USE` only exists in server engines. In SQLite, refer to `main.table` or `other_db.table` if attached.

* `USE` affects only the current connection/session. Applications typically set the database in the connection string so every client connection automatically uses the right DB.
* For cross-database queries on server engines you can use fully-qualified names: in MySQL/MariaDB a "database" and a "schema" are the same thing, so it's `db_name.table_name`; SQL Server uses `db_name.schema_name.table_name`; PostgreSQL uses `schema_name.table_name` and cannot query across databases directly (without extensions like `dblink`/`postgres_fdw`).

---

### Step 3 - Drop a database

**What & why:** Remove all data and tables inside a database - destructive!

**SQL**

```sql
-- MySQL / MariaDB
DROP DATABASE app_db;

-- SQLite: delete the file from disk outside SQL
```

**Example output**

```
Query OK, 0 rows affected
```

**Pitfall:** Always backup before dropping. File deletion for SQLite is irreversible without backup.

* **Replication & replication lag:** Dropping on a master can propagate to replicas. Be cautious in replicated environments.
* **Safety patterns:** Consider disabling writes, taking a final logical backup (`mysqldump`) and verifying it before `DROP`. Keep retention policies and backups automated.

---

# Tables (schema lifecycle: create, alter, constraints, drop, inspect)

### Step 1 - Create a minimal table (start small)

**What & why:** Start with only required columns. Easier to evolve the schema later.

**SQL**

```sql
-- cross-engine compatible example (SQLite affinity used; server engines accept same columns)
CREATE TABLE items (
  id INTEGER PRIMARY KEY,   -- SQLite: rowid; MySQL/MariaDB: INT PRIMARY KEY (use AUTO_INCREMENT)
  name TEXT NOT NULL
);
```

**Example output / effect**

* New table `items` created; `SELECT * FROM items;` returns zero rows.

**Details:**

* In MySQL/MariaDB prefer `id INT AUTO_INCREMENT PRIMARY KEY`.

* In SQLite `INTEGER PRIMARY KEY` is special and acts as the `rowid` (auto-incrementing).

* **Autoincrement differences:** In MySQL `AUTO_INCREMENT` uses the table's internal counter. In SQLite, `INTEGER PRIMARY KEY` uses the `rowid`. `AUTOINCREMENT` keyword in SQLite forces a monotonically increasing counter (never reuses ids) but can cause larger DB files - usually avoid unless you need that property.

* **Choosing column types:** Use `VARCHAR(n)` when you care about a maximum length and storage; `TEXT` for large free-form content. Use `DECIMAL` (fixed point) for money to avoid floating-point rounding errors.

---

### Step 2 - Add a column (ALTER TABLE ADD COLUMN)

**What & why:** Expand a table gradually when new data is needed.

**SQL**

```sql
ALTER TABLE items ADD COLUMN created_at DATETIME DEFAULT CURRENT_TIMESTAMP;
```

**Example schema output (conceptual)**

|     column |     type | default           |
| ---------: | -------: | ----------------- |
|         id |  INTEGER | PRIMARY KEY       |
|       name |     TEXT | NOT NULL          |
| created_at | DATETIME | CURRENT_TIMESTAMP |

**Details & pitfalls:**

* Adding a column with a default populates existing rows with the default (behavior varies slightly by engine/version).

* Some `ALTER` operations rewrite large tables - test on copy for big tables.

* **Blocking vs online DDL:** Modern MySQL/MariaDB/InnoDB versions support many online DDL operations; others still lock tables. Always check engine/version behavior and test on staging.

* **NULL vs NOT NULL:** Adding a `NOT NULL` column without a default fails on a non-empty table in SQLite and PostgreSQL (MySQL/MariaDB instead silently fill existing rows with the type's implicit default, e.g. `0` or `''`). Best practice: add the column nullable, backfill data, then alter to `NOT NULL`.

* **Backups & migrations:** For large tables, consider creating a shadow table, copying rows in chunks, then swapping names to minimize downtime.

---

### Step 3 - Rename table / column

**What & why:** Small refactors without copying data.

**SQL**

```sql
-- Rename table
ALTER TABLE items RENAME TO products;

-- Rename column (MySQL 8+, MariaDB, SQLite modern)
ALTER TABLE products RENAME COLUMN name TO title;
```

**Example output**

```
Query OK, 0 rows affected
```

**Pitfall:** Older SQLite versions lacked `RENAME COLUMN` - fallback is create-copy-drop-rename.

* **Migration tooling:** Prefer using versioned migrations (Flyway, Liquibase, or ORM migrations) so renames are tracked and reversible.
* **Client compatibility:** Renaming can break application code if column/table names are hard-coded; coordinate deploys (blue-green/deploy ordering) when necessary.

---

### Step 4 - Drop a column / drop table

**What & why:** Remove unused data; dropping tables removes structure & data.

**SQL**

```sql
-- Drop column (MySQL/MariaDB; modern SQLite supports this too)
ALTER TABLE products DROP COLUMN created_at;

-- Drop table
DROP TABLE products;
```

**Example output**

```
Query OK, 0 rows affected
```

**Details:** Dropping columns can be expensive. For critical apps, prefer migrations that create new tables and copy data to minimize risk.

* **Safe deprecation pattern:** Mark columns as deprecated, stop writing to them, backfill any reads from new columns, then remove in a follow-up migration with monitoring.
* **Rolling back drops:** Keep a copy of the schema and data (dump) before dropping so you can restore if needed.

---

### Step 5 - Constraints (primary key, unique, foreign key)

**What & why:** Constraints enforce rules: uniqueness, referential integrity, required fields.

**SQL**

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  username TEXT NOT NULL UNIQUE,
  email TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
  id INTEGER PRIMARY KEY,
  user_id INTEGER,
  title TEXT,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Example output (conceptual)**

* Inserting a post with `user_id` not present fails (if FK enforcement is ON). Attempting duplicate `username` fails.

**Engine notes & pitfalls:**

* SQLite requires `PRAGMA foreign_keys = ON;` per connection to enforce foreign keys.

* `UNIQUE` prevents duplicates; `PRIMARY KEY` implies unique + not null.

* `ON DELETE CASCADE` removes dependent rows automatically - use carefully.

* **Indexing FKs:** Index the referenced and referencing columns to speed joins and cascade operations.

* **Constraint checking behavior:** Some databases evaluate constraints at statement end vs transaction end; know your engine's semantics.

* **Deferrable constraints:** MySQL/MariaDB don't support deferrable constraints (checked at the end of the transaction), but PostgreSQL and SQLite do (`DEFERRABLE INITIALLY DEFERRED`). Keep these differences in mind when moving between RDBMSs.

---

### Step 6 - Indexes (create / drop / why)

**What & why:** Indexes speed lookups on columns used in `WHERE`, `JOIN`, or `ORDER BY`. Each index uses space and slows writes.

**SQL**

```sql
CREATE INDEX idx_users_username ON users(username);
CREATE UNIQUE INDEX ux_users_email ON users(email);

-- Drop an index (syntax varies)
DROP INDEX idx_users_username ON users;  -- MySQL/MariaDB (the table name is required)
DROP INDEX idx_users_username;           -- SQLite / PostgreSQL
```

**Example EXPLAIN snippet (conceptual)**

| id | table | type |                key | rows |
| -: | ----: | ---: | -----------------: | ---: |
|  1 | users |  ref | idx_users_username |    1 |

**Pitfalls:**

* `LIKE '%term%'` cannot use a leading-index (leading `%` prevents index use).

* Composite indexes help only when queries filter/order using the index column order.

* **Covering indexes:** If an index includes all columns used by a query (SELECT, WHERE), the DB can satisfy the request from the index without accessing the table (index-only scan).

* **Index maintenance:** Use `ANALYZE TABLE` (MySQL) or `ANALYZE` (SQLite) to refresh statistics so the optimizer makes better choices.

* **Index cost:** Each index increases write latency (INSERT/UPDATE/DELETE). Balance read speed vs write overhead.

---

# Table items / Records (SELECT → filter → sort → paginate → CRUD → joins → aggregates)

This whole section intentionally *builds* queries one keyword at a time.

### Step 1 - Read all rows: `SELECT *`

**What & why:** Quick way to inspect data. Useful for small tables or debugging.

**SQL**

```sql
SELECT * FROM users;
```

**Example output**

| id | username | email                                 | active | created_at          |
| -: | -------: | ------------------------------------- | :----: | ------------------- |
|  1 |    alice | [a@example.com](mailto:a@example.com) |    1   | 2026-03-01 09:00:00 |
|  2 |      bob | [b@example.com](mailto:b@example.com) |    1   | 2026-03-02 10:15:00 |

**Pitfall:** Returns everything - avoid in production.

* **SELECT * and schema evolution:** `SELECT *` returns columns in table order; adding/removing columns can break client code that relies on column ordering. Prefer explicit column lists in application queries.
* **Permissions:** `SELECT *` may return more data than a role should see; apply column-level permissions if supported.

---

### Step 2 - Projection: pick columns

**What & why:** Only fetch needed columns to reduce bandwidth & improve clarity.

**SQL**

```sql
SELECT id, username, email FROM users;
```

**Example output**

| id | username | email                                 |
| -: | -------: | ------------------------------------- |
|  1 |    alice | [a@example.com](mailto:a@example.com) |
|  2 |      bob | [b@example.com](mailto:b@example.com) |

* Project only what you need to reduce network I/O, serialization cost, and memory usage on the client. This is especially important for wide rows or large BLOB/TEXT columns.

---

### Step 3 - Filter: `WHERE` (single condition)

**What & why:** Return only rows meeting a condition. `WHERE` uses comparisons and boolean logic.

**SQL**

```sql
SELECT id, username FROM users WHERE active = 1;
```

**Example output**

| id | username |
| -: | -------- |
|  1 | alice    |
|  2 | bob      |

**Details:** Use `IS NULL` / `IS NOT NULL` for NULL checks. Combine with `AND`/`OR`.

* For performance, ensure columns used in `WHERE` are indexed if selective. Beware implicit type conversion in comparisons (e.g., comparing strings to numbers) which can prevent index usage.

---

### Step 4 - Sorting: `ORDER BY`

**What & why:** Determine the order of returned rows (important for UI & pagination).

**SQL**

```sql
SELECT id, username, created_at
FROM users
ORDER BY created_at DESC;  -- newest first

-- Ascending (oldest first) explicit example
SELECT id, username, created_at
FROM users
ORDER BY created_at ASC;   -- oldest first
```

**Example output**

| id | username | created_at          |
| -: | -------: | ------------------- |
|  2 |      bob | 2026-03-02 10:15:00 |
|  1 |    alice | 2026-03-01 09:00:00 |

**Tip:** Add secondary sort (e.g., `ORDER BY created_at DESC, id DESC`) for deterministic order.

* Sorting large result sets is expensive. Use indexes that match your `ORDER BY` to allow index scans rather than sorting in memory/disk.

---

### Step 5 - Limit / Offset for paging

**What & why:** Return a single page of rows. Simple pagination - fine for small offsets.

**SQL**

```sql
SELECT id, username FROM users ORDER BY created_at DESC LIMIT 2 OFFSET 0;
```

**Example output**

| id | username |
| -: | -------- |
|  2 | bob      |
|  1 | alice    |

**Pitfalls:** Large `OFFSET` is slow and can produce inconsistent results if data changes during paging.

* For large tables, prefer keyset pagination. OFFSET forces the DB to scan and discard the offset rows, which becomes slow as offset grows.

---

### Step 6 - Keyset (cursor) pagination - efficient paging

**What & why:** Use the last seen value (cursor) instead of `OFFSET` to efficiently fetch next pages. Uses indexed columns - scalable & stable.

**SQL**

```sql
-- Suppose last_seen_created_at = '2026-03-02 10:15:00'
SELECT id, username, created_at
FROM users
WHERE created_at < '2026-03-02 10:15:00'
ORDER BY created_at DESC
LIMIT 2;
```

**Example output**

| id | username | created_at          |
| -: | -------: | ------------------- |
|  1 |    alice | 2026-03-01 09:00:00 |

**Tip:** If timestamps can tie, include `id` as tie-breaker: `AND (created_at < :t OR (created_at = :t AND id < :last_id))`.

* **Stateless cursors:** Encode the cursor (e.g., base64 of timestamp+id) and use it for subsequent requests. This keeps APIs stateless.
* **Backwards paging:** Keyset is naturally forward-oriented; implementing bi-directional paging requires careful cursor logic.

---

### Step 7 - Pattern matching: `LIKE` (substring)

**What & why:** Find rows whose text column contains a substring. `%` = any sequence, `_` = single char.

**SQL**

```sql
SELECT id, username FROM users WHERE username LIKE '%al%';
```

**Example output**

| id | username |
| -: | -------- |
|  1 | alice    |

**Pitfalls:** Leading `%` prevents index use - slow on large tables. For serious search, use full-text.

* For case-insensitive `LIKE` behavior, collations matter. Some engines offer `ILIKE` (case-insensitive) (e.g., PostgreSQL), but MySQL uses collations.
* For even more flexible pattern or regex matching, many engines support `REGEXP` or `RLIKE` (e.g., MySQL `column REGEXP 'pattern'`) — syntax and features vary by engine.
* For very large text search, consider full-text indexes or an external search engine (Elasticsearch, Meilisearch).

---

### Step 8 - Joins: combine rows (INNER JOIN & LEFT JOIN)

**What & why:** Retrieve related data across tables.

**SQL (INNER JOIN)**

```sql
SELECT u.username, p.title
FROM users u
JOIN posts p ON p.user_id = u.id
WHERE u.active = 1;
```

**Example output**

| username | title         |
| -------: | ------------- |
|    alice | Welcome       |
|      bob | My first post |

**SQL (LEFT JOIN)**

```sql
SELECT u.username, p.title
FROM users u
LEFT JOIN posts p ON p.user_id = u.id;
```

**Example output**

| username | title   |
| -------: | ------- |
|    alice | Welcome |
|    carol | (NULL)  |

**Additional join types (opposites / complements):**

```sql
-- RIGHT JOIN (mirror of LEFT JOIN) - returns rows from the right table even if no match on left
SELECT u.username, p.title
FROM users u
RIGHT JOIN posts p ON p.user_id = u.id;

-- FULL OUTER JOIN (returns all rows from both sides, matched where possible) - not supported in MySQL/MariaDB; emulate with UNION of LEFT and RIGHT JOIN, or use databases that support it (PostgreSQL, SQL Server, SQLite 3.39+)
SELECT u.username, p.title
FROM users u
FULL OUTER JOIN posts p ON p.user_id = u.id;  -- dialect-dependent
```

**Pitfalls:** Missing or wrong `ON` clause can cause cartesian products (huge, incorrect result).

* **Join order & optimization:** The optimizer decides join order; indexing the join keys on both sides improves performance.
* **Detecting missing relations:** Use `LEFT JOIN ... WHERE p.id IS NULL` to find rows in the left table with no matching right-row.
* **Cross joins:** `CROSS JOIN` or missing ON in multi-table `FROM` lists produce Cartesian products - watch accidental omissions.

---

### Step 9 - Aggregation & grouping

**What & why:** Summarize rows (counts, sums, averages), grouped by a column.

**SQL**

```sql
SELECT active, COUNT(*) AS cnt
FROM users
GROUP BY active;
```

**Example output**

| active | cnt |
| :----: | --: |
|    0   |   2 |
|    1   |   3 |

**Details:** `COUNT(column)` counts non-NULL values; `COUNT(*)` counts rows. Use `HAVING` to filter groups (e.g., `HAVING COUNT(*) > 10`).

* **ONLY_FULL_GROUP_BY:** MySQL's `ONLY_FULL_GROUP_BY` SQL mode (enabled by default since MySQL 5.7.5) rejects queries where non-aggregated columns in `SELECT` are not in `GROUP BY` (or functionally dependent on it). Keep it enabled for predictable behavior; without it MySQL returns an arbitrary value from the group.
* **Aggregate performance:** Aggregations over large datasets benefit from pre-aggregated tables or indexed materialized views where supported.

---

### Step 10 - Insert single row

**What & why:** Add a single record. Most common create operation.

**SQL**

```sql
INSERT INTO users (username, email, active)
VALUES ('erin', 'e@example.com', 1);
```

**Example effect**

* Return from client: `1 row inserted, id = 5` (client libraries vary).
* `SELECT * FROM users WHERE id = 5;` shows the inserted row.

**Tip:** Always use parameterized queries in app code.

* **Retrieving generated ids:** In MySQL use `LAST_INSERT_ID()` or client library features. SQLite provides `last_insert_rowid()`. Client libraries often expose convenient functions.
* **Transactional inserts:** Wrap multi-row insert sequences in transactions to ensure atomicity and better write throughput.

---

### Step 11 - Insert multiple rows (batch)

**What & why:** Efficiently insert many rows in one statement.

**SQL**

```sql
INSERT INTO users (username, email, active) VALUES
('frank','f@example.com',1),
('gina','g@example.com',0);
```

**Example effect**

* Two rows added in one round-trip.

* **Batch size:** Very large single-statement batches can exceed packet/transaction size limits - chunk inserts in reasonable sizes and monitor performance.

* **Bulk loaders:** For massive imports, use bulk-load utilities if available to avoid transactional overhead per-row.

---

### Step 12 - Update rows

**What & why:** Modify existing records. Use `WHERE` to limit scope.

**SQL**

```sql
UPDATE users SET last_login = CURRENT_TIMESTAMP WHERE id = 2;
```

**Example effect**

* DB reports `1 row updated`. The `users` row with `id=2` now has a new `last_login`.

**Pitfall:** Omitting `WHERE` updates every row.

* **Optimistic concurrency:** Use version columns (e.g., `version` integer) and `WHERE version = :old` to prevent overwriting concurrent updates (check affected rows).
* **RETURNING:** PostgreSQL, SQLite (3.35+) and MariaDB (10.5+, for `INSERT`/`DELETE`) support `RETURNING` clauses to fetch changed rows; MySQL does **not** support `RETURNING`.

---

### Step 13 - Delete rows

**What & why:** Remove records. Prefer soft-delete if you may need to restore data.

**SQL**

```sql
DELETE FROM users WHERE id = 5;
```

**Example effect**

* DB reports `1 row deleted`. The row is gone.

**Engine note:** For server engines `TRUNCATE TABLE` is fast and resets auto-increment; SQLite uses `DELETE FROM` + sequence cleanup.

* **Soft deletes:** Add a boolean `deleted` or `deleted_at` timestamp to retain history and enable undelete. Implement query filters to exclude soft-deleted rows.
* **Cascading deletes:** Be careful with `ON DELETE CASCADE` on FKs - it can remove many dependent rows unexpectedly. Prefer explicit deletes in application logic for clarity when necessary.

---

### Step 14 - Upsert (insert-or-update)

**What & why:** Insert a row, or update it if it already exists - avoid racey read→write flows.

**SQL**

```sql
-- MySQL / MariaDB
INSERT INTO kv (k,v) VALUES ('site_hits', 1)
ON DUPLICATE KEY UPDATE v = VALUES(v);

-- SQLite
INSERT INTO kv (k,v) VALUES ('site_hits', 1)
ON CONFLICT(k) DO UPDATE SET v = excluded.v;
```

**Example behavior**

* If `k='site_hits'` exists → update `v`. If not → insert.

**Pitfall:** SQLite `INSERT OR REPLACE` deletes old row and inserts new - beware of FK cascades.

* **Race conditions & atomicity:** `ON DUPLICATE KEY` / `ON CONFLICT` are atomic upserts that avoid the classic `SELECT` then `INSERT/UPDATE` race.
* **MySQL `VALUES()` deprecation:** MySQL 8.0.20+ deprecates `VALUES(col)` inside `ON DUPLICATE KEY UPDATE`; use a row alias instead: `INSERT INTO kv (k,v) VALUES ('site_hits', 1) AS new ON DUPLICATE KEY UPDATE v = new.v;` (MariaDB does not support the alias syntax and still uses `VALUES(col)`).

---

# Transactions & Concurrency (atomicity, isolation, WAL)

### Step 1 - Basic transaction control

**What & why:** Make multiple statements atomic - either all apply or none.

**SQL**

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- or on error:
ROLLBACK;
```

**Alternative starts / notes:**

```sql
START TRANSACTION;  -- equivalent to BEGIN in many engines
SET autocommit = 0; -- turn off autocommit in some clients (MySQL) then COMMIT; SET autocommit = 1;
```

**Example effect**

* Both balances change or neither do. Client receives success/failure for the transaction.

**Details:** Always use transactions for multi-step state changes.

* **Autocommit default:** Many clients default to autocommit = ON (each statement is its own transaction). Be explicit about when you need multi-statement transactions.
* **Savepoints:** Use `SAVEPOINT name` and `ROLLBACK TO SAVEPOINT name` when you need partial rollbacks inside a larger transaction.

---

### Step 2 - Isolation (server engines) and SQLite concurrency

**What & why:** Isolation controls visibility of other transactions. MySQL/MariaDB offer `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ` (InnoDB default), `SERIALIZABLE`. SQLite allows many readers, single writer; use WAL for better concurrency.

**SQL (set isolation)**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- or per-transaction:
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

**SQLite concurrency tip**

```sql
PRAGMA journal_mode = WAL;
```

**Pitfall:** Higher isolation reduces anomalies but may reduce concurrency or increase locking.

* **InnoDB behavior:** MySQL/InnoDB default `REPEATABLE READ` historically prevents some anomalies using gap locks; however, it can produce lock waits or deadlocks under contention.
* **Deadlock handling:** Applications should detect deadlock errors (specific SQLSTATE codes) and retry transactions with backoff.
* **SQLite WAL specifics:** WAL improves concurrency but increases disk usage and requires checkpointing (copying the WAL file content back into the main DB file); this happens automatically, controlled by `PRAGMA wal_autocheckpoint` (default every 1000 pages), or manually with `PRAGMA wal_checkpoint`.

---

# Indexes & Performance

### Step 1 - When to index

**What & why:** Index columns used frequently in `WHERE`, `JOIN ON`, and `ORDER BY`. Don’t index low-cardinality columns (like boolean flags) unless they appear in selective composite indexes.

**Example:** `WHERE email = ?` → index on `email`.

* **Measure first:** Use slow query logs and EXPLAIN to find real hotspots before adding indexes. Blindly adding indexes increases write cost without guaranteed benefit.
* **Partial / filtered indexes:** Some engines support indexes on a subset of rows (e.g., `WHERE deleted = 0`) - useful for sparse data.

---

### Step 2 - Composite indexes and order

**What & why:** `INDEX(a,b)` helps queries filtering on `a` or `a`+`b`, and certain `ORDER BY` patterns. Order of columns matters.

**SQL**

```sql
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);
```

**Tip:** For `WHERE user_id = ? AND created_at > ? ORDER BY created_at DESC` the composite index `user_id, created_at` is ideal.

* **Left-most prefix rule:** `INDEX(a,b)` can be used for queries on `a` or `a,b` but not just `b` alone (without special index types).
* **Include columns (covering indexes):** Some DBs allow including non-key columns into the index to create covering indexes without affecting index order.

---

### Step 3 - Use EXPLAIN to inspect plans

**What & why:** `EXPLAIN` shows whether queries use indexes or perform full table scans.

**SQL**

```sql
EXPLAIN SELECT * FROM users WHERE email = 'a@example.com';
-- SQLite
EXPLAIN QUERY PLAN SELECT * FROM users WHERE email = 'a@example.com';
```

**Example EXPLAIN output (conceptual)**

| id | select_type | table | type | key             | rows |
| -: | ----------: | ----: | ---: | --------------- | ---: |
|  1 |      SIMPLE | users |  ref | idx_users_email |    1 |

**Pitfall:** If EXPLAIN shows `ALL` for `type`, it’s a full table scan - consider indexing.

* **EXPLAIN ANALYZE:** When available, `EXPLAIN ANALYZE` actually runs the query and returns runtime metrics. Use it in staging to measure real costs.
* **Interpreting output:** Look for operations with high row estimates, temporary files, or filesort steps - these indicate potential problems.

---

# Search & Full-Text

### Step 1 - `LIKE` vs full-text

**What & why:** `LIKE` is simple but slow for wide text. Full-text search builds an inverted index for fast, relevant queries.

**SQL (LIKE)**

```sql
SELECT id FROM posts WHERE body LIKE '%search%';
```

**SQL (MySQL full-text)**

```sql
CREATE FULLTEXT INDEX ft_posts_body ON posts(body);
SELECT id, MATCH(body) AGAINST('search') AS score FROM posts WHERE MATCH(body) AGAINST('search');
```

**SQL (SQLite FTS5)**

```sql
CREATE VIRTUAL TABLE posts_fts USING fts5(title, body);
SELECT rowid, * FROM posts_fts WHERE posts_fts MATCH 'search';
```

**Example output**

| id | score |
| -: | ----: |
| 10 |  2.45 |

**Pitfall:** Full-text syntax and behavior differ across engines (stopwords, stemming).

* **Ranking & relevance:** Full-text engines produce relevance scores and support boolean and phrase queries; tune stopword lists and tokenization for your language.
* **External search engines:** For large-scale, feature-rich search (faceting, fuzzy matching, synonyms), integrate with dedicated search systems.

---

# Backups & Maintenance

### Step 1 - Backups

**What & why:** Regular backups protect against data loss.

**Commands (conceptual)**

* MySQL/MariaDB: `mysqldump --single-transaction --databases app_db > dump.sql` (logical dump; `--single-transaction` gives a consistent snapshot of InnoDB tables without locking them) or physical backups (Percona XtraBackup / MariaDB Backup).

* SQLite: copy the DB file or use `sqlite3 db.sqlite ".backup backup.sqlite"` / backup API.

* **Backup strategy:** Keep multiple retention points, test restores regularly, and store backups offsite. Consider point-in-time recovery (binlogs) for MySQL by enabling binary logging.

* **Consistent backups:** For live systems, use consistent snapshot tools (LVM/ZFS snapshots) or logical dumps taken with minimal locking options.

---

### Step 2 - Vacuum & optimize (SQLite)

**What & why:** `VACUUM` rebuilds the SQLite file, reclaiming space and defragmenting.

**SQL**

```sql
VACUUM;
```

* `VACUUM` can be slow and may need up to twice the DB size of free disk space (it builds a temporary copy); schedule during maintenance windows.
* For MySQL, `OPTIMIZE TABLE` can be used to defragment and rebuild tables (behavior depends on engine).

---

# Utilities & Diagnostics

### Step 1 - Show schema & table info

**SQL**

```sql
-- MySQL / MariaDB
SHOW CREATE TABLE users;
DESCRIBE users;

-- SQLite
SELECT sql FROM sqlite_master WHERE name='users';
PRAGMA table_info('users');
```

**Example output (small)**

| Field |    Type | Null | Key | Default |
| ----: | ------: | :--: | :-: | ------: |
|    id | INTEGER |  NO  | PRI |         |

* Use `SHOW TABLE STATUS` (MySQL) or `PRAGMA page_count` / `PRAGMA page_size` (SQLite) to inspect space usage.
* `EXPLAIN`/`EXPLAIN QUERY PLAN` are essential for query diagnostics.

---

### Step 2 - Show indexes

**SQL**

```sql
SHOW INDEX FROM users;            -- MySQL/MariaDB
PRAGMA index_list('users');       -- SQLite
```

* Use `SHOW INDEX FROM table_name` and then `SHOW CREATE TABLE` to see indexed columns and their order. In SQLite use `PRAGMA index_info(index_name)` to inspect an index.
