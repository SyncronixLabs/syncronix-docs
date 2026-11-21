# Usage limits

SYNCRONIX is free to use, within limits that keep it available to
everyone and its costs predictable.

## Limits

Research runs and AI wallet analyses have separate allowances and separate budgets. Amounts are list-price
estimates of model and search usage; daily limits reset at 00:00 UTC.

| Limit | Research | AI wallet analysis |
|---|---|---|
| Per visitor per day | 1 run | 3 analyses |
| In progress at once (whole service) | 1 | 2 |
| Daily budget | $2.75 | $0.25 |
| Reserved while it runs | up to $0.20 (the per-run ceiling) | twice its maximum cost, at least $0.005 |

Shared by both:

| Limit | Value | Resets |
|---|---|---|
| Total daily budget | $3.00 | 00:00 UTC |
| Budget release | Each daily budget opens in 4 steps: a quarter at 00:00, 06:00, 12:00 and 18:00 UTC | each step |
| Monthly hard cap | $60, including the reserves of work in progress | 1st of the month, 00:00 UTC |
| AI usage per visitor per day | $0.30, including reserves | 00:00 UTC |
| AI runs per wider network per day | 6 (an IPv4 /24 or an IPv6 /48) | 00:00 UTC |

- A visitor is identified by network: an IPv4 address, or an IPv6 /64 prefix. People sharing one network (an office,
  a mobile carrier) share its allowances.
- Every budget check counts the reserves of work still in progress, so neither the daily budgets nor the monthly cap
  can be exceeded by work that started at the same time.
- A run that fails before costing anything meaningful (for example, an outage of the AI provider) does not use your
  allowance. A cancelled run that did research does.
- Asking the same question again within an hour replays your own saved report and does not use your run.
- The console shows what you have left. When a limit applies it says which one and when it resets. If today's budget
  is released in steps, it gives the time of the next step.

## Wallet lookups and AI analysis

Wallet Activity and Funding Trace use no AI and do not use any AI allowance. They have their own limits:

| Limit | Value |
|---|---|
| Chain lookups per visitor | 30 per minute |
| Funding traces per visitor | 6 per minute |
| Funding traces running across the service | 3 at once |
| Indexed reads per day (mainnet, whole service) | About 3,000 indexer calls; resets daily |

**Analyze with AI** is optional and runs only when you click it, after you have seen its estimated and maximum
cost. It uses one of your **3 daily AI wallet analyses**, separate from your research run, and is charged to the
AI wallet analysis budget ($0.25 per day). While it runs it holds twice its maximum cost (at least $0.005) in reserve.
The two analyses measured in release testing cost $0.0013 and $0.0014. Repeating your own analysis of the same
address with unchanged data within an hour replays it for free.

## What is stored

| Data | Purpose |
|---|---|
| Your task, the report, its sources and progress events | Your private history in this browser |
| Token usage, searches, pages read, estimated cost | Budget enforcement and cost reporting |
| A random owner token in an httpOnly cookie (only its hash is stored) | Private history; no account needed |
| A keyed hash of your network address and of its wider network (never the addresses themselves) | The per-visitor and per-network daily limits |
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
