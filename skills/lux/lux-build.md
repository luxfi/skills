# Lux Build - Builders Hub & Developer Portal

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-docs.md`, `lux/lux-sdk.md`, `lux/lux-node.md`

## Overview

Lux Build is the **developer portal and documentation hub** for the Lux Network. It hosts technical docs, Lux Academy courses, an integrations directory, and a blog. Deployed at build.lux.network. Built with Next.js 16 and Fumadocs (MDX-based documentation framework).

## Quick reference

| Item | Value |
|------|-------|
| Repo | `github.com/luxfi/build` |
| Branch | `main` |
| Package name | `lux-build` |
| Framework | Next.js 16 (Turbopack) |
| Docs engine | Fumadocs (fumadocs-core, fumadocs-mdx, fumadocs-ui, fumadocs-openapi) |
| Content format | MDX |
| Package manager | pnpm |
| Database | Prisma (PostgreSQL) |
| Auth | NextAuth |
| Image | `ghcr.io/hanzoai/lux-build:latest` |
| Domain | build.lux.network |
| Port | 3000 |

## Project Structure

```
build/
├── app/                    # Next.js app directory
│   ├── (home)/             # Landing pages
│   ├── academy/            # Lux Academy courses
│   ├── api/                # API routes
│   ├── blog/               # Blog pages
│   ├── console/            # Developer console
│   ├── docs/               # Documentation pages
│   ├── hackathons/         # Hackathon pages
│   ├── integrations/       # Integrations directory
│   ├── llms.txt/           # LLM-readable docs
│   └── llms-full.txt/      # Full LLM-readable docs
├── content/                # MDX content source
│   ├── academy/            # Academy course content
│   ├── blog/               # Blog posts
│   ├── common/             # Shared content
│   ├── docs/               # Technical documentation
│   ├── integrations/       # Integration guides
│   └── courses.tsx         # Course definitions
├── abi/                    # Smart contract ABIs
├── components/             # React components
├── constants/              # App constants
├── contracts/              # Contract interaction code
├── hooks/                  # React hooks
├── lib/                    # Shared libraries
├── prisma/                 # Database schema
├── server/                 # Server-side code
├── k8s/                    # Kubernetes manifests
│   └── lux-build.yaml      # Deployment + Service
├── Dockerfile              # Multi-stage build (node:22-alpine)
├── package.json            # Dependencies
├── source.config.ts        # Fumadocs config
├── next.config.mjs         # Next.js config
└── tsconfig.json           # TypeScript config
```

## Key Dependencies

```
@luxfi/cloud          — Lux Cloud SDK
@luxfi/core           — Lux Core library
@luxfi/lamport        — Lamport signature scheme
@luxfi/logo           — Lux logo assets
@luxfi/wallet-sdk     — Wallet SDK
luxfi                 — Core Lux package
fumadocs-core/ui/mdx  — Documentation framework
@ai-sdk/openai        — AI-powered features
viem                  — Ethereum interaction
@safe-global/*        — Safe multisig integration
```

## Development Commands

```bash
pnpm install                 # Install dependencies
pnpm dev                     # Start dev server (Turbopack)
pnpm build                   # Production build
pnpm lint                    # ESLint check
pnpm fumadocs-mdx            # Generate MDX content
npx tsx scripts/generate-event-signatures.mts  # Generate ABI signatures
```

## Content Sections

| Section | Path | Description |
|---------|------|-------------|
| Docs | `/content/docs/` | Technical documentation (chains, VMs, APIs, SDKs) |
| Academy | `/content/academy/` | Structured learning courses |
| Blog | `/content/blog/` | Engineering blog posts |
| Integrations | `/content/integrations/` | Third-party integration guides |
| Courses | `/content/courses.tsx` | Course catalog definitions |

## White-Label Branding

Lux Build supports per-brand deployments via `NEXT_PUBLIC_*` environment variables:

### Brand Environment Variables

```bash
NEXT_PUBLIC_BRAND_NAME=Lux          # Brand display name
NEXT_PUBLIC_BRAND_LOGO=/lux-logo.svg # Logo path
NEXT_PUBLIC_BRAND_THEME=lux          # Theme identifier
NEXT_PUBLIC_BRAND_DOMAIN=lux.build   # Canonical domain
NEXT_PUBLIC_BRAND_CHAIN_ID=96369     # Default chain ID
```

### Per-Brand .env Files

```
.env.example.zoo        # Zoo brand: build.zoo.network
.env.example.hanzo      # Hanzo brand: build.hanzo.ai
.env.example.pars       # Pars brand: build.pars.network
.env.example.liquidity  # Liquidity white-label brand
```

### Brand Hostnames

| Brand | Hostname |
|-------|----------|
| Lux | `lux.build` |
| Zoo | `build.zoo.network` |
| Hanzo | `build.hanzo.ai` |
| Pars | `build.pars.network` |

## Kubernetes Deployment

### Standard Deployment

```yaml
# k8s/lux-build.yaml
Namespace: hanzo
Image: ghcr.io/hanzoai/lux-build:latest
Port: 3000 (container) -> 80 (service)
Resources: 200m-1000m CPU, 256Mi-1Gi memory
Probes: HTTP GET / (liveness + readiness)
```

### White-Label Deployment

```yaml
# k8s/build-wl.yaml (in universe repo)
# Per-brand ConfigMaps + Deployments + IngressRoutes
# Each brand gets:
#   - ConfigMap with NEXT_PUBLIC_BRAND_* vars
#   - Deployment referencing that ConfigMap
#   - IngressRoute for the brand hostname
```

The `build-wl.yaml` manifest in `universe/k8s/` defines all brand variants as separate Deployments sharing the same image but with distinct ConfigMaps for branding.

## Docker Build

Multi-stage build:
1. **deps** -- pnpm install + prisma generate
2. **builder** -- fumadocs-mdx + ABI generation + next build
3. **runner** -- Standalone Next.js server (node:22-alpine, port 3000)

## Contributing

- Docs in `content/docs/` (MDX format)
- Images in `public/images/`
- Style guide at `style-guide.md`
- PRs against `main` branch
- Run `pnpm dev` to verify build passes before submitting

## Related Skills

- `lux/lux-docs.md` -- docs.lux.network (separate docs site)
- `lux/lux-sdk.md` -- Go SDK documentation
- `lux/lux-node.md` -- Node documentation

---

**Last Updated**: 2026-03-24
**Category**: Lux Ecosystem
**Related**: documentation, developer-portal, academy, build, MDX
