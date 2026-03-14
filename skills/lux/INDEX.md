# Lux Ecosystem Skills

**Complete guide to Lux blockchain infrastructure, cryptography, DeFi, and developer tools**

## Overview

Lux Network is a multi-chain blockchain with Quasar post-quantum consensus, 5 chain types (P/X/C/Q/M), sub-second 2-round finality, native DEX precompiles, and full FHE support. 60 skills cover every component.

### Key Rules

- **ALWAYS use luxfi packages** — NEVER go-ethereum or ava-labs
- **NEVER use EWOQ keys** — generate fresh keys
- **NEVER pkill luxd** — use `lux cli` properly
- **Consensus is Quasar** — NOT Snow, Snowball, or Avalanche BFT

## Skill Catalog

### Core Blockchain (7 skills)

**Lux Node** (`lux-node.md`)
Core validator node (`luxd`). Go 1.26.1. 5 chains: P/X/C/Q/M. Ports 9630/9631.

**Lux EVM** (`lux-evm.md`)
Unified Go EVM engine. `github.com/luxfi/evm` v0.8.35+. Coreth merged in. DEX precompiles at 0x0400-0x0403.

**Lux Quasar** (`lux-consensus.md`)
Post-quantum consensus engine with Photonic Selection. 2-round finality. Dual BLS+lattice certificates.

**Lux DEX** (`lux-dex.md`)
CLOB order matching engine. 434M+ orders/sec (GPU). Go + C++ + Rust + MLX. FIX protocol.

**Lux VM** (`lux-vm.md`)
VM types and interfaces. Dual-transport RPC (ZAP/gRPC). Plugin system. `github.com/luxfi/vm`.

**Lux Session** (`lux-session.md`)
Post-quantum secure messaging VM. ML-KEM-768 + ML-DSA-65. `github.com/luxfi/session`.

**Lux P2P** (`lux-p2p.md`)
Peer-to-peer networking: libp2p, mDNS, PubSub messaging.

### Cryptography & Security (7 skills)

**Lux Crypto** (`lux-crypto.md`)
BLS12-381, SLIP-10 HD, secp256k1, KZG, gnark-crypto, ZK (Ziren).

**Lux FHE** (`lux-fhe.md`)
TFHE (boolean gates) + CKKS (approximate arithmetic). GPU acceleration. EVM integrated.

**Lux FHEVM** (`lux-fhevm.md`)
Fully Homomorphic Encryption VM. Confidential smart contracts on EVM. Zama fork.

**Lux Lattice** (`lux-lattice.md`)
`github.com/luxfi/lattice/v7`. Full-RNS RLWE: BFV/BGV/CKKS schemes. Multiparty. EPFL origin.

**Lux Safe** (`lux-safe.md`)
Multisig smart accounts. Safe-Global fork v1.5.0. ERC-4337. Solidity 0.7.6 / Hardhat.

**Lux Precompile** (`lux-precompile.md`)
Custom EVM precompiles, SessionVM, ERC20 Go implementation.

**Lux MPC** (`lux-mpc.md`)
Distributed threshold signing service (`mpcd`). CGGMP21 (ECDSA) + FROST (EdDSA). 20+ chains.

### Threshold Signatures (2 skills)

**Lux Threshold** (`lux-threshold.md`)
Universal multi-chain threshold signature library. Go. Post-quantum. 20+ blockchains. Taurusgroup fork.

**Lux Teleport** (`lux-teleport.md`)
ZK MPC cross-chain bridge. Threshold signatures for asset transfers. Zero-knowledge privacy.

### SDK & Tools (5 skills)

**Lux SDK** (`lux-sdk.md`)
Official Go SDK. `github.com/luxfi/sdk`. All 5 chain types. Auto-selects CLI/netrunner/API backend.

**Lux CLI** (`lux-cli.md`)
CLI for chain creation, deployment, validator management. `github.com/luxfi/cli`.

**Lux Netrunner** (`lux-netrunner.md`)
Local network runner. gRPC + HTTP gateway. Snapshot save/load. `github.com/luxfi/netrunner`.

**Lux Build** (`lux-build.md`)
Developer portal and documentation hub. Next.js 16 + Fumadocs. build.lux.network.

**Lux Plugins** (`lux-plugins.md`)
VM & subnet plugin registry for Lux Plugin Manager (LPM). YAML definitions.

### Wallets (5 skills)

**Lux Wallet** (`lux-wallet.md`)
Multi-platform HD wallet. `@luxwallet/monorepo`. Yarn 4. Desktop/Web/Mobile/Extension.

**Lux Web Wallet** (`lux-wwallet.md`)
Web-based wallet (Vue.js). X/P/C-Chain + EVM. Mnemonic, Ledger, keystore. `@luxfi/wallet-sdk`.

**Lux xWallet** (`lux-xwallet.md`)
Browser extension wallet. Rabby fork. Chrome/Firefox/Safari. Multi-chain DeFi. `@luxwallet/x`.

**Lux Desktop Wallet** (`lux-dwallet.md`)
Electron desktop wallet. Rabby Desktop fork. Built-in dapp browser. IPFS support.

**Lux Finance** (`lux-finance.md`)
DeFi web interface (Svelte 4 + Vite). Swaps, liquidity, staking, bridging via Connext. lux.finance.

### DeFi & Markets (6 skills)

**Lux Exchange** (`lux-exchange.md`)
DEX frontend + omnichain routing. Uniswap-based with native precompiles + CLOB integration.

**Lux Market** (`lux-market.md`)
NFT marketplace.

**Lux Liquid** (`lux-liquid.md`)
DeFi lending protocol. Alchemist V3. Self-repaying synthetic assets. Foundry + Echidna + Medusa.

**Lux Oracle** (`lux-oracle.md`)
Optimistic oracle. UMA Protocol fork. AGPL-3.0. Solidity/Ganache.

**Lux Governance** (`lux-governance.md`)
On-chain voting (`@pars/vote`), treasury, DAO. LPs tracked at `github.com/luxfi/lps`.

**Lux DAO** (`lux-dao.md`)
Governor contracts & governance platform. Fractal DAOs. Karma reputation. lux.vote.

### Cross-Chain (3 skills)

**Lux Warp** (`lux-warp.md`)
Cross-chain messaging protocol V2. BLS signature aggregation. PQ ringtail validation. `github.com/luxfi/warp`.

**Lux Bridge** (`lux-bridge.md`)
MPC-based cross-chain bridge. TypeScript + Go. 2-of-3 threshold MPC nodes.

**Lux Tokens** (`lux-tokens.md`)
Canonical token registry. Bridge mappings ETH↔Lux. Token logos. Top 100/150 lists.

### GPU & Acceleration (2 skills)

**Lux GPU** (`lux-gpu.md`)
Go bindings for GPU ops. Metal/CUDA/Dawn/ONNX/CPU backends. Tensor ops, crypto, FHE, ML inference.

**Lux Accel** (`lux-accel.md`)
GPU, FPGA, Rust crypto acceleration. `github.com/luxfi/accel`.

### Infrastructure & Ops (7 skills)

**Lux Universe** (`lux-universe.md`)
Production K8s infrastructure (private repo).

**Lux Stack** (`lux-stack.md`)
Local dev environment: Docker Compose with all services.

**Lux Deploy** (`lux-deploy.md`)
Network automation: K8s operators, DevOps tooling.

**Lux Operator** (`lux-operator.md`)
Kubernetes operator (Rust). 6 CRDs: LuxNetwork, LuxChain, LuxIndexer, LuxExplorer, LuxGateway, LuxMPC.

**Lux Monitoring** (`lux-monitoring.md`)
Grafana + Prometheus + Loki + Promtail + Alertmanager. 7 dashboards. monitor.lux.network.

**Lux Database** (`lux-database.md`)
Storage layer: ZapDB, caching, compression.

**Lux Migrate** (`lux-migrate.md`)
Blockchain data migration framework. SubnetEVM (PebbleDB), C-Chain (BadgerDB). JSONL format.

### AI & Compute (2 skills)

**Lux AI** (`lux-ai.md`)
Decentralized AI compute network. OpenAI-compatible API. GPU mining. Tauri desktop app.

**Lux ADX** (`lux-adx.md`)
CTV ad exchange. 100M+ impressions/day. OpenRTB. ZK privacy auctions. FoundationDB.

### Data & Indexing (4 skills)

**Lux Explorer** (`lux-explorer.md`)
Blockscout fork. Elixir. Block explorer backend.

**Lux Explore** (`lux-explore.md`)
Blockscout frontend fork. Next.js 16. `@luxfi/explore`. explore.lux.network.

**Lux Indexer** (`lux-indexer.md`)
Blockchain data indexing, Uniswap subgraphs.

**Lux Graph** (`lux-graph.md`)
Graph Node (The Graph fork). Rust. Blockchain data indexing via GraphQL. Event-sourcing EVM chains.

### Identity & Auth (3 skills)

**Lux ID** (`lux-id.md`)
Web3 identity portal. Next.js 15.1.6. `@luxfi/ui` + `@hanzo/auth`. lux.id.

**Lux Login** (`lux-login.md`)
Custom auth portal for lux.id. Next.js 15, OAuth2 + password. `@luxfi/login`.

**Lux Brand** (`lux-brand.md`)
Brand assets and design system. `@luxfi/brand` npm package. Colors, typography, logos, themes.

### Frontend & Content (4 skills)

**Lux UI** (`lux-ui.md`)
React components, charts, status dashboard.

**Lux Docs** (`lux-docs.md`)
docs.lux.network. Next.js 16 + `@hanzo/docs`. pnpm.

**Lux Bitcoin** (`lux-bitcoin.md`)
Bitcoin integration marketing site. Next.js 15. MDX content. ESG/green-BTC messaging.

**Lux FX** (`lux-fx.md`)
Cryptocurrency pricing API gateway. Single-file Go. CoinGecko proxy + cache. fx.lux.network.

### Configuration & Upgrades (1 skill)

**Lux Upgrade** (`lux-upgrade.md`)
Network upgrade scheduling. Go package. Fork activation times. `github.com/luxfi/upgrade`.

### Research (2 skills)

**Lux Faucet** (`lux-faucet.md`)
Testnet token distribution.

**Lux Papers** (`lux-papers.md`)
Academic whitepapers: Quasar consensus proofs, PQ crypto protocols.

## Decision Tree

```
What do you need?
├── Run a node → lux-node.md
├── Smart contracts → lux-evm.md + lux-precompile.md
├── Confidential contracts → lux-fhevm.md + lux-fhe.md
├── Understand consensus → lux-consensus.md (Quasar, NOT Snow)
├── Build Go client → lux-sdk.md
├── Test locally → lux-cli.md + lux-netrunner.md
├── High-perf DEX → lux-dex.md (CLOB) + lux-exchange.md (AMM)
├── DeFi lending → lux-liquid.md
├── DeFi frontend → lux-finance.md
├── Cross-chain messaging → lux-warp.md + lux-bridge.md + lux-teleport.md
├── Threshold signing → lux-mpc.md + lux-threshold.md
├── Custom VM → lux-vm.md + lux-session.md
├── GPU acceleration → lux-gpu.md + lux-accel.md
├── FHE operations → lux-fhe.md + lux-lattice.md + lux-fhevm.md
├── Web3 identity → lux-id.md + lux-login.md
├── Wallet (web) → lux-wwallet.md
├── Wallet (extension) → lux-xwallet.md
├── Wallet (desktop) → lux-dwallet.md
├── Wallet (multi-platform) → lux-wallet.md
├── Multisig → lux-safe.md
├── NFT marketplace → lux-market.md
├── DAO governance → lux-dao.md + lux-governance.md
├── Data indexing → lux-graph.md + lux-indexer.md
├── Explorer → lux-explore.md (frontend) + lux-explorer.md (backend)
├── Token registry → lux-tokens.md
├── Deploy K8s → lux-operator.md + lux-deploy.md + lux-universe.md
├── Monitor → lux-monitoring.md
├── Data migration → lux-migrate.md
├── AI compute → lux-ai.md
├── Ad exchange → lux-adx.md
├── Brand/design → lux-brand.md
├── Developer docs → lux-build.md + lux-docs.md
├── VM plugins → lux-plugins.md
├── Fork scheduling → lux-upgrade.md
└── Pricing API → lux-fx.md
```

## Related Ecosystems

- **Hanzo AI** (`github.com/hanzoai/skills`) — AI infrastructure
- **Zoo Foundation** — Decentralized AI research

---

**Last Updated**: 2026-03-13
**Total Skills**: 60
**Gateway**: `discover-lux/SKILL.md`
