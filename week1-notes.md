# Week 1 Notes: Web Basics, HAR Logs, HTTP Codes & JSON

**Setup:** Arch Linux (KDE Plasma), Firefox/Chromium DevTools, Kate  
**Completed:** Days 1–5

---

## Day 1: Client vs. Server & How APIs Fit In

When a user clicks something in a SaaS app, four things happen in order:

1. **Browser (Client)** sends an HTTP request over the network.
2. **Server (Backend)** checks login/permissions and runs the app logic.
3. **Database** pulls or saves the actual record.
4. **Server** sends back a status code (`200`, `404`, `500`, etc.) + data (HTML or JSON).

**Rules to sort issues by urgency:**

- **Frontend issue:** Broken layout, frozen buttons, stale cache, or console JS errors. Usually fixed with a hard refresh, clearing cookies, or testing in an Incognito window.
- **Backend issue:** Login loops, permission blocks, failed syncs, or `5xx` errors. Check the Network tab and grab logs.

**What an API actually is:**  
An API is just a background listener on a server that lets two programs swap data directly without a human clicking around a webpage. Instead of typing 500 orders into a dashboard by hand, a customer's script hits our API URL with a secret **API key** and sends all 500 at once.

---

## Day 2: DevTools & HAR Files

A **HAR (HTTP Archive)** file is a plain-text JSON dump of everything the browser's Network tab recorded—exact URLs, headers, payloads, load times, and status codes. Engineers ask for HAR files because "the page broke" isn't enough to reproduce a bug.

**How to capture a clean HAR:**

1. Hit `F12` -> **Network** tab.
2. Turn on **Preserve log or Persist Logs(inside gear menu)** (so redirects don't wipe the log) and **Disable cache**.
3. Click the clear/trash icon to wipe old requests, then reproduce the bug.
4. Export/Save all as `.har`.
5. *Note:* HAR files contain live session tokens and cookies—never post them in public threads.

**Lab:** Forced a `404` on a fake `/missingpage` URL, exported `day2-404-error.har`, opened it in Kate, and searched (`Ctrl + F`) for `404` to find the exact failed request block.

---

## Day 3: HTTP Status Codes

The first digit tells you immediately who owns the problem:

- **`2xx` (Worked):** `200 OK` (standard success), `201 Created` (new record saved via `POST`).
- **`4xx` (Client / Customer side — Support helps fix):**
  - `400 Bad Request`: Broken JSON syntax or missing required field in what the customer sent.
  - `401 Unauthorized`: Server doesn't know who you are—missing, expired, or mistyped login token / API key.
  - `403 Forbidden`: Server knows who you are, but your account role isn't allowed there (e.g., regular user clicking Billing Settings -> tell them to ask their workspace admin for access).
  - `404 Not Found`: Typo in the URL path, or the record ID was deleted.
  - `429 Too Many Requests`: Customer's script is firing too fast and hit the rate limit. They need to slow down or add delays between calls.
- **`5xx` (Server / Our side — Escalate to Engineering):**
  - `500 Internal Server Error` / `502 Bad Gateway` / `504 Gateway Timeout`: Our backend crashed or timed out. Customer did nothing wrong; grab the HAR/request ID and escalate.

**Lab:** Tested `401`, `403`, `429`, and `500` endpoints on `httpbin.org` in DevTools and triaged a `403` Billing Settings scenario.

---

## Day 4: Reading & Fixing JSON

APIs talk in **JSON** (`"key": value` pairs). Five types to know:

- **String:** `"open"` — *must* use double quotes. Single quotes (`'open'`) break JSON immediately.
- **Number:** `42` — no quotes.
- **Boolean:** `true` or `false` — lowercase, no quotes.
- **Object `{ }`:** A group of key-value pairs. You can nest objects inside objects (like `"address": { ... }` inside a user record).
- **Array `[ ]`:** A comma-separated list (`["sso", "urgent"]`).

**Example Ticket JSON:**
```json
{
  "ticket_id": 1042,
  "subject": "SSO login failing",
  "urgent": true,
  "tags": ["sso", "enterprise"],
  "customer": {
    "company": "Acme Corp",
    "plan": "Enterprise"
  }
}
```

**The 3 typos that cause 90% of `400 Bad Request` errors:**

1. Using single quotes `'key'` instead of double quotes `"key"`.
2. Forgetting the comma `,` at the end of a line when another line follows.
3. Leaving a trailing comma `,` on the very last item before `}` or `]`.

**Lab:** Inspected nested `address` and `company` objects on `://typicode.com` and broke/fixed all three syntax errors on `jsonlint.com`.

---

## Day 5: Filtering Live API Calls & Git Setup

- **Catching background calls:** Modern apps load the page shell once and fetch data silently in the background. In DevTools -> **Network**, clicking the **`XHR`** (Firefox) or **`Fetch/XHR`** (Chromium) filter hides images, fonts, and CSS so only live API calls show up.
- **Lab:** Filtered `XHR` traffic on `en.wikipedia.org/wiki/Main_Page` while typing in the search bar and caught the background search call (`GET` -> `200`).
- **Git:** Ran `git init` in `~/saas-support-lab` to start tracking lab notes before pushing to GitHub.
