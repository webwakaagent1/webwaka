# CB-4 Inventory Management Capability - Architecture Document

**Date:** January 30, 2026  
**Version:** 1.0.0  
**Status:** Operational

---

## 1. Overview

The CB-4 Inventory Management Capability is a channel-agnostic inventory management service that serves as the **single source of truth** for inventory across all sales channels. It provides real-time stock tracking, multi-location support, inventory strategies (FIFO/LIFO), and channel subscription mechanisms for real-time synchronization.

### 1.1. Core Design Principles

1. **Single Source of Truth**: All channels subscribe to this service for inventory data
2. **Channel-Agnostic**: Works with any sales channel (ecommerce, POS, marketplace, wholesale, API)
3. **Real-Time Synchronization**: Event-driven updates to all subscribed channels
4. **Full Auditability**: Complete audit trail for all inventory movements
5. **Multi-Location Support**: Track inventory across warehouses, stores, and distribution centers

---

## 2. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   CB-4 Inventory Management Capability                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌────────────┐  │
│  │   Products   │  │  Locations   │  │  Inventory   │  │  Channels  │  │
│  │   Routes     │  │   Routes     │  │   Routes     │  │   Routes   │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘  │
│         │                 │                 │                 │         │
│  ┌──────▼─────────────────▼─────────────────▼─────────────────▼──────┐  │
│  │                         Service Layer                             │  │
│  ├────────────────┬────────────────┬────────────────┬───────────────┤  │
│  │ ProductService │ LocationService│ InventoryService│ ChannelService│  │
│  ├────────────────┼────────────────┼────────────────┼───────────────┤  │
│  │  AuditService  │  EventService  │                │               │  │
│  └────────────────┴────────────────┴────────────────┴───────────────┘  │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                       PostgreSQL Database                         │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │  │
│  │  │ cb4_products │ │cb4_locations │ │cb4_stock_    │              │  │
│  │  │              │ │              │ │    levels    │              │  │
│  │  └──────────────┘ └──────────────┘ └──────────────┘              │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │  │
│  │  │cb4_stock_    │ │cb4_stock_    │ │cb4_reserva-  │              │  │
│  │  │  movements   │ │  transfers   │ │    tions     │              │  │
│  │  └──────────────┘ └──────────────┘ └──────────────┘              │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │  │
│  │  │cb4_channels  │ │cb4_channel_  │ │cb4_inventory_│              │  │
│  │  │              │ │subscriptions │ │    events    │              │  │
│  │  └──────────────┘ └──────────────┘ └──────────────┘              │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘

                              ▼ Webhooks ▼
    ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
    │  Ecommerce  │  │     POS     │  │ Marketplace │  │  Wholesale  │
    │   Channel   │  │   Channel   │  │   Channel   │  │   Channel   │
    └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘
```

---

## 3. Data Model

### 3.1. Core Entities

#### Products
```typescript
interface Product {
  id: string;
  tenantId: string;
  sku: string;
  name: string;
  description?: string;
  category?: string;
  unitOfMeasure: string;
  trackInventory: boolean;
  allowNegativeStock: boolean;
  reorderPoint?: number;
  reorderQuantity?: number;
  inventoryStrategy: 'FIFO' | 'LIFO' | 'AVERAGE' | 'SPECIFIC';
  metadata?: Record<string, unknown>;
  isActive: boolean;
}
```

#### Locations
```typescript
interface Location {
  id: string;
  tenantId: string;
  code: string;
  name: string;
  locationType: 'warehouse' | 'store' | 'distribution_center' | 'virtual';
  address?: string;
  isActive: boolean;
  parentLocationId?: string;
}
```

#### Stock Levels
```typescript
interface StockLevel {
  id: string;
  tenantId: string;
  productId: string;
  locationId: string;
  quantityOnHand: number;
  quantityReserved: number;
  quantityAvailable: number;  // onHand - reserved
  quantityInTransit: number;
  lastCountedAt?: Date;
}
```

### 3.2. Database Schema

| Table | Purpose |
|-------|---------|
| `cb4_products` | Product catalog with inventory settings |
| `cb4_locations` | Warehouses, stores, distribution centers |
| `cb4_stock_levels` | Current stock per product/location |
| `cb4_stock_batches` | Batch tracking for FIFO/LIFO |
| `cb4_stock_movements` | Complete movement history |
| `cb4_stock_transfers` | Inter-location transfers |
| `cb4_reservations` | Reserved stock for orders |
| `cb4_channels` | Sales channel definitions |
| `cb4_channel_subscriptions` | Event subscriptions per channel |
| `cb4_inventory_events` | Event log for synchronization |
| `cb4_inventory_audit_log` | Full audit trail |

---

## 4. Inventory Strategies

### 4.1. FIFO (First In, First Out)
Oldest inventory is consumed first. Ideal for perishable goods.

### 4.2. LIFO (Last In, First Out)
Newest inventory is consumed first. Used for cost optimization in certain scenarios.

### 4.3. Average Cost
All units valued at weighted average cost.

### 4.4. Specific Identification
Track and consume specific batches by lot/batch number.

---

## 5. Stock Operations

### 5.1. Receive Stock
```
Receive → Update Stock Level → Record Movement → Emit Event → Check Reorder Point
```

### 5.2. Sell Stock
```
Validate Available → Consume Batch (FIFO/LIFO) → Update Stock → Record Movement → Emit Event
```

### 5.3. Adjust Stock
```
Validate Adjustment → Update Stock Level → Record Movement (increase/decrease) → Emit Event
```

### 5.4. Transfer Stock
```
Validate Source → Decrease Source → Add to In-Transit → Create Transfer Record
                                                              ↓
                          Complete Transfer → Increase Destination → Clear In-Transit
```

### 5.5. Reserve Stock
```
Validate Available → Increase Reserved → Decrease Available → Create Reservation → Emit Event
```

---

## 6. Channel Subscription & Synchronization

### 6.1. Channel Types
- **ecommerce**: Online store platforms
- **pos**: Point of sale systems
- **marketplace**: Third-party marketplaces
- **wholesale**: B2B wholesale channels
- **api**: Direct API integrations

### 6.2. Event Types
| Event | Trigger |
|-------|---------|
| `stock_updated` | Any stock level change |
| `stock_low` | Stock below reorder point |
| `stock_out` | Stock reaches zero |
| `reservation_created` | New reservation |
| `reservation_fulfilled` | Reservation fulfilled |
| `reservation_cancelled` | Reservation cancelled |
| `transfer_initiated` | Transfer started |
| `transfer_completed` | Transfer completed |
| `product_created` | New product added |
| `product_updated` | Product details changed |

### 6.3. Subscription Model
Channels subscribe to:
- Specific products or all products
- Specific locations or all locations
- Specific event types or all events

### 6.4. Webhook Delivery
```json
{
  "eventId": "uuid",
  "eventType": "stock_updated",
  "tenantId": "tenant-123",
  "payload": {
    "productId": "prod-456",
    "locationId": "loc-789",
    "quantityOnHand": 150,
    "quantityAvailable": 125
  },
  "timestamp": "2026-01-30T12:00:00Z"
}
```

---

## 7. API Reference

For complete API documentation, see: **[CB-4 Inventory API Documentation](/docs/api/CB4_INVENTORY_API.md)**

### 7.1. Products Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/products` | Create product |
| GET | `/api/v1/products` | List products |
| GET | `/api/v1/products/:id` | Get product |
| PUT | `/api/v1/products/:id` | Update product |
| DELETE | `/api/v1/products/:id` | Deactivate product |

### 7.2. Locations Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/locations` | Create location |
| GET | `/api/v1/locations` | List locations |
| GET | `/api/v1/locations/:id` | Get location |
| PUT | `/api/v1/locations/:id` | Update location |

### 7.3. Inventory Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/inventory/stock` | List stock levels |
| GET | `/api/v1/inventory/stock/:productId/:locationId` | Get stock level |
| POST | `/api/v1/inventory/receive` | Receive stock |
| POST | `/api/v1/inventory/sell` | Record sale |
| POST | `/api/v1/inventory/adjust` | Adjust stock |
| GET | `/api/v1/inventory/movements` | List movements |
| POST | `/api/v1/inventory/transfers` | Create transfer |
| GET | `/api/v1/inventory/transfers` | List transfers |
| POST | `/api/v1/inventory/transfers/:id/complete` | Complete transfer |
| POST | `/api/v1/inventory/reservations` | Create reservation |
| GET | `/api/v1/inventory/reservations` | List reservations |
| POST | `/api/v1/inventory/reservations/:id/fulfill` | Fulfill reservation |
| POST | `/api/v1/inventory/reservations/:id/cancel` | Cancel reservation |

### 7.4. Channels Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/channels` | Create channel |
| GET | `/api/v1/channels` | List channels |
| GET | `/api/v1/channels/:id` | Get channel |
| PUT | `/api/v1/channels/:id` | Update channel |
| POST | `/api/v1/channels/:channelId/subscriptions` | Create subscription |
| GET | `/api/v1/channels/:channelId/subscriptions` | List subscriptions |
| PATCH | `/api/v1/channels/subscriptions/:id/status` | Update subscription status |
| DELETE | `/api/v1/channels/subscriptions/:id` | Delete subscription |
| GET | `/api/v1/channels/events` | List inventory events |

---

## 8. Platform Invariants

| Invariant | Description | Enforcement |
|-----------|-------------|-------------|
| INV-002 | Tenant Isolation | All operations scoped by tenant_id |

---

## 9. Technology Stack

- **Runtime**: Node.js 20
- **Language**: TypeScript 5.x
- **Framework**: Express.js 4.x
- **Database**: PostgreSQL (Replit Postgres)
- **Decimal Math**: Decimal.js
- **Date Handling**: date-fns
- **UUID**: uuid v9

---

## 10. Related Documents

- [CB-4 API Documentation](/docs/api/CB4_INVENTORY_API.md)
- [CB-4 Operations Runbook](/docs/runbooks/CB4_INVENTORY_OPERATIONS.md)
- [Master Control Board](/docs/governance/WEBWAKA_MASTER_CONTROL_BOARD.md)
