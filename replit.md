# WebWaka Platform - Core Services & Capabilities

## Overview

WebWaka B2B SaaS platform with multi-tenancy, implementing reusable capabilities and core services following the Prompts-as-Artifacts (PaA) governance model.

## Active Service: CS-4 Pricing & Billing Service

Located at: `implementations/cs4-pricing-billing-service/`

### Features
- **Multi-Actor Pricing Authority**: Super Admin, Partners, Clients, Merchants, Agents, Staff
- **Composable Pricing Models**: Flat, Usage-Based, Tiered, Subscription, Revenue-Share, Hybrid
- **Decoupled Billing Engine**: Declarative pricing rules engine separate from billing execution
- **Deployment-Aware Pricing**: Shared SaaS, Partner-deployed, Self-hosted with inheritance/override
- **Auditability & Override Safety**: Versioned, auditable, and reversible pricing overrides

### API Endpoints
**Pricing:**
- `POST /api/v1/pricing/models` - Create pricing model
- `GET /api/v1/pricing/models` - List pricing models
- `POST /api/v1/pricing/models/:id/rules` - Create pricing rule
- `POST /api/v1/pricing/calculate` - Calculate price
- `POST /api/v1/pricing/scopes` - Create pricing scope
- `POST /api/v1/pricing/overrides` - Create override

**Billing:**
- `POST /api/v1/billing/cycles` - Create billing cycle
- `GET /api/v1/billing/cycles/:id/summary` - Get cycle summary
- `POST /api/v1/billing/cycles/:id/items` - Add billing item
- `GET /api/v1/billing/audit` - Search audit logs

## Completed Capabilities (Wave 1)

### CB-2 Reporting & Analytics
Located at: `implementations/CB-2_REPORTING_ANALYTICS_CAPABILITY/`
- Data aggregation engine, flexible query API, 10 pre-built reports
- Dashboard framework with 5 widget types, multi-format export

### CB-3 Content Management
Located at: `implementations/CB-3_CONTENT_MANAGEMENT_CAPABILITY/`
- Flexible content model, media management, versioning, workflows, localization

## Technology Stack

- **Runtime**: Node.js 20
- **Language**: TypeScript 5.x
- **Framework**: Express.js 4.x
- **Database**: PostgreSQL (Replit Postgres)
- **Decimal Math**: Decimal.js
- **Date Handling**: date-fns

## Platform Invariants Enforced

- **INV-001**: Pricing Flexibility - data-driven, declarative pricing
- **INV-002**: Tenant Isolation - all operations scoped by tenant_id
- **INV-005**: Partner-led operating model
- **INV-011**: Prompts-as-Artifacts execution
- **INV-012**: Single-Repository Topology

## Development

```bash
# CS-4 Pricing & Billing
cd implementations/cs4-pricing-billing-service
npm install
npm run dev   # Runs on port 5000
npm test      # 53 tests

# CB-2 Reporting & Analytics
cd implementations/CB-2_REPORTING_ANALYTICS_CAPABILITY
npm install
npm run dev   # Runs on port 5000
npm test      # 55 tests

# CB-3 Content Management
cd implementations/CB-3_CONTENT_MANAGEMENT_CAPABILITY
npm install
npm run dev   # Runs on port 5000
npm test      # 33 tests
```

## Architecture Documentation

- [CS-4 Architecture](/docs/architecture/ARCH_CS4_PRICING_BILLING.md)
- [CB-2 Architecture](/docs/architecture/ARCH_CB2_REPORTING_ANALYTICS.md)
- [CB-3 Architecture](/docs/architecture/ARCH_CB3_CONTENT_MANAGEMENT.md)
- [Master Control Board](/docs/governance/WEBWAKA_MASTER_CONTROL_BOARD.md)

## Runbooks

- [Configuring Pricing Models](/docs/runbooks/CS4_PRICING_MODELS.md)
- [Managing Billing Cycles](/docs/runbooks/CS4_BILLING_CYCLES.md)
- [Troubleshooting](/docs/runbooks/CS4_TROUBLESHOOTING.md)
