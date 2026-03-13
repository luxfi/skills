# Lux Ecosystem Skills

**Complete guide to Lux blockchain infrastructure, cryptography, and DeFi**

## Overview

Lux Network is a multi-chain blockchain with Quasar post-quantum consensus, 5 chain types (P/X/C/Q/M), sub-second 2-round finality, and native DEX precompiles. 31 skills cover every component.

### Key Rules

- **ALWAYS use luxfi packages** — NEVER go-ethereum or ava-labs
- **NEVER use EWOQ keys** — generate fresh keys
- **NEVER pkill luxd** — use `lux cli` properly
- **Consensus is Quasar** — NOT Snow, Snowball, or Avalanche BFT

## Skill Catalog

### Core Blockchain (5 skills)

**Lux Node** (`lux-node.md`)
Core validator node (`luxd`). Go 1.26.1. 5 chains: P/X/C/Q/M. Ports 9630/9631.
`cat skills/lux/lux-node.md`

**Lux EVM** (`lux-evm.md`)
Unified Go EVM engine. `github.com/luxfi/evm` v0.8.35+. Coreth merged in. DEX precompiles at 0x0400-0x0403.
`cat skills/lux/lux-evm.md`

**Lux Quasar** (`lux-consensus.md`)
Post-quantum consensus engine with Photonic Selection. 2-round finality. Dual BLS+lattice certificates.
`cat skills/lux/lux-consensus.md`

**Lux DEX** (`lux-dex.md`)
CLOB order matching engine. 434M+ orders/sec (GPU). Go + C++ + Rust + MLX. FIX protocol.
`cat skills/lux/lux-dex.md`

**Lux P2P** (`lux-p2p.md`)
Peer-to-peer networking: libp2p, mDNS, PubSub messaging.
`cat skills/lux/lux-p2p.md`

### Cryptography & Security (5 skills)

**Lux Crypto** (`lux-crypto.md`)
BLS12-381, SLIP-10 HD, secp256k1, KZG, gnark-crypto, ZK (Ziren).
`cat skills/lux/lux-crypto.md`

**Lux FHE** (`lux-fhe.md`)
TFHE (boolean gates) + CKKS (approximate arithmetic). GPU acceleration. EVM integrated.
`cat skills/lux/lux-fhe.md`

**Lux Lattice** (`lux-lattice.md`)
`github.com/luxfi/lattice/v7`. Full-RNS RLWE: BFV/BGV/CKKS schemes. Multiparty. EPFL origin.
`cat skills/lux/lux-lattice.md`

**Lux Safe** (`lux-safe.md`)
Multisig smart accounts. Safe-Global fork v1.5.0. ERC-4337. Solidity 0.7.6 / Hardhat.
`cat skills/lux/lux-safe.md`

**Lux Precompile** (`lux-precompile.md`)
Custom EVM precompiles, SessionVM, ERC20 Go implementation.
`cat skills/lux/lux-precompile.md`

### SDK & Tools (4 skills)

**Lux SDK** (`lux-sdk.md`)
Official Go SDK. `github.com/luxfi/sdk`. All 5 chain types. Auto-selects CLI/netrunner/API backend.
`cat skills/lux/lux-sdk.md`

**Lux CLI** (`lux-cli.md`)
CLI for chain creation, deployment, validator management. `github.com/luxfi/cli`.
`cat skills/lux/lux-cli.md`

**Lux Wallet** (`lux-wallet.md`)
Multi-platform HD wallet. `@luxwallet/monorepo`. Yarn 4. Desktop/Web/Mobile/Extension.
`cat skills/lux/lux-wallet.md`

**Lux Netrunner** (`lux-netrunner.md`)
Local network runner. gRPC + HTTP gateway. Snapshot save/load. `github.com/luxfi/netrunner`.
`cat skills/lux/lux-netrunner.md`

### DeFi & Markets (4 skills)

**Lux Exchange** (`lux-exchange.md`)
DEX frontend + omnichain routing. Uniswap-based with native precompiles + CLOB integration.
`cat skills/lux/lux-exchange.md`

**Lux Market** (`lux-market.md`)
NFT marketplace.
`cat skills/lux/lux-market.md`

**Lux Oracle** (`lux-oracle.md`)
Optimistic oracle. UMA Protocol fork. AGPL-3.0. Solidity/Ganache.
`cat skills/lux/lux-oracle.md`

**Lux Governance** (`lux-governance.md`)
On-chain voting (`@pars/vote`), treasury, DAO. LPs tracked at `github.com/luxfi/lps`.
`cat skills/lux/lux-governance.md`

### Cross-Chain (1 skill)

**Lux Bridge** (`lux-bridge.md`)
MPC-based cross-chain bridge. TypeScript + Go. 2-of-3 threshold MPC nodes.
`cat skills/lux/lux-bridge.md`

### Infrastructure & Ops (6 skills)

**Lux Universe** (`lux-universe.md`)
Production K8s infrastructure (private repo).
`cat skills/lux/lux-universe.md`

**Lux Stack** (`lux-stack.md`)
Local dev environment: Docker Compose with all services.
`cat skills/lux/lux-stack.md`

**Lux Deploy** (`lux-deploy.md`)
Network automation: K8s operators, DevOps tooling.
`cat skills/lux/lux-deploy.md`

**Lux Monitoring** (`lux-monitoring.md`)
Grafana + Prometheus + Loki + Promtail + Alertmanager. 7 dashboards. monitor.lux.network.
`cat skills/lux/lux-monitoring.md`

**Lux Database** (`lux-database.md`)
Storage layer: ZapDB, caching, compression.
`cat skills/lux/lux-database.md`

**Lux Accel** (`lux-accel.md`)
GPU, FPGA, Rust crypto acceleration. `github.com/luxfi/accel`.
`cat skills/lux/lux-accel.md`

### Data & Content (4 skills)

**Lux Explorer** (`lux-explorer.md`)
Blockscout fork. Elixir. Block explorer.
`cat skills/lux/lux-explorer.md`

**Lux Indexer** (`lux-indexer.md`)
Blockchain data indexing, Uniswap subgraphs.
`cat skills/lux/lux-indexer.md`

**Lux UI** (`lux-ui.md`)
React components, charts, status dashboard.
`cat skills/lux/lux-ui.md`

**Lux Docs** (`lux-docs.md`)
docs.lux.network. Next.js 16 + `@hanzo/docs`. pnpm.
`cat skills/lux/lux-docs.md`

### Research (2 skills)

**Lux Faucet** (`lux-faucet.md`)
Testnet token distribution.
`cat skills/lux/lux-faucet.md`

**Lux Papers** (`lux-papers.md`)
Academic whitepapers: Quasar consensus proofs, PQ crypto protocols.
`cat skills/lux/lux-papers.md`

## Decision Tree

```
What do you need?
├── Run a node → lux-node.md
├── Smart contracts → lux-evm.md + lux-precompile.md
├── Understand consensus → lux-consensus.md (Quasar, NOT Snow)
├── Build Go client → lux-sdk.md
├── Test locally → lux-cli.md + lux-netrunner.md
├── High-perf DEX → lux-dex.md (CLOB) + lux-exchange.md (AMM)
├── DeFi/DEX → lux-exchange.md + lux-oracle.md
├── Cross-chain → lux-bridge.md
├── Cryptography → lux-crypto.md + lux-fhe.md + lux-lattice.md
├── Wallet ops → lux-wallet.md + lux-safe.md
└── Deploy prod → lux-universe.md + lux-deploy.md
```

## Related Ecosystems

- **Hanzo AI** (`github.com/hanzoai/skills`) — AI infrastructure
- **Zoo Foundation** — Decentralized AI research

---

**Last Updated**: 2026-03-13
**Total Skills**: 31
**Gateway**: `discover-lux/SKILL.md`
