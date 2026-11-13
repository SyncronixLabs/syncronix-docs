# Research agent

The agent turns a plain-language task into a report whose key claims can be traced to pages it actually read.

## Research protocol

The model works through five steps, given to it as standing instructions:

1. **Plan** — the key facts, figures and dates the task needs, and what source would be authoritative for each.
2. **Search** — focused queries (usually 2–4), each looking for something new; no repeated or near-duplicate queries.
3. **Read** — open the 2–4 most relevant results. When an official or primary source exists (the organisation's own
   site, documentation, filings, standards, peer-reviewed papers), at least one page read must be that source;
   independent publications are added so at least two publishers are represented. If a page lacks the facts
   needed, the next best source is read instead of relying on snippets.
4. **Cross-check** — key claims confirmed against at least two independent sources where possible. Disagreements are
   reported with an explanation of which source is more trustworthy; claims with one source are marked
   *(single source)*; dates of time-sensitive facts are noted.
5. **Write** — Summary, Findings (each ending with links to its supporting pages), Analysis, Sources.

Web content is treated as untrusted data: instructions found in pages are never followed.

## Tools

| Tool | Version | Settings |
|---|---|---|
| `web_search` | `web_search_20250305` | up to 6 searches per run |
| `web_fetch` | `web_fetch_20250910` | up to 6 pages per run, ~12,000 tokens per page, passage citations on |

The direct-call tool versions are used deliberately. In side-by-side runs, the newer dynamic-filtering versions
re-issued identical searches and fetches from code, using up the per-run limits and costing more
([measurements](https://github.com/SyncronixLabs/syncronix-research)).

## Read gate

If the model tries to finish after reading fewer than two pages, it receives one follow-up instruction to read the
most relevant unread sources, cross-check and rewrite the report. This happens at most once per run and only if the
run has used less than half of its cost ceiling.

## Source verification and grounding

After the report is written:

- Every link in the report is traced to a real search result or fetched page. Untraceable links are flagged.
- Sources are marked **read** when the page itself was fetched, not just seen as a snippet; duplicates differing
  only by `www.`, trailing slash or `#fragment` are merged.
- A grounding line reports pages read, how many findings cite a page that was read, findings without a link,
  findings marked single-source, and the number of passage citations.

Example: `Grounding: 2 pages read in full · 6/8 findings cite a page that was read · 17 cited passages`

## Cost controls

- Each run reserves a **$0.20** ceiling from the daily budget before it starts. A run that pauses mid-turn is not
  resumed once its cost reaches the ceiling.
- Prompt caching serves most repeated context from cache across tool iterations.
- Identical tasks from the same browser within 60 minutes are replayed from the saved run at no cost.
- Measured cost of typical runs: **$0.04–$0.11** (claude-sonnet-5-5, list prices).
