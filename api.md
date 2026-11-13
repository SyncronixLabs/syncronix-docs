# Public API

Base URL: `https://syncronixai.xyz`. No key is needed during the public beta; [limits](public-beta.md) apply.
All responses are JSON unless noted. Times are UTC.

## `GET /api/health`

Readiness of the agent and its database.

```json
{ "ok": true, "agent": { "configured": true, "model": "claude-sonnet-5-5", "webResearch": true },
  "database": true, "publicBeta": true, "accessKeyRequired": false, "accessConfigured": true }
```

## `GET /api/runs/quota`

What this visitor can do today.

```json
{ "publicBeta": true, "admin": false, "runsPerDay": 1, "runsLeft": 1, "busy": false,
  "budgetUsedUp": false, "budgetHeld": false, "resetsAt": "2026-11-14T00:00:00.000Z" }
```

`busy`: a run is in progress (one runs at a time). `budgetHeld`: the rest of today's budget is reserved by a run in
progress. `budgetUsedUp`: no run can start until `resetsAt`.

## `POST /api/runs`

Starts a research run and streams its progress as **Server-Sent Events** (`text/event-stream`).

Request: `Content-Type: application/json`, body `{ "task": "…" }` (8–2,000 characters, written in words).
Optional `"fresh": true` skips replaying your own identical recent run. Other fields are rejected.

The first response sets an httpOnly `sx_owner` cookie (scoped to `/api`). Send it back to see your history.

### Events

| Event | Data |
|---|---|
| `run` | `{ id, model, task, createdAt, reused? }` |
| `phase` | `{ phase }` — `intent`, `plan`, `research`, `analyze`, `deliver` |
| `thinking` | `{ text }` — summarized reasoning (also streamed as `thinking_delta`) |
| `search` / `search_results` | `{ query }` / `{ query, results: [{ title, url, age }], error? }` |
| `fetch` / `fetch_result` | `{ url }` / `{ url, title, ok, error? }` |
| `text` | `{ delta }` — the report as it is written |
| `note` | `{ text }` — e.g. source check and grounding summaries |
| `error` | `{ code, message }` |
| `done` | `{ id, result, sources, usage, costUsd, sourceCheck, searches, fetches, durationMs, reused? }` |
| `failed` | `{ id, code, message }` |

`fetches` in `done` is the number of pages actually read. Each source has `{ url, title, verified, read }`.

### Refusals (before a run starts)

| HTTP | `error` | Meaning |
|---|---|---|
| 400 | `bad_json`, `bad_request`, `bad_task` | Body is not JSON, has unexpected fields, or the task is invalid |
| 403 | `cross_site` | Started from another website |
| 413 | `too_large` | Body over 16 KB |
| 415 | `bad_content_type` | Not JSON |
| 429 | `ip_quota` | Today's free run is used; includes `resetsAt` and `Retry-After` |
| 429 | `daily_budget` | Today's shared budget is used up; includes `resetsAt` |
| 503 | `busy`, `budget_reserved` | Another run is in progress; includes `Retry-After` |

Failures during a run arrive as a `failed` event, for example `timeout`, `rate_limited`, `provider_error`,
`budget` (cost ceiling), `cancelled`.

## `GET /api/runs`

Your recent runs (identified by the `sx_owner` cookie): `{ "runs": [ … ] }`. Without the cookie the list is empty.

## `GET /api/runs/:id`

One of your runs with its stored events. Other visitors' runs return 404.

## Example

```bash
curl -N https://syncronixai.xyz/api/runs \
  -H 'content-type: application/json' \
  -d '{"task":"What is the current Node.js Active LTS line and when does it reach end of life?"}'
```

More in [syncronix-examples](https://github.com/SyncronixLabs/syncronix-examples).
