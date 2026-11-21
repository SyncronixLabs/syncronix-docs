# Changelog

What changed in production at [syncronixai.xyz](https://syncronixai.xyz) and in this documentation, newest first.
Dates are UTC and come from the commit history of the SyncronixLabs repositories. An entry under *Production* means
it runs at syncronixai.xyz.

## 2026-11-21

**Documentation**

- [api.md](api.md): the transfer preview's refusals are now one complete table (it was missing `bad_network`,
  `bad_sender`, `empty`, `too_long`, `unrecognized`, `bad_json`, `bad_content_type`, `too_large`, `rate_limited`,
  `rpc_unavailable` and `disabled`). The table is checked against production by the new
  [API conformance benchmark](https://github.com/SyncronixLabs/syncronix-benchmarks#api-conformance) (30/30 cases).
- New benchmarks, a transfer example and a research note; see the entries in
  [syncronix-benchmarks](https://github.com/SyncronixLabs/syncronix-benchmarks/blob/main/CHANGELOG.md),
  [syncronix-examples](https://github.com/SyncronixLabs/syncronix-examples/blob/main/CHANGELOG.md) and
  [syncronix-research](https://github.com/SyncronixLabs/syncronix-research/blob/main/CHANGELOG.md).

No production change.

## 2026-11-20

**Production**

- **Wallet-approved transfers (Execution v1)** are live on Robinhood Chain mainnet (4663) and testnet (46630):
  - one native ETH transfer at a time;
  - the instruction is parsed by fixed rules;
  - the server previews it with the live balance and gas;
  - you confirm, sign in your own wallet, and the page checks the receipt.

  The mainnet limit is 0.05 ETH per transfer, and contract recipients are refused there. There is no AI and there
  are no server keys.
- **Verified on mainnet** with a real wallet-approved transfer through production. It was read back from the
  official RPC and sent exactly as previewed. A real testnet transfer was verified the same way before mainnet was
  enabled.

**Documentation**

- Added [wallet-transfers.md](wallet-transfers.md); updated the API, security and roadmap pages.
- Added the [technical whitepaper](whitepaper.md), [getting started](getting-started.md), this changelog and the
  verified milestones in the [roadmap](roadmap.md#verified-milestones).

## 2026-11-19

**Production**

- **Wallet Intelligence**: activity for any Robinhood Chain address, cross-checked against the official RPC, with
  Robinscan links.
- **Funding Trace**:
  - the first funder, with its proof status;
  - up to 5 hops upstream;
  - shared funders and comparison with up to 2 addresses;
  - a graph that keeps *verified* transfers apart from *inferred* links.
- **AI wallet analysis (opt-in)**: a `claude-haiku-5-5` brief written from a fact sheet built in code. It runs only
  after a cost estimate and an explicit click. Measured cost: $0.0013–$0.0014.
- **Website Docs page** refreshed with a quick start, examples and an FAQ; links between pages now scroll to the
  right section.

**Documentation**

- Added [wallet-intelligence.md](wallet-intelligence.md); updated the API, limits and security pages.
- Added the Wallet Intelligence harness and production measurements to
  [syncronix-benchmarks](https://github.com/SyncronixLabs/syncronix-benchmarks), and the first-funder note to
  [syncronix-research](https://github.com/SyncronixLabs/syncronix-research).

## 2026-11-17

**Production**

- **Multi-agent verification** on every run:
  - the Researcher (`claude-sonnet-5-5`) writes the report;
  - the Analyst (`claude-haiku-5-5`) extracts up to 8 claims, each with a verbatim quote;
  - the Verifier matches the quotes in code, cross-checks other domains and judges each claim from the evidence
    only.
- **Robinhood Chain data (read-only)**: the official RPC with a read-method allowlist and a chain ID check; live
  chain reads are added to research as evidence.
- The project was repositioned to the Robinhood ecosystem, with a non-affiliation notice. Beta labels were removed.

## 2026-11-13

**Production**

- **Research Agent** launched, free:
  - web search and page reading;
  - a research protocol and a read gate;
  - source tracing, *read* vs. *seen* labels and a grounding line;
  - live progress over Server-Sent Events;
  - a private history for each browser.
- **Cost and abuse controls**:
  - atomic admission in PostgreSQL with a $0.50 shared daily budget, a $0.20 reserve per run, one run at a time and
    one run per visitor per day;
  - same-site checks and input validation.

**Repositories**

- Published [syncronix-docs](https://github.com/SyncronixLabs/syncronix-docs),
  [syncronix-research](https://github.com/SyncronixLabs/syncronix-research),
  [syncronix-benchmarks](https://github.com/SyncronixLabs/syncronix-benchmarks) and
  [syncronix-examples](https://github.com/SyncronixLabs/syncronix-examples).

## Planned

Autonomous AI execution is not live. See [roadmap.md](roadmap.md#planned).
