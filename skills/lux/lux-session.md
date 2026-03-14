# Lux Session - Post-Quantum Secure Messaging VM

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-crypto.md`, `lux/lux-node.md`, `lux/lux-vm.md`

## Overview

SessionVM is a **pluggable virtual machine** for the Lux blockchain that provides end-to-end encrypted, post-quantum secure private messaging. It uses ML-KEM-768 (FIPS 203) for key encapsulation and ML-DSA-65 (FIPS 204) for digital signatures, with XChaCha20-Poly1305 for symmetric encryption.

## Quick reference

| Item | Value |
|------|-------|
| Module | `github.com/luxfi/session` |
| Go | 1.26.1 |
| VMID | `sessionvm` (Base58: `2ZbQaVuXHtT7vfJt8FmWEQKAT4NgtPqWEZHg5m3tUvEiSMnQNt`) |
| License | BSD-3-Clause |

## Cryptographic Primitives

| Algorithm | Purpose | Standard | Key Sizes |
|-----------|---------|----------|-----------|
| ML-KEM-768 | Key encapsulation | FIPS 203 | PK: 1184, SK: 2400, CT: 1088 |
| ML-DSA-65 | Digital signatures | FIPS 204 | PK: 1952, SK: 4032, Sig: 3309 |
| XChaCha20-Poly1305 | AEAD encryption | RFC 8439 | Key: 32, Nonce: 24 |
| Blake2b-256 | Hashing | RFC 7693 | Output: 32 |

## Session ID Format

Session IDs use a prefix system to identify the cryptographic suite:

- `07` -- Post-quantum (ML-KEM-768 + ML-DSA-65)
- `05` -- Legacy (X25519 + Ed25519)

Format: `<prefix>` + hex(Blake2b-256(KEM_pk || DSA_pk)) = 66 characters

## Architecture

```
github.com/luxfi/session
├── crypto/
│   ├── identity.go         # PQ identity generation (ML-KEM + ML-DSA)
│   └── identity_test.go    # Identity crypto tests
├── core/
│   ├── id.go               # Session ID type and parsing
│   ├── session.go          # Session lifecycle (pending/active/expired/closed)
│   └── step.go             # Protocol step execution
├── vm/
│   ├── vm.go               # SessionVM core (sessions, messages, channels)
│   ├── service.go          # JSON-RPC 2.0 service
│   ├── factory.go          # VM factory (creates SessionVM instances)
│   └── vm_test.go          # VM unit tests
├── network/
│   ├── peer.go             # Peer connection management
│   └── transport.go        # Network transport layer
├── protocol/
│   ├── attestation.go      # Message attestation
│   ├── domain.go           # Protocol domain separation
│   ├── oracle.go           # Oracle for external data
│   └── receipt.go          # Delivery receipts
├── storage/
│   ├── store.go            # Storage interface
│   ├── memory.go           # In-memory storage backend
│   └── encoding.go         # Message serialization
├── swarm/
│   ├── assignment.go       # Node assignment for message routing
│   └── registry.go         # Swarm node registry
├── daemon/
│   └── service.go          # Standalone daemon service
├── plugin/
│   └── main.go             # Lux node plugin entry point
├── cmd/                    # CLI entry points
├── e2e/                    # End-to-end tests
├── e2e_test.go             # E2E test runner
└── docs/                   # Documentation
```

## One-file quickstart

### Install

```bash
go get github.com/luxfi/session
```

### Generate identity and encrypt

```go
import (
    "github.com/luxfi/session/crypto"
    "github.com/luxfi/session/vm"
)

// Generate post-quantum identity
identity, err := crypto.GenerateIdentity()
// identity.SessionID: "07abc123..." (66 chars)
// identity.KEMPublicKey: 1184 bytes (ML-KEM-768)
// identity.DSAPublicKey: 1952 bytes (ML-DSA-65)

// Encrypt to recipient
ciphertext, err := crypto.EncryptToRecipient(recipientKEMPublicKey, plaintext)

// Sign message
signature, err := crypto.Sign(identity.DSASecretKey, message)

// Verify signature
valid := crypto.Verify(identity.DSAPublicKey, message, signature)
```

### Integrate with Pars node

```go
import "github.com/luxfi/session/vm"

provider, _ := vm.NewSessionProvider(logger)
identity, _ := provider.GenerateIdentity()
session, _ := provider.CreateSecureSession(ctx, identity, remotePublicKey)
```

## RPC Methods

The VM exposes JSON-RPC 2.0 over HTTP:

| Method | Description |
|--------|-------------|
| `CreateSession` | Create a new encrypted session |
| `GetSession` | Retrieve session by ID |
| `SendMessage` | Send encrypted message in session |
| `CloseSession` | Terminate a session |
| `Health` | Health check |

## Key Dependencies

```
github.com/luxfi/crypto@v1.17.38   — ML-KEM, ML-DSA, Blake2b (via cloudflare/circl)
github.com/luxfi/ids@v1.2.9        — ID types
github.com/luxfi/log@v1.4.1        — Structured logging
github.com/gorilla/rpc@v1.2.1      — JSON-RPC server
golang.org/x/crypto@v0.47.0        — XChaCha20-Poly1305
```

## Configuration

```json
{
  "sessionTTL": 86400,
  "maxMessages": 10000,
  "maxChannels": 1000,
  "retentionDays": 30,
  "idPrefix": "07"
}
```

## Benchmarks (Apple M1 Max)

```
BenchmarkGenerateIdentity:         268us/op
BenchmarkEncapsulateDecapsulate:   226us/op
BenchmarkSignVerify:               1.08ms/op
BenchmarkCreateSession:            3.8us/op
BenchmarkSendMessage:              1.9us/op
BenchmarkGetSession:               16ns/op
```

## Testing

```bash
# Run all tests with race detection
go test -v -race ./...

# Run benchmarks
go test -bench=. -benchmem ./...
```

## Security Properties

1. **Post-Quantum Security**: ML-KEM-768 and ML-DSA-65 provide NIST Level 3
2. **Forward Secrecy**: Each message uses fresh KEM encapsulation
3. **Authentication**: All messages signed with ML-DSA-65
4. **Confidentiality**: XChaCha20-Poly1305 authenticated encryption

## Related Repositories

- **luxcpp/session** -- C++ storage server with GPU acceleration (Metal)
- **luxfi/crypto** -- Cryptographic primitives (ML-KEM, ML-DSA, Blake2b)
- **parsdao/node** -- Pars blockchain node with SessionVM integration

## Related Skills

- `lux/lux-crypto.md` -- Underlying PQ cryptographic primitives
- `lux/lux-node.md` -- Host node that runs SessionVM
- `lux/lux-vm.md` -- VM interface that SessionVM implements
- `lux/lux-p2p.md` -- Network transport layer

---

**Last Updated**: 2026-03-13
**Category**: Lux Ecosystem
**Related**: session, messaging, post-quantum, ML-KEM, ML-DSA, privacy
**Prerequisites**: Go 1.26+
