# Lux Skills

Atomic skills for AI assistants working with the Lux blockchain ecosystem.

## Overview

This skills repository provides comprehensive, progressively-disclosed knowledge about every component of the Lux blockchain stack. Skills are designed for Claude Code and other AI assistants to load on-demand when working with Lux projects.

## Structure

```
skills/
├── lux/                    # Lux ecosystem product skills (30+)
│   ├── INDEX.md            # Full skill catalog
│   ├── lux-node.md         # Core validator node
│   ├── lux-evm.md          # EVM execution engine
│   ├── lux-consensus.md    # Snow consensus family
│   └── ...
├── discover-lux/           # Auto-discovery gateway skill
│   └── SKILL.md
└── _SKILL_TEMPLATE.md      # Template for new skills
```

## Progressive Loading

1. **Tier 1 - Gateway** (`discover-lux/SKILL.md`): Auto-activates on Lux keywords, ~200 lines
2. **Tier 2 - Index** (`lux/INDEX.md`): Full catalog with descriptions, ~500 lines
3. **Tier 3 - Individual Skills** (`lux/*.md`): Deep-dive per component, ~300 lines each

## Usage

```bash
# Auto-discovery (loaded by Claude Code on Lux keywords)
cat skills/discover-lux/SKILL.md

# Browse all Lux skills
cat skills/lux/INDEX.md

# Load specific skill
cat skills/lux/lux-node.md
```

## Key Principles

- **luxfi packages only** — NEVER use go-ethereum or ava-labs imports
- **NEVER use EWOQ keys** — always generate fresh keys
- **NEVER pkill luxd** — use `lux cli` for proper shutdown
- Real code examples with actual APIs and endpoints
- Security-first: all sensitive ops have warnings

## Related

- [Hanzo Skills](https://github.com/hanzoai/skills) — AI infrastructure skills
- [Lux Documentation](https://docs.lux.network) — Official docs
- [Lux Network](https://lux.network) — Main website

---

**Maintained by**: Lux Network / Hanzo AI
**License**: MIT
