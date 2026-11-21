# Public API

Base URL: `https://syncronixai.xyz`. No key is needed; [limits](limits.md) apply.
All responses are JSON unless noted. Times are UTC.

## Who can call what

The API is public: no key and no account. It is meant for terminals, scripts and your own server. The server sends no
CORS headers, and `POST` endpoints accept only requests from syncronixai.xyz itself or requests without browser
origin headers, so **web pages on other websites cannot call it from a visitor's browser**.

| Endpoint | curl / your server | Web page on another site | Calls a model |
|---|---|---|---|
| `GET /api/health`, `/api/runs/quota`, `/api/chain` | ✓ | ✗ (no CORS) | No |
| `POST /api/runs` | ✓ | ✗ `403 cross_site` | Yes, admitted against your daily research run and the research budget |
| `GET /api/runs`, `/api/runs/:id` | ✓ with your `sx_owner` cookie | ✗ | No |
| `GET /api/chain/wallet`, `GET /api/chain/analyze` | ✓ | ✗ (no CORS) | No |
| `POST /api/chain/analyze` | ✓ | ✗ `403 cross_site` | Yes, admitted against your daily AI wallet analyses and their budget |
| `POST /api/chain/execute` | ✓ (preview only) | ✗ `403 cross_site` | No; never signs or sends |

## `GET /api/health`

Readiness of the agent and its database. `publicBeta` in this and the quota response is a legacy field name kept for
existing clients; it means the free public tier is on.

```json
{ "ok": true, "agent": { "configured": true, "model": "claude-sonnet-5-5", "webResearch": true },
  "database": true, "publicBeta": true, "accessKeyRequired": false, "accessConfigured": true }
```

## `GET /api/runs/quota`

What this visitor can do now. The top-level fields describe research runs; `research` and `walletAi` describe both
kinds.

```json
{ "publicBeta": true, "admin": false, "runsPerDay": 1, "runsLeft": 1, "busy": false,
  "budgetUsedUp": false, "budgetHeld": false, "paced": false, "blocked": null, "resumesAt": null,
  "resetsAt": "2026-11-22T00:00:00.000Z",
  "research": { "perDay": 1, "left": 1, "available": true, "blocked": null, "resumesAt": null },
  "walletAi": { "perDay": 3, "left": 3, "available": true, "blocked": null, "resumesAt": null } }
```

- `blocked` is why that kind can't start now, or `null`: `busy`, `ip_quota`, `subnet_quota`, `visitor_budget`,
  `daily_budget`, `budget_reserved`, `budget_paced` or `monthly_budget` (see the refusal table below).
- `resumesAt` is when that limit lifts: the next budget release, the next 00:00 UTC, or the next month.
- `budgetUsedUp`, `budgetHeld` and `paced` repeat research's `blocked` as booleans, for older clients.

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
| 429 | `ip_quota` | Today's research run is used |
| 429 | `subnet_quota` | Your wider network (IPv4 /24, IPv6 /48) has used today's 6 AI runs |
| 429 | `visitor_budget` | Your network has used today's $0.30 of AI usage |
| 429 | `budget_paced` | Today's research budget is released in steps; `resetsAt` is the next step |
| 429 | `daily_budget` | Today's research budget is used up |
| 429 | `monthly_budget` | This month's $60 AI budget is used up; `resetsAt` is the 1st of next month |
| 503 | `busy`, `budget_reserved` | Another run is in progress, or the rest of the budget is held by work in progress; includes `Retry-After` |

Every 429 includes `resetsAt` and `Retry-After`.

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
  "quota": { "kind": "wallet_ai", "runsPerDay": 3, "runsLeft": 3, "busy": false, "budgetAvailable": true,
             "blocked": null, "resumesAt": null, "resetsAt": "…" } }
```

### `POST /api/chain/analyze`

Starts **one** billed AI analysis. The console sends it only when a visitor clicks **Analyze with AI**.
Body: `{ "network", "address", "compare"?, "fresh"? }` (JSON, same-site only). It is admitted like a research run:
it uses one of the visitor's 3 daily AI wallet analyses and the AI wallet analysis budget, separate from research. Response:
`{ id, result, costUsd, estimatedUsd, model, usage, check: { removedLinks, removedAddresses, links }, durationMs }`,
or `{ id, reused: true, result, costUsd: 0 }` when your identical analysis of unchanged data is replayed.

### `POST /api/chain/execute`

Preview of one wallet transfer. **Read-only: never signs or sends.** Same-site only.
Body (JSON, strings only): `{ "network", "from", "text" }` or `{ "network", "from", "to", "amount" }`.
Response: `{ intent, quote }` with `quote` = `{ network, chainId, from, to, valueWei, amountEth, gasEstimate, gasLimit,
maxFeePerGasWei, maxPriorityFeePerGasWei, estFeeEth, maxFeeEth, totalMaxEth, balanceEth, balanceAfterMinEth,
recipientIsContract, recipientUnused, warnings[], expiresAt }`.

Refusals, as `{ "error": code, "message": … }`:

| HTTP | `error` | Cause |
|---|---|---|
| 400 | `bad_network` | `network` is not `mainnet` or `testnet` |
| 403 | `execution_disabled`, `mainnet_disabled` | Transfers are switched off on that network |
| 400 | `bad_sender` | `from` is not a valid address (no wallet connected) |
| 400 | `empty`, `too_long`, `unrecognized` | No instruction, over 200 characters, or not one of the supported sentence forms |
| 400 | `unsupported`, `not_eth`, `multiple`, `ens` | Swaps, `all`/`max`, schedules and conditions; tokens; more than one recipient; `.eth` names |
| 400 | `bad_address`, `bad_checksum`, `zero_address` | Malformed recipient, failed [EIP-55](https://eips.ethereum.org/EIPS/eip-55) checksum, or the zero address |
| 400 | `bad_amount`, `over_limit` | Not a positive amount with up to 18 decimals; above the network's cap |
| 400 | `network_mismatch`, `self_transfer` | The instruction names the other network; sender and recipient are the same |
| 400 | `contract_recipient`, `insufficient_funds`, `estimate_failed` | After reading the chain: contract recipient on mainnet; balance below the amount plus the most the fee can cost; the network would reject it |
| 400 | `bad_json`, `bad_request` | Body is not JSON, has unexpected fields, or mixes `text` with `to`/`amount` |
| 403 | `cross_site` | Sent from a web page on another website |
| 415 | `bad_content_type` | Not `application/json` |
| 429 | `rate_limited` | Over 30 chain lookups per minute; includes `Retry-After` |
| 413 | `too_large` | Body over the size limit |
| 502 / 503 | `rpc_unavailable`, `disabled` | The official RPC could not be read; chain reads are switched off |

Every code above except `execution_disabled`, `mainnet_disabled`, `contract_recipient`, `insufficient_funds`,
`estimate_failed`, `rate_limited`, `too_large`, `rpc_unavailable` and `disabled` is checked against production by the
[API conformance benchmark](https://github.com/SyncronixLabs/syncronix-benchmarks#api-conformance). Details:
[wallet-transfers.md](wallet-transfers.md).

```bash
curl -s https://syncronixai.xyz/api/chain/execute -H 'content-type: application/json'   -d '{"network":"mainnet","from":"0x…","text":"Send 1 ETH to 0x…"}'
# {"error":"over_limit","message":"The limit is 0.05 ETH per transfer."}

curl -s https://syncronixai.xyz/api/chain/execute -H 'content-type: application/json'   -d '{"network":"testnet","from":"0x…","text":"Swap 1 ETH for USDC"}'
# {"error":"unsupported","message":"Only single, immediate ETH transfers are supported: no swaps, contracts, “all/max” amounts or scheduled transfers."}
```

`GET /api/chain?config=1` reports which networks accept transfers and their caps:
`{ "execution": { "testnet": true, "mainnet": true, "maxEth": { "testnet": "1", "mainnet": "0.05" } }, "networks": [ … ] }`.

### Wallet errors

| HTTP | `error` | Meaning |
|---|---|---|
| 400 | `bad_address`, `bad_compare` | Not a 0x address, or more than 2 compare addresses |
| 403 | `cross_site` | Analysis started from another website |
| 429 | `rate_limited` | Over 30 chain lookups or 6 traces per minute; includes `Retry-After` |
| 429 / 503 | `ip_quota`, `subnet_quota`, `visitor_budget`, `budget_paced`, `daily_budget`, `monthly_budget`, `busy`, `budget_reserved` | Analysis not admitted. Same codes as for research runs, but checked against the AI wallet analysis allowance (3 per day) and budget ($0.25 per day) |
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
