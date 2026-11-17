# Usage limits

The SYNCRONIX Research Agent is free to use, within limits that keep it available to
everyone and its costs predictable.

## Limits

| Limit | Value | Resets |
|---|---|---|
| Research runs per visitor | 1 per day | 00:00 UTC |
| Runs in progress at once (whole service) | 1 | when the run finishes |
| Shared daily budget | $0.50 of model and search usage | 00:00 UTC |
| Cost ceiling per run | $0.20 | — |

- A visitor is identified by network: an IPv4 address, or an IPv6 /64 prefix. People sharing one network (an office,
  a mobile carrier) share its daily run.
- A run that fails before costing anything meaningful (for example, an outage of the AI provider) does not use your
  daily run. A cancelled run that did research does.
- Asking the same question again within an hour replays your own saved report and does not use your run.
- The console shows your remaining run, and when a limit applies it explains which one and when it resets.

## What is stored

| Data | Purpose |
|---|---|
| Your task, the report, its sources and progress events | Your private history in this browser |
| Token usage, searches, pages read, estimated cost | Budget enforcement and cost reporting |
| A random owner token in an httpOnly cookie (only its hash is stored) | Private history; no account needed |
| A keyed hash of your network address (never the address itself) | The per-visitor daily limit |

Only the browser that created a run can list or open it. There is no account, no tracking script and no wallet.

## Fair use

Research and analysis only. The agent cannot sign, send, buy, trade or execute anything. Please do not submit
personal data about other people or secrets; tasks and reports are processed by the AI provider to produce results.
