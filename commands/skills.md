---
name: skills
description: Discover and load Lux blockchain skills
user_invocable: true
---

# /skills - Lux Skills Discovery

Discover relevant Lux blockchain skills based on your current context.

## Activation

When the user runs `/skills` or asks about Lux blockchain components:

1. **Detect context** from the current working directory and conversation:
   - If in `node/` → suggest lux-node.md
   - If in `evm/` → suggest lux-evm.md
   - If in `consensus/` → suggest lux-consensus.md
   - If in `crypto/`, `lamport/`, `lattice/` → suggest lux-crypto.md
   - If in `market/`, `exchange/` → suggest lux-market.md, lux-exchange.md
   - If in `bridge/`, `warp/`, `teleport/` → suggest lux-bridge.md
   - If in `universe/` → suggest lux-universe.md
   - If in `sdk/` → suggest lux-sdk.md

2. **Load the gateway skill** if no specific context:
   ```
   cat `github.com/luxfi/skills`skills/discover-lux/SKILL.md
   ```

3. **Load specific skill** if context is clear:
   ```
   cat `github.com/luxfi/skills`skills/lux/{skill-name}.md
   ```

## Arguments

- `/skills` — Show contextual recommendations
- `/skills lux` — Show all Lux skills
- `/skills {keyword}` — Search for specific skill

## Output

Present 3-5 most relevant skills with one-line descriptions and load commands.
