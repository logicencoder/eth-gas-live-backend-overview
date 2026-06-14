# ETH Gas Live — backend service

Private **FastAPI + Node SSR** stack that ingests Ethereum blocks, computes EIP-1559 gas tiers and analytics, persists history, and streams live data to the WordPress front end.

**Private code:** [logicencoder/eth-gas-live-backend](https://github.com/logicencoder/eth-gas-live-backend)

**Product overview (portfolio):** [eth-gas-live-plugin-overview](https://github.com/logicencoder/eth-gas-live-plugin-overview) — live app at [logicencoder.com/ethereum-gas-tracker/](https://logicencoder.com/ethereum-gas-tracker/)

## Service role

- **`gas_tracker.py`** — block ingest (local Geth or hosted RPC), tier math, IPI/spike/utilization, SQLite history, WebSocket `/ws/gas`, REST API for SPA and SSR
- **`gas_tracker_ssr-server.js`** — React SSR for SEO HTML from `/api/gas/ssr-data`
- **KLOD SEO templates** — crawler/embed HTML with live placeholder fill
- Optional push of realtime JSON to WordPress REST mirror (`ethgas/v1/realtime`)

Visitors never hit this service directly for marketing pages — the **plugin** hosts the public UI on logicencoder.com.

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
