# Runbook: Managing Billing Cycles

## Overview

This runbook provides instructions for creating, managing, and closing billing cycles in the CS-4 Pricing & Billing Service.

---

## 1. Creating a Billing Cycle

### Monthly Billing Cycle
```bash
curl -X POST http://localhost:5000/api/v1/billing/cycles \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "scopeId": "client-456",
    "scopeType": "client",
    "cycleType": "monthly",
    "startDate": "2026-02-01"
  }'
```

### Custom Date Range
```bash
curl -X POST http://localhost:5000/api/v1/billing/cycles \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "scopeId": "partner-123",
    "scopeType": "partner",
    "cycleType": "custom",
    "startDate": "2026-01-15",
    "endDate": "2026-02-14"
  }'
```

---

## 2. Adding Items to a Billing Cycle

### Add Subscription Fee
```bash
curl -X POST http://localhost:5000/api/v1/billing/cycles/{cycle-id}/items \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "pricingModelId": "{subscription-model-id}",
    "itemType": "subscription_fee",
    "quantity": 1,
    "scopeType": "client",
    "scopeId": "client-456",
    "description": "Monthly Premium Subscription"
  }'
```

### Add Usage-Based Charges
```bash
curl -X POST http://localhost:5000/api/v1/billing/cycles/{cycle-id}/items \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "pricingModelId": "{usage-model-id}",
    "itemType": "api_usage",
    "quantity": 5000,
    "scopeType": "client",
    "scopeId": "client-456",
    "description": "API Calls - January 2026",
    "metadata": {
      "usage_period": "2026-01"
    }
  }'
```

### Add Transaction Fees
```bash
curl -X POST http://localhost:5000/api/v1/billing/cycles/{cycle-id}/items \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "pricingModelId": "{revenue-share-model-id}",
    "itemType": "transaction_fee",
    "quantity": 150000,
    "scopeType": "merchant",
    "scopeId": "merchant-789",
    "description": "Transaction Revenue Share - 15%"
  }'
```

---

## 3. Viewing Billing Cycle Summary

### Get Summary
```bash
curl "http://localhost:5000/api/v1/billing/cycles/{cycle-id}/summary?tenantId=tenant-001"
```

### Response
```json
{
  "cycle": {
    "id": "cycle-123",
    "scopeId": "client-456",
    "scopeType": "client",
    "cycleType": "monthly",
    "startDate": "2026-02-01",
    "endDate": "2026-02-28",
    "status": "active"
  },
  "items": [...],
  "subtotal": 75000,
  "total": 75000,
  "currency": "NGN",
  "itemCount": 3
}
```

---

## 4. Closing a Billing Cycle

### Close Cycle
```bash
curl -X POST http://localhost:5000/api/v1/billing/cycles/{cycle-id}/close \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -H "X-User-Role: super_admin" \
  -d '{
    "tenantId": "tenant-001"
  }'
```

---

## 5. Updating Billing Cycle Status

### Mark as Invoiced
```bash
curl -X PATCH http://localhost:5000/api/v1/billing/cycles/{cycle-id}/status \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -H "X-User-Role: super_admin" \
  -d '{
    "tenantId": "tenant-001",
    "status": "invoiced"
  }'
```

### Mark as Paid
```bash
curl -X PATCH http://localhost:5000/api/v1/billing/cycles/{cycle-id}/status \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "status": "paid"
  }'
```

### Mark as Overdue
```bash
curl -X PATCH http://localhost:5000/api/v1/billing/cycles/{cycle-id}/status \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "status": "overdue"
  }'
```

---

## 6. Listing Billing Cycles

### All Active Cycles
```bash
curl "http://localhost:5000/api/v1/billing/cycles?tenantId=tenant-001&status=active"
```

### By Scope
```bash
curl "http://localhost:5000/api/v1/billing/cycles?tenantId=tenant-001&scopeId=client-456&scopeType=client"
```

### By Date Range
```bash
curl "http://localhost:5000/api/v1/billing/cycles?tenantId=tenant-001&fromDate=2026-01-01&toDate=2026-12-31"
```

---

## 7. Billing Cycle Status Flow

```
┌─────────┐     ┌────────┐     ┌──────────┐     ┌──────┐
│ active  │────▶│ closed │────▶│ invoiced │────▶│ paid │
└─────────┘     └────────┘     └──────────┘     └──────┘
                                    │
                                    ▼
                               ┌─────────┐
                               │ overdue │
                               └─────────┘
```

### Valid Transitions
- `active` → `closed`
- `closed` → `invoiced`
- `invoiced` → `paid`
- `invoiced` → `overdue`
- Any → `cancelled`

---

## 8. Billing Cycle Types

| Type | Duration | Use Case |
|------|----------|----------|
| `daily` | 1 day | High-frequency billing |
| `weekly` | 7 days | Weekly settlements |
| `monthly` | 1 month | Standard subscriptions |
| `quarterly` | 3 months | Enterprise contracts |
| `yearly` | 12 months | Annual subscriptions |
| `custom` | User-defined | Special billing periods |
