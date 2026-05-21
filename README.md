# ETH Gas Live — Backend Capabilities (Public Overview)

Backend capabilities for the live system at [logicencoder.com/ethereum-gas-tracker/](https://logicencoder.com/ethereum-gas-tracker/).

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

### Network stress signals

- **Transactions per minute** (estimated throughput)  
- **Block utilization** (how full recent blocks are)  
- **Inclusion Pressure Index (IPI)** — 0–100 score combining throughput, fullness, and tip spread  
- **Spike score** — 0–100 short-term regime vs recent hours  
- **Fee competition** — how wide priority-fee spreads are  
- **Block speed pressure** — share of blocks near capacity  

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
| Network load | `/ethereum-mempool-tracker/` |
| Network status | `/ethereum-network-status/` |
| Price history | `/ethereum-gas-price-history/` |
| Fee percentiles | `/ethereum-gas-percentiles/` |

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

The public WordPress site embeds the **same dashboard experience** and loads topic pages from the shared backend. WordPress overview: [eth-gas-live-plugin-overview](https://github.com/logicencoder/eth-gas-live-plugin-overview).

Full product narrative: [eth-gas-live-backend-overview](https://github.com/logicencoder/eth-gas-live-backend-overview).

---

## Live link

**https://logicencoder.com/ethereum-gas-tracker/**
