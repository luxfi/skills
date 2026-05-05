# Lux Broker - Multi-Provider Trading Router

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-regulated.md`, `lux/lux-cex.md`, `lux/lux-compliance.md`, `lux/lux-dex.md`

## Overview

Lux Broker (`brokerd`) is a Go service that provides a unified trading API across 16 execution providers. It implements the `provider.Provider` interface pattern: one base interface for all providers, optional capability interfaces detected at runtime via type assertions. The broker includes smart order routing (SOR), instant-buy settlement, consolidated market data, pre-trade risk checks, and a provider-agnostic frontend exchange API.

Compliance (KYC/AML) has been extracted to the separate `luxfi/compliance` module. The broker is a pure trading router.

## Quick Reference

| Item | Value |
|------|-------|
| Module | `github.com/luxfi/broker` |
| Go | 1.26.1 |
| Binary | `brokerd` |
| API Port | `:8080` (default) |
| Framework | go-chi/chi/v5 |
| Auth | API key + JWT (Hanzo IAM OIDC) |
| License | Apache 2.0 |

## Architecture

```
brokerd
├── cmd/brokerd/          Main entry point
├── pkg/
│   ├── provider/         Provider interface + Registry
│   │   ├── envconfig/    RegisterFromEnv (imports all 16 providers)
│   │   ├── alpaca/       Alpaca Markets (equities + crypto)
│   │   ├── ibkr/         Interactive Brokers (global multi-asset)
│   │   ├── sfox/         SFOX (crypto smart routing)
│   │   ├── coinbase/     Coinbase (crypto)
│   │   ├── binance/      Binance (crypto)
│   │   ├── kraken/       Kraken (crypto)
│   │   ├── gemini/       Gemini (crypto)
│   │   ├── bitgo/        BitGo (custody + prime)
│   │   ├── falcon/       Falcon (institutional OTC)
│   │   ├── fireblocks/   Fireblocks (MPC custody)
│   │   ├── circle/       Circle (USDC/EURC)
│   │   ├── finix/        Finix (ACH/wire/card)
│   │   ├── tradier/      Tradier (equities + options)
│   │   ├── polygon/      Polygon.io (market data)
│   │   ├── currencycloud/ CurrencyCloud (FX)
│   │   └── lmax/         LMAX (institutional CLOB)
│   ├── router/           Smart order routing + split execution
│   ├── settlement/       Instant-buy, pool capital, ACH lifecycle
│   ├── accounts/         User-to-provider account resolver
│   ├── api/              REST API server + frontend exchange handlers
│   ├── marketdata/       Consolidated book, BBO, tickers
│   ├── risk/             Pre-trade risk engine
│   ├── funding/          Deposit/withdraw via payment processors
│   ├── auth/             API key middleware
│   ├── admin/            JWT admin auth (HMAC-SHA256)
│   ├── audit/            Immutable audit log
│   ├── ws/               WebSocket server (SSE)
│   └── types/            Shared types
```

## Provider Interface

The core `provider.Provider` interface defines the baseline contract every provider must implement:

```go
type Provider interface {
    Name() string

    // Accounts
    CreateAccount(ctx context.Context, req *types.CreateAccountRequest) (*types.Account, error)
    GetAccount(ctx context.Context, providerAccountID string) (*types.Account, error)
    ListAccounts(ctx context.Context) ([]*types.Account, error)

    // Portfolio
    GetPortfolio(ctx context.Context, providerAccountID string) (*types.Portfolio, error)

    // Orders
    CreateOrder(ctx context.Context, providerAccountID string, req *types.CreateOrderRequest) (*types.Order, error)
    ListOrders(ctx context.Context, providerAccountID string) ([]*types.Order, error)
    GetOrder(ctx context.Context, providerAccountID, providerOrderID string) (*types.Order, error)
    CancelOrder(ctx context.Context, providerAccountID, providerOrderID string) error

    // Transfers
    CreateTransfer(ctx context.Context, providerAccountID string, req *types.CreateTransferRequest) (*types.Transfer, error)
    ListTransfers(ctx context.Context, providerAccountID string) ([]*types.Transfer, error)

    // Bank Relationships
    CreateBankRelationship(ctx context.Context, providerAccountID string, ownerName, accountType, accountNumber, routingNumber string) (*types.BankRelationship, error)
    ListBankRelationships(ctx context.Context, providerAccountID string) ([]*types.BankRelationship, error)

    // Assets
    ListAssets(ctx context.Context, class string) ([]*types.Asset, error)
    GetAsset(ctx context.Context, symbolOrID string) (*types.Asset, error)

    // Market Data
    GetSnapshot(ctx context.Context, symbol string) (*types.MarketSnapshot, error)
    GetSnapshots(ctx context.Context, symbols []string) (map[string]*types.MarketSnapshot, error)
    GetBars(ctx context.Context, symbol, timeframe, start, end string, limit int) ([]*types.Bar, error)
    GetLatestTrades(ctx context.Context, symbols []string) (map[string]*types.Trade, error)
    GetLatestQuotes(ctx context.Context, symbols []string) (map[string]*types.Quote, error)
    GetClock(ctx context.Context) (*types.MarketClock, error)
    GetCalendar(ctx context.Context, start, end string) ([]*types.MarketCalendarDay, error)
}
```

### Optional Capability Interfaces

Detected at runtime via type assertions (`if am, ok := p.(provider.AccountManager); ok { ... }`):

| Interface | Methods | Providers |
|-----------|---------|-----------|
| `TradingExtended` | ReplaceOrder, CancelAll, GetPosition, ClosePosition, CloseAllPositions, ListOrdersFiltered | alpaca |
| `AccountManager` | UpdateAccount, CloseAccount, GetAccountActivities | alpaca |
| `DocumentManager` | UploadDocument, ListDocuments, GetDocument, DownloadDocument | alpaca |
| `JournalManager` | CreateJournal, ListJournals, GetJournal, DeleteJournal, CreateBatchJournal, ReverseBatchJournal | alpaca |
| `TransferExtended` | CancelTransfer, DeleteACHRelationship, CreateRecipientBank, ListRecipientBanks, DeleteRecipientBank | alpaca |
| `CryptoDataProvider` | GetCryptoBars, GetCryptoQuotes, GetCryptoTrades, GetCryptoSnapshots | alpaca |
| `EventStreamer` | StreamTradeEvents, StreamAccountEvents, StreamTransferEvents, StreamJournalEvents | alpaca |
| `PortfolioAnalyzer` | GetPortfolioHistory | alpaca |
| `WatchlistManager` | CreateWatchlist, ListWatchlists, GetWatchlist, UpdateWatchlist, DeleteWatchlist, AddWatchlistAsset, RemoveWatchlistAsset | alpaca |

### Provider Registry

```go
registry := provider.NewRegistry()
registry.Register(alpaca.New(alpaca.Config{...}))
registry.Register(ibkr.New(ibkr.Config{...}))

// Or use envconfig for all 16 at once:
n := envconfig.RegisterFromEnv(registry)

p, err := registry.Get("alpaca")
names := registry.List() // []string{"alpaca", "ibkr", ...}
```

## The 16 Built-in Providers

| # | Provider | Module | Asset Classes | Key Features |
|---|----------|--------|-------------|-------------|
| 1 | alpaca | `provider/alpaca` | us_equity, crypto | Commission-free, margin, extended hours, ACH/wire, full capability set |
| 2 | ibkr | `provider/ibkr` | us_equity, option, futures, forex, bond, crypto | Global markets, algo orders, margin, short selling |
| 3 | sfox | `provider/sfox` | crypto | Smart routing, dark pool, 150+ pairs, OTC |
| 4 | coinbase | `provider/coinbase` | crypto | Staking, custody, 250+ pairs |
| 5 | binance | `provider/binance` | crypto | Margin, futures, 600+ pairs |
| 6 | kraken | `provider/kraken` | crypto | Staking, margin, futures, 200+ pairs |
| 7 | gemini | `provider/gemini` | crypto | Regulated custody, 100+ pairs |
| 8 | bitgo | `provider/bitgo` | crypto | Multisig custody, prime trading |
| 9 | falcon | `provider/falcon` | crypto | Institutional OTC, RFQ |
| 10 | fireblocks | `provider/fireblocks` | crypto | MPC custody, DeFi, 500+ assets |
| 11 | circle | `provider/circle` | stablecoin | USDC/EURC mint/burn, cross-chain |
| 12 | finix | `provider/finix` | payments | ACH, wire, card, Apple/Google Pay |
| 13 | tradier | `provider/tradier` | us_equity, option | Commission-free, streaming |
| 14 | polygon | `provider/polygon` | us_equity, crypto, forex, option | Market data only (bars, trades, quotes, snapshots) |
| 15 | currencycloud | `provider/currencycloud` | forex | FX trading, 35+ currencies, cross-border payments |
| 16 | lmax | `provider/lmax` | forex, crypto, commodity | Institutional CLOB, FCA regulated, sub-ms matching |

## envconfig Package

The `envconfig` package (`pkg/provider/envconfig/`) is the standard way to configure providers. It reads 16 environment variables and registers whichever are set:

```go
import (
    "github.com/luxfi/broker/pkg/provider"
    "github.com/luxfi/broker/pkg/provider/envconfig"
)

registry := provider.NewRegistry()
n := envconfig.RegisterFromEnv(registry) // returns count
```

**Design note**: `envconfig` is a sub-package of `provider` (not `provider` itself) to avoid import cycles. The provider sub-packages (`provider/alpaca`, etc.) import `provider` for compile-time interface assertions, so `provider` cannot import them back.

## Smart Order Routing (SOR)

The `router.Router` finds best execution across all providers:

```go
import "github.com/luxfi/broker/pkg/router"

sor := router.New(registry)

// Find best provider for a symbol
best, err := sor.FindBestProvider(ctx, "AAPL", "buy")
// best.Provider = "alpaca", best.NetPrice = 150.25, best.SpreadBps = 1.5

// Get all routes ranked by net execution price (price + fees)
routes, err := sor.GetAllRoutes(ctx, "BTC/USD", "buy")

// Split a large order across providers (minimize market impact)
plan, err := sor.BuildSplitPlan(ctx, "BTC/USD", "buy", "10.5")
// plan.Legs = [{Provider: "binance", Qty: "5.2"}, {Provider: "kraken", Qty: "5.3"}]
// plan.EstimatedVWAP, plan.EstimatedFees, plan.Savings (bps vs worst venue)

// Execute split plan in parallel
result, err := sor.ExecuteSplitPlan(ctx, plan, accountsByProvider)

// Route to best provider automatically
order, err := sor.SmartOrder(ctx, accountsByProvider, &types.CreateOrderRequest{
    Symbol: "AAPL", Qty: "100", Side: "buy", Type: "market",
})

// Aggregate all tradable assets across providers
assets, err := sor.AggregatedAssets(ctx)
```

### Fee-Aware Routing

Net-price routing factors in per-provider taker fees:

| Provider | Maker (bps) | Taker (bps) |
|----------|-------------|-------------|
| alpaca | 0 | 0 |
| ibkr | 0.5 | 0.5 |
| sfox | 15 | 25 |
| coinbase | 40 | 60 |
| binance | 10 | 10 |
| kraken | 16 | 26 |
| lmax | 2 | 3 |
| currencycloud | 3 | 5 |
| falcon | 5 | 10 |

## Settlement Service

Instant-buy backed by pool capital with ACH settlement lifecycle:

```go
import "github.com/luxfi/broker/pkg/settlement"

pool := settlement.NewPool(settlement.PoolConfig{
    MaxCapital:    1_000_000,
    MaxPerUser:    50_000,
    MaxPerTx:      25_000,
    MaxUtilization: 0.8,
})
svc := settlement.NewService(pool)

// KYC tier limits (USD)
// basic: $250, standard: $5,000, enhanced: $25,000, institutional: $250,000

result, err := svc.InstantBuy(ctx, &settlement.InstantBuyRequest{
    AccountID: "acct_123",
    Symbol:    "AAPL",
    Qty:       1000, // USD value
    Side:      "buy",
    KYCTier:   "standard",
})
// result.ReservationID, result.EstimatedSettlement (T+3)

// Track ACH lifecycle
svc.ProcessSettlement(result.ReservationID, settlement.SettlementEvent{
    Type: settlement.EventACHCleared, // -> releases pool credit
})

// Margin monitoring
alerts := svc.CheckMarginHealth(func(asset string) float64 {
    return currentPrice(asset)
})
```

## Account Resolver

Maps IAM user identities to provider account IDs:

```go
import "github.com/luxfi/broker/pkg/accounts"

resolver := accounts.NewResolver()
resolver.SetMapping("user_123", "org_456", "alpaca", "acct_789")

accountID, ok := resolver.ResolveAccount("user_123", "alpaca")
provName, acctID, ok := resolver.ResolveAnyAccount("user_123")

// Auto-discover from all providers
resolver.AutoDiscover(ctx, registry, "user_123", "org_456")

// Import from JWT claims (Hanzo IAM enrichment)
resolver.ImportFromJWT(tokenPayload)
```

## Frontend Exchange API

Provider-agnostic endpoints that resolve user identity from gateway headers (`X-Gateway-User-Id` / `X-Hanzo-User-Id`):

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/exchange/assets?type=crypto\|stocks\|bonds\|commodity\|forex` | Assets across all providers |
| GET | `/v1/exchange/crypto-prices` | Top crypto prices (BTC, ETH, SOL, ...) |
| GET | `/v1/exchange/charts/{symbol}?timeframe=1D&limit=100` | OHLCV bars |
| GET | `/v1/exchange/orders` | User's orders across all providers |
| POST | `/v1/exchange/orders` | Create order (auto-routes to provider) |
| GET | `/v1/exchange/positions` | User's positions across all providers |
| GET | `/v1/exchange/portfolio` | Aggregated portfolio snapshot |

Asset class mapping: `stocks` -> `us_equity`, `crypto` -> `crypto`, `bonds` -> `fixed_income`, `commodity` -> `commodities`, `forex` -> `forex`.

## Market Data

```go
import "github.com/luxfi/broker/pkg/marketdata"

feed := marketdata.NewFeed()

// Consolidated order book across providers
book := feed.GetBook("BTC/USD")
// book.Bids[0].Provider = "binance", .Price = 42500, .Size = 1.5

// Best bid/offer with provider attribution
ticker := feed.GetTicker("BTC/USD")
// ticker.BestBid, ticker.BestAsk, ticker.BestBidProvider, ticker.SpreadBps
```

### Market Data Routing

- Stock symbols (no `/`) route to `/v2/stocks/` APIs
- Crypto symbols (contain `/`) route to `/v1beta3/crypto/us/` APIs
- `isCryptoSymbol()` helper determines routing

## API Routes

### Core Trading API (`/v1`)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/v1/providers` | API key | List registered providers |
| GET | `/v1/providers/capabilities` | API key | Provider capabilities matrix |
| GET | `/v1/accounts` | API key | List accounts across providers |
| POST | `/v1/accounts` | API key | Create account |
| POST | `/v1/accounts/{provider}/{accountId}/orders` | API key | Place order |
| GET | `/v1/route/{symbol}` | API key | SOR routes |
| POST | `/v1/smart-order` | API key | Smart order (auto-route) |
| POST | `/v1/smart-order/split` | API key | Execute split plan |
| GET | `/v1/bbo/{symbol}` | API key | Best bid/offer |
| GET | `/v1/stream` | API key | SSE market data |
| GET | `/v1/risk/check` | API key | Pre-trade risk validation |
| GET | `/healthz` | none | Health check (returns provider list) |

## Adding a Custom Provider

1. Create `pkg/provider/myprovider/myprovider.go`
2. Implement `provider.Provider` interface
3. Add compile-time assertion: `var _ provider.Provider = (*Provider)(nil)`
4. Add env var block to `pkg/provider/envconfig/envconfig.go`
5. Optionally implement capability interfaces (`TradingExtended`, `AccountManager`, etc.)

```go
package myprovider

import (
    "context"
    "github.com/luxfi/broker/pkg/provider"
    "github.com/luxfi/broker/pkg/types"
)

var _ provider.Provider = (*Provider)(nil)

type Config struct {
    BaseURL string
    APIKey  string
}

type Provider struct {
    cfg Config
}

func New(cfg Config) *Provider {
    return &Provider{cfg: cfg}
}

func (p *Provider) Name() string { return "myprovider" }

// ... implement remaining Provider methods
```

## Related Skills

- `lux/lux-regulated.md` -- Assembling ATS/BD/TA products from broker + cex + compliance
- `lux/lux-cex.md` -- CLOB matching engine, uses broker for liquidity
- `lux/lux-compliance.md` -- KYC/AML (extracted from broker)
- `lux/lux-dex.md` -- High-perf DEX engine

---

**Category**: Lux Ecosystem
**Related**: broker, trading, sor, smart-order-routing, providers, settlement, market-data
**Prerequisites**: Go 1.26.1, understanding of brokerage/trading concepts
