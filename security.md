# Security model

## Assets

The AI provider API key, the blockchain indexer API key and database credentials (server-side only), the daily budget, visitors' private run
history, and the availability of the service.

## Controls

| Threat | Control |
|---|---|
| Secret exposure | Keys live only in a root-owned server environment file; never in client bundles, responses or logs. Automated checks scan built assets and API responses for secrets. |
| Budget abuse | Atomic admission in PostgreSQL: daily budget including reserves of runs in progress, one run at a time, one run per visitor per day, $0.20 ceiling per run. Concurrency tests confirm no over-admission under simultaneous requests. |
| Address rotation | IPv6 limits apply per /64 prefix. |
| Cross-site use of a visitor's run | `POST` requires the site's own origin (Origin / Sec-Fetch-Site); JSON-only bodies. |
| Malformed or hostile input | Field allow-list, size limits, task validation (control, zero-width and bidi characters stripped), parameterised SQL. |
| Reading other visitors' runs | History scoped to a random httpOnly owner token (hashed at rest); other runs return 404, indistinguishable from missing. Internal columns are never returned. |
| Prompt injection via web pages | Web content is declared untrusted in the system prompt; the agent has no tools that act, only search and fetch run by the AI provider. |
| Injected HTML in reports | Reports render as Markdown without raw HTML; only http(s) links; external links open with `noopener`. |
| Admin key guessing | Constant-time comparison; 10 failures from one network lock it out for 15 minutes; a wrong key is never treated as a visitor. |
| Transport and browser | HTTPS with HSTS, Content-Security-Policy, X-Frame-Options DENY, nosniff, strict referrer policy, nginx request rate limiting. |
| Data loss | Daily database backups with 14-day retention; immutable releases with one-step rollback. |
| Indexer key exposure | The Blockscout key is added to upstream requests on the server only. It is never sent to browsers, returned in responses or written to logs; automated tests check responses for it. |
| Indexer misuse (SSRF, cost) | Fixed allowlist of indexer paths and validated query parameters; per-lookup call budgets, a daily call allowance and a floor on the indexer's remaining credits; 30 lookups and 6 traces per minute per visitor, 3 traces at once. |
| Unwanted AI spend | Wallet data and graphs never call a model. `GET /api/chain/analyze` returns an estimate only; `POST` is same-site only, sent only from an explicit click, and admitted atomically against the per-visitor and daily budgets. |
| Hallucinated wallet facts | The AI receives only a fact sheet built in code; links and addresses in its answer that are not in the fact sheet are removed before display. Labels from explorers are sanitised and treated as data. |
| Wallet transfers | No server keys; the RPC client allows read methods only, so the server cannot sign or broadcast. The user's wallet signs after an explicit confirmation; one wallet request per click; previewed chain, account and values enforced; mainnet capped at 0.05 ETH, contract recipients refused, enabled only by two separate server settings. [Details](wallet-transfers.md). |

## Known limitations

- An actor controlling many networks could use up the shared daily budget, which locks the service for others until
  00:00 UTC. Spend can never exceed the budget.
- Visitors behind one shared address share one daily run.
- Indexed wallet data comes from a third party (Blockscout). It is cross-checked against the official RPC, but
  completeness depends on the indexer; mainnet internal transactions are currently reported as still processing.
- Wallet lookups are cached and counted in server memory. The indexer's own remaining-credit count, read on every
  call, is the backstop for the daily allowance.

## Reporting a vulnerability

Please report privately through GitHub: on the relevant SyncronixLabs repository, open **Security → Report a
vulnerability**. Do not open a public issue for security problems. Scope and what to include: [security policy](.github/SECURITY.md).

The controls above are tested by the project's own automated security, concurrency and browser suites. No external
security audit is claimed.
