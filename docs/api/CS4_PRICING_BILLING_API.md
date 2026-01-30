# CS-4 Pricing & Billing Service - API Documentation

**Version:** 1.0.0  
**Base URL:** `/api/v1`

---

## Authentication Headers

All requests require the following headers:

| Header | Required | Description |
|--------|----------|-------------|
| `X-User-Id` | Yes (for writes) | ID of the user making the request |
| `X-User-Role` | Yes (for writes) | Role of the user: `super_admin`, `partner`, `client`, `merchant`, `agent`, `staff` |

---

## Pricing Endpoints

### Create Pricing Model

Creates a new pricing model.

**Endpoint:** `POST /pricing/models`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "name": "string (required)",
  "modelType": "flat | usage_based | tiered | subscription | revenue_share | hybrid (required)",
  "description": "string (optional)",
  "config": {
    "basePrice": "number",
    "currency": "string (default: NGN)",
    "minimumCharge": "number (optional)",
    "maximumCharge": "number (optional)",
    "tiers": [
      {
        "minQuantity": "number",
        "maxQuantity": "number (optional)",
        "unitPrice": "number"
      }
    ],
    "usageMetric": "string",
    "usageUnit": "string",
    "subscriptionPeriod": "daily | weekly | monthly | quarterly | yearly",
    "revenueSharePercent": "number",
    "components": [
      {
        "type": "flat | usage_based | tiered | subscription | revenue_share",
        "weight": "number (default: 1)",
        "config": {}
      }
    ]
  },
  "isSystem": "boolean (default: false)"
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "tenantId": "string",
  "name": "string",
  "modelType": "string",
  "description": "string",
  "config": {},
  "isActive": true,
  "isSystem": false,
  "createdBy": "string",
  "createdByRole": "string",
  "version": 1,
  "createdAt": "ISO8601",
  "updatedAt": "ISO8601"
}
```

---

### List Pricing Models

Retrieves all pricing models for a tenant.

**Endpoint:** `GET /pricing/models`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |
| `modelType` | string | Optional. Filter by model type |
| `isActive` | boolean | Optional. Filter by active status |

**Response:** `200 OK`
```json
[
  {
    "id": "uuid",
    "tenantId": "string",
    "name": "string",
    "modelType": "string",
    "config": {},
    "isActive": true,
    "version": 1
  }
]
```

---

### Get Pricing Model

Retrieves a specific pricing model by ID.

**Endpoint:** `GET /pricing/models/:id`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "tenantId": "string",
  "name": "string",
  "modelType": "string",
  "description": "string",
  "config": {},
  "isActive": true,
  "isSystem": false,
  "createdBy": "string",
  "createdByRole": "string",
  "version": 1,
  "createdAt": "ISO8601",
  "updatedAt": "ISO8601"
}
```

---

### Update Pricing Model

Updates an existing pricing model.

**Endpoint:** `PUT /pricing/models/:id`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "name": "string (optional)",
  "description": "string (optional)",
  "config": {},
  "isActive": "boolean (optional)"
}
```

**Response:** `200 OK`

---

### Delete Pricing Model

Soft-deletes a pricing model (sets isActive to false).

**Endpoint:** `DELETE /pricing/models/:id`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |

**Response:** `200 OK`

---

### Create Pricing Rule

Creates a rule for a pricing model.

**Endpoint:** `POST /pricing/models/:id/rules`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "name": "string (required)",
  "ruleType": "string (required)",
  "conditions": [
    {
      "field": "string",
      "operator": "eq | neq | gt | gte | lt | lte | in | not_in | contains | between",
      "value": "any"
    }
  ],
  "actions": [
    {
      "type": "apply_discount | apply_surcharge | set_price | apply_multiplier | add_fee | skip",
      "value": "number",
      "unit": "percent | fixed (optional)",
      "reason": "string (optional)"
    }
  ],
  "priority": "number (default: 0)",
  "effectiveFrom": "ISO8601 (optional)",
  "effectiveTo": "ISO8601 (optional)"
}
```

**Response:** `201 Created`

---

### List Pricing Rules

Retrieves all rules for a pricing model.

**Endpoint:** `GET /pricing/models/:id/rules`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |

**Response:** `200 OK`
```json
[
  {
    "id": "uuid",
    "pricingModelId": "uuid",
    "name": "string",
    "ruleType": "string",
    "conditions": [],
    "actions": [],
    "priority": 0,
    "isActive": true
  }
]
```

---

### Calculate Price

Calculates the final price for an item using the pricing engine.

**Endpoint:** `POST /pricing/calculate`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "pricingModelId": "uuid (required)",
  "scopeType": "global | deployment | partner | client | merchant | agent | staff | individual | group | segment | contract",
  "scopeId": "string (optional)",
  "deploymentType": "shared_saas | partner_deployed | self_hosted (optional)",
  "itemType": "string (required)",
  "quantity": "number (required)",
  "metadata": {
    "customer_tier": "string",
    "region": "string"
  }
}
```

**Response:** `200 OK`
```json
{
  "basePrice": 1000,
  "adjustments": [
    {
      "type": "apply_discount",
      "amount": -100,
      "reason": "Premium customer discount",
      "ruleId": "uuid"
    }
  ],
  "finalPrice": 900,
  "currency": "NGN",
  "breakdown": [
    {
      "component": "Tier 1",
      "quantity": 100,
      "unitPrice": 10,
      "subtotal": 1000
    }
  ],
  "appliedRules": ["uuid"],
  "appliedOverrides": ["uuid"],
  "calculatedAt": "ISO8601"
}
```

---

### Create Pricing Scope

Assigns a pricing model to a scope.

**Endpoint:** `POST /pricing/scopes`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "pricingModelId": "uuid (required)",
  "scopeType": "global | deployment | partner | client | merchant | agent | staff | individual | group | segment | contract (required)",
  "scopeId": "string (optional, required for non-global scopes)",
  "deploymentType": "shared_saas | partner_deployed | self_hosted (optional)",
  "isOverride": "boolean (default: false)",
  "parentScopeId": "uuid (optional)"
}
```

**Response:** `201 Created`

---

### List Pricing Scopes

Retrieves all pricing scopes.

**Endpoint:** `GET /pricing/scopes`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |
| `scopeType` | string | Optional. Filter by scope type |
| `pricingModelId` | uuid | Optional. Filter by pricing model |

**Response:** `200 OK`

---

### Create Override

Creates a pricing override.

**Endpoint:** `POST /pricing/overrides`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "pricingModelId": "uuid (required)",
  "scopeType": "string (required)",
  "scopeId": "string (optional)",
  "overrideType": "price | discount | config (required)",
  "overrideValue": {},
  "reason": "string (required)",
  "effectiveFrom": "ISO8601 (optional)",
  "effectiveTo": "ISO8601 (optional)"
}
```

**Response:** `201 Created`

---

### List Overrides

Retrieves all pricing overrides.

**Endpoint:** `GET /pricing/overrides`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |
| `pricingModelId` | uuid | Optional. Filter by pricing model |
| `isActive` | boolean | Optional. Filter by active status |

**Response:** `200 OK`

---

### Approve Override

Approves a pending override.

**Endpoint:** `POST /pricing/overrides/:id/approve`

**Request Body:**
```json
{
  "tenantId": "string (required)"
}
```

**Response:** `200 OK`

---

### Deactivate Override

Deactivates an active override.

**Endpoint:** `POST /pricing/overrides/:id/deactivate`

**Request Body:**
```json
{
  "tenantId": "string (required)"
}
```

**Response:** `200 OK`

---

## Billing Endpoints

### Create Billing Cycle

Creates a new billing cycle.

**Endpoint:** `POST /billing/cycles`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "scopeId": "string (required)",
  "scopeType": "partner | client | merchant | agent | staff (required)",
  "cycleType": "daily | weekly | monthly | quarterly | yearly | custom (required)",
  "startDate": "ISO8601 (required)",
  "endDate": "ISO8601 (required for custom cycles)"
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "tenantId": "string",
  "scopeId": "string",
  "scopeType": "string",
  "cycleType": "string",
  "startDate": "ISO8601",
  "endDate": "ISO8601",
  "status": "active",
  "createdAt": "ISO8601"
}
```

---

### List Billing Cycles

Retrieves all billing cycles.

**Endpoint:** `GET /billing/cycles`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |
| `scopeId` | string | Optional. Filter by scope |
| `scopeType` | string | Optional. Filter by scope type |
| `status` | string | Optional. Filter by status |
| `fromDate` | ISO8601 | Optional. Filter by start date |
| `toDate` | ISO8601 | Optional. Filter by end date |

**Response:** `200 OK`

---

### Get Billing Cycle

Retrieves a specific billing cycle.

**Endpoint:** `GET /billing/cycles/:id`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |

**Response:** `200 OK`

---

### Get Billing Cycle Summary

Retrieves a summary of a billing cycle including all items and totals.

**Endpoint:** `GET /billing/cycles/:id/summary`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |

**Response:** `200 OK`
```json
{
  "cycle": {
    "id": "uuid",
    "scopeId": "string",
    "scopeType": "string",
    "cycleType": "string",
    "startDate": "ISO8601",
    "endDate": "ISO8601",
    "status": "string"
  },
  "items": [],
  "subtotal": 1000,
  "total": 1000,
  "currency": "NGN",
  "itemCount": 5
}
```

---

### Add Billing Item

Adds an item to a billing cycle.

**Endpoint:** `POST /billing/cycles/:id/items`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "pricingModelId": "uuid (required)",
  "itemType": "string (required)",
  "quantity": "number (required)",
  "scopeType": "string (required)",
  "scopeId": "string (required)",
  "description": "string (optional)",
  "metadata": {}
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "billingCycleId": "uuid",
  "pricingModelId": "uuid",
  "itemType": "string",
  "quantity": 10,
  "unitPrice": 100,
  "totalAmount": 1000,
  "currency": "NGN",
  "description": "string",
  "createdAt": "ISO8601"
}
```

---

### List Billing Items

Retrieves all items in a billing cycle.

**Endpoint:** `GET /billing/cycles/:id/items`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |

**Response:** `200 OK`

---

### Close Billing Cycle

Closes an active billing cycle (no more items can be added).

**Endpoint:** `POST /billing/cycles/:id/close`

**Request Body:**
```json
{
  "tenantId": "string (required)"
}
```

**Response:** `200 OK`

---

### Update Billing Cycle Status

Updates the status of a billing cycle.

**Endpoint:** `PATCH /billing/cycles/:id/status`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "status": "closed | invoiced | paid | overdue | cancelled (required)"
}
```

**Response:** `200 OK`

---

### Search Audit Logs

Searches the pricing audit log.

**Endpoint:** `GET /billing/audit`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |
| `entityType` | string | Optional. Filter by entity type |
| `entityId` | uuid | Optional. Filter by entity ID |
| `actorId` | string | Optional. Filter by actor |
| `action` | string | Optional. Filter by action |
| `fromDate` | ISO8601 | Optional. Start of date range |
| `toDate` | ISO8601 | Optional. End of date range |

**Response:** `200 OK`
```json
[
  {
    "id": "uuid",
    "tenantId": "string",
    "entityType": "pricing_model",
    "entityId": "uuid",
    "action": "create",
    "actorId": "string",
    "actorRole": "super_admin",
    "previousState": null,
    "newState": {},
    "reason": "string",
    "isReversible": true,
    "createdAt": "ISO8601"
  }
]
```

---

### Get Entity Audit History

Retrieves the complete audit history for an entity.

**Endpoint:** `GET /billing/audit/:entityType/:entityId`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required. Tenant identifier |

**Response:** `200 OK`

---

### Reverse Action

Reverses a previous audited action.

**Endpoint:** `POST /billing/audit/:id/reverse`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "reason": "string (required)"
}
```

**Response:** `200 OK`

---

## Error Responses

All endpoints return standard error responses:

**400 Bad Request:**
```json
{
  "error": "Description of validation error"
}
```

**404 Not Found:**
```json
{
  "error": "Resource not found"
}
```

**500 Internal Server Error:**
```json
{
  "error": "Internal server error message"
}
```

---

## Data Types Reference

### Model Types
- `flat` - Fixed price per unit
- `usage_based` - Price based on consumption
- `tiered` - Volume-based pricing tiers
- `subscription` - Recurring fixed fees
- `revenue_share` - Percentage of revenue
- `hybrid` - Combination of multiple models

### Scope Types
- `global` - Platform-wide default
- `deployment` - Deployment type scope
- `partner` - Partner organization
- `client` - Client organization
- `merchant` - Vendor/Merchant
- `agent` - Field agent
- `staff` - Internal staff
- `individual` - Specific user
- `group` - Organization group
- `segment` - Customer segment
- `contract` - Specific contract

### Actor Roles
- `super_admin` - Platform administrator
- `partner` - Partner administrator
- `client` - Client administrator
- `merchant` - Merchant/Vendor
- `agent` - Field agent
- `staff` - Staff member

### Billing Cycle Statuses
- `active` - Cycle is open for billing items
- `closed` - Cycle closed, no more items
- `invoiced` - Invoice generated
- `paid` - Payment received
- `overdue` - Payment overdue
- `cancelled` - Cycle cancelled

### Condition Operators
- `eq` - Equal to
- `neq` - Not equal to
- `gt` - Greater than
- `gte` - Greater than or equal
- `lt` - Less than
- `lte` - Less than or equal
- `in` - Value in array
- `not_in` - Value not in array
- `contains` - String contains
- `between` - Value between range

### Action Types
- `apply_discount` - Reduce price
- `apply_surcharge` - Increase price
- `set_price` - Override to fixed price
- `apply_multiplier` - Multiply price
- `add_fee` - Add additional fee
- `skip` - Skip billing item
