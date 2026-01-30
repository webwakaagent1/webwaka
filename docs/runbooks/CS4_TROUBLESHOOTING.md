# Runbook: Troubleshooting Pricing & Billing Issues

## Overview

This runbook provides guidance for diagnosing and resolving common issues in the CS-4 Pricing & Billing Service.

---

## 1. Price Calculation Issues

### Issue: "No applicable pricing model found"

**Cause:** No pricing scope matches the request parameters.

**Solution:**
1. Check if a scope exists for the given scopeType:
```bash
curl "http://localhost:5000/api/v1/pricing/scopes?tenantId={tenant-id}&scopeType={scope-type}"
```

2. Create a scope if missing:
```bash
curl -X POST http://localhost:5000/api/v1/pricing/scopes \
  -H "Content-Type: application/json" \
  -d '{
    "tenantId": "{tenant-id}",
    "pricingModelId": "{model-id}",
    "scopeType": "global"
  }'
```

### Issue: Unexpected price calculated

**Diagnosis:**
1. Check the calculation response for `breakdown`, `appliedRules`, and `appliedOverrides`
2. Verify the pricing model config:
```bash
curl "http://localhost:5000/api/v1/pricing/models/{model-id}?tenantId={tenant-id}"
```

3. Check active rules:
```bash
curl "http://localhost:5000/api/v1/pricing/models/{model-id}/rules?tenantId={tenant-id}"
```

4. Check active overrides:
```bash
curl "http://localhost:5000/api/v1/pricing/overrides?tenantId={tenant-id}&pricingModelId={model-id}&isActive=true"
```

---

## 2. Override Issues

### Issue: Override not being applied

**Checklist:**
1. Is the override approved?
```json
{
  "approvedBy": "admin-123",  // Must not be null
  "approvedAt": "2026-01-30T10:00:00Z"
}
```

2. Is the override active?
```json
{
  "isActive": true
}
```

3. Is the current date within the effective range?
```json
{
  "effectiveFrom": "2026-01-01",  // Must be <= today
  "effectiveTo": "2026-12-31"    // Must be >= today or null
}
```

### Issue: Cannot approve override

**Cause:** Override may already be approved.

**Solution:** Check current state:
```bash
curl "http://localhost:5000/api/v1/pricing/overrides?tenantId={tenant-id}&pricingModelId={model-id}"
```

---

## 3. Billing Cycle Issues

### Issue: Cannot close billing cycle

**Cause:** Cycle is not in `active` status.

**Solution:**
1. Check current status:
```bash
curl "http://localhost:5000/api/v1/billing/cycles/{cycle-id}?tenantId={tenant-id}"
```

2. Only `active` cycles can be closed.

### Issue: Cannot add items to cycle

**Possible Causes:**
1. Cycle is not `active`
2. Pricing model not found
3. Invalid scopeType

**Solution:** Verify the cycle status and pricing model exist.

---

## 4. Audit & Reversal Issues

### Issue: Cannot reverse action

**Possible Causes:**
1. Action marked as not reversible:
```json
{
  "isReversible": false
}
```

2. Action already reversed:
```json
{
  "reversedBy": "user-123",
  "reversedAt": "2026-01-30T15:00:00Z"
}
```

3. No previous state to restore:
```json
{
  "previousState": null
}
```

**Solution:** Check audit log entry:
```bash
curl "http://localhost:5000/api/v1/billing/audit/{entity-type}/{entity-id}?tenantId={tenant-id}"
```

---

## 5. Permission Issues

### Issue: "Only Super Admin can modify system pricing models"

**Cause:** Attempting to modify a system model without super_admin role.

**Solution:**
1. Use super_admin role header:
```bash
-H "X-User-Role: super_admin"
```

2. Or create a non-system model for customization.

---

## 6. Database Issues

### Issue: Database connection errors

**Diagnosis:**
1. Check health endpoint:
```bash
curl http://localhost:5000/health
```

2. Verify DATABASE_URL environment variable is set.

3. Check PostgreSQL logs for connection issues.

### Issue: Migration failures

**Solution:**
1. Restart the service to re-run migrations
2. Check for conflicting schema changes

---

## 7. Common Validation Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "tenantId is required" | Missing tenant ID in request | Add tenantId to request body or query |
| "Missing required fields" | Incomplete request body | Check API documentation for required fields |
| "Pricing model not found" | Invalid model ID | Verify model exists for tenant |
| "Invalid model type" | Unsupported model type | Use: flat, usage_based, tiered, subscription, revenue_share, hybrid |

---

## 8. Viewing Full Audit Trail

### Search all actions for an entity
```bash
curl "http://localhost:5000/api/v1/billing/audit/pricing_model/{model-id}?tenantId={tenant-id}"
```

### Search by actor
```bash
curl "http://localhost:5000/api/v1/billing/audit?tenantId={tenant-id}&actorId={user-id}"
```

### Search by date range
```bash
curl "http://localhost:5000/api/v1/billing/audit?tenantId={tenant-id}&fromDate=2026-01-01&toDate=2026-01-31"
```

---

## 9. Getting Help

If issues persist:
1. Check service logs for detailed error messages
2. Verify all platform invariants are being enforced (INV-001, INV-002)
3. Review the architecture documentation: `/docs/architecture/ARCH_CS4_PRICING_BILLING.md`
