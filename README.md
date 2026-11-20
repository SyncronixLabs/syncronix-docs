<p align="center">
  <img src="assets/banner.png" alt="SYNCRONIX / Documentation — architecture, guides, API, security" width="100%">
</p>

<h1 align="center">SYNCRONIX Documentation</h1>

<p align="center">
  How SYNCRONIX works: verified research, Wallet Intelligence, wallet-approved transfers and the public API, and what is live versus planned.<br>
  <a href="https://syncronixai.xyz">Website</a> · <a href="https://syncronixai.xyz/platform#console">Live console</a> · <a href="https://github.com/SyncronixLabs">SyncronixLabs</a>
</p>

---

> [!NOTE]
> **Live:** SYNCRONIX at [syncronixai.xyz](https://syncronixai.xyz): Researcher, Analyst and
> Verifier agents on every run, read-only Robinhood Chain data, Wallet Intelligence, a 5-hop Funding Trace and
> opt-in AI wallet analysis, and **wallet transfers**: one native ETH transfer at a time that you confirm and sign in
> your own wallet (mainnet and testnet, verified on mainnet). **Planned:** autonomous AI
> execution, where agents would plan actions on their own. SYNCRONIX holds no keys and never signs.
>
> SYNCRONIX is an independent project and is not affiliated with, endorsed by or partnered with Robinhood Markets, Inc. or its affiliates. Robinhood and Robinhood Chain are trademarks of their respective owners.

## Contents

| Document | You will find |
|---|---|
| 🚀 [Getting started](getting-started.md) | Five ways in: console, wallet lookup, a transfer, the API, the pattern in your own code; common problems |
| 📄 [Technical whitepaper](whitepaper.md) | The whole system in one document: design principles, verification, budgets, chain data, transfers, evaluation, limitations |
| 🏗️ [Architecture](architecture.md) | Components, request flow, data model, deployment shape |
| 🧠 [Research agent](research-agent.md) | Research protocol, tools, read gate, grounding and source verification, cost ceiling |
| 🔎 [Wallet Intelligence](wallet-intelligence.md) | Activity, first funder, 5-hop funding trace, shared funders, verified vs. inferred, opt-in AI analysis, indexer limits |
| 💸 [Wallet transfers](wallet-transfers.md) | Execution v1: instruction rules, checks, preview, confirmation, receipt check, limits |
| 🔌 [Public API](api.md) | Endpoints, Server-Sent Events, refusal codes, examples |
| 🎟️ [Usage limits](limits.md) | Limits, what is stored, privacy, fair use |
| 🛡️ [Security model](security.md) | Threats, controls, known limitations, reporting a vulnerability |
| 🗺️ [Roadmap](roadmap.md) | What is live, verified milestones, what is planned |
| 📝 [Changelog](CHANGELOG.md) | What changed in production and in these docs, by date |

## Start here

<details>
<summary><b>I want to use the agent</b></summary>

Open the [live console](https://syncronixai.xyz/platform#console), describe what you want researched and press
**Run agent**. You get one free run per day; the console shows your remaining run and when it resets. Your reports
stay in **History**, visible only in your browser. See [Usage limits](limits.md).

</details>

<details>
<summary><b>I want to call the API</b></summary>

```bash
curl -s https://syncronixai.xyz/api/runs/quota

curl -N https://syncronixai.xyz/api/runs \
  -H 'content-type: application/json' \
  -d '{"task":"What is the current Node.js Active LTS line and when does it reach end of life?"}'
```

The run streams as Server-Sent Events and ends with a `done` event carrying the report and its sources. Call the API
from a terminal or your own server; web pages on other websites cannot call it from a browser.
Reference: [api.md](api.md). Working clients: [syncronix-examples](https://github.com/SyncronixLabs/syncronix-examples).

</details>

<details>
<summary><b>I want to look up a wallet</b></summary>

Open the [platform page](https://syncronixai.xyz/platform), enter an address in **Address lookup** and pick
**Activity** or **Funding trace**. Both are free and use no AI. **Analyze with AI** shows a cost estimate first and
runs only when you click it. See [Wallet Intelligence](wallet-intelligence.md).

```bash
curl -s 'https://syncronixai.xyz/api/chain/wallet?network=mainnet&address=0x…&view=trace'
```

</details>

<details>
<summary><b>I want to send a transfer</b></summary>

In the Robinhood Chain panel on the [platform page](https://syncronixai.xyz/platform), connect your wallet and type
`Send 0.001 ETH to 0x…`. You see a preview with the fee and total, confirm it, and sign in your own wallet. One native
ETH transfer at a time; 0.05 ETH per transfer on mainnet. See [Wallet transfers](wallet-transfers.md).

</details>

<details>
<summary><b>I want to understand the design</b></summary>

Read the [whitepaper](whitepaper.md) for the whole picture, or [Architecture](architecture.md), then
[Research agent](research-agent.md). The measurements behind the design
decisions are in [syncronix-research](https://github.com/SyncronixLabs/syncronix-research).

</details>

## Key concepts

| Concept | Meaning |
|---|---|
| **Run** | One research task from admission to report, streamed live and saved to the visitor's private history |
| **Admission** | The atomic check that decides whether a run may start: shared budget incl. reserves, one active run, one run per visitor per day |
| **Reserve** | The $0.20 ceiling a run holds from the daily budget while active; replaced by its real cost when it ends |
| **Read gate** | One follow-up turn that asks the agent to read sources if it tried to finish after reading fewer than two pages |
| **Read vs. seen** | A *read* source was fetched and read; a *seen* source appeared only as a search result |
| **Grounding line** | Pages read · findings citing a read page · findings without a link · single-source findings · cited passages |
| **First funder** | The sender of an address's first ETH; *proven* only when the indexer's history up to that block is complete, otherwise *earliest found* |
| **Verified vs. inferred** | *Verified* = an actual on-chain transfer with its link; *inferred* = a pattern such as a shared funder, never proof of common ownership |

## Conventions

- All times are **UTC**; daily limits reset at **00:00 UTC**.
- Costs are **list-price estimates** from usage reported by the AI provider.
- Mainnet explorer links go to **Robinscan** (`robin.etherscan.io`); indexed data comes from the Blockscout API.
- A feature is documented as *live* only when it runs in production.

## Related

| Repository | |
|---|---|
| [syncronix-research](https://github.com/SyncronixLabs/syncronix-research) | Engineering notes with measurements |
| [syncronix-benchmarks](https://github.com/SyncronixLabs/syncronix-benchmarks) | Research-quality and Wallet Intelligence harnesses and results |
| [syncronix-examples](https://github.com/SyncronixLabs/syncronix-examples) | Working API and wallet clients and a minimal grounded agent |

<sub>Documentation licensed under [CC BY 4.0](LICENSE).</sub>
