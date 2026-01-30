# Runbook: Configuring Pricing Models

## Overview

This runbook provides step-by-step instructions for configuring pricing models in the CS-4 Pricing & Billing Service.

---

## 1. Creating a Flat Pricing Model

### Use Case
Fixed price per unit, regardless of quantity (e.g., per-item pricing).

### API Request
```bash
curl -X POST http://localhost:5000/api/v1/pricing/models \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -H "X-User-Role: super_admin" \
  -d '{
    "tenantId": "tenant-001",
    "name": "Standard Product Pricing",
    "modelType": "flat",
    "description": "Fixed price per product unit",
    "config": {
      "basePrice": 500,
      "currency": "NGN"
    }
  }'
```

---

## 2. Creating a Tiered Pricing Model

### Use Case
Volume discounts - lower unit prices for larger quantities.

### API Request
```bash
curl -X POST http://localhost:5000/api/v1/pricing/models \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -H "X-User-Role: super_admin" \
  -d '{
    "tenantId": "tenant-001",
    "name": "Volume Discount Pricing",
    "modelType": "tiered",
    "config": {
      "currency": "NGN",
      "tiers": [
        { "minQuantity": 1, "maxQuantity": 100, "unitPrice": 100 },
        { "minQuantity": 101, "maxQuantity": 500, "unitPrice": 80 },
        { "minQuantity": 501, "maxQuantity": 1000, "unitPrice": 60 },
        { "minQuantity": 1001, "unitPrice": 40 }
      ]
    }
  }'
```

---

## 3. Creating a Subscription Pricing Model

### Use Case
Recurring monthly/yearly subscriptions.

### API Request
```bash
curl -X POST http://localhost:5000/api/v1/pricing/models \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -H "X-User-Role: super_admin" \
  -d '{
    "tenantId": "tenant-001",
    "name": "Premium Plan",
    "modelType": "subscription",
    "config": {
      "basePrice": 9999,
      "currency": "NGN",
      "subscriptionPeriod": "monthly"
    }
  }'
```

---

## 4. Creating a Usage-Based Pricing Model

### Use Case
Pay-per-use (e.g., API calls, storage, transactions).

### API Request
```bash
curl -X POST http://localhost:5000/api/v1/pricing/models \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -H "X-User-Role: super_admin" \
  -d '{
    "tenantId": "tenant-001",
    "name": "API Usage Pricing",
    "modelType": "usage_based",
    "config": {
      "basePrice": 0.10,
      "currency": "NGN",
      "usageMetric": "api_calls",
      "usageUnit": "per_call",
      "minimumCharge": 100
    }
  }'
```

---

## 5. Creating a Revenue Share Model

### Use Case
Commission-based pricing (e.g., marketplace fees).

### API Request
```bash
curl -X POST http://localhost:5000/api/v1/pricing/models \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -H "X-User-Role: super_admin" \
  -d '{
    "tenantId": "tenant-001",
    "name": "Marketplace Commission",
    "modelType": "revenue_share",
    "config": {
      "revenueSharePercent": 15,
      "currency": "NGN"
    }
  }'
```

---

## 6. Creating a Hybrid Pricing Model

### Use Case
Combination of base subscription + usage fees + commission.

### API Request
```bash
curl -X POST http://localhost:5000/api/v1/pricing/models \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -H "X-User-Role: super_admin" \
  -d '{
    "tenantId": "tenant-001",
    "name": "Enterprise Plan",
    "modelType": "hybrid",
    "config": {
      "currency": "NGN",
      "components": [
        {
          "type": "subscription",
          "weight": 1,
          "config": { "basePrice": 50000, "subscriptionPeriod": "monthly" }
        },
        {
          "type": "usage_based",
          "weight": 1,
          "config": { "basePrice": 0.05, "usageMetric": "transactions" }
        },
        {
          "type": "revenue_share",
          "weight": 1,
          "config": { "revenueSharePercent": 2 }
        }
      ]
    }
  }'
```

---

## 7. Adding Pricing Rules

### Use Case
Apply discounts for premium customers.

### API Request
```bash
curl -X POST http://localhost:5000/api/v1/pricing/models/{model-id}/rules \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "name": "Premium Customer Discount",
    "ruleType": "customer_discount",
    "conditions": [
      { "field": "customer_tier", "operator": "in", "value": ["gold", "platinum"] }
    ],
    "actions": [
      { "type": "apply_discount", "value": 10, "unit": "percent", "reason": "Premium tier discount" }
    ],
    "priority": 100
  }'
```

---

## 8. Assigning Pricing to Scopes

### Assign to Partner
```bash
curl -X POST http://localhost:5000/api/v1/pricing/scopes \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "pricingModelId": "{model-id}",
    "scopeType": "partner",
    "scopeId": "partner-123"
  }'
```

### Assign to Deployment Type
```bash
curl -X POST http://localhost:5000/api/v1/pricing/scopes \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "pricingModelId": "{model-id}",
    "scopeType": "deployment",
    "deploymentType": "partner_deployed"
  }'
```

---

## 9. Calculating Prices

### API Request
```bash
curl -X POST http://localhost:5000/api/v1/pricing/calculate \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "pricingModelId": "{model-id}",
    "scopeType": "client",
    "scopeId": "client-456",
    "itemType": "product_sale",
    "quantity": 150,
    "metadata": {
      "customer_tier": "gold"
    }
  }'
```

### Response
```json
{
  "basePrice": 13000,
  "adjustments": [
    { "type": "apply_discount", "amount": -1300, "reason": "Premium tier discount" }
  ],
  "finalPrice": 11700,
  "currency": "NGN",
  "breakdown": [
    { "component": "Tier 1-100", "quantity": 100, "unitPrice": 100, "subtotal": 10000 },
    { "component": "Tier 101-500", "quantity": 50, "unitPrice": 80, "subtotal": 4000 }
  ],
  "appliedRules": ["rule-123"],
  "appliedOverrides": []
}
```
