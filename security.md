# Security model

## Assets

The AI provider API key and database credentials (server-side only), the daily budget, visitors' private run
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

## Known limitations

- An actor controlling many networks could use up the shared daily budget, which locks the beta for others until
  00:00 UTC. Spend can never exceed the budget.
- Visitors behind one shared address share one daily run.

## Reporting a vulnerability

Please report privately through GitHub: on the relevant SyncronixLabs repository, open **Security → Report a
vulnerability**. Do not open a public issue for security problems.
