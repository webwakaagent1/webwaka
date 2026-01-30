# CB-2: Reporting & Analytics Capability - Architecture Documentation

**Version:** 1.0  
**Date:** January 30, 2026  
**Status:** 🟢 Complete  

## Overview

CB-2 Reporting & Analytics Capability provides a reusable analytics engine that enables partners and clients to gain insights into their business operations. This capability offers data aggregation, flexible querying, pre-built reports, dashboard framework, and export functionality across the WebWaka platform.

## Architecture Decisions

### ADR-001: Metrics-Based Data Model

**Decision:** All analytics data is stored as timestamped metrics with dimensions.

**Rationale:**
- Consistent data model across all platform services
- Supports both counters (cumulative) and gauges (point-in-time)
- Dimensions enable flexible slicing and filtering
- Optimized for time-series analysis

**Consequences:**
- Services must emit metrics in the standard format
- Pre-aggregation needed for large datasets

### ADR-002: Pre-Computed Aggregations

**Decision:** Support both real-time queries and pre-computed aggregations.

**Rationale:**
- Real-time queries for up-to-date data
- Pre-computed aggregations for performance on historical data
- Batch processing for overnight rollups
- Granularity levels (minute, hour, day, week, month, year)

**Consequences:**
- Storage overhead for aggregated data
- Requires batch job scheduling

### ADR-003: Report-as-Configuration

**Decision:** Reports are defined as configuration objects, not code.

**Rationale:**
- Partners can create custom reports without code changes
- System reports provided as templates
- Report definitions are portable and version-controlled
- Enables report marketplace in future

**Consequences:**
- Query complexity is limited by configuration options
- Complex reports may require custom development

### ADR-004: Multi-Format Export Engine

**Decision:** Single export service supporting CSV, Excel, and PDF.

**Rationale:**
- Consistent export experience across formats
- PDF for formal reports with branding
- Excel for detailed analysis with formulas
- CSV for data integration with other tools

**Consequences:**
- PDF generation is CPU-intensive
- Large exports should be queued

## Component Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                 CB-2 Reporting & Analytics                       │
├─────────────────────────────────────────────────────────────────┤
│  API Layer (Express.js)                                          │
│  ├── /api/v1/metrics      - Metric recording and retrieval      │
│  ├── /api/v1/aggregations - Data aggregation                    │
│  ├── /api/v1/query        - Flexible query API                  │
│  ├── /api/v1/reports      - Report management and execution     │
│  ├── /api/v1/dashboards   - Dashboard management                │
│  └── /api/v1/export       - Data export (CSV, Excel, PDF)       │
├─────────────────────────────────────────────────────────────────┤
│  Service Layer                                                   │
│  ├── MetricsService       - Metric recording and retrieval      │
│  ├── AggregationService   - Real-time and batch aggregation     │
│  ├── QueryService         - Flexible query builder              │
│  ├── ReportService        - Report definitions and execution    │
│  ├── DashboardService     - Dashboard and widget management     │
│  └── ExportService        - Multi-format export engine          │
├─────────────────────────────────────────────────────────────────┤
│  Data Layer (PostgreSQL)                                         │
│  ├── cb2_metrics          - Raw metric events                   │
│  ├── cb2_aggregations     - Pre-computed aggregates             │
│  ├── cb2_report_definitions - Report templates                  │
│  ├── cb2_dashboards       - Dashboard configurations            │
│  ├── cb2_widgets          - Widget definitions                  │
│  ├── cb2_scheduled_reports - Scheduled report jobs              │
│  └── cb2_report_executions - Report execution history           │
└─────────────────────────────────────────────────────────────────┘
```

## Platform Invariant Compliance

### INV-002: Strict Tenant Isolation

All database tables include `tenant_id` column with mandatory filtering:
- All API endpoints require `tenantId` parameter
- All service methods enforce tenant scoping
- Indexes optimized for tenant-first queries
- No cross-tenant data access possible

### INV-005: Partner-Led Operating Model

Partners can operate independently:
- Create custom reports without WebWaka intervention
- Design custom dashboards with widgets
- Configure scheduled reports
- Export data in multiple formats

### INV-011: Prompts-as-Artifacts

Implementation follows PaA governance:
- Execution initiated via canonical prompt in CB-2 phase document
- All artifacts committed to repository
- Architecture documentation created and linked

## Database Schema

### Metrics
```sql
cb2_metrics (
  id UUID PRIMARY KEY,
  tenant_id VARCHAR(255) NOT NULL,
  metric_type VARCHAR(100) NOT NULL,  -- counter, gauge, histogram, summary
  metric_name VARCHAR(255) NOT NULL,
  value DECIMAL(20, 4) NOT NULL,
  dimensions JSONB DEFAULT '{}',
  timestamp TIMESTAMP WITH TIME ZONE,
  source VARCHAR(100),
  created_at TIMESTAMP WITH TIME ZONE
)
```

### Aggregations
```sql
cb2_aggregations (
  id UUID PRIMARY KEY,
  tenant_id VARCHAR(255) NOT NULL,
  aggregation_type VARCHAR(50) NOT NULL,
  metric_name VARCHAR(255) NOT NULL,
  period_start TIMESTAMP WITH TIME ZONE,
  period_end TIMESTAMP WITH TIME ZONE,
  granularity VARCHAR(20) NOT NULL,  -- minute, hour, day, week, month, year
  dimensions JSONB DEFAULT '{}',
  sum_value DECIMAL(20, 4),
  count_value INTEGER,
  min_value DECIMAL(20, 4),
  max_value DECIMAL(20, 4),
  avg_value DECIMAL(20, 4),
  UNIQUE(tenant_id, metric_name, period_start, granularity, dimensions)
)
```

### Report Definitions
```sql
cb2_report_definitions (
  id UUID PRIMARY KEY,
  tenant_id VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL,
  description TEXT,
  report_type VARCHAR(50) NOT NULL,  -- standard, custom, scheduled
  config JSONB NOT NULL,
  is_system BOOLEAN DEFAULT FALSE,
  created_by VARCHAR(255),
  UNIQUE(tenant_id, slug)
)
```

## Pre-Built Reports (10 System Reports)

| Report | Description | Metrics | Visualization |
|--------|-------------|---------|---------------|
| Sales Summary | Revenue, orders, average order value | revenue, order_count, order_value | Bar Chart |
| Inventory Status | Stock levels and movement | stock_level, stock_in, stock_out | Table |
| User Activity | Engagement metrics | active_users, page_views, session_duration | Line Chart |
| Financial Overview | Revenue, costs, margins | revenue, costs, profit, margin | KPI Card |
| Top Products | Best sellers by revenue/quantity | revenue, quantity_sold | Bar Chart |
| Customer Segments | Customer distribution | customer_count, lifetime_value | Pie Chart |
| Revenue Trends | Revenue over time | revenue | Line Chart |
| Conversion Funnel | Sales funnel metrics | visitors, leads, opportunities, customers | Bar Chart |
| Geographic Distribution | Regional performance | revenue, customer_count, order_count | Table |
| Performance Metrics | System KPIs | response_time, error_rate, uptime, throughput | KPI Card |

## Dashboard Framework (5 Widget Types)

| Widget Type | Description | Use Case |
|-------------|-------------|----------|
| Line Chart | Time-series visualization | Trends, patterns over time |
| Bar Chart | Comparison visualization | Category comparisons |
| Pie Chart | Distribution visualization | Proportional breakdown |
| Table | Tabular data display | Detailed data review |
| KPI Card | Single metric highlight | Key performance indicators |

## Query API Features

### Filters
- `eq` (equals), `neq` (not equals)
- `gt`, `gte`, `lt`, `lte` (comparisons)
- `in` (array membership)
- `like` (pattern matching)
- `between` (range)

### Date Range Presets
- `today`, `yesterday`
- `last_7_days`, `last_30_days`
- `this_month`, `last_month`
- `this_year`

### Grouping
- By metric dimensions
- By time granularity (minute to year)

### Ordering
- Ascending/descending on any field
- Multiple sort criteria

## Export Formats

| Format | Use Case | Features |
|--------|----------|----------|
| CSV | Data integration | Universal compatibility |
| Excel | Analysis | Formatted headers, styling |
| PDF | Formal reports | Title, pagination, totals |

## API Endpoints

### Metrics
- `POST /api/v1/metrics` - Record single metric
- `POST /api/v1/metrics/batch` - Record batch of metrics
- `GET /api/v1/metrics?tenantId=` - List metrics
- `GET /api/v1/metrics/names?tenantId=` - Get available metric names

### Aggregations
- `POST /api/v1/aggregations` - Compute aggregation
- `POST /api/v1/aggregations/batch` - Run batch aggregation

### Query
- `POST /api/v1/query` - Execute flexible query
- `GET /api/v1/query/dimensions?tenantId=` - Get available dimensions

### Reports
- `POST /api/v1/reports` - Create custom report
- `GET /api/v1/reports?tenantId=` - List reports
- `GET /api/v1/reports/:id?tenantId=` - Get report
- `PATCH /api/v1/reports/:id?tenantId=` - Update report
- `DELETE /api/v1/reports/:id?tenantId=` - Delete report
- `POST /api/v1/reports/:id/execute?tenantId=` - Execute report
- `POST /api/v1/reports/:id/export?tenantId=` - Export report
- `GET /api/v1/reports/executions/:id?tenantId=` - Get execution

### Dashboards
- `POST /api/v1/dashboards` - Create dashboard
- `GET /api/v1/dashboards?tenantId=` - List dashboards
- `GET /api/v1/dashboards/default?tenantId=` - Get default dashboard
- `GET /api/v1/dashboards/:id?tenantId=` - Get dashboard
- `PATCH /api/v1/dashboards/:id?tenantId=` - Update dashboard
- `DELETE /api/v1/dashboards/:id?tenantId=` - Delete dashboard
- `POST /api/v1/dashboards/:id/widgets?tenantId=` - Add widget
- `DELETE /api/v1/dashboards/:id/widgets/:widgetId?tenantId=` - Remove widget
- `GET /api/v1/widgets/system` - Get system widget templates

### Export
- `POST /api/v1/export` - Export data to format

## Dependencies

- **PF-1 (Foundational Extensions):** Database configuration and shared utilities
- **PostgreSQL:** Primary data storage
- **Express.js:** HTTP API framework
- **ExcelJS:** Excel file generation
- **PDFKit:** PDF document generation

## Future Enhancements

1. **Real-time Streaming:** WebSocket-based live dashboards
2. **Scheduled Reports:** Email delivery of reports on schedule
3. **Custom Visualizations:** Partner-defined chart types
4. **Data Connectors:** Import from external sources
5. **AI Insights:** Anomaly detection and trend prediction (PF-3)
6. **Report Templates:** Marketplace for report templates

---

**Backlink:** [CB-2 Phase Document](/docs/phases/CB-2_REPORTING_ANALYTICS_CAPABILITY.md)
