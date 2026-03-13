---
name: discover-lux
description: Auto-discover Lux blockchain skills based on context
---

# Discover Lux Ecosystem Skills

**Auto-activates when**: Lux, luxfi, luxd, subnet, validator, C-Chain, P-Chain, X-Chain, Q-Chain, M-Chain, Quasar consensus, Warp messaging, DEX, CLOB, or related blockchain keywords detected.

## Quick Reference

| Need | Skill | Load |
|------|-------|------|
| Run validator node | lux-node | `cat skills/lux/lux-node.md` |
| EVM / smart contracts | lux-evm | `cat skills/lux/lux-evm.md` |
| Quasar consensus | lux-consensus | `cat skills/lux/lux-consensus.md` |
| CLOB DEX engine | lux-dex | `cat skills/lux/lux-dex.md` |
| Go SDK client | lux-sdk | `cat skills/lux/lux-sdk.md` |
| CLI / local testing | lux-cli | `cat skills/lux/lux-cli.md` |
| Wallet / keys | lux-wallet | `cat skills/lux/lux-wallet.md` |
| Cryptography | lux-crypto | `cat skills/lux/lux-crypto.md` |
| FHE / privacy | lux-fhe | `cat skills/lux/lux-fhe.md` |
| Multisig | lux-safe | `cat skills/lux/lux-safe.md` |
| Cross-chain bridge | lux-bridge | `cat skills/lux/lux-bridge.md` |
| DEX / AMM | lux-exchange | `cat skills/lux/lux-exchange.md` |
| NFT marketplace | lux-market | `cat skills/lux/lux-market.md` |
| Block explorer | lux-explorer | `cat skills/lux/lux-explorer.md` |
| Monitoring | lux-monitoring | `cat skills/lux/lux-monitoring.md` |
| Production K8s | lux-universe | `cat skills/lux/lux-universe.md` |
| Network testing | lux-netrunner | `cat skills/lux/lux-netrunner.md` |

## Full Catalog

For complete skill listing with descriptions:
```
cat skills/lux/INDEX.md
```

## Key Rules (Always Apply)

1. **ALWAYS use luxfi packages** — NEVER go-ethereum or ava-labs
2. **NEVER use EWOQ keys** — generate fresh keys
3. **NEVER pkill luxd** — use `lux cli` for shutdown
4. **NEVER bump Go packages above v1.x.x**

## Context Detection

| Working Directory | Suggested Skills |
|-------------------|-----------------|
| ``github.com/luxfi/node`` | lux-node, lux-consensus |
| ``github.com/luxfi/evm`` | lux-evm, lux-precompile |
| ``github.com/luxfi/sdk`` | lux-sdk |
| ``github.com/luxfi/cli`` | lux-cli |
| ``github.com/luxfi/crypto`` | lux-crypto, lux-fhe, lux-lattice |
| ``github.com/luxfi/market`` | lux-market, lux-exchange |
| ``github.com/luxfi/bridge`` | lux-bridge |
| ``github.com/luxfi/universe`` | lux-universe, lux-deploy |
| ``github.com/luxfi/wallet`` | lux-wallet, lux-safe |

## Related Ecosystems

- **Hanzo AI**: `cat skills/hanzo/INDEX.md` (AI infrastructure)
- **Zoo Foundation**: Decentralized AI research

---

**Last Updated**: 2026-03-13
**Skills Count**: 30
**Format Version**: 1.0 (Gateway)
