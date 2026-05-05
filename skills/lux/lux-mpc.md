# Lux MPC - Multi-Party Computation Wallet Service

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-threshold.md`, `lux/lux-hsm.md`, `lux/lux-bridge.md`

## Overview

Lux MPC (`mpcd`) is a **distributed threshold signing service** for securely generating and managing cryptographic wallets across MPC nodes -- without ever exposing the full private key. It supports 5 signing protocols (CGGMP21, FROST, LSS, BLS, SR25519) across 7+ blockchain families (BTC, ETH, SOL, TON, XRP, DOT, KSM) with consensus-embedded transport (ZAP protocol) and per-tenant isolation via OrgID scoping.

## Quick reference

| Item | Value |
|------|-------|
| Module | `github.com/luxfi/mpc` |
| Go | 1.26.1 |
| Binaries | `mpcd` (daemon), `lux-mpc-cli` (tooling) |
| API Port | 8081 (dashboard API), 9651 (consensus transport) |
| Default Branch | `main` |
| License | Apache 2.0 |

## Hard requirements

1. ALWAYS use `github.com/luxfi/*` packages -- NEVER upstream forks (no `go-ethereum`, no `ava-labs`)
2. NEVER store passwords in plaintext -- ZapDB uses ChaCha20-Poly1305 encryption
3. ALWAYS use CBOR serialization for FROST/LSS configs -- JSON corrupts crypto types

## Architecture

```
mpcd
├── cmd/
│   ├── mpcd/              # Main MPC daemon (consensus-embedded)
│   └── lux-mpc-cli/       # CLI tools for config, keygen, identity
├── pkg/
│   ├── mpc/               # MPC engine (TSS protocols)
│   ├── transport/         # Consensus-embedded transport (ZAP + PoA)
│   ├── client/            # Go client library
│   ├── kvstore/           # ZapDB/BadgerDB encrypted storage
│   ├── identity/          # Ed25519 identity management
│   ├── protocol/          # Protocol session management
│   ├── api/               # Dashboard REST API
│   ├── backup/            # Encrypted backup/restore
│   ├── hsm/               # HSM integration layer
│   ├── kms/               # KMS integration
│   ├── messaging/         # NATS messaging (DEPRECATED)
│   ├── infra/             # Consul integration (DEPRECATED)
│   ├── settlement/        # Settlement attestation
│   ├── smart/             # Smart contract interaction
│   ├── threshold/         # Threshold protocol wrappers
│   ├── encryption/        # Encryption utilities
│   └── txtracker/         # Transaction tracking
├── e2e/                   # End-to-end tests
├── deployments/           # Bridge compatibility deployment
├── k8s/                   # Kubernetes manifests
└── dashboard/             # Dashboard UI
```

## Signing Protocols

| Protocol | Curve | Use Case |
|----------|-------|----------|
| CGGMP21 | secp256k1 | ECDSA signing for BTC (Legacy/SegWit), ETH/EVM, XRP, Lux |
| FROST | Ed25519 | EdDSA signing for SOL, TON, BTC (Taproot) |
| LSS | secp256k1 | Lightweight secret sharing (alternative ECDSA) |
| BLS | BLS12-381 | Aggregate threshold signatures |
| SR25519 | Ristretto | Substrate/Polkadot/Kusama signing |

## Multi-Chain Support

| Network | Curve | Protocol | Address Derivation |
|---------|-------|----------|--------------------|
| Bitcoin (Legacy/SegWit) | secp256k1 | CGGMP21/LSS | BIP32/BIP44 |
| Bitcoin (Taproot) | secp256k1 | FROST | BIP86 |
| Ethereum/EVM | secp256k1 | CGGMP21/LSS | BIP44 m/44'/60'/0'/0/i |
| Solana | Ed25519 | FROST | BIP44 m/44'/501'/0'/0' |
| TON | Ed25519 | FROST | TON-specific |
| XRPL | secp256k1 | CGGMP21/LSS | XRPL derivation |
| Polkadot/Kusama | Ristretto | SR25519 | Substrate SS58 |
| Lux Network | secp256k1 | CGGMP21/LSS | BIP44 m/44'/9000'/0'/0/i |

## Threshold Scheme

Uses **t-of-n** threshold signing with `t >= floor(n/2) + 1`:

- `n` = total MPC nodes (key shares)
- `t` = minimum nodes required to sign
- Full private key is **never reconstructed**

### Production Topology: 3-of-5

Standard deployment uses 3-of-5 threshold:
- **Customer operates 2 nodes** (self-custody partial control)
- **Lux operates 3 nodes** (infrastructure reliability)
- Any 3 nodes can sign (customer + 1 Lux, or 2 customer + 1 Lux)
- No single party can sign unilaterally

### Safe Multisig Integration

MPC-generated keys can be used as Safe multisig owners, combining MPC threshold signing with smart contract-level governance. The bridge and custody products use this pattern for institutional-grade security.

## Transport Modes

### Consensus-Embedded (Production Standard)

No external dependencies (no NATS, no Consul). Uses ZAP wire protocol with PoA membership:

```bash
mpcd start --mode consensus \
  --node-id node0 \
  --listen :9651 \
  --api :9800 \
  --data /data/mpc/node0 \
  --threshold 2 \
  --peer node1@127.0.0.1:9652 \
  --peer node2@127.0.0.1:9653
```

ZAP message types 60-79 for MPC operations (broadcast, direct, keygen, sign, reshare, result).

### Legacy (NATS + Consul) -- DEPRECATED

```bash
lux-mpc-cli generate-peers -n 3
lux-mpc-cli register-peers
lux-mpc-cli generate-initiator
mpcd start --mode legacy -n node0
```

## Multi-Tenant Isolation (OrgID)

All key material is scoped to an OrgID for per-tenant isolation:

```go
// GetKeyShareWithFallback tries org-scoped key first, falls back to global
func GetKeyShareWithFallback(db ZapDB, orgID, walletID string) (*KeyShare, error) {
    key := OrgScopedKey(orgID, walletID)  // "org:{orgID}:wallet:{walletID}"
    share, err := db.Get(key)
    if err == ErrNotFound {
        return db.Get(walletID)  // Legacy fallback (pre-org migration)
    }
    return share, err
}
```

- Keygen sessions tag shares with OrgID at creation time
- Signing sessions validate OrgID matches before proceeding
- Dashboard API filters entities by `orgId` from JWT claims
- Migration path: legacy keys without OrgID are accessible via fallback

## Secret Erasure

Key material is zeroed after use via `defer` patterns:

```go
defer func() {
    for i := range secretBytes {
        secretBytes[i] = 0
    }
}()
```

Note: `runtime/secret` package is a placeholder -- actual zeroing is done inline with defer. The Go runtime does not guarantee memory erasure, but this pattern prevents accidental retention in heap.

## Storage Layer (ZapDB)

Uses `github.com/luxfi/database` (ZapDB abstraction) -- NOT BadgerDB directly:

- ChaCha20-Poly1305 encryption at rest (password from env `BADGER_PASSWORD`)
- CBOR serialization for FROST/LSS configs (JSON corrupts crypto types)
- Automatic compaction and garbage collection
- Encrypted backup/restore with configurable intervals

## Authentication

JWT auth with KMS-sourced secrets:

- Dashboard API validates JWT tokens on every request
- JWT signing keys sourced from KMS (kms.hanzo.ai) -- never hardcoded
- No default/fallback secrets in production (`MPC_JWT_SECRET` must be set)
- Token claims include `orgId`, `role`, `permissions`

## One-file quickstart

### Build from source

```bash
git clone https://github.com/luxfi/mpc.git
cd mpc
make build
# Or install directly:
go install ./cmd/mpcd
go install ./cmd/lux-mpc-cli
```

### Configuration

```yaml
# config.yaml
nats:
  url: nats://127.0.0.1:4222
consul:
  address: localhost:8500
mpc_threshold: 2
environment: local
badger_password: "32-byte-password-for-AES-256..."
event_initiator_pubkey: "hex-encoded-ed25519-pubkey"
max_concurrent_keygen: 2
db_path: "."
backup_enabled: true
backup_period_seconds: 300
backup_dir: backups
```

### Go client

```go
import (
    "github.com/luxfi/mpc/pkg/client"
    "github.com/nats-io/nats.go"
)

natsConn, _ := nats.Connect("nats://localhost:4222")
mpcClient := client.NewMPCClient(client.Options{
    NatsConn: natsConn,
    KeyPath:  "./event_initiator.key",
})
mpcClient.OnWalletCreationResult(func(event event.KeygenSuccessEvent) {
    // Handle wallet creation
})
mpcClient.CreateWallet(walletID)
```

## Key Dependencies

```
github.com/luxfi/threshold@v1.5.5   -- CGGMP21, FROST, LSS protocols
github.com/luxfi/hsm@v1.1.0         -- HSM/KMS integration
github.com/luxfi/crypto@v1.17.40    -- BLS, secp256k1, certificates
github.com/luxfi/database@v1.17.43  -- ZapDB encrypted storage
github.com/luxfi/fhe@v1.7.6         -- FHE primitives
github.com/luxfi/lattice/v7@v7.0.0  -- Post-quantum lattice crypto
github.com/luxfi/log@v1.4.1         -- Structured logging
github.com/hanzoai/orm@v0.3.2       -- ORM for dashboard API
github.com/hanzoai/kv-go/v9         -- Valkey/Redis client
```

## Supported Networks (Legacy Table)

See "Multi-Chain Support" table above for the complete and current list.

## Production Deployment

- **Namespace**: `lux-mpc` (3 nodes, dashboard API, postgres, valkey)
- **Storage**: ZapDB with ChaCha20-Poly1305 encryption
- **Dashboard API**: Port 8081, enabled via `MPC_API_DB` env var
- **Multi-tenancy**: One postgres, `_entities` JSONB table, `kind` + `orgId` scoping
- **Binary distribution**: S3 bucket `lux-mpc-backups/binaries/`

## Performance

| Operation | Timing |
|-----------|--------|
| Key Generation | ~30s for 3 nodes |
| Signing | <1s for threshold signatures |
| Storage per node | ~100MB with backups |

## Testing

```bash
# Unit tests
go test ./... -v

# E2E tests
cd e2e && make test

# Coverage
make test-coverage
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Protocol message corruption | Using JSON for FROST/LSS configs | Use CBOR via `MarshalFROSTConfig()` |
| Party ID mismatch | Inconsistent ordering | Ensure `GetReadyPeersIncludeSelf()` sorts IDs |
| NATS topic mismatch | Wrong prefix | Use `mpc.mpc_keygen_result.<walletID>` |
| Self-message warnings | Normal pub/sub behavior | Ignore "Handler cannot accept message" logs |
| Session hangs | No timeout on protocol handler | Add context with timeout |

## Related Skills

- `lux/lux-threshold.md` -- Threshold signature library (CGGMP21, FROST, LSS)
- `lux/lux-hsm.md` -- HSM/KMS integration for key protection
- `lux/lux-bridge.md` -- Bridge integration using MPC signing

---

**Category**: Lux Ecosystem
**Related**: mpc, threshold, signing, custody, wallet
**Prerequisites**: Go 1.26+
