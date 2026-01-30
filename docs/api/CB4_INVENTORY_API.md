# CB-4 Inventory Management - API Documentation

**Version:** 1.0.0  
**Base URL:** `/api/v1`

---

## Authentication Headers

| Header | Required | Description |
|--------|----------|-------------|
| `X-User-Id` | Yes (for writes) | ID of the user performing the action |

---

## Products

### Create Product

**Endpoint:** `POST /products`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "sku": "string (required)",
  "name": "string (required)",
  "description": "string (optional)",
  "category": "string (optional)",
  "unitOfMeasure": "string (default: 'each')",
  "trackInventory": "boolean (default: true)",
  "allowNegativeStock": "boolean (default: false)",
  "reorderPoint": "number (optional)",
  "reorderQuantity": "number (optional)",
  "inventoryStrategy": "FIFO | LIFO | AVERAGE | SPECIFIC (default: FIFO)",
  "metadata": "object (optional)"
}
```

**Response:** `201 Created`

---

### List Products

**Endpoint:** `GET /products`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `category` | string | Filter by category |
| `isActive` | boolean | Filter by active status |
| `trackInventory` | boolean | Filter by inventory tracking |

---

### Get Product

**Endpoint:** `GET /products/:id`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |

---

### Update Product

**Endpoint:** `PUT /products/:id`

**Request Body:** Same as create, with `tenantId` required

---

### Delete Product

**Endpoint:** `DELETE /products/:id`

Soft-deletes (deactivates) the product.

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |

---

## Locations

### Create Location

**Endpoint:** `POST /locations`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "code": "string (required)",
  "name": "string (required)",
  "locationType": "warehouse | store | distribution_center | virtual (required)",
  "address": "string (optional)",
  "parentLocationId": "string (optional)"
}
```

---

### List Locations

**Endpoint:** `GET /locations`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `locationType` | string | Filter by type |
| `isActive` | boolean | Filter by status |
| `parentLocationId` | string | Filter by parent |

---

## Inventory Operations

### Get Stock Levels

**Endpoint:** `GET /inventory/stock`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `productId` | string | Filter by product |
| `locationId` | string | Filter by location |

**Response:**
```json
[
  {
    "id": "uuid",
    "productId": "uuid",
    "locationId": "uuid",
    "quantityOnHand": 100,
    "quantityReserved": 25,
    "quantityAvailable": 75,
    "quantityInTransit": 10,
    "updatedAt": "ISO8601"
  }
]
```

---

### Get Stock Level

**Endpoint:** `GET /inventory/stock/:productId/:locationId`

---

### Receive Stock

**Endpoint:** `POST /inventory/receive`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "productId": "uuid (required)",
  "locationId": "uuid (required)",
  "quantity": "number (required)",
  "costPerUnit": "number (required)",
  "batchNumber": "string (optional)",
  "expiryDate": "ISO8601 (optional)",
  "performedBy": "string (required)",
  "referenceType": "string (optional)",
  "referenceId": "string (optional)"
}
```

**Response:**
```json
{
  "stockLevel": { ... },
  "movement": { ... },
  "batch": { ... }
}
```

---

### Record Sale

**Endpoint:** `POST /inventory/sell`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "productId": "uuid (required)",
  "locationId": "uuid (required)",
  "channelId": "uuid (required)",
  "quantity": "number (required)",
  "performedBy": "string (required)",
  "referenceType": "string (optional)",
  "referenceId": "string (optional)"
}
```

---

### Adjust Stock

**Endpoint:** `POST /inventory/adjust`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "productId": "uuid (required)",
  "locationId": "uuid (required)",
  "quantityChange": "number (required, positive or negative)",
  "reason": "string (required)",
  "performedBy": "string (required)",
  "costPerUnit": "number (optional)"
}
```

---

### List Stock Movements

**Endpoint:** `GET /inventory/movements`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `productId` | string | Filter by product |
| `locationId` | string | Filter by location |
| `movementType` | string | Filter by type |
| `fromDate` | ISO8601 | Start date |
| `toDate` | ISO8601 | End date |

---

## Transfers

### Create Transfer

**Endpoint:** `POST /inventory/transfers`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "productId": "uuid (required)",
  "fromLocationId": "uuid (required)",
  "toLocationId": "uuid (required)",
  "quantity": "number (required)",
  "initiatedBy": "string (required)",
  "notes": "string (optional)"
}
```

---

### List Transfers

**Endpoint:** `GET /inventory/transfers`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `status` | string | pending, in_transit, completed, cancelled |
| `productId` | string | Filter by product |

---

### Complete Transfer

**Endpoint:** `POST /inventory/transfers/:id/complete`

**Request Body:**
```json
{
  "tenantId": "string (required)"
}
```

---

## Reservations

### Create Reservation

**Endpoint:** `POST /inventory/reservations`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "productId": "uuid (required)",
  "locationId": "uuid (required)",
  "channelId": "uuid (required)",
  "quantity": "number (required)",
  "referenceType": "string (required)",
  "referenceId": "string (required)",
  "expiresAt": "ISO8601 (optional)"
}
```

---

### List Reservations

**Endpoint:** `GET /inventory/reservations`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `status` | string | active, fulfilled, cancelled, expired |
| `productId` | string | Filter by product |
| `channelId` | string | Filter by channel |

---

### Fulfill Reservation

**Endpoint:** `POST /inventory/reservations/:id/fulfill`

**Request Body:**
```json
{
  "tenantId": "string (required)"
}
```

---

### Cancel Reservation

**Endpoint:** `POST /inventory/reservations/:id/cancel`

**Request Body:**
```json
{
  "tenantId": "string (required)"
}
```

---

## Channels

### Create Channel

**Endpoint:** `POST /channels`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "code": "string (required)",
  "name": "string (required)",
  "channelType": "ecommerce | pos | marketplace | wholesale | api (required)",
  "webhookUrl": "string (optional)",
  "apiKey": "string (optional)"
}
```

---

### List Channels

**Endpoint:** `GET /channels`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `channelType` | string | Filter by type |
| `isActive` | boolean | Filter by status |

---

### Create Subscription

**Endpoint:** `POST /channels/:channelId/subscriptions`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "productId": "uuid (optional, null for all products)",
  "locationId": "uuid (optional, null for all locations)",
  "eventTypes": ["stock_updated", "stock_low", "stock_out", ...]
}
```

---

### List Subscriptions

**Endpoint:** `GET /channels/:channelId/subscriptions`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `productId` | string | Filter by product |
| `status` | string | active, paused, cancelled |

---

### Update Subscription Status

**Endpoint:** `PATCH /channels/subscriptions/:id/status`

**Request Body:**
```json
{
  "tenantId": "string (required)",
  "status": "active | paused | cancelled"
}
```

---

### List Events

**Endpoint:** `GET /channels/events`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `eventType` | string | Filter by event type |
| `productId` | string | Filter by product |
| `locationId` | string | Filter by location |
| `fromDate` | ISO8601 | Start date |
| `toDate` | ISO8601 | End date |

---

## Audit

### Search Audit Logs

**Endpoint:** `GET /audit`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |
| `entityType` | string | product, location, stock_level, etc. |
| `entityId` | string | Specific entity |
| `action` | string | create, update, etc. |
| `performedBy` | string | User ID |
| `fromDate` | ISO8601 | Start date |
| `toDate` | ISO8601 | End date |

---

### Get Entity History

**Endpoint:** `GET /audit/:entityType/:entityId`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `tenantId` | string | Required |

---

## Data Types Reference

### Inventory Strategies
- `FIFO` - First In, First Out
- `LIFO` - Last In, First Out
- `AVERAGE` - Weighted average cost
- `SPECIFIC` - Specific batch identification

### Location Types
- `warehouse` - Storage facility
- `store` - Retail location
- `distribution_center` - Distribution hub
- `virtual` - Virtual/dropship location

### Channel Types
- `ecommerce` - Online store
- `pos` - Point of sale
- `marketplace` - Third-party marketplace
- `wholesale` - B2B wholesale
- `api` - Direct API integration

### Movement Types
- `receipt` - Stock received
- `sale` - Stock sold
- `transfer_out` - Transfer from location
- `transfer_in` - Transfer to location
- `adjustment_increase` - Positive adjustment
- `adjustment_decrease` - Negative adjustment
- `reservation` - Stock reserved
- `reservation_release` - Reservation cancelled
- `return` - Stock returned
- `write_off` - Stock written off

### Event Types
- `stock_updated` - Stock level changed
- `stock_low` - Below reorder point
- `stock_out` - Zero stock
- `reservation_created` - New reservation
- `reservation_fulfilled` - Reservation completed
- `reservation_cancelled` - Reservation cancelled
- `transfer_initiated` - Transfer started
- `transfer_completed` - Transfer finished
- `product_created` - New product
- `product_updated` - Product changed
