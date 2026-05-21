# ETH Gas Live — Backend Capabilities (Public Overview)

Backend capabilities for the live system at [logicencoder.com/ethereum-gas-tracker/](https://logicencoder.com/ethereum-gas-tracker/).  
No source code, RPC keys, or internal hostnames in this repository.

**Private code:** [eth-gas-live-backend](https://github.com/logicencoder/eth-gas-live-backend)  
**Website layer:** [eth-gas-live-plugin-overview](https://github.com/logicencoder/eth-gas-live-plugin-overview)

---

## Problem

Sending on Ethereum without context leads to:

- Paying **too much** because “fast” was selected blindly  
- Paying **too little** and waiting hours during congestion  
- Missing **quieter windows** when fees dip  
- Misreading a single GWEI number without tier breakdown or USD cost  

The backend turns raw chain data into **tiered, explained, historical, and time-aware** guidance updated every block (or near it on local Geth).

---

## Realtime fee tiers

### What

Three named paths updated continuously on each processed block:

| Tier | Typical use |
|------|-------------|
| **Base Route** | Minimum viable inclusion (base fee focused) |
| **Standard Way** | Balanced cost for most transfers and contract calls |
| **Faster Inclusion** | Higher tip profile when blocks are competitive |

Each tier is shown in **GWEI** (five decimal precision in SSR), estimated **ETH**, and **USD** from a live ETH price feed.

### Why it exists

Wallets often show one “gas price.” Users need **actionable tiers** mapped to confirmation expectations.

### Who benefits

Retail senders, DeFi users, and SEO visitors reading snapshot pages with the same numbers.

### How it fits

Computed in `gas_tracker.py`, stored in SQLite `gas_history`, broadcast on `/ws/gas` and exposed via `/api/gas/current`.

---

## Network stress signals

### What

- **Transactions per minute** — estimated from recent block tx counts  
- **Block utilization** — how full recent blocks are  
- **Inclusion Pressure Index (IPI)** — 0–100 combining throughput, fullness, tip spread (damped in very quiet markets)  
- **Spike score** — 0–100 short-term regime vs 1h/24h standard reference  
- **Fee competition** — spread between priority fee percentiles  
- **Block speed pressure** — share of blocks near capacity  

### Why

A single GWEI figure hides **regime change** (mempool heating up vs calm Sunday afternoon).

### Who

Anyone deciding “send now or wait.”

### How

`_compute_ipi()` and `_compute_spike_score()` in backend; surfaced in UI network grid and KLOD SEO placeholders.

---

## Historical and analytical views

### What

- **Price charts** — 1 hour through 30 days for all three tiers  
- **Heatmap** — hour-of-day patterns (browser shifts buckets to visitor timezone on the public site)  
- **Rolling averages** — 1h, 3h, 6h, 12h, 24h, 3d, 7d, 30d  
- **Faster-tier percentiles** — P50–P85  
- **Advanced statistics** — min/max/avg/median, volatility, best/worst hours in selected window  

### Why

Timing a non-urgent transfer benefits from **pattern memory**, not just the latest block.

### Who

Researchers, power users, and content readers on “price history” SEO URLs.

### How

SQLite queries on `gas_history`; cached REST endpoints `/api/gas/history`, `/api/gas/statistics`, `/api/gas/heatmap`.

---

## Gas Intelligence Hub

### What

Plain-language **insight line**, best/worst send windows (24h context), and **send now vs wait** recommendation from `/api/gas/predict`.

### Why

Translates metrics into a decision without reading charts.

### Who

Casual visitors who will not open the statistics tab.

### How

`_insight_text_one_liner()` plus predict endpoint comparing snapshot standard GWEI to `avg_24h`.

---

## Fee calculator

### What

Gas limit presets (transfer, ERC-20, swap, NFT, etc.) plus custom limit; price tied to Base / Standard / Faster / custom GWEI.

### Why

Users think in **“how much for my swap”**, not raw GWEI.

### Who

DeFi and NFT senders evaluating total cost.

### How

Client-side math using live tier values from WebSocket; dedicated SEO page `/ethereum-gas-calculator/`.

---

## Featured transaction costs

### What

Estimates for **18 action types** (transfer, approve, swap, NFT, bridge, staking, deploy, …) at each tier in ETH and USD.

### Why

Comparisons across action types drive organic search and wallet planning.

### Who

Visitors comparing “cost to swap vs cost to bridge.”

### How

`GET /api/gas/featured-actions` with fixed gas units per action recomputed each block.

---

## Custom alerts

### What

Per-browser session alerts when Standard GWEI crosses **above** or **below** a threshold; optional desktop notifications.

### Why

Users want to **act when fees drop** without staring at the chart.

### Who

Patient senders waiting for sub-threshold fees.

### How

`alerts` SQLite table; `POST /api/alerts`; WS `type: alert` events.

---

## WebSocket live stream

### What

`/ws/gas` pushes the full enriched gas object each block plus 30s keepalive pings.

### Why

HTTP polling lags behind block time on local Geth.

### Who

Live dashboard and WordPress embed using the same SPA.

### How

FastAPI websocket manager; same payload shape as `/api/gas/current`.

---

## Indexable educational pages (KLOD)

### What

Separate URL paths with HTML templates filled from live `klod_seo` placeholders (fees, IPI, utilization, tx/min).

| Topic | Path on LogicEncoder |
|-------|----------------------|
| Live tracker hub | `/ethereum-gas-tracker/` |
| Fees today | `/ethereum-gas-fees-today/` |
| Why fees are high | `/why-are-ethereum-gas-fees-high/` |
| Best time to send | `/best-time-to-send-ethereum/` |
| Gas calculator | `/ethereum-gas-calculator/` |
| Transaction costs | `/ethereum-transaction-costs/` |
| Swap gas | `/ethereum-swap-gas-fees/` |
| Network load | `/ethereum-mempool-tracker/` |
| Network status | `/ethereum-network-status/` |
| Price history | `/ethereum-gas-price-history/` |
| Percentiles | `/ethereum-gas-percentiles/` |

### Why

JavaScript-only homepages rank poorly; **static HTML with live numbers** answers long-tail queries.

### Who

SEO visitors and AI crawlers.

### How

Python serves `ssr_klod_files/*.html` on backend; WordPress plugin serves `seo-templates/` filled from `GET /api/gas/ssr-data`.

---

## Node SSR layer

### What

React server renders `/gas` HTML using bundle from `/api/gas/ssr-data` (Schema.org JSON-LD, meta, five-decimal GWEI).

### Why

Some deployments want **one HTML document** per request for crawlers without PHP template fill.

### Who

Operators running SSR on port 3001 beside Python 8031.

### How

`gas_tracker_ssr-server.js` — optional; WordPress path uses JSON placeholders instead of fetching Node HTML.

---

## Block ingestion

### What

Preferred: **local Geth** HTTP + optional `newHeads` WebSocket. Alternatives: Ankr, Infura, Alchemy, QuickNode, DRPC via CLI/env.

### Why

Local node gives ~1s refresh and full fee history sampling; hosted RPC adds latency and caps.

### Who

LogicEncoder production on SOL hardware.

### How

`GasTracker` loop in private repo; `block_trigger_mode` exposed in monitoring API.

---

## ETH/USD price feed

### What

ETH priced in USD via MEXC WebSocket protobuf ticker with Binance REST fallback.

### Why

Tier cards show **dollar cost** — Geth does not provide reliable USD.

### Who

All dashboard visitors.

### How

Separate async task; not mixed into gas tier math.

---

## WordPress realtime mirror (optional)

### What

Backend may POST JSON to WordPress `ethgas/v1/realtime` so the plugin can serve last payload if WS blocked.

### Why

Shared hosting or CDN sometimes blocks browser WebSockets.

### Who

Public site visitors on LogicEncoder.com.

### How

Env-configured push URL + API key; plugin stores 90s transient.

---

## Operator monitoring

### What

`GET /api/monitoring/overview` — uptime, fetch success rates, WS client count, DB row counts, cache hits, last errors, sparkline arrays.

### Why

Mission Control in wp-admin and SSH-free health checks.

### Who

Maintainers.

### How

Private `ARCHITECTURE.md` lists full JSON sections; not repeated here.

---

## Data retention

### What

~**30 days** of per-block rows in SQLite; alerts until fired or deleted.

### Why

Bounded disk on SOL; enough for monthly charts.

### Who

Long-range analysts (30d view).

### How

Hourly purge job in ingest loop.

---

## Who benefits (summary)

| Audience | Value |
|----------|--------|
| **Retail senders** | Cost before confirming |
| **DeFi users** | Tier comparison for swaps/approvals |
| **SEO visitors** | Topic pages with current numbers |
| **Operators** | Monitoring without log tail |

---

## Repository map

| Repo | Role |
|------|------|
| `eth-gas-live-backend` | Private implementation |
| `eth-gas-live-backend-overview` | This file |
| `eth-gas-live-plugin` | WordPress bridge |
| `eth-gas-live-plugin-overview` | Site-facing story |

Legacy `eth-gas-tracker-legacy-*` repos are archived — not used.

---

## Live link

**https://logicencoder.com/ethereum-gas-tracker/**
