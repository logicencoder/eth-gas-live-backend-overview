# ETH Gas Live — Backend Capabilities (Public Overview)

This document describes **what the live Ethereum Gas backend does** and **who it helps**. It matches the production system behind [logicencoder.com/ethereum-gas-tracker/](https://logicencoder.com/ethereum-gas-tracker/). No source code or credentials are published in this repository.

---

## Problem

Sending on Ethereum without context leads to:

- Paying **too much** because “fast” was selected blindly  
- Paying **too little** and waiting hours during congestion  
- Missing **quieter windows** when fees dip  
- Misreading a single GWEI number without tier breakdown or USD cost  

The backend exists to turn raw chain data into **tiered, explained, historical, and time-aware** guidance updated every block (or near it on local Geth).

---

## What the system delivers

### Realtime fee tiers

Three named paths updated continuously:

| Tier | Typical use |
|------|-------------|
| **Base Route** | Minimum viable inclusion (base fee only) |
| **Standard Way** | Balanced cost for most transfers and interactions |
| **Faster Inclusion** | Higher tip profile when blocks are competitive |

Each tier is shown in **GWEI**, estimated **ETH**, and **USD** (live ETH price feed).

### Network stress signals (not a mempool counter)

Public RPC endpoints rarely expose trustworthy global mempool depth. The product therefore focuses on metrics that **are** observable from recent blocks:

- **Transactions per minute** (estimated throughput)  
- **Block utilization** (how full recent blocks are)  
- **Inclusion Pressure Index (IPI)** — 0–100 score combining throughput, fullness, and tip spread  
- **Spike score** — 0–100 short-term regime vs recent hours  
- **Fee competition** — how wide priority-fee spreads are  
- **Block speed pressure** — share of blocks near capacity  

Together these answer: *“Is now a bad time to send?”* without pretending to know an exact pending-tx count.

### Historical and analytical views

- **Price charts** — 1 hour through 30 days for all three tiers  
- **Heatmap** — hour-of-day patterns (browser uses local timezone on the public site)  
- **Rolling averages** — 1h, 3h, 6h, 12h, 24h, 3d, 7d, 30d  
- **Percentiles** on the faster tier (P50–P85)  
- **Advanced statistics** — min/max/avg/median, volatility, best/worst hours in a selected window  

### Planning tools

- **Gas Intelligence Hub** — narrative insight, best/worst send windows (24h context), **send now vs wait** recommendation  
- **Fee calculator** — gas limit presets + custom limit; price tied to Base / Standard / Faster / custom GWEI  
- **Featured transaction costs** — estimates for 18 common actions (transfer, swap, NFT, bridge, DeFi ops, contract deploy, etc.) at each tier  

### Alerts

Browser-session alerts when Standard (or configured) GWEI crosses **above** or **below** a threshold, with optional desktop notifications.

### Indexable educational pages

Separate URLs (served for search engines and deep links) explain focused topics. Content is filled with **live numbers** (fees, IPI, utilization, tx/min) so pages stay relevant as the market moves.

| Page topic | URL path (on LogicEncoder) |
|------------|----------------------------|
| Live tracker hub | `/ethereum-gas-tracker/` |
| Fees today | `/ethereum-gas-fees-today/` |
| Why fees are high | `/why-are-ethereum-gas-fees-high/` |
| Best time to send | `/best-time-to-send-ethereum/` |
| Gas calculator | `/ethereum-gas-calculator/` |
| Transaction costs | `/ethereum-transaction-costs/` |
| Swap gas costs | `/ethereum-swap-gas-fees/` |
| Network load & throughput | `/ethereum-mempool-tracker/` *(educational “network load” page — not a live mempool depth widget)* |
| Network status | `/ethereum-network-status/` |
| Price history | `/ethereum-gas-price-history/` |
| Fee percentiles | `/ethereum-gas-percentiles/` |

The live app’s in-page hub links to these topics; the hub strip intentionally omits a separate “mempool” tab because the live dashboard does not show mempool size.

---

## Who benefits

| Audience | Value |
|----------|--------|
| **Retail senders** | See cost before confirming a wallet transaction |
| **Active traders / DeFi users** | Compare tier costs for swaps, approvals, bridges |
| **Content / SEO visitors** | Land on specific questions (“fees today”, “best time”) with current data |
| **Operators** | Health telemetry (uptime, fetch success, WS clients) without reading logs |

---

## How it connects to the website

The public WordPress site embeds the **same dashboard experience** and loads topic pages from the shared backend. WordPress overview: [eth-gas-live-plugin-overview-public](https://github.com/logicencoder/eth-gas-live-plugin-overview-public).

Full product narrative: [eth-gas-tracker-overview](https://github.com/logicencoder/eth-gas-tracker-overview).

---

## What this repository is

Documentation and high-level capability description for portfolios and evaluation. Implementation lives in private repositories (by invitation).

---

## Live link

**https://logicencoder.com/ethereum-gas-tracker/**
