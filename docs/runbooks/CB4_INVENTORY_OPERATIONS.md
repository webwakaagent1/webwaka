# Runbook: CB-4 Inventory Operations

## Overview

This runbook provides operational guidance for the CB-4 Inventory Management Capability.

---

## 1. Setting Up Products and Locations

### Create a Warehouse Location
```bash
curl -X POST http://localhost:5000/api/v1/locations \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -d '{
    "tenantId": "tenant-001",
    "code": "WH-LAGOS",
    "name": "Lagos Main Warehouse",
    "locationType": "warehouse",
    "address": "123 Industrial Estate, Lagos"
  }'
```

### Create a Product with FIFO Strategy
```bash
curl -X POST http://localhost:5000/api/v1/products \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin-123" \
  -d '{
    "tenantId": "tenant-001",
    "sku": "WIDGET-001",
    "name": "Premium Widget",
    "category": "Electronics",
    "unitOfMeasure": "each",
    "trackInventory": true,
    "allowNegativeStock": false,
    "reorderPoint": 50,
    "reorderQuantity": 200,
    "inventoryStrategy": "FIFO"
  }'
```

---

## 2. Receiving Stock

### Receive with Batch Tracking
```bash
curl -X POST http://localhost:5000/api/v1/inventory/receive \
  -H "Content-Type: application/json" \
  -H "X-User-Id: warehouse-staff" \
  -d '{
    "tenantId": "tenant-001",
    "productId": "{product-id}",
    "locationId": "{location-id}",
    "quantity": 500,
    "costPerUnit": 150.00,
    "batchNumber": "BATCH-2026-001",
    "expiryDate": "2027-01-30",
    "performedBy": "warehouse-staff",
    "referenceType": "purchase_order",
    "referenceId": "PO-12345"
  }'
```

---

## 3. Recording Sales

### Record a Sale from Ecommerce
```bash
curl -X POST http://localhost:5000/api/v1/inventory/sell \
  -H "Content-Type: application/json" \
  -H "X-User-Id: system" \
  -d '{
    "tenantId": "tenant-001",
    "productId": "{product-id}",
    "locationId": "{location-id}",
    "channelId": "{ecommerce-channel-id}",
    "quantity": 5,
    "performedBy": "ecommerce-system",
    "referenceType": "order",
    "referenceId": "ORD-98765"
  }'
```

---

## 4. Stock Adjustments

### Positive Adjustment (Found Stock)
```bash
curl -X POST http://localhost:5000/api/v1/inventory/adjust \
  -H "Content-Type: application/json" \
  -H "X-User-Id: supervisor" \
  -d '{
    "tenantId": "tenant-001",
    "productId": "{product-id}",
    "locationId": "{location-id}",
    "quantityChange": 10,
    "reason": "Stock found during audit",
    "performedBy": "supervisor"
  }'
```

### Negative Adjustment (Damaged Stock)
```bash
curl -X POST http://localhost:5000/api/v1/inventory/adjust \
  -H "Content-Type: application/json" \
  -H "X-User-Id: supervisor" \
  -d '{
    "tenantId": "tenant-001",
    "productId": "{product-id}",
    "locationId": "{location-id}",
    "quantityChange": -5,
    "reason": "Water damage during storage",
    "performedBy": "supervisor"
  }'
```

---

## 5. Inter-Location Transfers

### Initiate Transfer
```bash
curl -X POST http://localhost:5000/api/v1/inventory/transfers \
  -H "Content-Type: application/json" \
  -H "X-User-Id: logistics" \
  -d '{
    "tenantId": "tenant-001",
    "productId": "{product-id}",
    "fromLocationId": "{warehouse-id}",
    "toLocationId": "{store-id}",
    "quantity": 50,
    "initiatedBy": "logistics",
    "notes": "Weekly store replenishment"
  }'
```

### Complete Transfer (at Destination)
```bash
curl -X POST http://localhost:5000/api/v1/inventory/transfers/{transfer-id}/complete \
  -H "Content-Type: application/json" \
  -H "X-User-Id: store-manager" \
  -d '{
    "tenantId": "tenant-001"
  }'
```

---

## 6. Reservations

### Create Reservation for Order
```bash
curl -X POST http://localhost:5000/api/v1/inventory/reservations \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "tenant-001",
    "productId": "{product-id}",
    "locationId": "{location-id}",
    "channelId": "{channel-id}",
    "quantity": 3,
    "referenceType": "order",
    "referenceId": "ORD-54321",
    "expiresAt": "2026-02-01T00:00:00Z"
  }'
```

### Fulfill Reservation (Order Shipped)
```bash
curl -X POST http://localhost:5000/api/v1/inventory/reservations/{reservation-id}/fulfill \
  -H "Content-Type: application/json" \
  -H "X-User-Id: fulfillment" \
  -d '{
    "tenantId": "tenant-001"
  }'
```

### Cancel Reservation (Order Cancelled)
```bash
curl -X POST http://localhost:5000/api/v1/inventory/reservations/{reservation-id}/cancel \
  -H "Content-Type: application/json" \
  -H "X-User-Id: customer-service" \
  -d '{
    "tenantId": "tenant-001"
  }'
```

---

## 7. Channel Subscription Setup

### Create Ecommerce Channel
```bash
curl -X POST http://localhost:5000/api/v1/channels \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin" \
  -d '{
    "tenantId": "tenant-001",
    "code": "SHOPIFY-MAIN",
    "name": "Main Shopify Store",
    "channelType": "ecommerce",
    "webhookUrl": "https://mystore.example.com/webhooks/inventory",
    "apiKey": "webhook-secret-key"
  }'
```

### Subscribe to All Stock Updates
```bash
curl -X POST http://localhost:5000/api/v1/channels/{channel-id}/subscriptions \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin" \
  -d '{
    "tenantId": "tenant-001",
    "eventTypes": ["stock_updated", "stock_low", "stock_out"]
  }'
```

### Subscribe to Specific Product
```bash
curl -X POST http://localhost:5000/api/v1/channels/{channel-id}/subscriptions \
  -H "Content-Type: application/json" \
  -H "X-User-Id: admin" \
  -d '{
    "tenantId": "tenant-001",
    "productId": "{product-id}",
    "eventTypes": ["stock_updated"]
  }'
```

---

## 8. Monitoring & Troubleshooting

### Check Current Stock Level
```bash
curl "http://localhost:5000/api/v1/inventory/stock/{product-id}/{location-id}?tenantId=tenant-001"
```

### View Recent Movements
```bash
curl "http://localhost:5000/api/v1/inventory/movements?tenantId=tenant-001&productId={product-id}&limit=50"
```

### View Audit Trail
```bash
curl "http://localhost:5000/api/v1/audit?tenantId=tenant-001&entityType=stock_level&fromDate=2026-01-01"
```

### Check Pending Transfers
```bash
curl "http://localhost:5000/api/v1/inventory/transfers?tenantId=tenant-001&status=pending"
```

### Check Active Reservations
```bash
curl "http://localhost:5000/api/v1/inventory/reservations?tenantId=tenant-001&status=active"
```

---

## 9. Common Issues

### Issue: "Insufficient stock for sale"
**Cause:** Trying to sell more than available quantity.

**Solution:**
1. Check current stock level
2. Check for active reservations reducing available quantity
3. Check for pending transfers

### Issue: "Insufficient stock for reservation"
**Cause:** Available quantity is less than requested reservation.

**Solution:**
1. Check quantityAvailable (not quantityOnHand)
2. Consider cancelling expired reservations

### Issue: Webhook not receiving events
**Cause:** Channel webhook configuration issue.

**Solution:**
1. Verify webhookUrl is correct and accessible
2. Check apiKey is set for signature verification
3. Check subscription status is 'active'
4. Review events table for delivery status
