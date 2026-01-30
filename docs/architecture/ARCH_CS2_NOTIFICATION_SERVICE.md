# CS-2: Notification Service - Architecture Documentation

**Version:** 1.0.0  
**Date:** January 30, 2026  
**Status:** 🟢 **Complete**

---

## 1. Overview

The CS-2 Notification Service provides a unified, multi-channel notification infrastructure for the WebWaka platform. It enables all platform components to send notifications via email, SMS, and push channels with support for templating, localization, delivery tracking, and user preferences.

---

## 2. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CS-2 Notification Service                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────────┐    ┌───────────────────┐    ┌───────────────────┐   │
│  │   REST API Layer  │───▶│ Notification Svc  │───▶│ Channel Providers │   │
│  │  (Express.js)     │    │  (Queue + Retry)  │    │  Email/SMS/Push   │   │
│  └───────────────────┘    └───────────────────┘    └───────────────────┘   │
│           │                        │                        │               │
│           ▼                        ▼                        ▼               │
│  ┌───────────────────┐    ┌───────────────────┐    ┌───────────────────┐   │
│  │  Template Service │    │ Preference Service│    │ Delivery Service  │   │
│  │  (Handlebars)     │    │ (User Settings)   │    │ (Tracking/Stats)  │   │
│  └───────────────────┘    └───────────────────┘    └───────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
                          ┌───────────────────────┐
                          │      PostgreSQL       │
                          │  (Notifications DB)   │
                          └───────────────────────┘
```

---

## 3. Core Components

### 3.1 Notification Service

The central orchestrator that:
- Receives notification requests via API
- Resolves templates and renders content
- Checks user preferences and quiet hours
- Routes to appropriate channel providers
- Handles retries with exponential backoff
- Tracks delivery status

### 3.2 Template Service

Manages notification templates with:
- Handlebars-based templating engine
- Dynamic variable substitution
- Custom helpers (formatCurrency, formatDate, uppercase, etc.)
- Multi-locale support
- Template versioning per tenant

### 3.3 Preference Service

User preference management:
- Per-channel enable/disable
- Notification frequency (realtime, daily, weekly, never)
- Quiet hours configuration
- Timezone-aware scheduling

### 3.4 Delivery Service

Delivery tracking and analytics:
- Status tracking (pending → sent → delivered)
- Open tracking (for email)
- Click tracking
- Delivery statistics and reporting

### 3.5 Channel Providers

Abstracted channel implementations:

| Channel | Providers Supported | Status |
|---------|---------------------|--------|
| Email   | SMTP, SendGrid, SES, Mailgun | ✅ Implemented |
| SMS     | Twilio, Africa's Talking, Termii | ✅ Implemented |
| Push    | Firebase, OneSignal | ✅ Implemented |
| WhatsApp| (Placeholder) | 🔜 Planned |

---

## 4. Data Model

### 4.1 Tables

```sql
notification_templates    -- Template definitions
user_preferences         -- User notification settings
notifications           -- Notification queue and history
delivery_logs           -- Delivery tracking logs
```

### 4.2 Key Relationships

- Templates are tenant-scoped and channel-specific
- User preferences are unique per tenant/user/channel
- Notifications reference templates (optional)
- Delivery logs track each notification attempt

---

## 5. API Endpoints

### 5.1 Notifications

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/notifications` | Send notification |
| GET | `/api/v1/notifications` | List notifications |
| GET | `/api/v1/notifications/:id` | Get notification |
| DELETE | `/api/v1/notifications/:id` | Cancel notification |
| GET | `/api/v1/notifications/stats` | Get statistics |
| GET | `/api/v1/notifications/:id/delivery-logs` | Get delivery logs |
| POST | `/api/v1/notifications/:id/track/open` | Track open |
| POST | `/api/v1/notifications/:id/track/click` | Track click |

### 5.2 Templates

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/templates` | Create template |
| GET | `/api/v1/templates` | List templates |
| GET | `/api/v1/templates/:id` | Get template |
| GET | `/api/v1/templates/by-slug/:slug` | Get by slug |
| PUT | `/api/v1/templates/:id` | Update template |
| DELETE | `/api/v1/templates/:id` | Delete template |
| POST | `/api/v1/templates/preview` | Preview render |

### 5.3 Preferences

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/preferences` | Create/update preference |
| GET | `/api/v1/preferences/user/:userId` | Get user preferences |
| GET | `/api/v1/preferences/:id` | Get preference |
| PUT | `/api/v1/preferences/:id` | Update preference |
| DELETE | `/api/v1/preferences/:id` | Delete preference |
| POST | `/api/v1/preferences/bulk` | Bulk update |

---

## 6. Architecture Decisions

### ADR-001: Channel Abstraction

**Decision:** Use abstract base provider class with channel-specific implementations.

**Rationale:** Enables easy addition of new channels without modifying core logic.

**Consequences:** Slightly more complex initial setup, but highly extensible.

### ADR-002: Template Engine

**Decision:** Use Handlebars for templating.

**Rationale:** 
- Well-established, widely used
- Supports custom helpers
- Secure by default (escapes HTML)
- Easy to learn for non-developers

### ADR-003: Retry Strategy

**Decision:** Exponential backoff with max 3 retries.

**Rationale:** Handles transient failures without overwhelming providers.

**Configuration:**
- Base delay: 5 seconds
- Max retries: 3 (configurable)
- Backoff multiplier: 2x

### ADR-004: Nigeria-First Providers

**Decision:** Prioritize Nigerian SMS providers (Termii, Africa's Talking) alongside global providers.

**Rationale:** 
- Cost optimization for Nigerian market
- Better delivery rates locally
- Regulatory compliance (NCC)

### ADR-005: Tenant Isolation

**Decision:** All queries include tenant_id as mandatory filter.

**Rationale:** INV-002 compliance - strict tenant isolation.

**Implementation:** Database constraints and application-level enforcement.

---

## 7. Invariant Compliance

| Invariant | Compliance | Notes |
|-----------|------------|-------|
| INV-002: Tenant Isolation | ✅ | All operations scoped by tenant_id |
| INV-010: Realtime Optional | ✅ | Async by design, realtime is best-effort |
| INV-011: PaA Compliance | ✅ | Execution follows governance model |

---

## 8. Standard Templates

Five standard templates included:

1. **welcome** - Welcome message for new users
2. **password-reset** - Password reset instructions
3. **order-confirmation** - Order confirmation receipt
4. **payment-receipt** - Payment receipt notification
5. **system-alert** - System alerts and warnings

Each template available for email, SMS (where applicable), and push channels.

---

## 9. Configuration

### Environment Variables

```env
# Database
DATABASE_URL=postgresql://...

# Email (SMTP)
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=user
SMTP_PASSWORD=password
EMAIL_FROM_ADDRESS=noreply@webwaka.com
EMAIL_FROM_NAME=WebWaka

# SMS
SMS_PROVIDER=termii|twilio|africastalking
SMS_API_KEY=...
SMS_SENDER_ID=WebWaka

# Push
PUSH_PROVIDER=firebase|onesignal
FIREBASE_SERVER_KEY=...

# Retry Configuration
NOTIFICATION_MAX_RETRIES=3
NOTIFICATION_RETRY_DELAY_MS=5000
```

---

## 10. Security Considerations

- All API endpoints protected by rate limiting
- Helmet.js for HTTP security headers
- Input validation with Joi
- Template sandboxing via Handlebars
- No raw SQL - parameterized queries only

---

## 11. Performance Characteristics

- Async notification processing
- Connection pooling for database
- Lazy loading of channel providers
- In-memory template caching

---

## 12. Link to Phase Document

- **Phase Document:** [CS-2: Notification Service](../phases/CS-2_NOTIFICATION_SERVICE.md)
- **Master Control Board:** [§7.3 CS-2](../governance/WEBWAKA_MASTER_CONTROL_BOARD.md#cs-2-notification-service)

---

**End of Document**
