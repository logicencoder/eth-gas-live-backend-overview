# ETH Gas Live Backend — Overview (Public)

This repository is a public overview only and excludes private implementation details.

![ETH Gas Live Backend UI](screenshot.png)

## Purpose
Provide realtime Ethereum gas intelligence through a FastAPI backend used by local UI and WordPress plugin clients.

## Architecture
- Ingestion: Ethereum RPC + mempool/block metrics
- Processing: fee tiers, percentile/volatility/time analysis
- Storage: SQLite historical persistence
- Delivery:
  - WebSocket (`/ws/gas`) for realtime updates
  - REST (`/api/gas/*`) for analytics/history
  - Monitoring (`/api/monitoring/overview`) for operations visibility
- WordPress bridge: optional push mirror to plugin realtime cache endpoint

## Operational model
- Backend runs independently
- Cloudflare Tunnel provides stable public routing
- Plugin consumes backend with websocket-only realtime mode

## Security note
No secrets, private source internals, or credentials are published here.
