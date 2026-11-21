# REST API Testing & Debugging Cheatsheet (`curl` + `jq`)

A practical command-line reference for testing REST API endpoints, inspecting HTTP headers, filtering JSON payloads, and reproducing common customer API bugs.

---

## 1. Basic `GET` Requests & Headers

* **Fetch raw response body** (sends full HTML or JSON payload to the terminal):
  ```bash
  curl https://jsonplaceholder.typicode.com/users/1
  ```

* **Include HTTP status line and response headers (`-i`):**
  ```bash
   curl -i https://jsonplaceholder.typicode.com/users/1 
  ```

* **Pretty-print JSON cleanly (`-s` hides progress bar/errors, piped into `jq`):**
  ```bash
  curl -s https://jsonplaceholder.typicode.com/users/1 | jq
  ```

---

## 2. Filtering JSON Arrays with `jq`

* **Extract a specific array item (`'.[n]'` returns the `(n+1)`th item):**
  ```bash
  curl -s "https://jsonplaceholder.typicode.com/users" | jq '.[1]'
  ```

* **Extract specific fields across an entire list:**
  ```bash
  curl -s "https://jsonplaceholder.typicode.com/users" | jq '.[] | {name: .name, email: .email}'
  ```

---

## 3. Modifying Data (`POST`, `PATCH`, `DELETE`)

Use `-X` to specify the HTTP method, `-H` to set request headers, `-d` to attach the JSON payload, and `\` to split long commands across readable lines.

* **Create a new record (`POST` — expects `201 Created`):**
  ```bash
  curl -i -X POST https://jsonplaceholder.typicode.com/posts \
    -H "Content-Type: application/json" \
    -d '{"title":"Login Bug","body":"401 on SSO","userId":101}'
  ```

* **Partially update an existing record (`PATCH` — expects `200 OK`):**
  ```bash
  curl -i -X PATCH https://jsonplaceholder.typicode.com/posts/1 \
    -H "Content-Type: application/json" \
    -d '{"title":"Resolved: Login Bug"}'
  ```

* **Delete a specific record (`DELETE` — expects `200 OK`):**
  ```bash
  curl -i -X DELETE https://jsonplaceholder.typicode.com/posts/1
  ```

---

## 4. Common Customer API Bugs & Triage Notes

| Symptom / Error | Root Cause | Fix |
| :--- | :--- | :--- |
| **`200 OK` / `201 Created`, but record is blank or unchanged** | Omitted `-H "Content-Type: application/json"`. The server treats `-d` as raw form text, so the JSON parser ignores the fields. | Always include `-H "Content-Type: application/json"` when sending a JSON payload. |
| **`400 Bad Request` or `500 Internal Server Error` on `POST`/`PATCH`** | Malformed JSON syntax inside `-d` (e.g., a trailing comma before `}`, single quotes, or a missing comma). | Validate the JSON payload structure before sending. |

