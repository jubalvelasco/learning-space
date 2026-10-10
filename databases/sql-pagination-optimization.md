# SQL Pagination Optimization

## 1) Basic pagination query

### Example 1: order by one column

```sql
SELECT id, title, created_at
FROM posts
ORDER BY created_at DESC
LIMIT 20 OFFSET 40;
```

### Example 2: order by two columns

```sql
SELECT id, title, created_at
FROM posts
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 40;
```

**Why it is slow:** the database still has to scan and sort a large portion of the result set before it can skip the first 40 rows. As the offset grows, the query usually gets slower because more rows must be read and discarded.

## 2) Deferred join pagination

Deferred join works in two steps. First, the database runs the inner query to find just the IDs for the requested page. That inner query can often use the ordering index efficiently and returns a small list of row identifiers instead of full rows. Then the outer query joins those IDs back to the main table to fetch the full records only for that small page. This avoids reading a large amount of data for rows that will be skipped.

### Example 1: order by one column

```sql
SELECT p.id, p.title, p.created_at
FROM posts p
JOIN (
  SELECT id
  FROM posts
  ORDER BY created_at DESC
  LIMIT 20 OFFSET 40
) page ON page.id = p.id
ORDER BY p.created_at DESC;
```

### Example 2: order by two columns

```sql
SELECT p.id, p.title, p.created_at
FROM posts p
JOIN (
  SELECT id
  FROM posts
  ORDER BY created_at DESC, id DESC
  LIMIT 20 OFFSET 40
) page ON page.id = p.id
ORDER BY p.created_at DESC, p.id DESC;
```

**What happens here:** the subquery inside `JOIN (...)` produces a temporary result set containing only the IDs for page 3, for example. The database then matches those IDs against `posts` in the outer query and returns the full rows. This is useful when the original table has many columns, large text fields, or other expensive data that you do not want to load for every skipped row.

**JOIN (...)** means joining against a derived table or subquery. In this case, `page` is not a real table; it is the result of the query inside the parentheses, treated like a table for the rest of the statement.

**ON** defines the matching rule for the join. `page.id = p.id` tells the database to pair each ID from the derived page result with the row in `posts` that has the same ID. Without the `ON` condition, the database would not know which rows belong together.

**When to use it:** use deferred join when the table is wide or expensive to read, but you still need offset-based paging. The subquery first finds only the page of IDs, then the outer query fetches the full rows for just those IDs.

## 3) Keyset pagination (cursor-based pagination)

### Example 1: order by one column

```sql
SELECT id, title, created_at
FROM posts
WHERE created_at < '2026-03-24 10:00:00'
ORDER BY created_at DESC
LIMIT 20;
```

### Example 2: order by two columns

```sql
SELECT id, title, created_at
FROM posts
WHERE (created_at < '2026-03-24 10:00:00')
   OR (created_at = '2026-03-24 10:00:00' AND id < 12345)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

In PostgreSQL and MySQL/MariaDB the same condition can be written more compactly with a row value comparison (PostgreSQL can use the `(created_at, id)` index directly for this form):

```sql
WHERE (created_at, id) < ('2026-03-24 10:00:00', 12345)
```

**When to use it:** use keyset pagination when you need fast, stable paging over large or changing datasets. It avoids large offsets and usually performs much better for deep pages. The ordering columns must be unique together (that's why `id` is added as a tie-breaker), otherwise rows with the same `created_at` could be skipped or repeated.

## 4) Indexing to speed things up

Indexes help the database find the ordered rows faster and avoid full scans. For pagination queries, a composite index that matches the `ORDER BY` clause is often best.

### Example 1: one-column ordering

```sql
CREATE INDEX idx_posts_created_at_desc ON posts (created_at DESC);
```

### Example 2: two-column ordering

```sql
CREATE INDEX idx_posts_created_at_id_desc ON posts (created_at DESC, id DESC);
```

For keyset pagination, this kind of index is especially helpful because the database can jump directly to the cursor position and continue reading in order.

NOTE: a B-tree index can also be scanned backwards, so a plain ascending index `(created_at, id)` works for `ORDER BY created_at DESC, id DESC` too. The `DESC` in the index definition only matters when the columns are ordered in mixed directions (e.g. `ORDER BY created_at DESC, id ASC`).

## Which method fits which navigation style?

* **Jump to a specific page number:** basic pagination or deferred join are the better fits, because they work with `OFFSET` and can request page 1, page 2, page 50, and so on directly.
* **Next / previous page navigation:** keyset pagination is usually the best choice, because it uses a cursor from the current page instead of counting through earlier rows. It is faster and more stable for moving forward or backward through large result sets.

In practice, `OFFSET` is simpler for direct page numbers, while keyset pagination is usually better for infinite scroll, “next page”, and “previous page” buttons.
