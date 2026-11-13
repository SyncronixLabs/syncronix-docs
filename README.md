<p align="center">
  <img src="https://raw.githubusercontent.com/SyncronixLabs/SyncronixLabs/main/assets/syncronix-banner.jpg" alt="SYNCRONIX" width="100%">
</p>

<h1 align="center">SYNCRONIX Documentation</h1>

<p align="center">
  How the SYNCRONIX Research Agent works, how to use its public API, and what is live versus planned.<br>
  <a href="https://syncronixai.xyz">Website</a> · <a href="https://syncronixai.xyz/platform#console">Live console</a> · <a href="https://github.com/SyncronixLabs">SyncronixLabs</a>
</p>

---

> [!NOTE]
> **Live:** the SYNCRONIX Research Agent, a free public beta at [syncronixai.xyz](https://syncronixai.xyz).
> **Planned:** execution and settlement on Kinetiq Elysium. The console has no wallet and sends no transactions.

## Contents

| Document | You will find |
|---|---|
| 🏗️ [Architecture](architecture.md) | Components, request flow, data model, deployment shape |
| 🧠 [Research agent](research-agent.md) | Research protocol, tools, read gate, grounding and source verification, cost ceiling |
| 🔌 [Public API](api.md) | Endpoints, Server-Sent Events, refusal codes, examples |
| 🎟️ [Public beta](public-beta.md) | Limits, what is stored, privacy, fair use |
| 🛡️ [Security model](security.md) | Threats, controls, known limitations, reporting a vulnerability |
| 🗺️ [Roadmap](roadmap.md) | What is live, what is planned |

## Start here

<details>
<summary><b>I want to use the agent</b></summary>

Open the [live console](https://syncronixai.xyz/platform#console), describe what you want researched and press
**Run agent**. You get one free run per day; the console shows your remaining run and when it resets. Your reports
stay in **History**, visible only in your browser. See [Public beta](public-beta.md).

</details>

<details>
<summary><b>I want to call the API</b></summary>

```bash
curl -s https://syncronixai.xyz/api/runs/quota

curl -N https://syncronixai.xyz/api/runs \
  -H 'content-type: application/json' \
  -d '{"task":"What is the current Node.js Active LTS line and when does it reach end of life?"}'
```

The run streams as Server-Sent Events and ends with a `done` event carrying the report and its sources.
Reference: [api.md](api.md). Working clients: [syncronix-examples](https://github.com/SyncronixLabs/syncronix-examples).

</details>

<details>
<summary><b>I want to understand the design</b></summary>

Read [Architecture](architecture.md), then [Research agent](research-agent.md). The measurements behind the design
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

## Conventions

- All times are **UTC**; daily limits reset at **00:00 UTC**.
- Costs are **list-price estimates** from usage reported by the AI provider.
- A feature is documented as *live* only when it runs in production.

## Related

| Repository | |
|---|---|
| [syncronix-research](https://github.com/SyncronixLabs/syncronix-research) | Engineering notes with measurements |
| [syncronix-benchmarks](https://github.com/SyncronixLabs/syncronix-benchmarks) | Research-quality harness and results |
| [syncronix-examples](https://github.com/SyncronixLabs/syncronix-examples) | Working API clients and a minimal grounded agent |

<sub>Documentation licensed under [CC BY 4.0](LICENSE).</sub>
