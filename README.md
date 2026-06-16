# ETH Gas Live — backend service

**FastAPI + SQLite + Node SSR backend** for [logicencoder.com/ethereum-gas-tracker/](https://logicencoder.com/ethereum-gas-tracker/) and eleven related SEO URLs. The private service ingests Ethereum blocks, computes EIP-1559 fee tiers and congestion metrics, stores rolling history, streams live updates over WebSocket, fills SEO data bundles, and optionally mirrors JSON to WordPress. Visitors use the dashboard through the plugin; this repo documents the compute and API layer behind that product.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| Ingest | Python 3, FastAPI, uvicorn, web3.py, local Geth `newHeads` or hosted RPC fallback |
| Storage | SQLite rolling history (~30 days), alert rules, in-memory caches for history/heatmap/stats |
| Realtime | WebSocket `/ws/gas` — full gas object each block plus alert events |
| Analytics | Rolling 1h–30d windows, hourly heatmap buckets, IPI/spike scoring, predict/send hints |
| Price feed | MEXC protobuf WebSocket for ETH/USD with Binance REST fallback |
| SEO feed | `klod_seo` placeholder bundle via `/api/gas/ssr-data`; Node Express + React SSR for proxied crawler routes |
| WordPress mirror | Optional authenticated POST to plugin REST transient |
| Ops | `/api/monitoring/overview` — loop timing, fetch/push stats, WS clients, cache hit ratios |
| Hosting | Self-hosted Linux for ingest and fan-out; WordPress on shared hosting for public pages |

**Typical operator flows:** deploy new `gas_tracker.html` → copy shell to plugin repo → bump plugin version → purge WP cache → confirm **Mission Control** fetch latency and push HTTP status. Users report stale tiers → check monitoring **last push** and **fetch failures** before restarting Geth. SEO page looks wrong → hit SSR data bundle freshness (transient on WP side) and verify `klod_seo` keys in monitoring JSON dump.

## Block ingest and fee tiers

Each new head triggers fee sampling from **local Geth** (preferred — `newHeads` subscription or ~1s poll) or hosted RPC when the execution client is unavailable. The ingest loop enriches one **`current_gas`** dict per block: three tiers, network stress fields, rolling averages, and SEO placeholders.

| Tier | Role |
|------|------|
| **Base Route** | Base fee path with minimal priority tip — economical when blocks are calm |
| **Standard Way** | Typical inclusion from recent block fee sampling — default comparison for wallets |
| **Faster Inclusion** | Higher percentile tips with congestion caps — competitive mempools and time-sensitive sends |

Each tier exposes gwei (base + priority), ETH and USD estimates for a reference transfer, and confirmation hints derived from block timing (~12s target per block). **ETH/USD** is refreshed from exchange feeds, not from the execution client.

When a wallet “medium” disagrees with the public site → compare against **Standard Way** on `/api/gas/current` or the WS payload — wallets often collapse tiers into one number; the backend keeps base and priority components separate for EIP-1559 accuracy.

## Network stress signals

Every block computes metrics the UI network grid and Intelligence Hub consume:

- **Tx / minute** — estimated from recent block transaction counts  
- **Block utilization** — current and rolling average fullness  
- **Inclusion Pressure Index (IPI)** — 0–100 composite of throughput, fullness, and tip spread (damped in very quiet markets)  
- **Spike score** — 0–100 short-term regime vs 1h/24h Standard reference  
- **Fee competition** — spread between high and median priority fees  
- **Block speed pressure** — share of recent blocks above 90% full  
- **Network status** — NORMAL / LOW / HIGH / SPIKE labels derived from the above  

`/api/gas/predict` says “send now” but **IPI** and **SPIKE** are elevated on the same payload → treat predict as relative to 24h average only; cross-check tx/min and utilization before a large batch — use statistics/heatmap endpoints for hourly context.

## History charts and heatmap

Each stored block appends to the rolling history store. Retention is roughly **30 days** with purge during ingest so queries stay bounded.

**History API** serves Chart.js ranges from **1 hour through 30 days** for all three tiers, optional smoothing on the client, and utilization overlay series. Short windows stream live over WebSocket; longer windows read SQLite with server-side caching and background prewarm after each block.

**Heatmap API** pre-aggregates **hourly average Standard gwei** by calendar day in SQL (`GROUP BY day, hour`), returns up to **30 days**, and caches results — the browser shifts hours to the visitor timezone. Default UI window is **seven days** unless the user selects the **30 Days** chart range.

When an operator sees rising **heatmap_requests** and cache misses after deploy → cold cache, not broken ingest; wait for prewarm or one user-driven request to populate. Researcher pulls **7d heatmap** + **168h statistics** → same hourly buckets, different shape (visual grid vs tabular best/worst hour).

## Gas Intelligence and predict

The **`/api/gas/predict`** endpoint returns send-now vs wait guidance, plain-language message, current vs 24h average Standard gwei, and trend — computed from **`current_gas` rolling fields**, not a heavy hourly SQL scan. The SPA **Gas Intelligence Hub** and SSR **`klod_seo`** insight strings share the same enrichment path (`_enrich_gas_aliases`, `_ssr_snapshot_send_hint`, `_insight_text_one_liner`).

SEO “best time to send” page and live hub show aligned **Best Time (Last 24h)** because both read the same bundle from **`/api/gas/ssr-data`**; WordPress templates fill `{{PLACEHOLDER}}` keys from that JSON on a short transient cache.

## Featured transaction costs

**`/api/gas/featured-actions`** recomputes **nineteen action types** each tick — transfers, approvals, swaps, NFT mint/sale, bridging, lending/borrowing, staking flows, liquidity add/remove, governance, multi-send, and contract deploy — each with low/avg/high gas limits mapped to **Base / Standard / Faster** gwei and USD. The SPA list and calculator presets stay in sync with the same limits defined in backend code.

When planning approve + swap + bridge → sum three **standard** USD fields from featured-actions JSON for a budget check before opening the wallet; no manual gas-limit math on the client beyond display.

## Threshold alerts

**POST `/api/alerts`** stores rules keyed by **browser session id** with **over** or **under** threshold gwei. On each block, the ingest loop evaluates active rules; triggers broadcast on **`/ws/gas`** as alert events and can surface browser notifications in the SPA. **DELETE** removes rules per id; **GET** lists rules for a session.

When a user sets an UNDER threshold → walks away → WS pushes alert payload when Standard drops → SPA toast + optional Notification API — backend holds state so refresh does not lose the rule until session expires or user deletes it.

## WebSocket broadcast

Clients connect to **`/ws/gas`** for the full enriched gas object every block plus alert events. An internal client list fans out JSON to every session; monitoring tracks connected count, messages per minute, and broadcast failures. REST **`/api/gas/current`** mirrors the latest object for bootstrap and corporate networks that block WebSocket.

WS connected but REST bootstrap stale → check **`last_push`** to WordPress and **`fetch_failures`** in monitoring — ingest may be healthy while the WP mirror path is not.

## SSR data bundle and Node helper

**`/api/gas/ssr-data`** returns one JSON document: current tiers, network metrics, rolling averages, heatmap headline keys, predict hint, and the full **`klod_seo`** placeholder map for eleven SEO templates. WordPress fills local HTML templates from this bundle; the plugin does not recompute gas in PHP.

A companion **`gas_tracker_ssr-server.js`** (Express + React) renders indexable HTML for routes proxied by FastAPI (`/ssr_gas/page`, `/ethereum-gas/`, and related paths) using the same bundle — no duplicate tier math in Node. Static **`ssr_klod_files/`** HTML supports crawler and embed routes aligned with logicencoder.com SEO URLs.

## WordPress mirror push

When configured, an async task POSTs the enriched JSON to the plugin **`ethgas/v1/realtime`** route with a shared API key. WordPress stores a short-lived transient so REST readers and cache-bypass fallback pages see fresh tiers without hitting Python on every page view. Mission Control surfaces push latency, HTTP status, and last success timestamp.

When public site tiers are frozen but monitoring shows healthy fetch → inspect **last_push_http_status** and plugin push key mismatch before restarting uvicorn.

## Operator monitoring

**`/api/monitoring/overview`** aggregates uptime, ingest loop ms (last/avg/max), RPC fetch success rate, WebSocket client counts, SQLite row counts, history/heatmap/statistics cache hit ratios, WordPress push stats, and recent error strings. Sparkline arrays back wp-admin Mission Control charts.

When loop **avg ms** is climbing while **fetch_hard_timeouts** increment → RPC or Geth stress; **broadcast_failures** climbing with high client count → payload or network issue — check SQLite/cache next.

## Shared hosting headroom

The public gas product is published on **WordPress shared hosting** for the SPA shell, SEO templates, cache bypass, and REST mirror. Block-by-block RPC ingest, SQLite growth, WebSocket fan-out, and tier math run on **self-hosted Linux** with async workers and optional Geth; compact JSON pushes to WordPress. Visitors still get per-block updates over WebSocket; REST and the WP transient cover strict networks. Shared-hosting CPU and memory stay well below plan limits while the live tracker runs — same split pattern as other Logic Encoder realtime products on the same host.

Private code: [eth-gas-live-backend](https://github.com/logicencoder/eth-gas-live-backend)

Plugin overview: [eth-gas-live-plugin-overview](https://github.com/logicencoder/eth-gas-live-plugin-overview)

See [REPOS.md](REPOS.md).

---

## Feature examples (two per capability)

#### Block ingest and fee tiers
1. You start the gas service on your execution client and confirm each new head triggers a fresh Base Route, Standard Way, and Faster Inclusion tier on the public dashboard.
2. You compare wallet “medium” to Standard Way on the live API and explain to a user why EIP-1559 base and priority components differ from a single collapsed number.

#### ETH/USD price feed
1. You watch the header ETH price update from the exchange WebSocket feed while tiers recalculate in USD on the same block.
2. MEXC feed stalls and you confirm Binance REST fallback keeps USD estimates on the page until the primary feed recovers.

#### Rolling history and charts
1. You select seven days in the SPA and load history charts with utilization overlay for Standard gwei across the week.
2. You enable clamp smoothing on the client and reprocess cached series without another server round-trip during a demo.

#### Network stress and Intelligence Hub
1. Spike score crosses your mental threshold and the status label flips to SPIKE with a congestion reason you can quote in chat support.
2. You open the Intelligence Hub and read the one-line insight plus send-now/wait hint after comparing current Standard gwei to the 24h average.

#### WebSocket live updates
1. You connect to the gas WebSocket and receive the full enriched object on connect, then on every subsequent block without refreshing the page.
2. Corporate Wi-Fi blocks WS briefly and you fall back to polling `/api/gas/current` until the socket reconnects.

#### Featured transaction costs
1. You open featured actions and read Base/Standard/Faster USD for Swap (DEX) using the preset gas limits baked into the product.
2. You click a featured row in the SPA and the fee calculator preset jumps to that action’s standard gas limit automatically.

#### In-app fee calculator
1. You choose the Swap (DEX) preset, switch price mode to Faster Inclusion, and watch ETH and USD totals update from live tier prices.
2. You type a custom gas limit and gwei for a niche contract call and compare sidebar totals across all three tiers.

#### Threshold alerts
1. You create an “under” alert for Standard gwei and receive a WebSocket alert event the first time the network dips below your threshold.
2. The alert fires once and deactivates — you create a new rule when you want to watch the same condition again.

#### KLOD SEO pages and in-app hub
1. Googlebot requests a best-time-to-send URL and receives filled static HTML with live placeholder values — not an empty marketing shell.
2. You browse the same topic inside the SPA via the SEO hub tab strip without a full page reload.

#### WordPress realtime mirror
1. You configure the push URL and confirm enriched JSON reaches the plugin transient on the configured interval.
2. Push HTTP errors increment in monitoring overview — you fix the API key mismatch before restarting the execution client.

#### Operator monitoring
1. You poll monitoring overview before a deploy and record fetch success rate, WebSocket client count, and cache hit ratios as a baseline.
2. Loop average milliseconds climb while fetch timeouts increment — you check RPC health before blaming the WordPress plugin.

#### WSL database merge utility
1. You dry-run the merge script and review how many history rows would insert from laptop to server without writing anything.
2. You run a real merge after a dev session and confirm backups exist and only missing rows append — never overwrites.


---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
