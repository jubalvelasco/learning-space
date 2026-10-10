# SQL Practice - Questions with Answers

This file contains the same questions as the practice file, with model-correct answers provided. Use this to self-check after attempting the questions.

---

1. **Database creation:** In one sentence, explain the difference between creating a database on MySQL/MariaDB and creating/opening a database in SQLite.

**Answer:** On MySQL/MariaDB `CREATE DATABASE` makes a named namespace on the server containing tables (in MySQL/MariaDB "database" and "schema" mean the same thing); in SQLite a "database" is a file on disk, so creating/opening it is creating/opening a file rather than issuing a server-side `CREATE` statement.

2. **Charset recommendation:** Fill in the blank (one keyword): The recommended character set for full Unicode support in MySQL/MariaDB is `utf8mb4`.

3. **Create database SQL:** Write the single-line SQL command to create a database named `app_db` with `utf8mb4` and `utf8mb4_unicode_ci` collation (MySQL syntax).

**Answer:**

```sql
CREATE DATABASE app_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

4. **SQLite attach:** Provide the one-line SQLite command to attach a file `other.db` under the alias `other_db`.

**Answer:**

```sql
ATTACH DATABASE 'other.db' AS other_db;
```

5. **Permissions:** In one or two sentences, describe why application accounts should have limited privileges rather than global privileges.

**Answer:** Application accounts should have the minimal privileges necessary (principle of least privilege) to reduce blast radius if credentials are compromised and to limit accidental or malicious changes to unrelated databases or server settings.

6. **Storage engines:** Briefly explain, in a sentence, the main practical difference between InnoDB and MyISAM and which one you should prefer for modern apps.

**Answer:** InnoDB is transactional, supports row-level locking and foreign keys, while MyISAM is non-transactional with table-level locking; prefer InnoDB for modern applications.

7. **USE statement:** Provide the one-line SQL command used on server engines to set the current database to `app_db`.

**Answer:**

```sql
USE app_db;
```

8. **Cross-database queries:** In a sentence, explain how to reference a table in another database/schema on server engines.

**Answer:** Use fully-qualified names: `db_name.table_name` in MySQL/MariaDB (where database = schema), or `db_name.schema_name.table_name` in engines like SQL Server.

9. **Drop database caution:** Give two concise safety steps you should perform before running `DROP DATABASE` on a production server.

**Answer:** Take a verified backup (e.g., `mysqldump`) and disable writes or put the service into maintenance mode; verify the backup restore process and ensure replicas/replication are considered.

10. **Minimal table creation:** Write a short SQL snippet (one `CREATE TABLE` statement) that creates a minimal `items` table with an integer primary key and a NOT NULL `name` column, compatible with both SQLite and MySQL.

**Answer:**

```sql
CREATE TABLE items (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL
);
```

11. **SQLite special PK:** One sentence: what is special about `INTEGER PRIMARY KEY` in SQLite?

**Answer:** In SQLite `INTEGER PRIMARY KEY` makes the column an alias for the internal `rowid` and provides automatic integer auto-increment behavior.

12. **Autoincrement differences:** In one sentence, contrast `AUTO_INCREMENT` behavior in MySQL with the `AUTOINCREMENT` keyword in SQLite.

**Answer:** MySQL's `AUTO_INCREMENT` uses a per-table counter (before MySQL 8.0 the InnoDB counter was recalculated as `MAX(id)+1` on server restart, so ids of deleted rows at the end could be reused); SQLite's `AUTOINCREMENT` forces a monotonically increasing counter that never reuses old `rowid`s but can increase file size and is rarely necessary.

13. **Adding a column:** Provide the one-line `ALTER TABLE` command to add a `created_at` DATETIME column defaulting to the current timestamp to the `items` table.

**Answer:**

```sql
ALTER TABLE items ADD COLUMN created_at DATETIME DEFAULT CURRENT_TIMESTAMP;
```

14. **ALTER pitfalls:** In one or two sentences, explain why adding a `NOT NULL` column without a default may fail on a populated table.

**Answer:** Existing rows would have no value for the new column, violating the `NOT NULL` constraint; databases typically require a default or require the column be nullable, backfilled, and then altered to `NOT NULL`.

15. **Renaming table and column:** Give the two one-line SQL statements to rename table `items` to `products` and then rename column `name` to `title` on supported engines.

**Answer:**

```sql
ALTER TABLE items RENAME TO products;
ALTER TABLE products RENAME COLUMN name TO title;
```

16. **Dropping columns:** In a sentence, explain why dropping columns can be risky and what safer migration pattern the cheat-sheet recommends.

**Answer:** Dropping columns can be destructive and risky for large/critical apps; the cheat-sheet recommends deprecating the column, stop writing to it, backfill reads to new columns, and then remove using a controlled migration (create-copy-swap) with backups.

17. **Primary vs unique:** One sentence: what does a `PRIMARY KEY` imply that a `UNIQUE` constraint does not explicitly require?

**Answer:** A `PRIMARY KEY` implies both uniqueness and `NOT NULL`, while a `UNIQUE` constraint allows NULLs unless explicitly combined with `NOT NULL`.

18. **Foreign keys in SQLite:** Fill in the blank: To enforce foreign keys in SQLite per connection you must run `PRAGMA foreign_keys = ON;`.

19. **Index creation:** Provide a one-line SQL command to create a non-unique index on `users(username)` named `idx_users_username`.

**Answer:**

```sql
CREATE INDEX idx_users_username ON users(username);
```

20. **Index pitfalls:** In two short sentences, explain why `LIKE '%term%'` is slow and when indexes are not used.

**Answer:** Leading `%` prevents the DB from using a left-anchored index because the starting position is unknown, forcing a full scan. Indexes are also not used when implicit type conversions occur or when query predicates don't match index column order or types.

21. **SELECT * pitfall:** In one sentence, explain a problem of using `SELECT *` in production code related to schema evolution.

**Answer:** `SELECT *` returns columns in table order, so adding or removing columns can change result shape and break client code that depends on column ordering; explicit column lists are safer.

22. **Projection:** Write the one-line SQL to select only `id`, `username`, and `email` from `users`.

**Answer:**

```sql
SELECT id, username, email FROM users;
```

23. **WHERE NULL checks:** In one sentence, state how you check for NULL values in SQL.

**Answer:** Use `IS NULL` or `IS NOT NULL` (e.g., `WHERE column IS NULL`).

24. **ORDER BY determinism:** Give a short SQL example (two columns in `ORDER BY`) that produces deterministic ordering using `created_at` and `id` descending.

**Answer:**

```sql
ORDER BY created_at DESC, id DESC
```

25. **Pagination (limit/offset):** Write a simple one-line SQL that returns the first page of 10 users ordered by `created_at` descending.

**Answer:**

```sql
SELECT id, username FROM users ORDER BY created_at DESC LIMIT 10 OFFSET 0;
```

26. **Keyset pagination:** In one or two sentences, explain why keyset pagination is preferred over large offsets.

**Answer:** Keyset pagination uses the last-seen indexed value as a cursor, avoiding scanning and discarding many rows as `OFFSET` does, so it scales better and provides more consistent results under concurrent modifications.

27. **LIKE wildcard meaning:** Fill in the blanks: In `LIKE` patterns, `%` matches any sequence of characters and `_` matches any single character.

28. **INNER vs LEFT JOIN:** In one sentence each, state the difference between `INNER JOIN` and `LEFT JOIN`.

**Answer:** `INNER JOIN` returns only rows with matching keys on both sides. `LEFT JOIN` returns all rows from the left table and matching rows from the right, with NULLs when there is no match.

29. **Detect missing relations:** Provide the one-line SQL pattern (conceptual) that uses `LEFT JOIN` to find rows in `users` with no corresponding `posts`.

**Answer:**

```sql
SELECT u.* FROM users u LEFT JOIN posts p ON p.user_id = u.id WHERE p.id IS NULL;
```

30. **Aggregation basics:** Write the one-line SQL that counts users grouped by the `active` flag and names the count column `cnt`.

**Answer:**

```sql
SELECT active, COUNT(*) AS cnt FROM users GROUP BY active;
```

31. **ONLY_FULL_GROUP_BY:** In one sentence, explain what `ONLY_FULL_GROUP_BY` enforces in MySQL.

**Answer:** It enforces that any non-aggregated column in `SELECT` must appear in the `GROUP BY` clause, preventing ambiguous queries that rely on nondeterministic column selection.

32. **Insert single row:** Provide a one-line `INSERT` statement to add user `erin` with email `e@example.com` and active `1` into `users` (column list included).

**Answer:**

```sql
INSERT INTO users (username, email, active) VALUES ('erin', 'e@example.com', 1);
```

33. **Retrieve last id:** In one sentence, name the function to get the last inserted id in SQLite and MySQL respectively.

**Answer:** SQLite: `last_insert_rowid()`; MySQL: `LAST_INSERT_ID()`.

34. **Batch insert:** Provide a one-line SQL inserting two rows into `users` (`frank` and `gina`) with their emails and active flags as shown in the cheat-sheet.

**Answer:**

```sql
INSERT INTO users (username, email, active) VALUES ('frank','f@example.com',1), ('gina','g@example.com',0);
```

35. **Update caution:** One sentence: what happens if you run `UPDATE users SET last_login = CURRENT_TIMESTAMP;` without a `WHERE` clause?

**Answer:** It updates the `last_login` column on every row in the `users` table.

36. **Delete vs soft-delete:** In two sentences, explain the difference between a hard delete and a soft delete and why you might prefer soft deletes.

**Answer:** A hard delete (`DELETE`) removes rows permanently from the table, while a soft delete marks rows as deleted (e.g., `deleted` boolean or `deleted_at` timestamp) and keeps them for possible recovery or auditing. Soft deletes are preferred when you need an undo/restore path, auditability, or to avoid accidental irreversible data loss.

37. **Upsert MySQL:** Provide the one-line MySQL `INSERT ... ON DUPLICATE KEY UPDATE` statement shown in the cheat-sheet for key `site_hits`.

**Answer:**

```sql
INSERT INTO kv (k,v) VALUES ('site_hits', 1)
ON DUPLICATE KEY UPDATE v = VALUES(v);
```

38. **Transaction basics:** Write the minimal SQL sequence (BEGIN/COMMIT) to transfer 100 from account 1 to account 2 using two `UPDATE` statements as in the cheat-sheet.

**Answer:**

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

39. **Isolation levels:** List three isolation levels mentioned in the cheat-sheet that MySQL/MariaDB support.

**Answer:** `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`.

40. **WAL in SQLite:** In one sentence, explain why turning on `PRAGMA journal_mode = WAL;` can improve concurrency in SQLite and one caveat to be aware of.

**Answer:** WAL mode allows many concurrent readers while a writer appends to the WAL, reducing writer-reader contention and improving concurrency; caveat: WAL increases disk usage and requires checkpointing and careful management of auto-checkpoints.

41. **When to index:** In two short sentences, explain when you should add an index and one case where you should avoid indexing.

**Answer:** Add indexes on columns frequently used in `WHERE`, `JOIN`, and `ORDER BY` when they are selective and improve query performance. Avoid indexing very low-cardinality columns (like boolean flags) or adding indexes blindly without measuring, because each index increases write cost.

42. **Composite index left-most rule:** In one sentence, state the left-most prefix rule for composite indexes.

**Answer:** A composite index on `(a,b)` can be used for queries filtering on `a` or on both `a` and `b`, but not for queries that filter only on `b` (unless other index types exist).

43. **EXPLAIN usage:** Provide the one-line SQL to explain the query `SELECT * FROM users WHERE email = 'a@example.com';` on MySQL, and the equivalent for SQLite.

**Answer:**

MySQL:

```sql
EXPLAIN SELECT * FROM users WHERE email = 'a@example.com';
```

SQLite:

```sql
EXPLAIN QUERY PLAN SELECT * FROM users WHERE email = 'a@example.com';
```

44. **Full-text vs LIKE:** In one sentence, explain the advantage of full-text search over `LIKE` for large text columns.

**Answer:** Full-text search builds an inverted index and provides fast, relevance-ranked searches (with stemming, stopwords, and phrase support), which is far faster and more featureful than scanning large text fields with `LIKE`.

45. **Backup strategies:** List three essential elements of a good backup strategy mentioned in the cheat-sheet (short bullet-style answer acceptable).

**Answer:**

* Regular automated backups with multiple retention points.
* Offsite/isolated storage and tested restores.
* Use point-in-time recovery (binary logs) or snapshots for consistent restores on live systems.

46. **SQLite VACUUM:** One sentence: what does `VACUUM;` do in SQLite and what is a maintenance consideration when running it?

**Answer:** `VACUUM` rebuilds the SQLite database file to reclaim space and defragment it; it can be slow and may need up to twice the DB size of free disk space (it builds a temporary copy), so run during maintenance windows.

47. **Show schema:** Provide the one-line MySQL command to show table creation SQL for `users`, and the SQLite query to get the `sql` from `sqlite_master` for `users`.

**Answer:**

MySQL:

```sql
SHOW CREATE TABLE users;
```

SQLite:

```sql
SELECT sql FROM sqlite_master WHERE name='users';
```

48. **Show indexes:** Provide the single-line commands to list indexes for `users` in MySQL and in SQLite.

**Answer:**

MySQL:

```sql
SHOW INDEX FROM users;
```

SQLite:

```sql
PRAGMA index_list('users');
```

49. **Practical troubleshooting:** In two sentences, explain why running `ANALYZE TABLE` or `ANALYZE` can help query performance.

**Answer:** `ANALYZE` updates table and index statistics that the query optimizer uses to choose efficient plans, helping avoid poor plans like full table scans. Keeping statistics fresh ensures the optimizer's cost estimates reflect current data distribution.

50. **Extra credit (short SQL):** Write a one-line SQL using `GROUP BY` and `HAVING` to find usernames that appear more than once in `users` (assume `username` may not be unique).

**Answer:**

```sql
SELECT username, COUNT(*) AS cnt FROM users GROUP BY username HAVING COUNT(*) > 1;
```
