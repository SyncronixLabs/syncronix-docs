# Public API

Base URL: `https://syncronixai.xyz`. No key is needed; [limits](limits.md) apply.
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

## Wallet Intelligence

Read-only. Explorer links in responses point to **Robinscan** (`https://robin.etherscan.io`) on mainnet and to the
testnet explorer on testnet. Method and limits: [wallet-intelligence.md](wallet-intelligence.md).

### `GET /api/chain/wallet?network=mainnet|testnet&address=0x…&view=intel`

Activity, computed without AI. Main fields:

| Field | Content |
|---|---|
| `account` | `balanceEth`, `nonce`, `isContract` from the official RPC |
| `coverage` | `transactions`, `transactionsComplete`, `internalTransfers`, `internalComplete`, `internalPending`, `from`, `to` |
| `flows` | `inEth`, `outEth`, `netEth`, top-level / internal split, `feesEth`, `txIn`, `txOut`, `failed` |
| `contracts`, `counterparties`, `tokens` | Grouped interactions, each with an explorer `url` |
| `transactions`, `internalTransfers`, `timeline` | Recent items, each with an explorer `url` |
| `crossCheck` | `{ ok, reason, url }`: the newest transaction re-read from the official RPC |
| `source` | `{ indexer, rpc, indexerCalls }` |

### `GET /api/chain/wallet?network=…&address=0x…&view=trace[&compare=0x…,0x…]`

Funding trace, computed without AI. Up to 2 addresses in `compare`.

| Field | Content |
|---|---|
| `steps[]` | One per hop: `hop`, `to`, `funder`, `kind` (`transfer`, `internal`, `bridge_deposit`, `unattributed`), `valueEth`, `ts`, `url`, `verifiedBy` (`rpc`, `indexer+rpc_tx`, `indexer`), `proven`, `note`; bridge deposits add `l1Sender` |
| `proven`, `note` | Whether the first funder is proven, and why not |
| `timeline[]` | The address's earliest ETH inflows |
| `stop`, `stopReason` | Why the trace ended: `max_hops`, `contract`, `bridge`, `system`, `cycle`, `never_funded`, `not_found`, `no_history`, `unattributed`, `budget` |
| `shared` | `{ funder, sampled, distinctRecipients, hub, wallets[] }`: other wallets the first funder paid (**inferred**) |
| `compared[]` | `{ address, steps, stopReason, common[] }`: shared upstream funders are **inferred** |
| `graph` | `nodes[]` (`target`, `funder`, `sibling`, `compare`, `compare_funder`, `shared`) and `edges[]` with `evidence`: `verified` or `inferred` |

Abridged response (testnet, addresses shortened):

```json
{ "network": "testnet", "chainId": 46630, "proven": true,
  "steps": [ { "hop": 1, "funder": "0x38D0…c101", "to": "0xa7F3…7DA2", "kind": "transfer", "valueEth": "0.03",
               "verifiedBy": "rpc", "proven": true, "url": "https://explorer.testnet.chain.robinhood.com/tx/0x0073…" } ],
  "stop": "max_hops", "stopReason": "Stopped after 5 hops, the maximum.",
  "shared": { "sampled": 50, "distinctRecipients": 7, "hub": false },
  "graph": { "nodes": [ … 12 ], "edges": [ … 11 verified, 6 inferred ] } }
```

### `GET /api/chain/analyze?network=…&address=0x…[&compare=…]`

The cost estimate for an AI analysis. **Makes no model call.**

```json
{ "model": "claude-haiku-5-5", "inputTokens": 4361, "maxOutputTokens": 3000, "estimatedUsd": 0.00114,
  "maxUsd": 0.00202, "reserveUsd": 0.005, "facts": { "lines": 29, "links": 15 }, "aiConfigured": true,
  "quota": { "runsPerDay": 1, "runsLeft": 1, "busy": false, "budgetAvailable": true, "resetsAt": "…" } }
```

### `POST /api/chain/analyze`

Starts **one** billed AI analysis. The console sends it only when a visitor clicks **Analyze with AI**.
Body: `{ "network", "address", "compare"?, "fresh"? }` (JSON, same-site only). It is admitted like a research run:
it uses the visitor's daily run and the shared daily budget. Response:
`{ id, result, costUsd, estimatedUsd, model, usage, check: { removedLinks, removedAddresses, links }, durationMs }`,
or `{ id, reused: true, result, costUsd: 0 }` when your identical analysis of unchanged data is replayed.

### Wallet errors

| HTTP | `error` | Meaning |
|---|---|---|
| 400 | `bad_address`, `bad_compare` | Not a 0x address, or more than 2 compare addresses |
| 403 | `cross_site` | Analysis started from another website |
| 429 | `rate_limited` | Over 30 chain lookups or 6 traces per minute; includes `Retry-After` |
| 429 / 503 | `ip_quota`, `daily_budget`, `busy`, `budget_reserved` | Analysis not admitted, as for research runs |
| 503 | `busy` | 3 traces already running; includes `Retry-After` |
| 503 | `daily_limit`, `budget`, `indexer_unavailable`, `indexer_rate_limited` | Today's indexed-read allowance is used, or the indexer is unavailable |
| 502 | `indexer_unreachable`, `indexer_error` | The indexer could not be reached or returned an error |
| 502 | `incomplete` | The AI answer was empty or cut off; recorded as failed with its real cost |

## Example

```bash
curl -N https://syncronixai.xyz/api/runs \
  -H 'content-type: application/json' \
  -d '{"task":"What is the current Node.js Active LTS line and when does it reach end of life?"}'
```

More in [syncronix-examples](https://github.com/SyncronixLabs/syncronix-examples).
