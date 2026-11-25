# Week 5 Notes: SQL Joins, Linux Log Investigation & End-to-End Outage Triage

## 1. SQL `LEFT JOIN` & Null Record Detection
Used when verifying whether a user has any matching billing/invoice rows (`invoices.id IS NULL` is preferred over checking nullable columns like `amount`):

```sql
SELECT
  users.id,
  users.name,
  users.email,
  users.plan,
  users.status,
  invoices.id AS invoice_id
FROM users
LEFT JOIN invoices
  ON users.id = invoices.user_id
WHERE invoices.id IS NULL;
```

---

## 2. Linux Log Investigation (`journalctl`, `grep`, `tail`, Pipes)

```bash
# View the last 20 systemd journal entries without pager lock
journalctl -n 20 --no-pager

# Filter systemd journal for errors only (last 15 lines)
journalctl -p err -n 15 --no-pager

# Find recent package upgrades (case-insensitive) and show last 10 lines
grep -i "upgraded" /var/log/pacman.log | tail -n 10

# Count occurrences of warnings in a log file
grep -i -c "warning" /var/log/pacman.log

# Search with line numbers (-n)
grep -i -n "dbeaver" /var/log/pacman.log

# Search multiple patterns with Extended Regex (-E)
grep -i -E "upgraded (systemd|linux) " /var/log/pacman.log | tail -n 5

# Filter billing log for a specific user ID (last 10 lines)
grep -i "user_id=2" /var/log/saas-billing.log | tail
```

---

## 3. End-to-End Outage Triage Drill (User ID `2`)
* **Symptom:** Customer reported upgrading to Pro, but features stayed locked and no `invoice.paid` webhook fired.
* **Database Check (`users` + `LEFT JOIN invoices`):** Confirmed `plan = 'free'`, `status = 'suspended'`, and `invoice_id = NULL`.
* **API Check:** Upgrade call (`POST /api/v1/subscriptions/upgrade`) returned **`HTTP 401 Unauthorized`** due to an expired auth token, rejecting the request before payment or webhook dispatch occurred.
* **Webhook Verification Command:**
  ```bash
  curl -i -X POST <webhook-url> \
    -H "Content-Type: application/json" \
    -d '{"event":"invoice.paid","user_id":2}'
  ```
