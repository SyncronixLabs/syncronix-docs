# Usage limits

SYNCRONIX is free to use, within limits that keep it available to
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

## Wallet lookups and AI analysis

Wallet Activity and Funding Trace use no AI and do not use your daily run. They have their own limits:

| Limit | Value |
|---|---|
| Chain lookups per visitor | 30 per minute |
| Funding traces per visitor | 6 per minute |
| Funding traces running across the service | 3 at once |
| Indexed reads per day (mainnet, whole service) | About 3,000 indexer calls; resets daily |

**Analyze with AI** is optional and runs only when you click it, after you have seen its estimated and maximum
cost. It **counts as your daily run** (the same one research runs use) and is charged to the shared daily budget,
with a reserve of twice its maximum cost (at least $0.005) held while it runs. The two analyses measured in
release testing cost $0.0013 and $0.0014. Repeating your own analysis of the same address with unchanged data within an hour replays it for free.

## What is stored

| Data | Purpose |
|---|---|
| Your task, the report, its sources and progress events | Your private history in this browser |
| Token usage, searches, pages read, estimated cost | Budget enforcement and cost reporting |
| A random owner token in an httpOnly cookie (only its hash is stored) | Private history; no account needed |
| A keyed hash of your network address (never the address itself) | The per-visitor daily limit |
| AI wallet analyses: the address analysed, the brief and its cost | Your private history and budget accounting |

Wallet lookups without AI are not stored; their results are cached in server memory for up to a few minutes
(24 hours for a proven first funder) to save indexer calls.

Only the browser that created a run can list or open it. There is no account and no tracking script. A wallet is
needed only to sign a transfer, and it is connected in your browser; SYNCRONIX never receives its keys.

## Fair use

The research agents and the AI wallet brief cannot sign, send, buy, trade or execute anything. The only way ETH
moves is a [wallet transfer](wallet-transfers.md) that you start, preview, confirm and sign in your own wallet.
Wallet analysis describes on-chain activity; it never identifies who owns an address. Please do not submit
personal data about other people or secrets; tasks and reports are processed by the AI provider to produce results.
