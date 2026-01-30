# WebWaka Platform - Core Services & Capabilities

## Overview

WebWaka B2B SaaS platform with multi-tenancy, implementing reusable capabilities and core services following the Prompts-as-Artifacts (PaA) governance model.

## Active Capability: CB-4 Inventory Management

Located at: `implementations/cb4-inventory-management/`

### Features
- **Core Inventory Service**: Products, SKUs, stock levels, multi-location support
- **Inventory Strategies**: FIFO, LIFO, Average Cost, Specific Identification
- **Channel Subscription**: Real-time event-based synchronization to sales channels
- **Stock Operations**: Receive, Sell, Adjust, Transfer, Reserve
- **Full Auditability**: Complete audit trail for all inventory movements

### API Endpoints
**Products:** `/api/v1/products`
**Locations:** `/api/v1/locations`
**Inventory:** `/api/v1/inventory/stock`, `/receive`, `/sell`, `/adjust`, `/movements`, `/transfers`, `/reservations`
**Channels:** `/api/v1/channels`, `/subscriptions`, `/events`
**Audit:** `/api/v1/audit`

## Completed Services & Capabilities

### CS-4 Pricing & Billing Service
Located at: `implementations/cs4-pricing-billing-service/`
- Multi-actor pricing authority, 6 composable pricing models
- Decoupled billing engine, deployment-aware pricing, auditable overrides

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
# CB-4 Inventory Management
cd implementations/cb4-inventory-management
npm install
npm run dev   # Runs on port 5000
npm test      # 35 tests

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

- [CB-4 Architecture](/docs/architecture/ARCH_CB4_INVENTORY_MANAGEMENT.md)
- [CS-4 Architecture](/docs/architecture/ARCH_CS4_PRICING_BILLING.md)
- [CB-2 Architecture](/docs/architecture/ARCH_CB2_REPORTING_ANALYTICS.md)
- [CB-3 Architecture](/docs/architecture/ARCH_CB3_CONTENT_MANAGEMENT.md)
- [Master Control Board](/docs/governance/WEBWAKA_MASTER_CONTROL_BOARD.md)

## API Documentation

- [CB-4 Inventory API](/docs/api/CB4_INVENTORY_API.md)
- [CS-4 Pricing & Billing API](/docs/api/CS4_PRICING_BILLING_API.md)

## Runbooks

- [CB-4 Inventory Operations](/docs/runbooks/CB4_INVENTORY_OPERATIONS.md)
- [CS-4 Pricing Models](/docs/runbooks/CS4_PRICING_MODELS.md)
- [CS-4 Billing Cycles](/docs/runbooks/CS4_BILLING_CYCLES.md)
- [CS-4 Troubleshooting](/docs/runbooks/CS4_TROUBLESHOOTING.md)
