# Lux Skills - AI Assistant Guide

## Overview

This is the Lux blockchain skills repository. It provides atomic, progressively-disclosed knowledge about every component of the Lux ecosystem for AI coding assistants.

## Key Rules

1. **ALWAYS use luxfi packages** — NEVER use go-ethereum or ava-labs imports
2. **NEVER use EWOQ keys** — always generate fresh keys for testing
3. **NEVER pkill luxd** — use `lux cli` for proper node management
4. **NEVER bump Go packages above v1.x.x**
5. Use `compose.yml` not `docker-compose.yml`

## Directory Structure

```
skills/
├── lux/              # 30+ Lux ecosystem skills
├── discover-lux/     # Auto-discovery gateway
└── _SKILL_TEMPLATE.md
```

## Skill Categories

### Core Blockchain
- lux-node.md — Validator node (Go)
- lux-evm.md — EVM execution engine
- lux-consensus.md — Snow consensus family

### Cryptography
- lux-crypto.md — Primitives and hashing
- lux-fhe.md — Fully Homomorphic Encryption
- lux-lattice.md — Post-quantum lattice crypto
- lux-safe.md — Multisig + FROST threshold

### Networking
- lux-p2p.md — P2P layer, libp2p, mDNS, PubSub

### DeFi & Markets
- lux-market.md — NFT marketplace
- lux-exchange.md — DEX + AMM
- lux-oracle.md — Price feeds

### Infrastructure
- lux-universe.md — Production K8s (lux-k8s cluster)
- lux-stack.md — Local dev environment
- lux-deploy.md — Deployment automation
- lux-monitoring.md — Prometheus + Grafana

### Tools & SDKs
- lux-sdk.md — Go SDK
- lux-cli.md — CLI tool
- lux-wallet.md — HD wallets
- lux-netrunner.md — Network testing

## Related Ecosystems

- **Hanzo AI** (`github.com/hanzoai/skills`) — AI infrastructure
- **Zoo Foundation** — Decentralized AI research
