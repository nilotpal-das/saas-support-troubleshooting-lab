# Week 4 Notes: Relational Databases & Core SQL Diagnostics

## 1. Relational Schema Basics (SaaS Data Model)
- **Primary Key (`id`):** Uniquely identifies each record in a table (e.g., `users.id`).
- **Foreign Key (`user_id`):** Links a child record back to its parent record (e.g., `invoices.user_id` -> `users.id`).
- **Data Type Hygiene:** Match column types in `WHERE` clauses—use unquoted integers for IDs (`WHERE user_id = 1`) and single quotes for strings (`WHERE status = 'failed'`) to avoid type-mismatch errors or index misses in PostgreSQL/MySQL.

---

## 2. Core Diagnostic Queries (`SELECT`, `WHERE`, `ORDER BY`, `LIMIT`)

### Filter Active vs. Suspended Accounts
```sql
SELECT id, name, email, plan, status
FROM users
WHERE status = 'suspended'
LIMIT 50;
```

### Pattern Matching & Sorting
```sql
SELECT id, email, plan
FROM users
WHERE email LIKE '%@example.com'
ORDER BY id DESC
LIMIT 50;
```

---

## 3. Aggregations (`COUNT`, `SUM`, `GROUP BY`, `HAVING`)

### Count Users by Subscription Plan
```sql
SELECT plan, COUNT(*) AS user_count
FROM users
GROUP BY plan
HAVING COUNT(*) > 0;
```

### Total Paid vs. Failed Invoice Revenue
```sql
SELECT status, COUNT(*) AS invoice_count, SUM(amount) AS total_amount
FROM invoices
GROUP BY status;
```

---

## 4. Safe Read-Only Production SQL Rules
1. **Read-Only Access Only:** Never run `UPDATE`, `DELETE`, `INSERT`, `DROP`, or `ALTER` on production data without engineering approval. Configure SQL clients (like DBeaver) in **Read-only connection** mode.
2. **Always Append `LIMIT`:** Always cap exploratory queries (e.g., `LIMIT 50`) to prevent accidental full-table network dumps.
3. **Filter by Indexed IDs First:** Narrow queries using specific ticket identifiers (`id`, `user_id`, `email`) before scanning broad text fields.
4. **Select Explicit Columns:** Avoid `SELECT *` in production to prevent exposing sensitive fields (tokens, password hashes, PII) in logs or screenshots.

### Example Production-Safe Investigation Query
```sql
SELECT id, user_id, amount, status
FROM invoices
WHERE user_id = 1
LIMIT 50;
```
