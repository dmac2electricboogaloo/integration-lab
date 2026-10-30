# integration-lab

A hands-on lab for building integration skills, one worksheet at a time. The worksheets follow a "faded example" progression: the first ones are typed exactly as written, later ones give only a spec, and the last one gives only a goal. The aim is to move from SAP Ariba / S/4HANA integration *knowledge* to integration *building* (no-code, low-code and code).

Progress report last updated: 2026-10-30.

## Status at a glance

| # | Worksheet | Level | Status |
| --- | --- | --- | --- |
| 1 | HTTP and JSON by hand (curl + Postman) | 1: type it exactly | Done (2026-10-30) |
| 2 | First script (Python + `requests`) | 1: type it exactly | Up next |
| 3 | Production habits (OAuth2, pagination, retries against a mock API) | 2: change it | Not started |
| 4 | SAP Business Partner API sandbox | 2: change it | Not started |
| 5 | First iFlow (SAP Integration Suite trial) | 2: change it | Not started |
| 6 | cXML to JSON mapping iFlow | 3: rebuild from spec | Not started |
| 7 | Low-code rebuild (Workato or n8n) | 3: rebuild from spec | Not started |
| 8 | MCP server wrapping the mock supplier API | 3: rebuild from spec | Not started |
| 9 | Capstone: supplier sync with validation, idempotency, retries, alerting, runbook | 4: goal only | Not started |

## Repository layout

```
integration-lab/
  README.md                         this report
  01-http-json/
    worksheet-1.md                  explain-it-back answers
    Worksheet 1.postman_collection.json   exported Postman collection (3 requests)
```

## Worksheet 1: HTTP and JSON by hand

**Practice API:** [JSONPlaceholder](https://jsonplaceholder.typicode.com/guide/), a free fake REST API. Writes are accepted but not saved.

### What was built

- Ran the five basic HTTP operations with `curl` against `/posts`: read (GET), create (POST), replace (PUT), partial update (PATCH) and delete (DELETE).
- Rebuilt three of those calls in Postman as a collection named `Worksheet 1`: `01 GET post 1`, `02 POST create post`, `03 DELETE delete post 1`.
- Added a Postman post-response test to the POST request: `pm.test("status is 201", ...)`. It passes.
- Exported the collection and committed it to this repo.

### Observed results

| Step | Request | Result |
| --- | --- | --- |
| 1 | `GET /posts/1` | `200 OK`, JSON with `userId`, `id`, `title`, `body` |
| 2 | `GET /posts/1` with `-i` | Status line `HTTP/1.1 200 OK`; `Content-Type: application/json; charset=utf-8`; rate-limit headers (`x-ratelimit-limit`, `x-ratelimit-remaining`) |
| 3 | `GET /posts?userId=1` | 10 records (ids 1 to 10) |
| 4 | `GET /posts/99999` | `404 Not Found`, body `{}` (`Content-Length: 2`) |
| 5 | `POST /posts` | `201 Created`, new `id` 101, `location` header pointing to `/posts/101` |
| 6 | `PUT /posts/1` (all four fields) | `200 OK`, full record returned with new values |
| 7 | `PATCH /posts/1` (title only) | `200 OK`, full record returned with only `title` changed |
| 8 | `DELETE /posts/1` | `200 OK`, body `{}` |
| 8b | `GET /posts/1` again | Still `200 OK` with the original record (the API is fake and does not persist writes) |

### Things that went wrong, and what each taught

| What happened | Error seen | Lesson |
| --- | --- | --- |
| Typed the URL without `curl` in front | `bash: https://...: No such file or directory` | The shell treated the URL as a program name. Read the error before changing anything. |
| Typed `.../typicode/posts/1` (missing `.com`) | `curl: (6) Could not resolve host` | "Could not resolve host" means check the hostname first. |
| Sent `userID` instead of `userId` | `201 Created`, field echoed back as typed | This fake API does not validate field names. A real system would reject or silently drop the field. JSON field names are case-sensitive. |
| JSON with `;` and `,` where `:` belonged | `500 Internal Server Error`, an HTML stack trace (`Expected ':' after property name in JSON at position 5`) | Malformed input should be a 400, not a 500, and error bodies are not always JSON. Integration code must check the status first and the body second. |
| Used `-X PATH` instead of `-X PATCH` | `400 Bad Request`, empty body | A 400 means the caller must fix the request; retrying will not help. A 500 may be worth retrying. |
| Used `http://` instead of `https://` | Worked, but unencrypted | Real APIs expect HTTPS. |

### Concepts covered

- **Status codes carry the outcome.** `200` means the request worked, `201` means a new record was created, `404` means the record does not exist, `400` means a bad request, `500` means a server fault.
- **A `404` can come with a perfectly valid JSON body (`{}`).** A program that only parses the body would mistake it for an empty record and report a false success. Check the status code first.
- **PUT replaces the whole record; PATCH changes only the fields you send.**
- **A success status is not proof the data changed.** The fake API returned `200` for a delete and still returned the record afterwards. Real integrations confirm with a follow-up read.
- **Headers vs fields.** Headers are the metadata lines (for example `Content-Type`). Fields are the keys inside the JSON body (for example `title`).
- **Production integrations need more than these requests had:** authentication, retries and error handling, logging and alerting, idempotency, negative testing and a staging environment.

### Explain-it-back

Answers to the four worksheet questions are in [`01-http-json/worksheet-1.md`](01-http-json/worksheet-1.md).

## Environment and tooling notes

- **Machine:** Windows, working in an MSYS2 **UCRT64** terminal (bash).
- **curl:** 8.23.0 (MSYS2 build).
- **Postman:** desktop app, free plan. In current versions, tests live under *Scripts, then Post-response* (older versions call it the *Tests* tab).
- **Python:** 3.14.7 is installed. The `requests` library is not installed yet; Worksheet 2 starts by installing it with `pacman -S mingw-w64-ucrt-x86_64-python-requests`.
- **Git:** commits use the GitHub noreply address, so a personal email address does not appear in the public history.
- **Git on Windows, two things to remember:**
  - The first push was rejected because GitHub had already created a starter commit. Fix: `git pull --rebase origin main`, then `git push`.
  - During the rebase, Windows refused to delete a folder while File Explorer had it open. Close Explorer and Notepad windows on the repo before pulling.
- **Shells:** bash commands belong in the UCRT64 terminal, not PowerShell. `cygpath` and the `curl` quoting in the worksheets only work in UCRT64.

## What's next

1. **Worksheet 2:** install `requests`, then write five short Python scripts that repeat the curl calls, check the status code before using the body, and create a record with `json=`.
2. **Worksheet 3:** a client for a local mock supplier API with OAuth2, pagination and retry with backoff. Baseball statistics (the Lahman database) are planned as the sample data wherever a sheet does not need a real ERP shape.
3. **Worksheet 4 onward:** the SAP Business Partner API sandbox, then SAP Integration Suite. The Integration Suite trial is described by SAP as a 30-day experience, so it should be started only at the beginning of Worksheet 5.
