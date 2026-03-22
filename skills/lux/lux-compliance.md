# Lux Compliance - KYC, AML, and Regulatory Framework

**Category**: Lux Ecosystem
**Related Skills**: `lux/lux-regulated.md`, `lux/lux-broker.md`, `lux/lux-cex.md`

## Overview

Lux Compliance (`github.com/luxfi/compliance`) is a standalone Go module providing KYC orchestration, AML screening and monitoring, Jube fraud detection integration, multi-jurisdiction regulatory rules, and payment compliance. It was extracted from the broker module so that any regulated product (ATS, BD, TA, MSB) can import it independently.

## Quick Reference

| Item | Value |
|------|-------|
| Module | `github.com/luxfi/compliance` |
| Go | 1.26.1 |
| License | Apache 2.0 |

## Package Structure

```
github.com/luxfi/compliance/
├── pkg/
│   ├── idv/           Identity verification providers (Jumio, Onfido, Plaid)
│   ├── kyc/           KYC orchestration, verification lifecycle
│   ├── aml/           Sanctions screening + transaction monitoring
│   ├── jube/          Jube AML/fraud engine HTTP client
│   ├── entity/        Regulated entity types (ATS, BD, TA, MSB)
│   ├── regulatory/    Multi-jurisdiction compliance rules
│   ├── payments/      Travel rule, CTR, payment compliance
│   └── webhook/       Webhook routing for IDV callbacks
```

## Identity Verification (pkg/idv)

Three IDV providers with a unified interface:

```go
type Provider interface {
    Name() string
    Initiate(ctx context.Context, req *VerificationRequest) (*VerificationResult, error)
    GetStatus(ctx context.Context, verificationID string) (*VerificationResult, error)
    HandleWebhook(ctx context.Context, payload json.RawMessage, signature string) (*WebhookEvent, error)
}
```

### Providers

| Provider | Package | Auth Env Vars | Capabilities |
|----------|---------|--------------|-------------|
| **Jumio** | `idv.NewJumioProvider()` | `JUMIO_API_TOKEN`, `JUMIO_API_SECRET` | Document verification, liveness, address extraction |
| **Onfido** | `idv.NewOnfidoProvider()` | `ONFIDO_API_TOKEN` | Document check, facial similarity, watchlist |
| **Plaid** | `idv.NewPlaidProvider()` | `PLAID_CLIENT_ID`, `PLAID_SECRET` | Bank account verification, identity match, income |

### Verification Status

```go
const (
    StatusPending  VerificationStatus = "pending"
    StatusApproved VerificationStatus = "approved"
    StatusDeclined VerificationStatus = "declined"
    StatusExpired  VerificationStatus = "expired"
    StatusError    VerificationStatus = "error"
)
```

### Verification Request

```go
type VerificationRequest struct {
    ApplicationID string   `json:"application_id"`
    GivenName     string   `json:"given_name"`
    FamilyName    string   `json:"family_name"`
    DateOfBirth   string   `json:"date_of_birth,omitempty"`
    Email         string   `json:"email"`
    Phone         string   `json:"phone,omitempty"`
    Country       string   `json:"country,omitempty"`
    IPAddress     string   `json:"ip_address,omitempty"`
    Street        []string `json:"street,omitempty"`
    City          string   `json:"city,omitempty"`
    State         string   `json:"state,omitempty"`
    PostalCode    string   `json:"postal_code,omitempty"`
    TaxID         string   `json:"tax_id,omitempty"`
    TaxIDType     string   `json:"tax_id_type,omitempty"` // ssn, itin, ein
    DocumentType  string   `json:"document_type,omitempty"` // passport, drivers_license, id_card
}
```

## KYC Orchestration (pkg/kyc)

Manages the full verification lifecycle across multiple IDV providers:

```go
import (
    "github.com/luxfi/compliance/pkg/kyc"
    "github.com/luxfi/compliance/pkg/idv"
)

svc := kyc.NewService()

// Register IDV providers
svc.RegisterProvider("jumio", idv.NewJumioProvider(idv.JumioConfig{
    APIToken:  os.Getenv("JUMIO_API_TOKEN"),
    APISecret: os.Getenv("JUMIO_API_SECRET"),
}))
svc.RegisterProvider("onfido", idv.NewOnfidoProvider(idv.OnfidoConfig{
    APIToken: os.Getenv("ONFIDO_API_TOKEN"),
}))

// Set default provider
svc.SetDefault("jumio")

// Initiate verification
v, err := svc.Initiate(ctx, &kyc.InitiateRequest{
    ApplicationID: "app_123",
    Provider:      "jumio", // or "" for default
    Request:       &idv.VerificationRequest{...},
})
// v.ID, v.RedirectURL (for user to complete verification)

// Handle webhook callback
event, err := svc.HandleWebhook(ctx, "jumio", payload, signature)

// Check status
v, err := svc.Get(ctx, verificationID)
// v.Status, v.RiskScore, v.Checks

// List by application
verifications := svc.ListByApplication("app_123")

// List by org
verifications := svc.ListByOrg("org_456")
```

### Verification Lifecycle

```
Initiate -> pending -> (webhook callback) -> approved | declined | error
                                          -> expired (if no callback within TTL)
```

## AML Screening (pkg/aml)

### Sanctions Screening

Checks against OFAC SDN, EU sanctions, UK HMT, PEP databases, and adverse media:

```go
import "github.com/luxfi/compliance/pkg/aml"

screener := aml.NewScreeningService()

result, err := screener.Screen(ctx, &aml.ScreeningRequest{
    GivenName:  "John",
    FamilyName: "Doe",
    Country:    "US",
})
// result.RiskLevel: "low" | "medium" | "high" | "critical"
// result.Matches: [{ListSource: "ofac_sdn", MatchType: "fuzzy", Score: 0.85, ...}]
```

**List sources**:
- `ofac_sdn` -- OFAC Specially Designated Nationals
- `eu_sanctions` -- EU Consolidated Sanctions List
- `uk_hmt` -- UK HM Treasury
- `pep` -- Politically Exposed Persons
- `adverse_media` -- Adverse media screening

**Match types**: `exact`, `fuzzy`, `partial`

**Risk levels**: `low`, `medium`, `high`, `critical`

### Transaction Monitoring

Real-time monitoring with configurable rules:

```go
monitorSvc := aml.NewMonitoringService()

// Install the 3 FinCEN-mandated baseline rules
aml.InstallDefaultRules(monitorSvc)

// Or add custom rules
monitorSvc.AddRule(aml.Rule{
    ID:              "custom-geographic",
    Type:            aml.RuleGeographic,
    Enabled:         true,
    Severity:        aml.SeverityHigh,
    Description:     "Flag transactions involving high-risk jurisdictions",
})

// Screen a transaction
alerts, err := monitorSvc.Evaluate(ctx, &aml.Transaction{
    AccountID: "acct_123",
    Type:      "wire",
    Direction: "outbound",
    Amount:    15000,
    Currency:  "USD",
    Country:   "IR",
})
// alerts: [{RuleID: "ctr-threshold", Severity: "high", ...}]
```

### DefaultMonitoringRules

The 3 standard rules mandated by FinCEN:

| Rule ID | Type | Threshold | Reference | Description |
|---------|------|-----------|-----------|-------------|
| `ctr-threshold` | `single_amount` | $10,000 | 31 CFR 1010.311 | Flag transactions over $10,000 (CTR) |
| `structuring` | `structuring` | $9,000 | 31 USC 5324 | Detect $9,000-$9,999 pattern |
| `velocity-24h` | `velocity` | $50,000 / 100 txns / 24h | FinCEN | Block if 24h volume exceeds threshold |

### Rule Types

```go
const (
    RuleSingleAmount   RuleType = "single_amount"
    RuleDailyAggregate RuleType = "daily_aggregate"
    RuleVelocity       RuleType = "velocity"
    RuleGeographic     RuleType = "geographic"
    RuleStructuring    RuleType = "structuring"
)
```

### Alert Lifecycle

```go
const (
    AlertOpen         AlertStatus = "open"
    AlertInvestigating AlertStatus = "investigating"
    AlertEscalated    AlertStatus = "escalated"
    AlertClosed       AlertStatus = "closed"
    AlertFiled        AlertStatus = "filed" // SAR filed
)
```

## Jube Integration (pkg/jube)

Jube is a C# transaction monitoring engine. The client calls its HTTP API for real-time scoring:

```go
import "github.com/luxfi/compliance/pkg/jube"

client, err := jube.NewClient(jube.Config{
    BaseURL:  "http://jube.svc.cluster.local:5001",
    ModelID:  os.Getenv("JUBE_MODEL_ID"),
    FailOpen: false, // reject when Jube is unreachable
    Timeout:  5 * time.Second,
})

// Score a transaction
result, err := client.ScoreTransaction(ctx, &jube.TransactionRequest{
    AccountID:     "acct_123",
    TransactionID: "tx_456",
    Amount:        15000,
    Currency:      "USD",
    Direction:     "outbound",
    Country:       "US",
})
// result.Score, result.ResponseElevation, result.ActivationAlerts, result.SanctionsMatches
```

**API**: `POST /api/invoke/EntityAnalysisModel/{modelGUID}`

**FailOpen behavior**:
- `true`: Transactions allowed through when Jube is down (risky)
- `false`: Transactions rejected when Jube is unavailable (safe default)

### Pre-Trade Compliance with Jube

```go
import "github.com/luxfi/compliance/pkg/jube"

pretrade := jube.NewPreTradeChecker(client)

decision, err := pretrade.Check(ctx, &jube.PreTradeRequest{
    AccountID: "acct_123",
    Symbol:    "BTC/USD",
    Side:      "buy",
    Qty:       10.5,
    Price:     42500,
    KYCLevel:  2,
})
// decision: "approve" | "review" | "reject"
```

## Regulatory Frameworks (pkg/regulatory)

Multi-jurisdiction compliance rules:

```go
type Jurisdiction interface {
    Name() string
    Code() string // ISO 3166-1 alpha-2
    Requirements() []Requirement
    ValidateApplication(app *ApplicationData) []Violation
    TransactionLimits() *Limits
}
```

### Supported Jurisdictions

| Code | Name | Regulator | Implementation |
|------|------|-----------|---------------|
| `US` | United States | SEC / FINRA / FinCEN | `regulatory.NewUSA()` |
| `UK` | United Kingdom | FCA | `regulatory.NewUK()` |
| `IM` | Isle of Man | IOMFSA | `regulatory.NewIOM()` |

### Transaction Limits

```go
type Limits struct {
    SingleTransactionMax float64 // max single tx
    DailyMax             float64
    MonthlyMax           float64
    AnnualMax            float64
    CTRThreshold         float64 // Currency Transaction Report threshold
    TravelRuleMin        float64 // FATF travel rule applies above this
    Currency             string
}
```

### Application Validation

```go
usa := regulatory.NewUSA()
violations := usa.ValidateApplication(&regulatory.ApplicationData{
    GivenName:  "John",
    FamilyName: "Doe",
    TaxID:      "", // missing — violation
    Country:    "US",
})
// violations: [{RequirementID: "us-tax-id", Field: "tax_id", Message: "SSN/ITIN required for US persons"}]
```

## Payment Compliance (pkg/payments)

Travel rule enforcement and per-jurisdiction payment validation:

```go
import "github.com/luxfi/compliance/pkg/payments"

svc := payments.NewService(screeningSvc, jurisdictions)

decision, err := svc.ValidatePayment(ctx, &payments.PaymentRequest{
    Direction:     payments.PaymentPayout,
    Amount:        5000,
    Currency:      "USD",
    Country:       "US",
    Type:          "wire",
    AccountID:     "acct_123",

    // Travel rule fields (required for wire > $3,000)
    OriginatorName:    "John Doe",
    OriginatorAccount: "acct_123",
    BeneficiaryName:    "Jane Smith",
    BeneficiaryAccount: "ext_456",
})
// decision.Decision: "approve" | "decline" | "review"
// decision.Violations: [{Rule: "travel_rule", Message: "beneficiary address required for wire > $3,000"}]
```

### Payment Decisions

```go
const (
    DecisionApprove PaymentDecision = "approve"
    DecisionDecline PaymentDecision = "decline"
    DecisionReview  PaymentDecision = "review"
)
```

### Travel Rule

FATF Travel Rule requires originator and beneficiary information for transfers above jurisdiction thresholds:
- **US**: $3,000 (31 CFR 1010.410)
- **EU**: EUR 1,000 (TFR)
- **UK**: GBP 1,000

## Entity Types (pkg/entity)

Defines regulated financial entity types with their license requirements, reporting obligations, capital requirements, and operational requirements:

```go
type RegulatedEntity interface {
    Name() string
    Type() string
    Jurisdiction() string
    RequiredLicenses() []License
    ReportingObligations() []ReportingObligation
    CapitalRequirements() *CapitalRequirement
    OperationalRequirements() []OperationalRequirement
}
```

### Supported Entity Types

| Type | Go Type | Jurisdiction | Key Licenses |
|------|---------|-------------|-------------|
| ATS | `entity.ATS` | US | SEC Reg ATS, FINRA membership |
| BD | `entity.BD` | US | SEC Section 15, FINRA membership |
| TA | `entity.TA` | US | SEC Rule 17Ad |
| MSB | `entity.MSB` | US | FinCEN registration, state MTLs |

```go
import "github.com/luxfi/compliance/pkg/entity"

ats := &entity.ATS{}
licenses := ats.RequiredLicenses()
// [{Name: "Reg ATS Registration", Regulator: "SEC", Reference: "17 CFR 242.301"}]

capital := ats.CapitalRequirements()
// {MinNetCapital: 250000, Currency: "USD", CalculationRule: "Rule 15c3-1"}

reporting := ats.ReportingObligations()
// [{Name: "Form ATS-N", Frequency: "quarterly", Regulator: "SEC/FINRA"}]
```

## Stablecoin Compliance (pkg/payments)

Special handling for stablecoin transfers via Circle:

```go
import "github.com/luxfi/compliance/pkg/payments"

// Stablecoin-specific compliance checks
result := payments.ValidateStablecoinTransfer(&payments.StablecoinRequest{
    Token:     "USDC",
    Chain:     "ethereum",
    Amount:    100000,
    Direction: "mint",
})
```

## Integration Pattern

Full compliance stack for any regulated product:

```go
package main

import (
    "github.com/luxfi/compliance/pkg/aml"
    "github.com/luxfi/compliance/pkg/idv"
    "github.com/luxfi/compliance/pkg/jube"
    "github.com/luxfi/compliance/pkg/kyc"
    "github.com/luxfi/compliance/pkg/payments"
    "github.com/luxfi/compliance/pkg/regulatory"
)

func setupCompliance() {
    // 1. KYC with multiple IDV providers
    kycSvc := kyc.NewService()
    kycSvc.RegisterProvider("jumio", idv.NewJumioProvider(...))
    kycSvc.RegisterProvider("onfido", idv.NewOnfidoProvider(...))
    kycSvc.SetDefault("jumio")

    // 2. AML screening + monitoring
    screener := aml.NewScreeningService()
    monitor := aml.NewMonitoringService()
    aml.InstallDefaultRules(monitor) // FinCEN baseline

    // 3. Jube integration (real-time fraud scoring)
    jubeClient, _ := jube.NewClient(jube.Config{
        BaseURL:  os.Getenv("JUBE_BASE_URL"),
        ModelID:  os.Getenv("JUBE_MODEL_ID"),
        FailOpen: false,
    })

    // 4. Jurisdiction rules
    jurisdictions := map[string]regulatory.Jurisdiction{
        "US": regulatory.NewUSA(),
        "UK": regulatory.NewUK(),
        "IM": regulatory.NewIOM(),
    }

    // 5. Payment compliance (travel rule, CTR)
    paymentSvc := payments.NewService(screener, jurisdictions)
}
```

## Related Skills

- `lux/lux-regulated.md` -- How to assemble ATS/BD/TA from these components
- `lux/lux-broker.md` -- Broker module that compliance integrates with
- `lux/lux-cex.md` -- CEX with pre/post-trade compliance hooks

---

**Last Updated**: 2026-03-22
**Category**: Lux Ecosystem
**Related**: compliance, kyc, aml, sanctions, travel-rule, jube, jumio, onfido, plaid, finra, sec
**Prerequisites**: Go 1.26.1, understanding of KYC/AML/regulatory compliance
