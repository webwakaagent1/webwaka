# CB-3: Content Management Capability - Architecture Documentation

**Version:** 1.0  
**Date:** January 30, 2026  
**Status:** 🟢 Complete  

## Overview

CB-3 Content Management Capability provides a reusable CMS engine that allows partners and clients to create, manage, and publish content across the WebWaka platform. This capability delivers a flexible content model, media management, versioning, workflow support, and localization.

## Architecture Decisions

### ADR-001: Schema-Driven Content Types

**Decision:** Content types are defined as JSON schemas with typed fields, stored in the database.

**Rationale:**
- Partners can define custom content structures without code changes
- Field validation is performed at runtime based on schema
- Supports content type evolution without migrations
- Enables content modeling UI for non-technical users

**Consequences:**
- Validation logic is more complex than static types
- Field type changes require careful migration planning

### ADR-002: Version-First Content Management

**Decision:** Every content change creates a new version, with full rollback support.

**Rationale:**
- Complete audit trail of all content changes
- Safe editing with ability to restore any previous state
- Supports collaboration with conflict resolution
- Enables content preview of specific versions

**Consequences:**
- Increased storage requirements (all versions stored)
- Query complexity for current version vs. version history

### ADR-003: Pluggable Workflow Engine

**Decision:** Workflows are configurable per tenant with system defaults.

**Rationale:**
- Partners can customize approval processes
- Supports compliance requirements (editorial review, legal approval)
- System-provided workflows reduce setup friction
- Tenant isolation ensures workflow independence

**Consequences:**
- Workflow engine adds complexity
- Step configuration requires careful validation

### ADR-004: File-Based Media Storage with CDN-Ready URLs

**Decision:** Media assets stored on filesystem with CDN-compatible URL structure.

**Rationale:**
- Simple implementation suitable for development and small deployments
- URL structure supports CDN integration for production
- Tenant-isolated storage directories
- Easy migration to object storage (S3, GCS) later

**Consequences:**
- Horizontal scaling requires shared storage or object storage migration
- Backup strategy needed for media files

## Component Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    CB-3 Content Management                       │
├─────────────────────────────────────────────────────────────────┤
│  API Layer (Express.js)                                          │
│  ├── /api/v1/content-types  - Content type definitions          │
│  ├── /api/v1/content        - Content item CRUD + versioning    │
│  ├── /api/v1/media          - Media asset management            │
│  ├── /api/v1/locales        - Locale configuration              │
│  └── /api/v1/workflows      - Workflow management               │
├─────────────────────────────────────────────────────────────────┤
│  Service Layer                                                   │
│  ├── ContentTypeService     - Schema-driven content types       │
│  ├── ContentItemService     - Content CRUD with versioning      │
│  ├── MediaService           - File upload and organization      │
│  ├── LocaleService          - Multi-language support            │
│  └── WorkflowService        - Publishing workflows              │
├─────────────────────────────────────────────────────────────────┤
│  Data Layer (PostgreSQL)                                         │
│  ├── content_types          - Content schema definitions        │
│  ├── content_items          - Content instances                 │
│  ├── content_versions       - Version history                   │
│  ├── media_assets           - File metadata                     │
│  ├── media_folders          - Folder organization               │
│  ├── locales                - Language configurations           │
│  ├── workflow_definitions   - Workflow templates                │
│  └── workflow_instances     - Active workflows                  │
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
- Custom content types per tenant
- Custom workflows per tenant
- Custom locales per tenant
- No WebWaka intervention required for content management

### INV-007: Data Residency Governance

Content storage respects data residency:
- Media files organized by tenant directory
- Content data scoped to tenant-specific rows
- Supports future multi-region deployment
- No cross-border data movement without explicit configuration

### INV-011: Prompts-as-Artifacts

Implementation follows PaA governance:
- Execution initiated via canonical prompt in CB-3 phase document
- All artifacts committed to repository
- Architecture documentation created and linked

## Database Schema

### Content Types
```sql
content_types (
  id UUID PRIMARY KEY,
  tenant_id VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL,
  description TEXT,
  fields JSONB NOT NULL,        -- Array of field definitions
  is_system BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP,
  updated_at TIMESTAMP,
  UNIQUE(tenant_id, slug)
)
```

### Content Items
```sql
content_items (
  id UUID PRIMARY KEY,
  tenant_id VARCHAR(255) NOT NULL,
  content_type_id UUID REFERENCES content_types(id),
  slug VARCHAR(500) NOT NULL,
  title VARCHAR(500) NOT NULL,
  status VARCHAR(50) DEFAULT 'draft',  -- draft, in_review, approved, published, archived
  data JSONB NOT NULL,                 -- Field values
  localized_data JSONB NOT NULL,       -- {locale_code: {field_values}}
  author_id VARCHAR(255) NOT NULL,
  published_at TIMESTAMP,
  published_version INTEGER,
  current_version INTEGER DEFAULT 1,
  metadata JSONB,
  created_at TIMESTAMP,
  updated_at TIMESTAMP,
  UNIQUE(tenant_id, content_type_id, slug)
)
```

### Content Versions
```sql
content_versions (
  id UUID PRIMARY KEY,
  content_item_id UUID REFERENCES content_items(id),
  version INTEGER NOT NULL,
  data JSONB NOT NULL,
  localized_data JSONB NOT NULL,
  author_id VARCHAR(255) NOT NULL,
  change_log TEXT,
  created_at TIMESTAMP,
  UNIQUE(content_item_id, version)
)
```

## Field Types

| Type | Description | Validation |
|------|-------------|------------|
| `text` | Single-line text | Max length, pattern |
| `richtext` | Multi-line formatted text | Max length |
| `number` | Numeric value | Min, max, integer |
| `boolean` | True/false | - |
| `date` | Date only | Min, max date |
| `datetime` | Date and time | Min, max datetime |
| `select` | Single choice from options | Options list |
| `multiselect` | Multiple choices | Options list |
| `media` | Reference to media asset | Media type filter |
| `reference` | Reference to other content | Content type filter |
| `json` | Arbitrary JSON data | JSON schema |

## Workflow System

### Default Workflows (System-Provided)

1. **Standard Publishing** (draft → review → publish)
   - Draft: Initial authoring
   - Review: Editorial review with 1 approver
   - Publish: Auto-transition on approval

2. **Direct Publishing** (draft → publish)
   - Draft: Authoring
   - Publish: Direct publish without review

3. **Editorial Review** (draft → editorial → approval → publish)
   - Draft: Initial authoring
   - Editorial Review: Content review
   - Approval: Final sign-off
   - Publish: Auto-transition on approval

### Workflow States

| Status | Description |
|--------|-------------|
| `pending` | Workflow created but not started |
| `in_progress` | Workflow actively being processed |
| `completed` | All steps approved, content published |
| `rejected` | Workflow rejected, content returned to draft |

## API Endpoints

### Content Types
- `POST /api/v1/content-types` - Create content type
- `GET /api/v1/content-types?tenantId=` - List content types
- `GET /api/v1/content-types/:id?tenantId=` - Get content type
- `PATCH /api/v1/content-types/:id?tenantId=` - Update content type
- `DELETE /api/v1/content-types/:id?tenantId=` - Delete content type

### Content Items
- `POST /api/v1/content` - Create content item
- `GET /api/v1/content?tenantId=` - List content items
- `GET /api/v1/content/:id?tenantId=` - Get content item
- `PATCH /api/v1/content/:id?tenantId=&authorId=` - Update content item
- `DELETE /api/v1/content/:id?tenantId=` - Delete content item
- `POST /api/v1/content/:id/publish?tenantId=` - Publish content
- `POST /api/v1/content/:id/archive?tenantId=` - Archive content
- `GET /api/v1/content/:id/versions?tenantId=` - Get version history
- `POST /api/v1/content/:id/rollback?tenantId=&authorId=` - Rollback to version
- `POST /api/v1/content/:id/workflow/start?tenantId=` - Start workflow
- `GET /api/v1/content/:id/workflow?tenantId=` - Get active workflow

### Media
- `POST /api/v1/media/upload?tenantId=` - Upload media asset
- `GET /api/v1/media?tenantId=` - List media assets
- `GET /api/v1/media/:id?tenantId=` - Get media asset
- `PATCH /api/v1/media/:id?tenantId=` - Update media metadata
- `DELETE /api/v1/media/:id?tenantId=` - Delete media asset
- `POST /api/v1/media/folders` - Create folder
- `GET /api/v1/media/folders/list?tenantId=` - List folders
- `DELETE /api/v1/media/folders/:id?tenantId=` - Delete folder

### Locales
- `POST /api/v1/locales` - Create locale
- `GET /api/v1/locales?tenantId=` - List locales
- `GET /api/v1/locales/default?tenantId=` - Get default locale
- `GET /api/v1/locales/:id?tenantId=` - Get locale
- `PATCH /api/v1/locales/:id?tenantId=` - Update locale
- `DELETE /api/v1/locales/:id?tenantId=` - Delete locale

### Workflows
- `POST /api/v1/workflows` - Create workflow definition
- `GET /api/v1/workflows?tenantId=` - List workflow definitions
- `GET /api/v1/workflows/default?tenantId=` - Get default workflow
- `GET /api/v1/workflows/:id?tenantId=` - Get workflow definition
- `GET /api/v1/workflows/instances/:id?tenantId=` - Get workflow instance
- `POST /api/v1/workflows/instances/:id/approve?tenantId=` - Approve/reject step
- `POST /api/v1/workflows/instances/:id/comment?tenantId=` - Add comment
- `POST /api/v1/workflows/instances/:id/cancel?tenantId=` - Cancel workflow

## Dependencies

- **PF-1 (Foundational Extensions):** Database configuration and shared utilities
- **PostgreSQL:** Primary data storage
- **Express.js:** HTTP API framework

## Future Enhancements

1. **CDN Integration:** Replace file storage with S3/GCS for production scalability
2. **Full-Text Search:** Elasticsearch/Meilisearch integration for content search
3. **Scheduled Publishing:** Publish content at scheduled times
4. **Content Relationships:** Rich linking between content items
5. **SEO Metadata:** Built-in SEO field support
6. **A/B Testing:** Content variant testing

---

**Backlink:** [CB-3 Phase Document](/docs/phases/CB-3_CONTENT_MANAGEMENT_CAPABILITY.md)
