# Lux Cloud Skills

Agent Skills for integrating Lux Cloud blockchain APIs into AI-native workflows.

## Skills

### lux-api

Use the traditional API-key approach for full-featured access to 100+ chains.

- **Auth:** API key in header (`X-API-Key`) or URL path
- **Setup:** Create free key at [cloud.lux.network/dashboard](https://cloud.lux.network/dashboard)
- **Entry point:** `skills/lux-api/SKILL.md`

### lux-gateway

Keyless access via SIWE authentication + x402 USDC micropayments. No account needed.

- **Auth:** SIWE token + x402 USDC payment per request
- **Setup:** Generate wallet, fund with USDC on Base
- **Entry point:** `skills/lux-gateway/SKILL.md`

## Which skill should I use?

| Scenario | Skill |
|----------|-------|
| I have a Lux Cloud API key | `lux-api` |
| I want keyless access with USDC | `lux-gateway` |
| Building a server-side app | `lux-api` |
| Building an autonomous agent | `lux-gateway` |
| Need webhooks or websockets | `lux-api` |
| Need pay-per-request billing | `lux-gateway` |

## Installation

**Open standard** (works with any agent):
```bash
npx skills add luxfi/skills --yes
```

**Hanzo Bot** (installs to `~/.hanzo/skills/` and symlinks to all agents):
```bash
npx @hanzo/bot skills add luxfi/skills --yes
```

Skills are installed to `~/.hanzo/skills/luxfi-skills/` and automatically symlinked to:
- `~/.claude/skills/` (Claude Code)
- `~/.agents/skills/` (Codex, Openclaw)
- `~/.cursor/skills/` (Cursor)
- `~/.hanzo/bot/skills/` (Hanzo Bot)

## Available on all platforms

These skills work on any Bootnode-powered platform:

| Platform | Domain | Install |
|----------|--------|---------|
| Bootnode | bootno.de | `npx skills add bootnode/skills` |
| Hanzo Web3 | web3.hanzo.ai | `npx skills add hanzoai/skills` |
| Lux Cloud | cloud.lux.network | `npx skills add luxfi/skills` |
| Zoo Labs | web3.zoo.ngo | `npx skills add zooai/skills` |
| Pars Cloud | cloud.pars.network | `npx skills add parsnetwork/skills` |

## Compatible with

- [Hanzo Bot](https://hanzo.bot) (`npx @hanzo/bot`)
- Claude Code
- Codex
- Cursor
- Openclaw
- Any agent supporting the [Agent Skills spec](https://agentskills.io/specification)

## Architecture

See [HIP-009: Unified Agent Skills Architecture](https://github.com/hanzoai/hips/blob/main/HIP-009-agent-skills.md) for the full design rationale.

`~/.hanzo/skills/` is the canonical source of truth. All agents see the same skills via symlinks.

## Powered by

[Hanzo AI](https://hanzo.ai) - Enterprise AI infrastructure

## Specification

Follows the [Agent Skills spec](https://agentskills.io/specification).

## License

MIT
