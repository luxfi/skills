# Lux Bootnode - Bootstrap Node Infrastructure

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-node.md`, `lux/lux-deploy.md`, `lux/lux-universe.md`

## Overview

Lux Bootnode provides bootstrap node infrastructure for the Lux Network. Bootstrap nodes are the initial peers that new validators contact to discover the rest of the network. The Bootnode service also powers the multi-chain RPC API infrastructure at `bootno.de`, providing managed blockchain access (100+ chains) for the Lux, Hanzo, Zoo, and Pars ecosystems.

## When to use

- Configuring bootstrap nodes for new Lux network deployments
- Setting up initial peer discovery for mainnet, testnet, or devnet validators
- Deploying or managing the Bootnode RPC API service
- Integrating with the Bootnode multi-chain API (bootno.de)

## Quick reference

| Item | Value |
|------|-------|
| Dashboard | https://bootno.de/dashboard |
| API base | https://api.bootno.de/v1 |
| Agentic gateway | https://gateway.bootno.de/v1 |
| WebSocket | wss://ws.bootno.de/v1 |
| Status | https://status.bootno.de |
| Docs | https://bootno.de/docs |
| K8s namespace | `bootnode` |
| K8s manifests | `~/work/lux/universe/k8s/bootnode/` |

## Bootstrap Nodes

Bootstrap nodes are regular Lux validators designated as initial contact points. New validators connect to them first to discover the full peer set.

### Configuration

```bash
# Point a new validator at bootstrap nodes
./build/luxd \
  --network-id=mainnet \
  --bootstrap-ips=<bootstrap-ip-1>:9631,<bootstrap-ip-2>:9631 \
  --bootstrap-ids=<bootstrap-node-id-1>,<bootstrap-node-id-2>
```

### Production Bootstrap

Production bootstrap nodes run on the `lux-k8s` DOKS cluster. K8s manifests are in `~/work/lux/universe/k8s/bootnode/`. Secrets (staking keys, TLS certs) are synced from `kms.hanzo.ai` via `KMSSecret` CRDs.

## Bootnode Multi-Chain API (bootno.de)

The Bootnode API provides managed RPC access to 100+ chains. It is the same infrastructure that powers the white-label platforms:

| Platform | Domain |
|----------|--------|
| Bootnode | bootno.de |
| Hanzo Web3 | web3.hanzo.ai |
| Lux Cloud | cloud.lux.network |
| Zoo Labs | web3.zoo.ngo |
| Pars Cloud | cloud.pars.network |

### Access Methods

1. **API Key** -- Traditional approach via `X-API-Key` header. Get a key at https://bootno.de/dashboard
2. **Agentic Gateway** -- Keyless via x402 USDC micropayments at `gateway.bootno.de/v1`

### Skills

For detailed API usage, see the Bootnode skills repo (`github.com/bootnode/skills`):
- `bootnode-api` -- Full API-key access (RPC, data, webhooks, smart wallets)
- `agentic-gateway` -- Keyless x402 USDC micropayment access

## Deployment

### K8s (Production)

```bash
# Deploy bootstrap nodes
cd ~/work/lux/universe/k8s/bootnode
kubectl kustomize . | kubectl apply -f -
```

### Images

- Node: `ghcr.io/luxfi/node:<tag>` (always `--platform linux/amd64`)
- Bootnode API: `ghcr.io/hanzoai/bootnode-api:<tag>`

## Related Skills

- `lux/lux-node.md` -- Core validator node that bootstrap nodes run
- `lux/lux-deploy.md` -- Network deployment automation
- `lux/lux-universe.md` -- Production K8s infrastructure (contains bootnode manifests)
- `lux/lux-p2p.md` -- P2P networking and peer discovery

---

**Category**: Lux Ecosystem
**Related**: bootnode, bootstrap, peer-discovery, rpc-api, infrastructure
**Prerequisites**: K8s access for deployment, or API key for RPC access
