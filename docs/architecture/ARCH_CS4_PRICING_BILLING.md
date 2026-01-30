# CS-4 Pricing & Billing Service - Architecture Document

**Date:** January 30, 2026  
**Version:** 1.0.0  
**Status:** Operational

---

## 1. Overview

The CS-4 Pricing & Billing Service is a flexible, data-driven pricing engine that supports multi-actor pricing authority, composable pricing models, a decoupled billing engine, deployment-aware pricing, and auditable overrides. The engine is fully declarative - no suite may embed its own billing logic.

### 1.1. Core Design Principles

1. **Declarative over Imperative**: All pricing logic is configuration-driven, not hard-coded
2. **Multi-Actor Authority**: Different actors can configure pricing at different scopes
3. **Composable Models**: Pricing models can be combined for hybrid pricing scenarios
4. **Decoupled Architecture**: Pricing rules engine is separate from billing execution
5. **Full Auditability**: All changes are versioned, logged, and reversible

---

## 2. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         CS-4 Pricing & Billing Service                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │  Pricing Routes  │    │  Billing Routes  │    │   Audit Routes   │  │
│  └────────┬─────────┘    └────────┬─────────┘    └────────┬─────────┘  │
│           │                       │                       │            │
│  ┌────────▼─────────────────────────────────────────────────────────┐  │
│  │                         Service Layer                             │  │
│  ├─────────────────┬─────────────────┬─────────────────┬────────────┤  │
│  │ PricingModel    │ PricingCalc     │ BillingEngine   │ Audit      │  │
│  │ Service         │ ulator          │                 │ Service    │  │
│  ├─────────────────┼─────────────────┼─────────────────┼────────────┤  │
│  │ ScopeService    │ OverrideService │                 │            │  │
│  └─────────────────┴─────────────────┴─────────────────┴────────────┘  │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                       PostgreSQL Database                         │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │  │
│  │  │pricing_models│ │pricing_rules │ │pricing_scopes│              │  │
│  │  └──────────────┘ └──────────────┘ └──────────────┘              │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │  │
│  │  │billing_cycles│ │billing_items │ │pricing_audit │              │  │
│  │  └──────────────┘ └──────────────┘ │    _log      │              │  │
│  │  ┌───────────────────────────────┐ └──────────────┘              │  │
│  │  │      pricing_overrides        │                               │  │
│  │  └───────────────────────────────┘                               │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Data Model

### 3.1. Core Entities

#### 3.1.1. Pricing Models

```typescript
interface PricingModel {
  id: string;
  tenantId: string;
  name: string;
  modelType: 'flat' | 'usage_based' | 'tiered' | 'subscription' | 'revenue_share' | 'hybrid';
  config: PricingConfig;
  isActive: boolean;
  isSystem: boolean;
  createdBy: string;
  createdByRole: ActorRole;
  version: number;
}
```

#### 3.1.2. Pricing Rules

```typescript
interface PricingRule {
  id: string;
  pricingModelId: string;
  name: string;
  ruleType: string;
  conditions: RuleCondition[];  // Field, operator, value
  actions: RuleAction[];        // Discounts, surcharges, multipliers
  priority: number;
  effectiveFrom?: Date;
  effectiveTo?: Date;
}
```

#### 3.1.3. Pricing Scopes

```typescript
interface PricingScope {
  id: string;
  pricingModelId: string;
  scopeType: 'global' | 'deployment' | 'partner' | 'client' | 'merchant' | 'agent' | 'staff' | 'individual' | 'group' | 'segment' | 'contract';
  scopeId?: string;
  deploymentType?: 'shared_saas' | 'partner_deployed' | 'self_hosted';
  isOverride: boolean;
  parentScopeId?: string;
}
```

### 3.2. Database Schema

| Table | Purpose |
|-------|---------|
| `pricing_models` | Stores pricing model definitions |
| `pricing_rules` | Declarative rules attached to models |
| `pricing_scopes` | Maps models to actors/deployments |
| `billing_cycles` | Tracks billing periods |
| `billing_items` | Individual billable items |
| `pricing_audit_log` | Complete audit trail |
| `pricing_overrides` | Versioned price overrides |

---

## 4. Pricing Model Types

### 4.1. Flat Pricing
Fixed price per unit, regardless of quantity.

```json
{
  "modelType": "flat",
  "config": {
    "basePrice": 100,
    "currency": "NGN"
  }
}
```

### 4.2. Usage-Based Pricing
Price based on consumption of a metric.

```json
{
  "modelType": "usage_based",
  "config": {
    "basePrice": 0.05,
    "usageMetric": "api_calls",
    "usageUnit": "per_call"
  }
}
```

### 4.3. Tiered Pricing
Different prices at different quantity levels.

```json
{
  "modelType": "tiered",
  "config": {
    "tiers": [
      { "minQuantity": 1, "maxQuantity": 100, "unitPrice": 10 },
      { "minQuantity": 101, "maxQuantity": 500, "unitPrice": 8 },
      { "minQuantity": 501, "unitPrice": 5 }
    ]
  }
}
```

### 4.4. Subscription Pricing
Fixed recurring fee for a period.

```json
{
  "modelType": "subscription",
  "config": {
    "basePrice": 999,
    "subscriptionPeriod": "monthly"
  }
}
```

### 4.5. Revenue Share Pricing
Percentage of revenue or transaction value.

```json
{
  "modelType": "revenue_share",
  "config": {
    "revenueSharePercent": 15
  }
}
```

### 4.6. Hybrid Pricing
Combination of multiple pricing models.

```json
{
  "modelType": "hybrid",
  "config": {
    "components": [
      { "type": "subscription", "config": { "basePrice": 500 } },
      { "type": "usage_based", "config": { "basePrice": 0.01 } },
      { "type": "revenue_share", "config": { "revenueSharePercent": 5 } }
    ]
  }
}
```

---

## 5. Multi-Actor Pricing Authority

### 5.1. Actor Roles

| Role | Authority Level | Typical Use |
|------|-----------------|-------------|
| `super_admin` | Platform-wide | Default pricing, system models |
| `partner` | Partner tenant | Partner-specific pricing |
| `client` | Client organization | Client contract pricing |
| `merchant` | Vendor/Merchant | Product-level pricing |
| `agent` | Field agent | Agent commission rates |
| `staff` | Internal staff | Limited overrides |

### 5.2. Scope Resolution Order

When calculating a price, scopes are resolved in order:
1. Individual (specific user)
2. Contract (negotiated terms)
3. Segment (customer segment)
4. Group (organization group)
5. Actor-specific scope (partner/client/merchant)
6. Deployment (SaaS/partner-deployed/self-hosted)
7. Global (platform default)

---

## 6. Pricing Rules Engine

### 6.1. Condition Operators

| Operator | Description |
|----------|-------------|
| `eq` | Equal to |
| `neq` | Not equal to |
| `gt` | Greater than |
| `gte` | Greater than or equal |
| `lt` | Less than |
| `lte` | Less than or equal |
| `in` | In array |
| `not_in` | Not in array |
| `contains` | String contains |
| `between` | Between two values |

### 6.2. Action Types

| Action | Description |
|--------|-------------|
| `apply_discount` | Reduce price (percent or fixed) |
| `apply_surcharge` | Increase price (percent or fixed) |
| `set_price` | Override to fixed price |
| `apply_multiplier` | Multiply price by factor |
| `add_fee` | Add additional fee |
| `skip` | Skip this item |

---

## 7. Deployment-Aware Pricing

### 7.1. Deployment Types

| Type | Description |
|------|-------------|
| `shared_saas` | Multi-tenant shared infrastructure |
| `partner_deployed` | Partner-managed whitelabel instance |
| `self_hosted` | Fully isolated self-hosted deployment |

### 7.2. Inheritance Model

Each deployment can:
- **Inherit** platform defaults
- **Override** specific pricing elements
- **Define** deployment-specific models

---

## 8. Audit & Override Safety

### 8.1. Audit Trail

Every action is logged with:
- Entity type and ID
- Action performed
- Actor ID and role
- Previous state (for rollback)
- New state
- Reason/justification
- Reversibility flag

### 8.2. Override Workflow

1. Create override with reason
2. Optional approval workflow
3. Effective date range
4. Version tracking
5. Deactivation capability
6. Full reversal support

---

## 9. API Reference

For complete API documentation including request/response schemas, see:
**[CS4 Pricing & Billing API Documentation](/docs/api/CS4_PRICING_BILLING_API.md)**

### 9.1. Pricing Endpoints Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/pricing/models` | Create pricing model |
| GET | `/api/v1/pricing/models` | List pricing models |
| GET | `/api/v1/pricing/models/:id` | Get pricing model |
| PUT | `/api/v1/pricing/models/:id` | Update pricing model |
| DELETE | `/api/v1/pricing/models/:id` | Delete pricing model |
| POST | `/api/v1/pricing/models/:id/rules` | Create pricing rule |
| GET | `/api/v1/pricing/models/:id/rules` | List pricing rules |
| POST | `/api/v1/pricing/calculate` | Calculate price |
| POST | `/api/v1/pricing/scopes` | Create pricing scope |
| GET | `/api/v1/pricing/scopes` | List scopes |
| POST | `/api/v1/pricing/overrides` | Create override |
| GET | `/api/v1/pricing/overrides` | List overrides |
| POST | `/api/v1/pricing/overrides/:id/approve` | Approve override |
| POST | `/api/v1/pricing/overrides/:id/deactivate` | Deactivate override |

### 9.2. Billing Endpoints Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/billing/cycles` | Create billing cycle |
| GET | `/api/v1/billing/cycles` | List billing cycles |
| GET | `/api/v1/billing/cycles/:id` | Get billing cycle |
| GET | `/api/v1/billing/cycles/:id/summary` | Get cycle summary |
| POST | `/api/v1/billing/cycles/:id/items` | Add billing item |
| GET | `/api/v1/billing/cycles/:id/items` | List billing items |
| POST | `/api/v1/billing/cycles/:id/close` | Close billing cycle |
| PATCH | `/api/v1/billing/cycles/:id/status` | Update cycle status |
| GET | `/api/v1/billing/audit` | Search audit logs |
| GET | `/api/v1/billing/audit/:entityType/:entityId` | Get entity audit history |
| POST | `/api/v1/billing/audit/:id/reverse` | Reverse action |

---

## 10. Platform Invariants

| Invariant | Description | Enforcement |
|-----------|-------------|-------------|
| INV-001 | Pricing Flexibility | Declarative config, no hard-coded pricing |
| INV-002 | Tenant Isolation | All operations scoped by tenant_id |
| INV-012 | Single Repository | All code in webwaka repository |

---

## 11. Technology Stack

- **Runtime**: Node.js 20
- **Language**: TypeScript 5.x
- **Framework**: Express.js 4.x
- **Database**: PostgreSQL (Replit Postgres)
- **Decimal Math**: Decimal.js
- **Date Handling**: date-fns

---

## 12. Testing

- **Unit Tests**: 53 tests covering all pricing models, rules, and billing calculations
- **Coverage**: Comprehensive coverage of pricing logic and audit trails

---

## 13. Related Documents

- [Master Control Board](/docs/governance/WEBWAKA_MASTER_CONTROL_BOARD.md)
- [CS-4 Execution Prompt](/docs/phases/CS-4_PRICING_BILLING_SERVICE.md)
