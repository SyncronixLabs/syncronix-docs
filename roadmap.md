# Roadmap

Status as of November 2026. Items move from *planned* to *live* only when they run in production.

## Live

- **Research agent, free** — web search, page reading, cross-checking, grounded reports with source
  verification, live progress streaming, private per-browser history.
- **Multi-agent runs** — Researcher, Analyst and Verifier on every run, inside one per-run cost ceiling.
- **Output verification** — the Analyst extracts checkable claims with verbatim quotes; the Verifier matches each
  quote against the fetched page text in code, cross-checks numbers and dates across independent domains, and an
  independent model call judges support from that evidence only.
- **Robinhood Chain reads** — read-only access through the official public RPC (mainnet chain ID 4663, testnet
  46630), with a fixed allowlist of read methods and the endpoint's chain ID checked before any data is used.
- **Wallet Intelligence** — activity for any Robinhood Chain address: transaction history, ETH inflows and outflows
  including internal transfers, contract interactions, counterparties, with an RPC cross-check and Robinscan links.
- **Funding Trace** — first funder with proof status, funding timeline, upstream path up to 5 hops, shared-funder
  analysis, comparison with up to 2 addresses, interactive graph separating verified transfers from inferred links.
- **AI wallet analysis (opt-in)** — a brief written from the deterministic data, only after an explicit click and a
  cost estimate, inside the same per-visitor and daily budgets.
- **Wallet transfers (Execution v1)** — one native ETH transfer at a time on mainnet and testnet: deterministic
  instruction parsing, server preview with live balance and gas, explicit confirmation, signing only in the user's
  own wallet, receipt checked against the preview. [Details](wallet-transfers.md).
- **Cost and abuse controls** — atomic daily budget with per-run reserves, one run at a time, one run per visitor
  per day, origin and input validation.
- **Operations** — HTTPS, daily database backups, immutable releases with rollback.

## Planned

- **Autonomous AI execution on Robinhood Chain** — policy-bound execution of agent-planned, approved actions on Robinhood Chain, an Ethereum
  Layer 2 built on Arbitrum, with simulation before signing and a receipt for every action.
- **Accounts and higher limits** beyond the free tier.

Follow updates on [X / @SyncronixHq](https://x.com/SyncronixHq).
