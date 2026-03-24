# ETH Gas Live Backend — Overview (Public)

This repository is a public overview only and excludes private implementation details.

![ETH Gas Live Backend UI](screenshot.png)

## Purpose
Provide realtime Ethereum gas intelligence through a FastAPI backend used by local UI and WordPress plugin clients.

Live production page:

- `https://logicencoder.com/ethereum-gas-tracker/`

Public backend host:

- Deployment-specific (configure your own backend host/domain)

## Recent UI Improvements

- **Fully responsive mobile layout** — adapts seamlessly from desktop to phone without requiring a hard refresh
- **Custom chart legend with click-to-toggle** — click any legend item (Base Route / Standard / Faster Inclusion / Avg Utilization) to show or hide that dataset on the chart
- **Chart timestamps aligned to plot area** — timeline ticks and legend are dynamically padded to stay inside the chart borders at all screen sizes (via Chart.js `afterRender` plugin)
- **Local timezone everywhere** — all hourly statistics, heatmap day/hour grid, and Best/Worst time to send are converted to the visitor's local timezone (no hardcoded UTC labels)
- **Heatmap midnight crossover fix** — `transformHeatmapToLocal()` correctly shifts UTC data across day boundaries so each local calendar day shows its correct 24 hours
- **Rolling 25-hour statistics window** — statistics always cover the last 25 hours from now regardless of time of day (not just since UTC midnight)
- **Hourly averages chronological order** — the hourly averages panel lists hours starting from 25 hours ago through the current hour, giving a true rolling timeline view
- **Heatmap auto-refresh on hour boundary** — detected via incoming WebSocket timestamps with `Date.now()` fallback
- **Unified dropdown styling** — all select menus share consistent dark theme with no double-border artefacts

## Architecture
- Ingestion: Ethereum RPC + mempool/block metrics
- Processing: fee tiers, percentile/volatility/time analysis
- Storage: SQLite historical persistence
- Delivery:
  - WebSocket (`/ws/gas`) for realtime updates
  - REST (`/api/gas/*`) for analytics/history
  - Monitoring (`/api/monitoring/overview`) for operations visibility
- WordPress bridge: optional push mirror to plugin realtime cache endpoint

## Backup Node Support

The backend auto-detects the Ethereum node host at startup:

1. Attempts to connect to `127.0.0.1:8545` (local Geth)
2. If unreachable, falls back to `GETH_FALLBACK_HOST` from `.env`
3. `GETH_HOST` in `.env` overrides both auto-detection steps

This makes the same codebase portable across machines — set `GETH_FALLBACK_HOST` on any machine that doesn't run a local node.

## Runtime Domains

### 1) Realtime domain

- websocket snapshots (`WS /ws/gas`)
- alert events in same stream channel
- block-by-block updates

### 2) Analytics domain

- historical series APIs
- statistics and prediction APIs
- heatmap and featured-action APIs

### 3) Operations domain

- health endpoint
- runtime observability endpoint (`/api/monitoring/overview`)
- cache/db telemetry and loop timing signals

## Monitoring Payload Surface

Mission Control consumes these payload sections:

- `runtime`
- `current_state`
- `fetch_pipeline`
- `websocket`
- `alerts`
- `wordpress_push`
- `api_cache`
- `database`
- `timeseries`

This enables operational visibility without exposing private code internals.

## End-to-End Data Flow

1. Node data is fetched and normalized into gas snapshot payloads.
2. Snapshots are persisted to SQLite.
3. Snapshots are broadcast to websocket clients.
4. Analytics endpoints read from cached + persisted datasets.
5. Optional signed mirror payloads are pushed to WordPress cache bridge.
6. Mission Control polls monitoring endpoint for runtime diagnostics.

## Operational model
- Backend runs independently
- Cloudflare Tunnel provides stable public routing
- Plugin consumes backend with websocket-only realtime mode

## Stack

- Python
- FastAPI
- AsyncIO
- Web3 RPC integration
- SQLite
- WebSocket + REST
- Cloudflare Tunnel

## Security note
No secrets, private source internals, or credentials are published here.
