# Lux Exchange - DEX Frontend & Omnichain Routing

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-dex.md`, `lux/lux-evm.md`, `lux/lux-oracle.md`

## Overview

Lux Exchange is the **DEX frontend and routing layer** — a Uniswap-based interface with native EVM DEX precompiles and CLOB integration via `github.com/luxfi/dex`. Supports AMM swaps, limit orders via CLOB, and cross-chain routing via Warp/Teleport.

This is the **frontend/routing layer**. The high-performance order matching engine is at `github.com/luxfi/dex` (separate skill).

## Quick reference

| Item | Value |
|------|-------|
| Repo | `github.com/luxfi/exchange` |
| Language | TypeScript |
| Runtime | Bun (required) |
| Build | Nx workspace |
| URL | lux.exchange |
| License | GPL-3.0-or-later |

## Components

| Component | Repo | Purpose |
|-----------|------|---------|
| exchange | `github.com/luxfi/exchange` | DEX web/mobile UI |
| exchange-api | `github.com/luxfi/exchange-api` | DEX API service |
| exchange-proxy | `github.com/luxfi/exchange-proxy` | DEX router |
| exchange-sdk | `github.com/luxfi/exchange-sdk` | Exchange SDK |

## DEX Precompiles (Native EVM)

Lux C-Chain includes native DEX precompiles at reserved addresses:

| Precompile | Address | Purpose |
|------------|---------|---------|
| **PoolManager** | `0x0400` | AMM pool creation and management |
| **SwapRouter** | `0x0401` | Swap routing (AMM + CLOB) |
| **HooksRegistry** | `0x0402` | Custom swap hooks (fees, oracles, etc.) |
| **FlashLoan** | `0x0403` | Flash loan facility |

These precompiles execute at native speed (no EVM interpretation overhead).

## Omnichain Routing

```
User Order
    ↓
┌──────────────────┐
│  Omnichain Router │
├──────────────────┤
│ 1. Check AMM pools│ → PoolManager (0x0400)
│ 2. Check CLOB     │ → luxfi/dex engine
│ 3. Cross-chain    │ → Warp/Teleport messaging
│ 4. Best execution │ → Route to optimal venue
└──────────────────┘
    ↓
Execution Report
```

## Key Dependencies

```
@uniswap/router-sdk    — Uniswap routing logic
@uniswap/sdk-core      — Core DEX types
@uniswap/v2-sdk        — V2 AMM integration
react@19               — UI framework
react-native@0.79.5    — Mobile app
viem@2.30.5            — Ethereum client
tamagui                — Cross-platform UI
```

## DeFi Ecosystem

| Component | Repo | Description |
|-----------|------|-------------|
| Markets | `github.com/luxfi/markets` | Morpho Blue lending/borrowing |
| V2 Subgraph | `github.com/luxfi/uni-v2-subgraph` | V2 AMM indexer |
| V3 Subgraph | `github.com/luxfi/uni-v3-subgraph` | V3 concentrated liquidity indexer |
| V4 Subgraph | `github.com/luxfi/uni-v4-subgraph` | V4 hooks indexer |

## Related Skills

- `lux/lux-dex.md` — CLOB engine (434M orders/sec)
- `lux/lux-evm.md` — EVM with DEX precompiles
- `lux/lux-oracle.md` — Price feeds
- `lux/lux-bridge.md` — Cross-chain routing

---

**Last Updated**: 2026-03-13
**Category**: Lux Ecosystem
**Related**: dex, defi, amm, swap, uniswap
**Prerequisites**: TypeScript, Bun, DeFi concepts
