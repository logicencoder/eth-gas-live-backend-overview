# ETH Gas Live — Backend Overview (Public)

Public-facing summary of the **Ethereum Gas Live** backend.  
No private source code, credentials, or deployment secrets are published here.

![ETH Gas Live — dashboard](screenshot.png)

**Live page:** https://logicencoder.com/ethereum-gas-tracker/

---

## What problem does this solve?

Sending an Ethereum transaction at the wrong moment can cost far more than necessary. Fees move block by block, mempools fill up, and “recommended gas” from a single number is often misleading.

This backend powers a **real-time gas intelligence dashboard** that helps you:

- See **current fee tiers** in plain language (economical, standard, faster)  
- Understand **network stress** before you send  
- Use **charts, heatmaps, and rolling statistics** to pick a better time window  
- Estimate costs for **common actions** (transfer, swap, approve, bridge, NFT, etc.)  
- Set **alerts** when fees cross your threshold  
- Operate a **live public site** with fast updates over WebSocket  

---

## Who is it for?

- Traders and operators who send transactions often  
- Developers evaluating realtime dashboard + blockchain data products  
- Teams who want a **self-hosted** gas monitor tied to their own node or RPC  
- Visitors to LogicEncoder’s live gas tracker page  

---

## What you get (capabilities)

### Real-time monitoring

- Live connection and network status  
- Current block and mempool visibility  
- Safe / standard / fast style tiers with GWEI, ETH, and USD estimates  
- Short **guidance text** (pressure + spike awareness) so you know if waiting is reasonable  

### Analytics & decisions

- Historical charts with selectable time ranges  
- **Heatmap** by day and hour (shown in the visitor’s local timezone on the live UI)  
- Rolling **~25 hour** statistics window  
- **Fee calculator** with presets for typical transaction types  
- **Featured actions** — ballpark costs for swaps, transfers, bridges, and more  

### Reliability & operations (conceptual)

- WebSocket stream for low-latency updates  
- REST endpoints for history and statistics when a snapshot is enough  
- Health and **mission-style monitoring** surface for uptime and pipeline visibility  
- Historical storage so charts stay useful after restarts  

### Discoverability (live product)

- Indexable **SEO pages** (fees today, calculator, network status, mempool, etc.) backed by the same live data  

---

## How it fits the live product

```
Visitor browser
    → WordPress page (native dashboard UI)
    → Backend over WebSocket + REST (realtime gas data)
    → Optional SEO pages (server-rendered summaries for search)
```

The WordPress plugin is documented separately:  
[eth-gas-live-plugin-overview-public](https://github.com/logicencoder/eth-gas-live-plugin-overview-public)

---

## Technology (high level only)

- Python, FastAPI, async I/O  
- Ethereum RPC / node integration  
- SQLite for history  
- WebSocket + REST  
- Companion SSR layer for SEO HTML  
- Cloudflare Tunnel (or equivalent) for stable public access in production  

---

## Related public repos

| Repository | Contents |
|------------|----------|
| [eth-gas-tracker-overview](https://github.com/logicencoder/eth-gas-tracker-overview) | Whole-product story (standalone + live) |
| [eth-gas-live-plugin-overview-public](https://github.com/logicencoder/eth-gas-live-plugin-overview-public) | WordPress-facing overview |

Private implementation: `logicencoder/eth-gas-live-backend-private` (not public).

---

## Disclosure

This repository is intentionally **documentation and screenshots only**.  
It exists so recruiters, collaborators, and users can understand **what the system does** without exposing how every line is implemented.
