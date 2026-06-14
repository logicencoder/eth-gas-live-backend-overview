# ETH Gas Live — backend service

Private **FastAPI + Node SSR** service that powers the live gas dashboard and SEO pages on logicencoder.com. It ingests Ethereum blocks, computes EIP-1559 tiers and congestion metrics, stores history, streams updates over WebSocket, and optionally mirrors JSON to WordPress.

**Private code:** [logicencoder/eth-gas-live-backend](https://github.com/logicencoder/eth-gas-live-backend)

**Public product (portfolio):** [eth-gas-live-plugin-overview](https://github.com/logicencoder/eth-gas-live-plugin-overview) — [logicencoder.com/ethereum-gas-tracker/](https://logicencoder.com/ethereum-gas-tracker/)

Visitors use the **WordPress plugin** for the public UI. This backend is the compute and API layer behind that app.

## Role in the stack

The plugin serves `gas_tracker.html`, routes SEO templates, and hosts Mission Control. All gas math, block ingest, SQLite history, WebSocket broadcast, and SSR data generation run here — not in PHP.

Optional push posts enriched JSON to WordPress REST **`GET/POST /wp-json/ethgas/v1/realtime`** so the plugin can serve a transient mirror when configured.

## Block ingest and tiers

Each new block triggers fee sampling from **local Geth** (preferred, `newHeads` WebSocket or ~1s poll) or hosted RPC (Ankr, Infura, Alchemy, and others via environment variables).

Three tiers map to the live UI cards:

| Tier | Meaning |
|------|---------|
| **Base Route** | Base fee path with minimal priority tip |
| **Standard Way** | Typical inclusion from recent block fee sampling |
| **Faster Inclusion** | Higher percentile tips with congestion caps |

Each payload includes GWEI, priority component, ETH and USD estimates, and estimated confirmation windows (~12s per block).

**ETH/USD** comes from MEXC protobuf WebSocket with Binance REST fallback — not from the execution client.

## Network metrics computed per block

Inclusion Pressure Index (**IPI** 0–100), **Spike** score, tx/minute estimate, block utilization, fee competition (median priority spread), network status (SPIKE / HIGH / LOW / NORMAL), and rolling 1h/24h averages for charts and “predict send / wait” guidance.

## Persistence and retention

SQLite **`gas_history`** plus alerts table — roughly **30 days** retention with purge on ingest. Powers Chart.js ranges (1h–30d), heatmaps (hour buckets shifted to visitor timezone in the browser), statistics endpoints, and featured-action cost tables (**18 action types** recomputed each tick).

## Live API surface

| Type | Path | Role |
|------|------|------|
| WebSocket | `/ws/gas` | Full gas object each block + alert events |
| REST | `/api/gas/current`, `/history`, `/heatmap`, `/statistics`, `/predict` | SPA fallback and analytics |
| REST | `/api/gas/ssr-data` | Single bundle for SSR and SEO placeholder fill |
| REST | `/api/gas/featured-actions` | Tier costs for common transaction types |
| REST | `/api/monitoring/overview` | Operator health (WS clients, fetch stats, cache) |
| REST | `/api/alerts*` | Threshold-based alert configuration |
| Proxy | `/ssr_gas/page`, `/ethereum-gas/` | Forwards to Node SSR server |
| Static | `/` | Serves `gas_tracker.html` SPA shell |

**KLOD SEO HTML** from `ssr_klod_files/` — crawler and embed routes with live placeholder substitution aligned to the eleven public SEO URLs on logicencoder.com.

## Node SSR helper

**`gas_tracker_ssr-server.js`** (Express + React) renders indexable HTML from `/api/gas/ssr-data` for routes proxied by FastAPI. Same data model as the browser SPA — no duplicate gas logic in WordPress.

## Key files

| File | Role |
|------|------|
| `gas_tracker.py` | FastAPI app, ingest loop, SQLite, WS, REST |
| `gas_tracker_ssr-server.js` | React SSR |
| `gas_tracker.html`, `tracker.css` | SPA shell (also copied to plugin repo) |
| `ssr_klod_files/` | SEO HTML templates |

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
