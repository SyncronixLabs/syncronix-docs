# Getting started

Five ways to use SYNCRONIX, from the quickest to the most involved. All of them are free; [limits](limits.md) apply.

| Path | You need | Time |
|---|---|---|
| [1. Run research in the console](#1-run-research-in-the-console) | A browser | 1–2 min per run |
| [2. Look up a wallet](#2-look-up-a-wallet) | A browser and a Robinhood Chain address | seconds |
| [3. Send a wallet-approved transfer](#3-send-a-wallet-approved-transfer) | MetaMask (or another EIP-1193 wallet) with ETH on Robinhood Chain | 1 min |
| [4. Call the API](#4-call-the-api) | `curl` or Node.js 18+ | minutes |
| [5. Build the pattern yourself](#5-build-the-pattern-yourself) | Node.js 18+ and your own Anthropic API key | minutes |

## 1. Run research in the console

1. Open the [live console](https://syncronixai.xyz/platform#console).
2. Describe what you want researched, for example *"What is the current Node.js Active LTS line and when does it
   reach end of life?"*, and press **Run agent**.
3. Watch the progress (plan → research → analyze → deliver). The report ends with the Verifier's line, for example
   `Verifier: 5/6 claims verified · quotes matched 6/6 · corroborated by an independent source 4/6`, and a
   grounding line.
4. Reopen it later under **History**. Only this browser can see it.

You get one research run per day; it resets at 00:00 UTC. How checking works: [research-agent.md](research-agent.md).

## 2. Look up a wallet

1. On the [platform page](https://syncronixai.xyz/platform), enter an address in **Address lookup**.
2. Pick **Activity** (history, inflows and outflows, contracts) or **Funding trace** (first funder and up to 5 hops
   upstream). No AI is used, and it does not use your daily run.
3. Optional: **Analyze with AI** first shows the estimated and maximum cost. It makes one model call only if you click
   it, and it counts as your daily run.

Read *verified* edges as on-chain transfers with a link, and *inferred* edges as patterns, never as proof of
ownership. Details: [wallet-intelligence.md](wallet-intelligence.md).

## 3. Send a wallet-approved transfer

1. Add Robinhood Chain to your wallet. The values are also offered by the page itself:

   | | Mainnet | Testnet |
   |---|---|---|
   | Chain ID | 4663 | 46630 |
   | RPC | `https://rpc.mainnet.chain.robinhood.com` | `https://rpc.testnet.chain.robinhood.com` |
   | Explorer | `https://robin.etherscan.io` | `https://explorer.testnet.chain.robinhood.com` |
   | Currency | ETH | ETH |

2. In the Robinhood Chain panel on the [platform page](https://syncronixai.xyz/platform), connect your wallet and type
   `Send 0.001 ETH to 0x…`. Add `on testnet` to use testnet.
3. Read the preview: amount, estimated fee, the most it can cost and your balance afterwards. On mainnet, tick the
   acknowledgement.
4. Press **Confirm in wallet** and approve the transfer in your wallet. The page checks the receipt against the
   preview.

Start with a small amount, or with testnet. The mainnet limit is 0.05 ETH per transfer. Transfers are final.
Details: [wallet-transfers.md](wallet-transfers.md).

## 4. Call the API

No key is needed. Call the API from a terminal or your own server. Web pages on other websites cannot call it from a
browser: see [Who can call what](api.md#who-can-call-what).

```bash
# What can I do today?
curl -s https://syncronixai.xyz/api/runs/quota

# Start a research run and stream it (uses your daily run)
curl -N https://syncronixai.xyz/api/runs \
  -H 'content-type: application/json' \
  -d '{"task":"What is the current Node.js Active LTS line and when does it reach end of life?"}'

# Wallet activity and funding trace (no AI, does not use your run)
curl -s 'https://syncronixai.xyz/api/chain/wallet?network=mainnet&address=0x…&view=intel'
curl -s 'https://syncronixai.xyz/api/chain/wallet?network=mainnet&address=0x…&view=trace'
```

Ready-made Node.js clients: [syncronix-examples](https://github.com/SyncronixLabs/syncronix-examples). Reference:
[api.md](api.md).

## 5. Build the pattern yourself

[syncronix-examples/grounded-agent](https://github.com/SyncronixLabs/syncronix-examples/tree/main/grounded-agent)
is the core research pattern in one file: Claude with web search and web fetch, passage citations, a read gate and a
grounding check. It runs on your own Anthropic API key, billed to your account.

## Common problems

| You see | Why | What to do |
|---|---|---|
| `429 ip_quota` | Today's run is used | Wait until 00:00 UTC; `GET /api/runs/quota` shows `resetsAt` |
| `503 busy` | One run at a time across the service | Retry after `Retry-After` seconds |
| `429 daily_budget` | The shared daily budget is used | Wait until 00:00 UTC |
| `403 cross_site` | Called from a web page on another website | Call from a terminal or your own server |
| `429 rate_limited` on wallets | Over 30 lookups or 6 traces per minute | Retry after `Retry-After` seconds |
| First funder shows *earliest found* | The indexer's history is not complete up to that block | Expected on mainnet for now; see [wallet-intelligence.md](wallet-intelligence.md) |
| `over_limit` on a transfer | Above 0.05 ETH (mainnet) or 1 ETH (testnet) | Send a smaller amount |
| `unsupported` on a transfer | Swaps, tokens, `all`/`max`, schedules are not supported | Use one plain ETH amount and one address |
| Wallet on the wrong network | The previewed chain must match your wallet | Switch the network in your wallet; the preview refreshes |

<sub>Documentation licensed under [CC BY 4.0](LICENSE).</sub>
