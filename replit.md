# WebWaka Platform - Wave 1 Capabilities

## Overview

WebWaka B2B SaaS platform with multi-tenancy, implementing reusable capabilities following the Prompts-as-Artifacts (PaA) governance model. Wave 1 includes CB-2 Reporting & Analytics and CB-3 Content Management capabilities.

## Active Capability: CB-2 Reporting & Analytics

Located at: `implementations/CB-2_REPORTING_ANALYTICS_CAPABILITY/`

### Features
- **Data Aggregation Engine**: Real-time and batch aggregation with multiple granularities (minute to year)
- **Flexible Query API**: Filters, grouping, sorting, pagination, date range presets
- **10 Pre-built Reports**: Sales Summary, Inventory Status, User Activity, Financial Overview, Top Products, Customer Segments, Revenue Trends, Conversion Funnel, Geographic Distribution, Performance Metrics
- **Dashboard Framework**: 5 widget types (line_chart, bar_chart, pie_chart, table, kpi_card)
- **Multi-format Export**: CSV, Excel, PDF

### API Endpoints
- `GET /` - API info
- `GET /health` - Health check
- `POST /api/v1/metrics` - Record metric
- `GET /api/v1/metrics?tenantId=` - List metrics
- `POST /api/v1/query` - Execute flexible query
- `GET /api/v1/reports?tenantId=` - List reports
- `POST /api/v1/dashboards` - Create dashboard
- `POST /api/v1/export` - Export data

## Available Capability: CB-3 Content Management

Located at: `implementations/CB-3_CONTENT_MANAGEMENT_CAPABILITY/`

### Features
- Flexible schema-driven content model (11 field types)
- Media management with validation
- Version history for all content
- Configurable workflows (3 system workflows)
- Multi-language localization support

## Technology Stack

- **Runtime**: Node.js 20
- **Language**: TypeScript 5.x
- **Framework**: Express.js 4.x
- **Database**: PostgreSQL (Replit Postgres)
- **Export Libraries**: ExcelJS, PDFKit

## Platform Invariants Enforced

- **INV-002**: Strict tenant isolation (all tables have tenant_id, all queries filter by tenant)
- **INV-005**: Partner-led operating model (partners can create custom reports, dashboards)
- **INV-011**: Prompts-as-Artifacts execution (documented in governance docs)

## Development

```bash
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

- [CB-2 Architecture](/docs/architecture/ARCH_CB2_REPORTING_ANALYTICS.md)
- [CB-3 Architecture](/docs/architecture/ARCH_CB3_CONTENT_MANAGEMENT.md)
- [Master Control Board](/docs/governance/WEBWAKA_MASTER_CONTROL_BOARD.md)
