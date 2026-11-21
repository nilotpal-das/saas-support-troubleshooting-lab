# Week 2 Lab Notes (Days 6–10): REST APIs & `curl` Diagnostics

## Day 6: REST APIs, URL Structure & Query Parameters
* Mapped standard app actions (CRUD) to HTTP methods: `POST` (Create), `GET` (Read), `PUT` (Full Update), `PATCH` (Partial Update), and `DELETE` (Remove).
* Broke down API URLs (`https://jsonplaceholder.typicode.com/posts/1?status=open&limit=10`):
  * **Subdomain (`jsonplaceholder.`):** Points to the dedicated API server rather than the main site (`typicode.com`).
  * **Collection (`/posts`) vs. Item ID (`/1`):** `POST` targets `/posts` since no ID exists yet, while `PATCH` and `DELETE` require `/1` to specify which row to change.
  * **Query Parameters (`?status=open&limit=10`):** Filters joined by `&`. Always wrap URLs containing `&` in quotes in Linux terminals.
* **Security Rule:** Never put passwords or API keys in URL query parameters (like `?passwd=...`)—they get logged in plain text in browser history, server logs, and `.har` files. Keep credentials in HTTP headers.

## Day 7: Live `GET` Requests & `jq` Filtering
* Used `curl -i` to inspect HTTP status lines (`HTTP/2 200`) and response headers alongside the JSON body.
* Used `curl -s` piped into `jq` to format JSON cleanly, pull individual array items (`jq '.[0]'`), and extract specific fields across a list (`jq '.[] | {name: .name, email: .email}'`).

## Day 8: Modifying Records (`POST`, `PATCH`, `DELETE`)
* Verified expected status codes in Konsole: **`POST` returned `201 Created`** (generating `"id": 101`), while **`PATCH` and `DELETE` returned `200 OK`**.
* Observed that JSONPlaceholder is a stateless mock API—it simulates responses without saving changes to a real database, so refreshing `/posts` still shows only the original 100 items.

## Day 9: Deliberate API Failure Labs
* **Missing `Content-Type` Header:** Omitting `-H "Content-Type: application/json"` still returns `201`/`200`, but `curl` sends `-d` as plain form text so the server ignores the payload and leaves the record blank/unchanged.
* **Broken JSON Syntax:** Keeping `-H` but sending a trailing comma (`'{"title":"Test",}'`) crashes the server's JSON parser with **`HTTP/2 500`**.
* **Customer Explanation (Missing Header Fix):**
  > *"Your `POST` request succeeded with `201 Created`, which is why no error appeared, but our server ignored your JSON fields because the `Content-Type` header was missing. Without that header, `curl` sends the payload as plain text rather than JSON, creating a blank record. Please add `-H "Content-Type: application/json"` to your request and try again so your fields save properly."*

## Day 10: GitHub Portfolio Setup
* Organized Week 1–2 diagnostic files (`curl-cheatsheet.md` and `week2-notes.md`) in `~/saas-support-lab` and pushed them to my public GitHub repository (`saas-support-troubleshooting-lab`).

