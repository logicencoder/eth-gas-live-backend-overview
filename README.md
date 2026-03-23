# ETH Gas Live Backend — Overview (Public)

This repository is a public overview only and excludes private implementation details.

![ETH Gas Live Backend UI](screenshot.png)

## Purpose
Provide realtime Ethereum gas intelligence through a FastAPI backend used by local UI and WordPress plugin clients.

Live production page:

- `https://logicencoder.com/ethereum-gas-tracker/`

Public backend host:

- Deployment-specific (configure your own backend host/domain)

## Architecture
- Ingestion: Ethereum RPC + mempool/block metrics
- Processing: fee tiers, percentile/volatility/time analysis
- Storage: SQLite historical persistence
- Delivery:
  - WebSocket (`/ws/gas`) for realtime updates
  - REST (`/api/gas/*`) for analytics/history
  - Monitoring (`/api/monitoring/overview`) for operations visibility
- WordPress bridge: optional push mirror to plugin realtime cache endpoint

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
