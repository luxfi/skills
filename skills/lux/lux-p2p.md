# Lux P2P - Peer-to-Peer Networking

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-node.md`, `lux/lux-consensus.md`

## Overview

Lux P2P networking layer — libp2p integration, mDNS peer discovery, publish-subscribe messaging for consensus and state propagation.

## Components

| Component | Path | Purpose |
|-----------|------|---------|
| net/ | ``github.com/luxfi/net`` | Core networking |
| p2p/ | ``github.com/luxfi/p2p`` | P2P protocols |
| libp2p/ | ``github.com/libp2p/go-libp2p`` | libp2p integration |
| mdns/ | ``github.com/luxfi/mdns`` | mDNS peer discovery |
| pubsub/ | ``github.com/luxfi/pubsub`` | PubSub messaging |

## Key Concepts

- **Peer discovery**: mDNS for local, DHT for global
- **Message routing**: Gossip protocol for block/tx propagation
- **PubSub topics**: Per-chain topic subscriptions
- **Connection management**: Peer scoring, rate limiting

---

**Last Updated**: 2026-03-13
**Category**: Lux Ecosystem
