# Lux Login - Custom Auth Portal for lux.id

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-brand.md`

## Overview

Lux Login is the **custom authentication portal** for Lux Network, providing a monochrome dark-themed sign-in experience at `lux.id`. It supports both direct password login (via Casdoor/IAM API) and OAuth2 Authorization Code flow against `lux.id` IAM.

## Quick Reference

| Item | Value |
|------|-------|
| Repo | `github.com/luxfi/login` |
| Package | `@luxfi/login` v0.1.0 |
| Default branch | `main` |
| Framework | Next.js 15 (App Router) |
| Package manager | pnpm 9.15.4 |
| Node | 22 |
| Docker image | `ghcr.io/luxfi/login:latest` |
| Port | 3000 |

## Architecture

```
login/
├── app/
│   ├── page.tsx            # Login page (email/password + OAuth)
│   ├── signup/page.tsx     # Sign up page
│   ├── forgot/page.tsx     # Forgot password page
│   ├── callback/page.tsx   # OAuth callback handler
│   ├── layout.tsx          # Root layout (dark theme)
│   ├── globals.css         # Tailwind styles
│   ├── api/
│   │   ├── login/route.ts  # POST /api/login (Casdoor proxy)
│   │   ├── signup/route.ts # POST /api/signup
│   │   └── forgot/route.ts # POST /api/forgot
├── components/
│   └── lux-wordmark.tsx    # LUX SVG wordmark component
├── lib/
│   └── oauth.ts            # OAuth2 helpers (authorize URL, token exchange)
├── Dockerfile              # Multi-stage Node 22 Alpine build
├── next.config.ts
├── tailwind.config.ts
└── package.json
```

## Auth Flows

### Direct Password Login
1. User submits email + password on `/`
2. `POST /api/login` proxies to IAM: `${IAM_URL}/api/login`
3. Payload: `{ application: "app-lux", organization: "lux", username, password, type: "login" }`
4. On success, redirects to `NEXT_PUBLIC_REDIRECT_URL` (default: `cloud.lux.network`)

### OAuth2 Authorization Code
1. User clicks "Sign in with Lux ID"
2. Redirects to `lux.id/oauth/authorize` with `response_type=code`
3. After IAM login, callback to `/callback` with authorization code
4. `/callback` exchanges code for token via `lux.id/oauth/token`
5. Stores `access_token` + `refresh_token` in sessionStorage
6. Redirects to target URL

## Environment Variables

```bash
# IAM backend (server-side)
IAM_URL=https://iam.hanzo.ai          # Casdoor API endpoint
IAM_APP=app-lux                        # Casdoor application name

# OAuth (client-side)
NEXT_PUBLIC_IAM_CLIENT_ID=lux-cloud-client-id
NEXT_PUBLIC_IAM_AUTHORIZE_URL=https://lux.id/oauth/authorize
NEXT_PUBLIC_IAM_TOKEN_URL=https://lux.id/oauth/token
NEXT_PUBLIC_REDIRECT_URL=https://cloud.lux.network
NEXT_PUBLIC_IAM_ORG=lux
```

## Commands

```bash
pnpm install
pnpm dev              # Next.js dev server
pnpm build            # Production build
pnpm start            # Start production server
pnpm typecheck        # TypeScript check
pnpm lint             # ESLint
```

## CI/CD

`.github/workflows/ci.yml`:
- Triggers on push/PR to `main`
- Runs `pnpm typecheck` + `pnpm build`
- On main push: builds and pushes Docker image to GHCR
- Tags: `ghcr.io/luxfi/login:latest` + `ghcr.io/luxfi/login:<sha>`

## Design

- **Theme**: Monochrome dark (black background, white text)
- **Branding**: Production LUX wordmark SVG (126x34)
- **UI**: Tailwind CSS, minimal, centered card layout
- **Pages**: Login, Sign up, Forgot password, OAuth callback

## Related Skills

- `lux/lux-brand.md` -- Brand assets and design system

---

**Last Updated**: 2026-03-13
**Category**: Lux Ecosystem
