# Lux CEX - Centralized Exchange Engine

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-regulated.md`, `lux/lux-broker.md`, `lux/lux-compliance.md`, `lux/lux-dex.md`

## Overview

Lux CEX (`github.com/luxfi/cex`) is a regulated centralized exchange engine built on top of the broker's provider registry. It provides a CLOB matching engine, multi-protocol gateway (REST/FIX/WS/ZAP), pre/post-trade compliance hooks, market surveillance, and FINRA regulatory reporting. The CEX imports `luxfi/broker` for provider liquidity and `luxfi/dex` for high-performance matching.

## Quick Reference

| Item | Value |
|------|-------|
| Module | `github.com/luxfi/cex` |
| Go | 1.26.1 |
| Binary | `cexd` |
| Dependencies | `luxfi/broker` v0.2.1, `luxfi/dex` v1.2.1 |
| API Port | `:8080` (default) |
| Auth | JWT (Hanzo IAM OIDC via `CEX_JWT_SECRET`) |
| Framework | go-chi/chi/v5, gorilla/websocket |
| Database | PostgreSQL (pgx/v5) |
| License | Proprietary |

## Architecture

```
cexd
├── cmd/cexd/             Main entry point
├── pkg/
│   ├── engine/           CLOB matching engine + order book
│   ├── markets/          Market registration from broker providers
│   ├── compliance/       Pre/post-trade compliance hooks
│   ├── surveillance/     Market abuse detection (wash, spoof, layer)
│   ├── reporting/        FINRA trade reports + ATS-N filings
│   ├── gateway/          Multi-protocol API gateway
│   ├── protocol/         Protocol handlers (FIX, JSON-RPC, WS, ZAP)
│   ├── execution/        Order execution routing
│   ├── custody/          Custody integration
│   ├── store/            Persistent storage (PostgreSQL)
│   └── types/            Shared types (orders, trades, markets, asset classes)
├── proto/                gRPC proto definitions
├── web/                  Admin dashboard frontend
├── k8s/                  Kubernetes manifests
└── deploy/               Deployment configs
```

## Matching Engine (pkg/engine)

CLOB with price-time priority:

```go
import "github.com/luxfi/cex/pkg/engine"

eng := engine.New()

// Register a market
eng.RegisterMarket(&types.Market{
    Symbol:       "BTC-USD",
    AssetClass:   types.AssetClassCrypto,
    BaseCurrency: "BTC",
    QuoteCurrency: "USD",
    Status:       "active",
    TickSize:     "0.01",
    LotSize:      "0.00001",
    MakerFee:     "0.001",
    TakerFee:     "0.002",
    Tradable:     true,
})

// Submit order
order, err := eng.PlaceOrder(ctx, &types.PlaceOrderRequest{
    AccountID: "acct_123",
    Symbol:    "BTC-USD",
    Side:      types.SideBuy,
    Type:      types.OrderTypeLimit,
    Price:     "42500.00",
    Qty:       "1.5",
    TIF:       types.TIFGTC,
})

// Get order book depth
book := eng.GetOrderBook("BTC-USD", 10) // top 10 levels

// Get market info
market, exists := eng.GetMarket("BTC-USD")
```

### Order Types

```go
const (
    OrderTypeMarket    OrderType = "market"
    OrderTypeLimit     OrderType = "limit"
    OrderTypeStop      OrderType = "stop"
    OrderTypeStopLimit OrderType = "stop_limit"
)
```

### Time-in-Force

```go
const (
    TIFDay TimeInForce = "day"
    TIFGTC TimeInForce = "gtc"  // Good Till Cancelled
    TIFIOC TimeInForce = "ioc"  // Immediate Or Cancel
    TIFFOK TimeInForce = "fok"  // Fill Or Kill
)
```

### Order Status Lifecycle

```
new -> pending_approval -> open -> partial_fill -> filled
                       -> rejected              -> cancelled
                       -> suspended (by surveillance)
                                               -> expired
```

```go
const (
    OrderStatusNew             OrderStatus = "new"
    OrderStatusPendingApproval OrderStatus = "pending_approval"
    OrderStatusOpen            OrderStatus = "open"
    OrderStatusPartialFill     OrderStatus = "partial_fill"
    OrderStatusFilled          OrderStatus = "filled"
    OrderStatusCancelled       OrderStatus = "cancelled"
    OrderStatusRejected        OrderStatus = "rejected"
    OrderStatusExpired         OrderStatus = "expired"
    OrderStatusSuspended       OrderStatus = "suspended"
)
```

## Market Registration (pkg/markets)

### RegisterFromProviders

Auto-registers CEX markets from broker provider assets:

```go
import (
    "github.com/luxfi/broker/pkg/provider"
    "github.com/luxfi/broker/pkg/provider/envconfig"
    "github.com/luxfi/cex/pkg/engine"
    "github.com/luxfi/cex/pkg/markets"
)

registry := provider.NewRegistry()
envconfig.RegisterFromEnv(registry)

eng := engine.New()
n := markets.RegisterFromProviders(ctx, registry, eng)
// Queries each provider's ListAssets(), normalizes symbols, registers markets
// Crypto symbols: "/" -> "-" (e.g. "BTC/USD" -> "BTC-USD")
// Asset classes normalized via types.MapAssetClass()
```

### types.MapAssetClass

Maps provider-specific asset class names to canonical CEX types:

```go
import "github.com/luxfi/cex/pkg/types"

class := types.MapAssetClass("us_equity") // -> AssetClassUSEquity
```

### Asset Classes

```go
const (
    // Equities
    AssetClassUSEquity    AssetClass = "us_equity"
    AssetClassIntlEquity  AssetClass = "intl_equity"

    // Digital
    AssetClassCrypto AssetClass = "crypto"

    // FX
    AssetClassForex AssetClass = "forex"

    // Derivatives
    AssetClassOptions AssetClass = "options"
    AssetClassFutures AssetClass = "futures"

    // Fixed income
    AssetClassFixedIncome   AssetClass = "fixed_income"
    AssetClassMunicipal     AssetClass = "municipal"
    AssetClassStructured    AssetClass = "structured"
    AssetClassCorporateDebt AssetClass = "corporate_debt"
    AssetClassConsumerDebt  AssetClass = "consumer_debt"

    // Real assets
    AssetClassRealEstate     AssetClass = "real_estate"
    AssetClassCommodities    AssetClass = "commodities"
    AssetClassPreciousMetals AssetClass = "precious_metals"

    // Tokenized
    AssetClassTokenizedEquity AssetClass = "tokenized_equity"
    AssetClassTokenizedDebt   AssetClass = "tokenized_debt"
    AssetClassTokenizedRE     AssetClass = "tokenized_re"
)
```

## Pre/Post-Trade Compliance (pkg/compliance)

KYC-level gating and jurisdiction-aware trade validation:

```go
import "github.com/luxfi/cex/pkg/compliance"

comp := compliance.NewService()

// Register account with KYC level + jurisdiction
comp.RegisterAccount("acct_123", compliance.KYCStandard, compliance.JurisdictionUS)

// Pre-trade check
decision, err := comp.PreTradeCheck(ctx, &types.PlaceOrderRequest{
    AccountID: "acct_123",
    Symbol:    "AAPL",
    Side:      types.SideBuy,
    Qty:       "100",
    Price:     "150.00",
})
// decision: "approve" | "review" | "reject"

// Post-trade check (after fill)
comp.PostTradeCheck(ctx, trade)
```

### KYC Levels

```go
const (
    KYCNone     KYCLevel = 0 // no trading allowed
    KYCBasic    KYCLevel = 1 // email + phone verified
    KYCStandard KYCLevel = 2 // ID verified
    KYCEnhanced KYCLevel = 3 // accredited investor
)
```

### Jurisdiction Codes

The compliance service supports 25+ jurisdictions:

| Region | Codes | Regulators |
|--------|-------|-----------|
| Americas | US, CA, BR, MX, KY, BM, BS | SEC/FINRA, CSA, CVM, CNBV, CIMA, BMA, SCB |
| Europe | UK, EU, CH, IM, GI, LI | FCA, ESMA/MiFID II/MiCA, FINMA, IOM FSA, GFSC, FMA |
| Asia-Pacific | SG, HK, JP, AU, KR, IN | MAS, SFC, FSA/JFSA, ASIC, FSC, SEBI |
| MENA | AE, SA, BH, QA, KW | DFSA/VARA/SCA, CMA, CBB, QFC, CMA |

## Surveillance (pkg/surveillance)

Detects market abuse patterns in real time:

```go
import "github.com/luxfi/cex/pkg/surveillance"

surv := surveillance.NewService(surveillance.DefaultKYTConfig())

// Process trade through surveillance
alerts := surv.CheckTrade(ctx, trade)
for _, alert := range alerts {
    // alert.Type, alert.Severity, alert.AccountID, alert.Details
}
```

### Alert Types

| Alert | Type | Description |
|-------|------|-------------|
| Wash trading | `wash_trading` | Same account on both sides of a trade |
| Spoofing | `spoofing` | Placing and quickly cancelling large orders |
| Layering | `layering` | Multiple orders at different prices to create false depth |
| Front running | `front_running` | Trading ahead of known large orders |
| Insider trading | `insider_trading` | Unusual activity before material events |
| Market manipulation | `market_manipulation` | Coordinated price manipulation |
| Structuring | `structuring` | Multiple transactions just under CTR threshold |
| Velocity | `velocity` | Unusual trade frequency |
| Large trade | `large_trade` | Trade above notional threshold |
| Price spike | `price_spike` | Abnormal price acceleration (parabolic/exponential) |

### KYT (Know Your Transaction) Config

```go
cfg := surveillance.DefaultKYTConfig()
// CTRThreshold:        $10,000
// LargeTradeThreshold: configurable
// VelocityWindow:      1 hour
// VelocityMaxTrades:   50
// StructuringWindow:   24 hours
// StructuringMinTxns:  3
// PriceSpikeWindow:    5 minutes
// PriceSpikeMaxPct:    10%
// PriceSpikeAccelPct:  5% (second derivative threshold)
```

## Regulatory Reporting (pkg/reporting)

### FINRA Trade Reports

10-second reporting obligation for ATS trades:

```go
import "github.com/luxfi/cex/pkg/reporting"

rep := reporting.NewService(eng)

// Report a trade to FINRA TRF/ADF/ORF
report, err := rep.ReportTrade(ctx, &reporting.TradeReport{
    TradeID:       trade.ID,
    Symbol:        trade.Symbol,
    Side:          trade.Side,
    Qty:           trade.Qty,
    Price:         trade.Price,
    ExecutionTime: trade.Timestamp,
    Capacity:      "agency", // principal, agency, riskless_principal
    ContraParty:   "NSDQ",
})
// report.Status: "pending" -> "accepted" | "rejected"
```

### FINRA Report Fields

```go
type FINRAReport struct {
    ID            string    // internal ID
    TradeID       string    // trade reference
    Symbol        string
    Side          string
    Qty           string
    Price         string
    Venue         string    // execution venue
    ExecutionTime time.Time
    ReportTime    time.Time
    ReportType    string    // "last_sale" | "ats"
    MediatelyType string    // "as_of" | "late" | "on_time"
    Capacity      string    // "principal" | "agency" | "riskless_principal"
    ContraParty   string
    SpecialCond   string    // "sold_short" | "avg_price" | etc.
    Status        string    // "pending" | "accepted" | "rejected"
    AckID         string    // FINRA acknowledgment reference
}
```

### Quarterly ATS-N Reports

```go
report, err := rep.GenerateATSReport("2026-Q1", "My ATS", "CRD12345")
// report.TotalVolume, report.TotalTrades, report.UniqueSymbols
// report.ByAssetClass: {"crypto": {Volume: 1000000, Trades: 5000, Symbols: 50}}
```

## Multi-Protocol Gateway (pkg/gateway)

The gateway serves REST, FIX, WebSocket, and ZAP protocols:

```go
import (
    "github.com/luxfi/cex/pkg/gateway"
    "github.com/luxfi/cex/pkg/compliance"
    "github.com/luxfi/cex/pkg/reporting"
    "github.com/luxfi/cex/pkg/surveillance"
)

gw := gateway.New(eng, comp, rep, surv, ":8080")
log.Fatal(gw.ListenAndServe())
```

### REST API Routes

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/v1/markets` | JWT | List all markets |
| GET | `/v1/markets/{symbol}` | JWT | Market details |
| GET | `/v1/markets/{symbol}/book` | JWT | Order book depth |
| POST | `/v1/accounts/{id}/register` | JWT | Register account |
| POST | `/v1/orders` | JWT | Place order |
| DELETE | `/v1/orders/{id}` | JWT | Cancel order |
| GET | `/v1/orders` | JWT | List orders |
| GET | `/v1/trades` | JWT | Trade history |
| GET | `/v1/surveillance/alerts` | JWT | Surveillance alerts |
| GET | `/v1/reports` | JWT | Regulatory reports |
| GET | `/healthz` | none | Health check |

### JWT Auth

When `CEX_JWT_SECRET` is set, all `/v1/*` routes require JWT authentication. Issuer defaults to `https://hanzo.id` (configurable via `CEX_JWT_ISSUER`).

### Protocol Handlers (pkg/protocol)

| Protocol | File | Port | Use Case |
|----------|------|------|----------|
| REST/JSON | `gateway.go` | 8080 | Standard API clients |
| FIX | `fix.go` | 9878 | Institutional connectivity |
| WebSocket | `websocket.go` | 8080 (upgrade) | Real-time streaming |
| JSON-RPC 2.0 | `jsonrpc.go` | 8080 | Programmatic access |
| ZAP | `zap.go` | 9879 | Ultra-low-latency HFT |

## Assembly (Full ATS)

```go
package main

import (
    "context"
    "log"
    "os"

    "github.com/luxfi/broker/pkg/provider"
    "github.com/luxfi/broker/pkg/provider/envconfig"
    "github.com/luxfi/cex/pkg/engine"
    "github.com/luxfi/cex/pkg/markets"
    "github.com/luxfi/cex/pkg/compliance"
    "github.com/luxfi/cex/pkg/reporting"
    "github.com/luxfi/cex/pkg/surveillance"
    "github.com/luxfi/cex/pkg/gateway"
    amlpkg "github.com/luxfi/compliance/pkg/aml"
)

func main() {
    ctx := context.Background()

    // 1. Provider registry (liquidity sources)
    registry := provider.NewRegistry()
    n := envconfig.RegisterFromEnv(registry)
    log.Printf("providers: %d", n)

    // 2. Matching engine
    eng := engine.New()

    // 3. Auto-register markets from providers
    m := markets.RegisterFromProviders(ctx, registry, eng)
    log.Printf("markets: %d", m)

    // 4. Compliance, surveillance, reporting
    comp := compliance.NewService()
    surv := surveillance.NewService(surveillance.DefaultKYTConfig())
    rep := reporting.NewService(eng)

    // 5. AML baseline
    amlSvc := amlpkg.NewMonitoringService()
    amlpkg.InstallDefaultRules(amlSvc)

    // 6. Start gateway
    addr := os.Getenv("CEX_LISTEN_ADDR")
    if addr == "" {
        addr = ":8080"
    }
    gw := gateway.New(eng, comp, rep, surv, addr)
    log.Fatal(gw.ListenAndServe())
}
```

## Environment Variables

```bash
# Provider config (via broker envconfig)
ALPACA_API_KEY=xxx
BINANCE_API_KEY=xxx
# ... (see lux-broker.md for all 16)

# CEX-specific
CEX_LISTEN_ADDR=:8080
CEX_JWT_SECRET=xxx           # enables JWT auth
CEX_JWT_ISSUER=https://hanzo.id

# Database
CEX_DATABASE_URL=postgres://user:pass@host:5432/cex

# Surveillance
CEX_KYT_CTR_THRESHOLD=10000
CEX_KYT_VELOCITY_WINDOW=1h
CEX_KYT_VELOCITY_MAX=50
```

## Related Skills

- `lux/lux-regulated.md` -- Assembling ATS/BD/TA products
- `lux/lux-broker.md` -- Provider registry that feeds markets into the CEX
- `lux/lux-compliance.md` -- KYC/AML module (separate from CEX compliance hooks)
- `lux/lux-dex.md` -- High-perf DEX engine (434M orders/sec) — different from CEX

---

**Last Updated**: 2026-03-22
**Category**: Lux Ecosystem
**Related**: cex, exchange, clob, matching-engine, surveillance, finra, reporting, fix, websocket, zap
**Prerequisites**: Go 1.26.1, understanding of exchange mechanics and US securities regulation
