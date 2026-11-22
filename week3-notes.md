# Week 3 Lab Notes (Days 11–15): Bruno API Client, Auth/JWTs, Webhooks & Rate Limits

**Setup:** Arch Linux (KDE Plasma), Bruno v4.3.0, Firefox/Chromium, Konsole (`curl`), `jwt.io`, `webhook.site`  
**Completed:** Days 11–15

---

## Day 11: Visual API Testing with Bruno
* Installed **Bruno** (local-first, Git-friendly open-source API client) on Arch Linux and created the **`SaaS Support API Lab`** collection inside `~/saas-support-lab/bruno-collection`.
* Saved and verified four core CRUD requests (`GET Get User 1`, `POST Create Ticket Post`, `PATCH Update Ticket Title`, and `DELETE Delete Ticket`), confirming `201 Created` on `POST` and `200 OK` on `GET`/`PATCH`/`DELETE`.
* Observed that selecting **JSON** in Bruno's Body tab automatically attaches the `Content-Type: application/json` header—preventing the blank-record bug reproduced in Day 9.

## Day 12: Environment Variables (`Sandbox` vs. `Production`)
* Created a **`Test`** environment with `base_url = https://typicode.com` (no trailing slash) and replaced hardcoded domains across all requests with **`{{base_url}}`**.
* **Why it matters:** Support engineers switch environments (`Test/Sandbox` vs. `Production`) with one dropdown click rather than editing URLs manually, preventing accidental edits to live customer data.

## Day 13: API Authentication & Decoding JWTs
* Mapped the three primary authentication methods behind `401 Unauthorized` tickets:
  1. **API Key (`X-API-Key`):** Static secret key; breaks when a Sandbox key is used in Production or extra whitespace is copied.
  2. **Bearer Token / JWT (`Authorization: Bearer <token>`):** Signed token containing user identity and expiry; breaks when the `Bearer ` prefix is omitted or the token expires.
  3. **OAuth 2.0:** App-to-app delegated access; breaks when an admin revokes permissions or a refresh token expires.
* **Lab (`jwt.io`):** Decoded a sample JWT payload (`email`, `role`, `iat`, `exp`) and verified how converting the **`exp` (Expiration)** Unix timestamp immediately confirms whether a customer's `401` error is caused by an expired token—without needing the server's secret key.

## Day 14: Webhooks (Reverse APIs) & Live Payload Testing
* Contrasted **API Polling** (client repeatedly asks the server for updates, wasting rate limits) with **Webhooks** (server pushes an instant HTTP `POST` JSON event to a customer's callback URL the moment an event occurs).
* **Lab (`webhook.site` + `curl`):** Generated a temporary callback URL and fired a simulated `invoice.paid` JSON payload (`amount: 4999`, `currency: INR`) from Konsole, verifying live header and JSON body delivery in the browser.

## Day 15: Pagination, Rate Limits (`429`) & GitHub Push
* **Pagination (`?page=2&per_page=100`):** When a customer reports that an API list endpoint returned only 100 records instead of thousands, the API is paginating results to protect performance; instruct them to iterate through subsequent pages using `page` or cursor parameters.
* **Rate Limiting (`429 Too Many Requests`):** Occurs when automated scripts exceed allowed requests per minute; instruct the customer to inspect the **`Retry-After`** response header and add backoff delays between calls.
* **Git:** Committed the `bruno-collection/` folder and `week3-notes.md` to `saas-support-troubleshooting-lab`.
