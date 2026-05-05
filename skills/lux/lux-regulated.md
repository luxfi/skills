# Lux Regulated Products - Building ATS, BD, and TA from OSS

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-broker.md`, `lux/lux-cex.md`, `lux/lux-compliance.md`, `lux/lux-dex.md`, `lux/lux-mpc.md`

## Overview

Lux provides the building blocks to assemble three classes of regulated financial products from open-source Go modules. Each product composes the same core packages differently depending on its regulatory category. This skill describes the product types, their building blocks, and how to wire them together.

## Product Types

| Product | Full Name | Regulator | Module Composition |
|---------|-----------|-----------|-------------------|
| **ATS** | Alternative Trading System | SEC / FINRA (Reg ATS, Reg SCI) | broker + cex + compliance + mpc |
| **BD** | Broker-Dealer | SEC / FINRA (Rule 15c3-1) | broker + compliance + bank |
| **TA** | Transfer Agent | SEC (Rule 17Ad) | broker + compliance + mpc + bank |

### ATS (Alternative Trading System)

Operates a matching engine for securities. Requires FINRA membership, Form ATS-N filing, and Reg SCI compliance for large venues.

**Required modules**: `luxfi/broker` (provider registry, SOR), `luxfi/cex` (CLOB matching engine, surveillance, FINRA reporting), `luxfi/compliance` (KYC/AML, entity rules), `luxfi/mpc` (custody).

**Key obligations**: Trade reporting to FINRA TRF/ADF/ORF within 10 seconds, quarterly Form ATS-N, surveillance for wash trading / spoofing / layering.

### BD (Broker-Dealer)

Routes orders to execution venues on behalf of customers. Requires SEC registration, FINRA membership, net capital requirements (Rule 15c3-1).

**Required modules**: `luxfi/broker` (provider registry, SOR, settlement), `luxfi/compliance` (KYC/AML/travel rule), bank integration (Finix/CurrencyCloud for ACH/wire).

**Key obligations**: Best execution (Reg NMS Rule 606), customer protection (Rule 15c3-3), net capital maintenance, SAR filing.

### TA (Transfer Agent)

Maintains shareholder records and processes transfers. Requires SEC registration under Rule 17Ad.

**Required modules**: `luxfi/broker` (provider registry, account resolver), `luxfi/compliance` (KYC/AML), `luxfi/mpc` (token custody), bank integration.

**Key obligations**: Transfer turnaround times (Rule 17Ad-2), recordkeeping (Rule 17Ad-6/7), lost securities (Rule 17Ad-17).

## Building Blocks

| Block | Module | Package | Purpose |
|-------|--------|---------|---------|
| **Broker** | `github.com/luxfi/broker` | `pkg/provider` | Unified provider interface, 16 providers, registry |
| | | `pkg/provider/envconfig` | `RegisterFromEnv()` — reads env vars, registers all configured providers |
| | | `pkg/router` | Smart order routing, split plans, TWAP |
| | | `pkg/settlement` | Instant-buy with pool capital, ACH lifecycle, margin monitoring |
| | | `pkg/accounts` | User-to-provider account resolver, JWT import, auto-discovery |
| | | `pkg/api` | REST API server, frontend exchange API |
| | | `pkg/marketdata` | Consolidated book, BBO, ticker aggregation |
| | | `pkg/risk` | Pre-trade risk checks, per-account limits |
| | | `pkg/funding` | Deposit/withdraw via payment processors |
| **CEX** | `github.com/luxfi/cex` | `pkg/engine` | CLOB matching engine, order book, matcher |
| | | `pkg/markets` | `RegisterFromProviders()` — auto-register markets from broker providers |
| | | `pkg/compliance` | Pre/post-trade compliance hooks, KYC level gates |
| | | `pkg/surveillance` | Wash trading, spoofing, layering, KYT monitoring |
| | | `pkg/reporting` | FINRA trade reports, quarterly ATS-N filings |
| | | `pkg/gateway` | Multi-protocol API gateway (REST + FIX + WS + ZAP) |
| | | `pkg/protocol` | Protocol handlers: FIX, JSON-RPC, WebSocket, ZAP |
| **Compliance** | `github.com/luxfi/compliance` | `pkg/kyc` | KYC orchestration across IDV providers |
| | | `pkg/idv` | Identity verification (Jumio, Onfido, Plaid) |
| | | `pkg/aml` | Sanctions screening (OFAC/EU/UK/PEP), transaction monitoring |
| | | `pkg/jube` | Jube AML/fraud engine integration |
| | | `pkg/entity` | Regulated entity types (ATS, BD, TA, MSB) with license/reporting requirements |
| | | `pkg/regulatory` | Multi-jurisdiction rules (USA/UK/IOM) |
| | | `pkg/payments` | Travel rule, CTR, payment compliance |
| **MPC** | `github.com/luxfi/mpc` | `pkg/mpc` | Threshold signing (CGGMP21 + FROST) |
| | | `pkg/settlement` | Settlement attestation |
| **DEX** | `github.com/luxfi/dex` | `pkg/lx` | High-perf CLOB (434M orders/sec GPU) |
| | | `pkg/gateway` | Multi-provider liquidity aggregation |
| **Bank** | `luxfi/broker` providers | `provider/finix` | ACH, wire, card payments |
| | | `provider/currencycloud` | FX, cross-border, 35+ currencies |
| | | `provider/circle` | USDC/EURC mint/burn |

## Assembly Patterns

### ATS Assembly

```go
package main

import (
    "context"
    "log"

    "github.com/luxfi/broker/pkg/provider"
    "github.com/luxfi/broker/pkg/provider/envconfig"
    "github.com/luxfi/cex/pkg/engine"
    "github.com/luxfi/cex/pkg/markets"
    "github.com/luxfi/cex/pkg/compliance"
    "github.com/luxfi/cex/pkg/reporting"
    "github.com/luxfi/cex/pkg/surveillance"
    "github.com/luxfi/cex/pkg/gateway"
    compliancemod "github.com/luxfi/compliance/pkg/aml"
)

func main() {
    ctx := context.Background()

    // 1. Register liquidity providers from environment
    registry := provider.NewRegistry()
    n := envconfig.RegisterFromEnv(registry)
    log.Printf("registered %d providers", n)

    // 2. Build matching engine
    eng := engine.New()

    // 3. Auto-register markets from provider assets
    m := markets.RegisterFromProviders(ctx, registry, eng)
    log.Printf("registered %d markets", m)

    // 4. Compliance (pre/post-trade)
    comp := compliance.NewService()

    // 5. Surveillance (wash trading, spoofing, layering)
    surv := surveillance.NewService(surveillance.DefaultKYTConfig())

    // 6. FINRA reporting
    rep := reporting.NewService(eng)

    // 7. Install AML monitoring rules
    amlSvc := compliancemod.NewMonitoringService()
    compliancemod.InstallDefaultRules(amlSvc)

    // 8. Start gateway (REST + FIX + WS)
    gw := gateway.New(eng, comp, rep, surv, ":8080")
    log.Fatal(gw.ListenAndServe())
}
```

### BD Assembly

```go
package main

import (
    "log"

    "github.com/luxfi/broker/pkg/provider"
    "github.com/luxfi/broker/pkg/provider/envconfig"
    "github.com/luxfi/broker/pkg/api"
    compliancemod "github.com/luxfi/compliance/pkg/aml"
)

func main() {
    // 1. Register providers
    registry := provider.NewRegistry()
    n := envconfig.RegisterFromEnv(registry)
    log.Printf("registered %d providers", n)

    // 2. Install AML rules
    amlSvc := compliancemod.NewMonitoringService()
    compliancemod.InstallDefaultRules(amlSvc)

    // 3. Start broker API (includes SOR, settlement, risk, market data)
    srv := api.NewServer(registry, ":8080")
    log.Fatal(srv.ListenAndServe())
}
```

### TA Assembly

```go
package main

import (
    "context"
    "log"

    "github.com/luxfi/broker/pkg/provider"
    "github.com/luxfi/broker/pkg/provider/envconfig"
    "github.com/luxfi/broker/pkg/accounts"
    "github.com/luxfi/broker/pkg/api"
    compliancemod "github.com/luxfi/compliance/pkg/aml"
    "github.com/luxfi/compliance/pkg/kyc"
    "github.com/luxfi/compliance/pkg/idv"
)

func main() {
    ctx := context.Background()

    // 1. Register providers (custody + payments)
    registry := provider.NewRegistry()
    envconfig.RegisterFromEnv(registry)

    // 2. KYC service with IDV providers
    kycSvc := kyc.NewService()
    kycSvc.RegisterProvider("jumio", idv.NewJumioProvider(idv.JumioConfig{
        APIToken:  os.Getenv("JUMIO_API_TOKEN"),
        APISecret: os.Getenv("JUMIO_API_SECRET"),
    }))

    // 3. AML monitoring
    amlSvc := compliancemod.NewMonitoringService()
    compliancemod.InstallDefaultRules(amlSvc)

    // 4. Account resolver (user -> provider account mapping)
    resolver := accounts.NewResolver()

    // 5. Start API
    srv := api.NewServer(registry, ":8080")
    log.Fatal(srv.ListenAndServe())
}
```

## Key Functions

### `envconfig.RegisterFromEnv(registry)`

Reads all 16 provider environment variables and registers configured providers. This is the single entry point for provider configuration in any product type.

```go
// Environment variables (set any combination):
// ALPACA_API_KEY, IBKR_ACCESS_TOKEN, BITGO_ACCESS_TOKEN,
// FALCON_API_KEY, FINIX_USERNAME, SFOX_API_KEY,
// COINBASE_API_KEY, BINANCE_API_KEY, KRAKEN_API_KEY,
// GEMINI_API_KEY, FIREBLOCKS_API_KEY, CIRCLE_API_KEY,
// TRADIER_ACCESS_TOKEN, POLYGON_API_KEY,
// CURRENCYCLOUD_LOGIN_ID, LMAX_API_KEY
```

### `markets.RegisterFromProviders(ctx, registry, eng)`

Queries every registered provider for tradable assets and registers them as CEX markets. Uses `types.MapAssetClass()` to normalize asset classes across providers.

### `aml.InstallDefaultRules(svc)`

Installs the 3 FinCEN-mandated monitoring rules:
- CTR threshold: Flag transactions over $10,000 (31 CFR 1010.311)
- Structuring: Detect $9,000-$9,999 patterns (31 USC 5324)
- Velocity: Block if 24h volume exceeds $50,000 without enhanced KYC

### Jube Integration

```go
import "github.com/luxfi/compliance/pkg/jube"

client, err := jube.NewClient(jube.Config{
    BaseURL:  "http://jube.svc.cluster.local:5001",
    ModelID:  os.Getenv("JUBE_MODEL_ID"),
    FailOpen: false, // reject on Jube unavailability
    Timeout:  5 * time.Second,
})
// Pre-trade: score order before matching
result, err := client.ScoreTransaction(ctx, &jube.TransactionRequest{...})
if result.ResponseElevation > 0 {
    // reject or flag for review
}
```

## Provider Matrix

| Provider | Asset Classes | Use Case |
|----------|-------------|----------|
| alpaca | us_equity, crypto | Commission-free equities + crypto |
| ibkr | us_equity, option, futures, forex, bond, crypto | Global multi-asset |
| sfox | crypto | Smart routing, dark pool, 150+ pairs |
| coinbase | crypto | Retail crypto, staking, 250+ pairs |
| binance | crypto | High volume, 600+ pairs |
| kraken | crypto | Staking, margin, futures |
| gemini | crypto | Regulated custody, 100+ pairs |
| bitgo | crypto | Institutional custody, multisig |
| falcon | crypto | Institutional OTC, RFQ |
| fireblocks | crypto | MPC custody, DeFi, 500+ assets |
| circle | stablecoin | USDC/EURC mint/burn |
| finix | payments | ACH, wire, card, Apple/Google Pay |
| tradier | us_equity, option | Commission-free equities + options |
| polygon | us_equity, crypto, forex, option | Market data only |
| currencycloud | forex | FX trading, 35+ currencies |
| lmax | forex, crypto, commodity | Institutional CLOB, FCA regulated |

## Regulatory Requirements by Product

### ATS Requirements

| Requirement | Reference | Implementation |
|-------------|-----------|---------------|
| Form ATS-N filing | Reg ATS Rule 304 | `reporting.Service.GenerateATSReport()` |
| Trade reporting (10s) | FINRA Rules 6380A/B | `reporting.Service.ReportTrade()` |
| Fair access | Reg ATS Rule 301(b)(5) | Compliance policy |
| Surveillance | FINRA Rule 3110 | `surveillance.Service` |
| Net capital | Rule 15c3-1 | Entity capital tracking |
| Books & records | Rule 17a-4 | Audit log |

### BD Requirements

| Requirement | Reference | Implementation |
|-------------|-----------|---------------|
| Best execution | Reg NMS / Rule 5310 | `router.Router.FindBestProvider()` |
| Customer protection | Rule 15c3-3 | Settlement service |
| Net capital | Rule 15c3-1 | Risk engine |
| SAR filing | 31 CFR 1010.320 | `aml.MonitoringService` |
| KYC/CIP | 31 CFR 1010.220 | `kyc.Service` |
| Order routing disclosure | Rule 606 | Audit log |

### TA Requirements

| Requirement | Reference | Implementation |
|-------------|-----------|---------------|
| Transfer turnaround | Rule 17Ad-2 | SLA monitoring |
| Recordkeeping | Rule 17Ad-6/7 | Persistent store |
| Lost securities | Rule 17Ad-17 | Search/inquiry |
| Annual reporting | Rule 17Ad-13 | Reporting service |
| KYC | 31 CFR 1010.220 | `kyc.Service` |

## Environment Variables

All products share the same env var pattern. Set only what you need:

```bash
# Execution providers (set any combination)
ALPACA_API_KEY=xxx       ALPACA_API_SECRET=xxx
IBKR_ACCESS_TOKEN=xxx    IBKR_ACCOUNT_ID=xxx
SFOX_API_KEY=xxx
COINBASE_API_KEY=xxx     COINBASE_API_SECRET=xxx
BINANCE_API_KEY=xxx      BINANCE_API_SECRET=xxx
KRAKEN_API_KEY=xxx       KRAKEN_API_SECRET=xxx
GEMINI_API_KEY=xxx       GEMINI_API_SECRET=xxx

# Custody
FIREBLOCKS_API_KEY=xxx   FIREBLOCKS_PRIVATE_KEY=xxx
BITGO_ACCESS_TOKEN=xxx

# Payments
FINIX_USERNAME=xxx       FINIX_PASSWORD=xxx
CIRCLE_API_KEY=xxx
CURRENCYCLOUD_LOGIN_ID=xxx CURRENCYCLOUD_API_KEY=xxx

# Market data
POLYGON_API_KEY=xxx
LMAX_API_KEY=xxx         LMAX_USERNAME=xxx       LMAX_PASSWORD=xxx

# KYC/IDV
JUMIO_API_TOKEN=xxx      JUMIO_API_SECRET=xxx
ONFIDO_API_TOKEN=xxx
PLAID_CLIENT_ID=xxx      PLAID_SECRET=xxx

# AML
JUBE_BASE_URL=http://jube.svc.cluster.local:5001
JUBE_MODEL_ID=xxx

# Auth
ADMIN_SECRET=xxx         BROKER_API_KEY=xxx
CEX_JWT_SECRET=xxx       CEX_JWT_ISSUER=https://hanzo.id
```

## Related Skills

- `lux/lux-broker.md` -- Provider interface, SOR, settlement, API details
- `lux/lux-cex.md` -- CLOB engine, surveillance, reporting, gateway protocols
- `lux/lux-compliance.md` -- KYC/AML/IDV, regulatory frameworks, entity types
- `lux/lux-dex.md` -- High-performance DEX engine (434M orders/sec)
- `lux/lux-mpc.md` -- Threshold signing for custody
- Docs: https://docs.lux.network/regulated/

---

**Category**: Lux Ecosystem
**Related**: ats, broker-dealer, transfer-agent, regulated, finra, sec, compliance, kyc, aml
**Prerequisites**: Go 1.26.1, understanding of US securities regulation
