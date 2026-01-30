# ARCH_PF1_FOUNDATIONAL_EXTENSIONS.md

**WebWaka Platform Foundation Phase 1: Foundational Extensions**

**Version:** 1.0  
**Date:** January 30, 2026  
**Author:** Manus AI  
**Status:** 🟡 In Progress  
**Canonical Reference:** [PF-1: Foundational Extensions](../phases/PF-1_FOUNDATIONAL_EXTENSIONS.md)

---

## 1. Executive Summary

This document provides the complete technical architecture and implementation specification for **PF-1: Foundational Extensions**, the first platform foundation phase of the WebWaka platform. This phase establishes the critical infrastructure primitives required to support stateful compute, instance orchestration, and the Super Admin control plane.

PF-1 is designed to enable all future capabilities, including the "Compile & Deploy" automation model, multi-instance management, and advanced service orchestration, while strictly adhering to the platform's core invariants.

### 1.1. Core Deliverables

1. **Stateful Compute Infrastructure** - Job queue, scheduler, and worker orchestration for long-running background tasks
2. **Instance Orchestration Service** - Multi-instance provisioning, configuration, and lifecycle management
3. **Super Admin Control Plane** - Dedicated, audited control plane for platform operations
4. **Version & Upgrade Control Service** - Platform version management with update channel policies

### 1.2. Platform Invariants Enforced

- **INV-002: Strict Tenant Isolation** - Super Admin control plane is architecturally separated from tenant operations
- **INV-003: Audited Super Admin Access** - All Super Admin actions are logged and auditable
- **INV-004: Layered Dependency Rule** - Foundation-layer primitives with no higher-layer dependencies
- **INV-008: Update Policy as Governed Lifecycle** - Support for all update channel policies and version pinning

---

## 2. Architectural Principles

### 2.1. Design Philosophy

The PF-1 architecture is guided by the following principles:

1. **Separation of Concerns** - Clear boundaries between compute, orchestration, and control plane
2. **Auditability by Default** - All administrative actions are logged and traceable
3. **Graceful Degradation** - System continues to operate even when non-critical components fail
4. **Scalability First** - Designed to handle thousands of instances and millions of jobs
5. **Security in Depth** - Multiple layers of security controls and isolation
6. **Operational Excellence** - Built-in monitoring, alerting, and operational runbooks

### 2.2. Technology Stack

| Component | Technology | Justification |
|-----------|-----------|---------------|
| **Job Queue** | BullMQ + Redis | Battle-tested, supports priority queues, retries, and delayed jobs |
| **Scheduler** | Node-cron + Custom Orchestrator | Flexible scheduling with distributed coordination |
| **Database** | PostgreSQL | ACID compliance, excellent JSON support, mature ecosystem |
| **Message Bus** | Redis Pub/Sub | Low-latency, simple, reliable for internal messaging |
| **API Framework** | Express.js + TypeScript | Type-safe, well-documented, extensive middleware ecosystem |
| **Container Orchestration** | Docker + Docker Compose | Standard containerization, easy local development |
| **Monitoring** | Prometheus + Grafana | Industry standard, powerful querying, beautiful dashboards |
| **Logging** | Winston + Elasticsearch | Structured logging, powerful search, long-term retention |

### 2.3. Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Super Admin Control Plane                │
│  (Isolated Network, Audited Access, Dedicated Database)     │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ Secure API Gateway
                              │
┌─────────────────────────────────────────────────────────────┐
│                  Instance Orchestration Layer                │
│  - Instance Provisioning Service                             │
│  - Configuration Management                                  │
│  - Health Monitoring                                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              │
┌─────────────────────────────────────────────────────────────┐
│                   Stateful Compute Layer                     │
│  - Job Queue (BullMQ)                                        │
│  - Scheduler (Node-cron)                                     │
│  - Worker Pool (Auto-scaling)                                │
└─────────────────────────────────────────────────────────────┘
                              │
                              │
┌─────────────────────────────────────────────────────────────┐
│                    Platform Instances                        │
│  (Tenant-Isolated, Independently Deployable)                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Component Architecture

### 3.1. Stateful Compute Infrastructure

The Stateful Compute Infrastructure provides the foundation for long-running, stateful background jobs and scheduled tasks.

#### 3.1.1. Job Queue System

**Technology:** BullMQ + Redis

**Features:**
- Priority-based job execution
- Automatic retry with exponential backoff
- Job progress tracking
- Delayed job execution
- Job result persistence
- Dead letter queue for failed jobs

**Job Schema:**
```typescript
interface Job {
  id: string;
  type: string;
  priority: number; // 1-10, higher = more urgent
  payload: Record<string, any>;
  status: 'pending' | 'active' | 'completed' | 'failed' | 'delayed';
  attempts: number;
  maxAttempts: number;
  createdAt: Date;
  startedAt?: Date;
  completedAt?: Date;
  failedAt?: Date;
  error?: string;
  result?: any;
  metadata: {
    createdBy: string;
    instanceId: string;
    tenantId?: string;
  };
}
```

**Queue Configuration:**
```typescript
const queueConfig = {
  defaultJobOptions: {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 2000,
    },
    removeOnComplete: false,
    removeOnFail: false,
  },
  settings: {
    stalledInterval: 30000,
    maxStalledCount: 2,
  },
};
```

#### 3.1.2. Scheduler System

**Technology:** Node-cron + Custom Distributed Coordinator

**Features:**
- Cron-based scheduling
- One-time scheduled jobs
- Recurring jobs with flexible intervals
- Timezone-aware scheduling
- Leader election for distributed scheduling
- Schedule persistence and recovery

**Schedule Schema:**
```typescript
interface Schedule {
  id: string;
  name: string;
  description: string;
  cronExpression: string;
  timezone: string;
  jobType: string;
  jobPayload: Record<string, any>;
  enabled: boolean;
  lastRunAt?: Date;
  nextRunAt: Date;
  metadata: {
    createdBy: string;
    instanceId: string;
    tenantId?: string;
  };
}
```

**Scheduler Architecture:**
```
┌──────────────────────────────────────────────────────────┐
│                    Scheduler Coordinator                  │
│  - Leader Election (Redis-based)                          │
│  - Schedule Registry                                      │
│  - Execution Tracker                                      │
└──────────────────────────────────────────────────────────┘
                        │
                        │ Distributes Jobs
                        │
┌──────────────────────────────────────────────────────────┐
│                      Job Queue                            │
│  (BullMQ)                                                 │
└──────────────────────────────────────────────────────────┘
                        │
                        │ Consumes Jobs
                        │
┌──────────────────────────────────────────────────────────┐
│                    Worker Pool                            │
│  - Auto-scaling Workers                                   │
│  - Job Execution Engine                                   │
│  - Result Persistence                                     │
└──────────────────────────────────────────────────────────┘
```

#### 3.1.3. Worker Orchestration

**Features:**
- Auto-scaling based on queue depth
- Worker health monitoring
- Graceful shutdown
- Job timeout enforcement
- Resource limit enforcement

**Worker Configuration:**
```typescript
interface WorkerConfig {
  concurrency: number; // Max concurrent jobs per worker
  minWorkers: number;
  maxWorkers: number;
  scaleUpThreshold: number; // Queue depth to trigger scale-up
  scaleDownThreshold: number; // Queue depth to trigger scale-down
  jobTimeout: number; // Max execution time per job (ms)
  memoryLimit: string; // e.g., "512M"
  cpuLimit: string; // e.g., "1.0"
}
```

---

### 3.2. Instance Orchestration Service

The Instance Orchestration Service manages the lifecycle of platform instances, enabling the "Compile & Deploy" model for self-hosted clients.

#### 3.2.1. Instance Model

**Instance Schema:**
```typescript
interface Instance {
  id: string;
  name: string;
  type: 'shared-saas' | 'partner-deployed' | 'self-hosted-enterprise';
  status: 'provisioning' | 'active' | 'suspended' | 'terminated';
  version: string;
  updateChannel: 'auto-update' | 'manual-approval' | 'frozen';
  configuration: {
    region: string;
    dataResidency: string[];
    enabledSuites: string[];
    enabledCapabilities: string[];
    customDomain?: string;
    sslEnabled: boolean;
  };
  resources: {
    cpu: string;
    memory: string;
    storage: string;
    database: string;
  };
  metadata: {
    createdAt: Date;
    createdBy: string;
    partnerId?: string;
    tenantId?: string;
  };
  health: {
    lastHealthCheck: Date;
    status: 'healthy' | 'degraded' | 'unhealthy';
    uptime: number;
    metrics: Record<string, any>;
  };
}
```

#### 3.2.2. Provisioning Workflow

```
┌──────────────────────────────────────────────────────────┐
│  1. Instance Request                                      │
│     - Validate configuration                              │
│     - Check resource availability                         │
│     - Generate instance ID                                │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  2. Infrastructure Provisioning                           │
│     - Allocate compute resources                          │
│     - Provision database                                  │
│     - Configure networking                                │
│     - Set up storage                                      │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  3. Platform Deployment                                   │
│     - Deploy application containers                       │
│     - Run database migrations                             │
│     - Configure environment variables                     │
│     - Set up SSL certificates                             │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  4. Configuration & Initialization                        │
│     - Enable requested suites                             │
│     - Enable requested capabilities                       │
│     - Configure data residency rules                      │
│     - Set up monitoring & logging                         │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  5. Health Check & Activation                             │
│     - Verify all services are running                     │
│     - Run smoke tests                                     │
│     - Mark instance as active                             │
│     - Send activation notification                        │
└──────────────────────────────────────────────────────────┘
```

#### 3.2.3. Instance Lifecycle Management

**Supported Operations:**
- **Provision** - Create new instance
- **Configure** - Update instance configuration
- **Upgrade** - Update platform version
- **Suspend** - Temporarily disable instance
- **Resume** - Reactivate suspended instance
- **Terminate** - Permanently delete instance
- **Backup** - Create instance backup
- **Restore** - Restore from backup
- **Scale** - Adjust resource allocation
- **Migrate** - Move to different region/infrastructure

#### 3.2.4. Health Monitoring

**Health Check System:**
```typescript
interface HealthCheck {
  instanceId: string;
  timestamp: Date;
  status: 'healthy' | 'degraded' | 'unhealthy';
  checks: {
    database: HealthCheckResult;
    api: HealthCheckResult;
    workers: HealthCheckResult;
    storage: HealthCheckResult;
    network: HealthCheckResult;
  };
  metrics: {
    cpu: number;
    memory: number;
    disk: number;
    requestRate: number;
    errorRate: number;
    latency: number;
  };
}

interface HealthCheckResult {
  status: 'pass' | 'warn' | 'fail';
  message?: string;
  responseTime: number;
}
```

---

### 3.3. Super Admin Control Plane

The Super Admin Control Plane is a dedicated, audited interface for platform-wide operations.

#### 3.3.1. Architecture

**Key Principles:**
- **Isolation** - Separate network, database, and authentication
- **Auditability** - All actions logged with full context
- **Security** - Multi-factor authentication, IP whitelisting, session management
- **Separation** - Architecturally separated from tenant operations

**Control Plane Components:**
```
┌──────────────────────────────────────────────────────────┐
│                  Super Admin API Gateway                  │
│  - Authentication & Authorization                         │
│  - Rate Limiting                                          │
│  - Audit Logging                                          │
│  - IP Whitelisting                                        │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│              Control Plane Service Layer                  │
│  - Platform Configuration Service                         │
│  - Partner Lifecycle Service                              │
│  - Tenant Lifecycle Service                               │
│  - Capability Management Service                          │
│  - Version Control Service                                │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│            Control Plane Database (Isolated)              │
│  - Platform Configuration                                 │
│  - Partner Registry                                       │
│  - Tenant Registry                                        │
│  - Audit Logs                                             │
└──────────────────────────────────────────────────────────┘
```

#### 3.3.2. Audit Logging System

**Audit Log Schema:**
```typescript
interface AuditLog {
  id: string;
  timestamp: Date;
  actor: {
    id: string;
    type: 'super-admin' | 'system';
    email: string;
    ipAddress: string;
    userAgent: string;
  };
  action: string; // e.g., "instance.provision", "partner.create"
  resource: {
    type: string; // e.g., "instance", "partner", "tenant"
    id: string;
    name?: string;
  };
  changes?: {
    before: Record<string, any>;
    after: Record<string, any>;
  };
  result: 'success' | 'failure';
  error?: string;
  metadata: Record<string, any>;
}
```

**Audit Requirements:**
- All Super Admin actions MUST be logged
- Logs MUST be immutable (append-only)
- Logs MUST be retained for minimum 7 years
- Logs MUST include full context (who, what, when, where, why)
- Logs MUST be searchable and exportable

#### 3.3.3. Super Admin API Endpoints

**Platform Configuration:**
```
POST   /api/superadmin/config                    # Update platform configuration
GET    /api/superadmin/config                    # Get platform configuration
GET    /api/superadmin/config/history            # Get configuration history
```

**Partner Lifecycle:**
```
POST   /api/superadmin/partners                  # Create partner
GET    /api/superadmin/partners                  # List partners
GET    /api/superadmin/partners/:id              # Get partner details
PATCH  /api/superadmin/partners/:id              # Update partner
DELETE /api/superadmin/partners/:id              # Delete partner
POST   /api/superadmin/partners/:id/suspend      # Suspend partner
POST   /api/superadmin/partners/:id/resume       # Resume partner
```

**Tenant Lifecycle:**
```
POST   /api/superadmin/tenants                   # Create tenant
GET    /api/superadmin/tenants                   # List tenants
GET    /api/superadmin/tenants/:id               # Get tenant details
PATCH  /api/superadmin/tenants/:id               # Update tenant
DELETE /api/superadmin/tenants/:id               # Delete tenant
POST   /api/superadmin/tenants/:id/suspend       # Suspend tenant
POST   /api/superadmin/tenants/:id/resume        # Resume tenant
```

**Instance Management:**
```
POST   /api/superadmin/instances                 # Provision instance
GET    /api/superadmin/instances                 # List instances
GET    /api/superadmin/instances/:id             # Get instance details
PATCH  /api/superadmin/instances/:id             # Update instance
DELETE /api/superadmin/instances/:id             # Terminate instance
POST   /api/superadmin/instances/:id/suspend     # Suspend instance
POST   /api/superadmin/instances/:id/resume      # Resume instance
POST   /api/superadmin/instances/:id/upgrade     # Upgrade instance
POST   /api/superadmin/instances/:id/backup      # Backup instance
POST   /api/superadmin/instances/:id/restore     # Restore instance
```

**Capability Management:**
```
POST   /api/superadmin/capabilities              # Register capability
GET    /api/superadmin/capabilities              # List capabilities
PATCH  /api/superadmin/capabilities/:id          # Update capability
POST   /api/superadmin/instances/:id/capabilities/:capId/enable   # Enable capability
POST   /api/superadmin/instances/:id/capabilities/:capId/disable  # Disable capability
```

**Audit Logs:**
```
GET    /api/superadmin/audit-logs                # Query audit logs
GET    /api/superadmin/audit-logs/:id            # Get audit log details
POST   /api/superadmin/audit-logs/export         # Export audit logs
```

---

### 3.4. Version & Upgrade Control Service

The Version & Upgrade Control Service manages platform versions, suite versions, and update channel policies.

#### 3.4.1. Version Model

**Version Schema:**
```typescript
interface Version {
  id: string;
  number: string; // Semantic versioning: major.minor.patch
  type: 'platform' | 'suite' | 'capability';
  releaseDate: Date;
  status: 'draft' | 'beta' | 'stable' | 'deprecated';
  changelog: string;
  breaking: boolean;
  securityPatch: boolean;
  dependencies: {
    [key: string]: string; // dependency: version
  };
  artifacts: {
    docker: string[]; // Docker image tags
    migrations: string[]; // Database migration scripts
    config: string[]; // Configuration files
  };
  metadata: {
    author: string;
    approvedBy?: string;
    approvalDate?: Date;
  };
}
```

#### 3.4.2. Update Channel Policies

**Supported Update Channels:**

1. **Auto-Update**
   - Automatically apply all stable releases
   - Security patches applied immediately
   - Non-breaking updates applied during maintenance window
   - Breaking updates require explicit approval

2. **Manual-Approval**
   - All updates require explicit approval
   - Security patches flagged for urgent review
   - Update notifications sent to administrators
   - Approval workflow with rollback plan

3. **Frozen**
   - No automatic updates
   - Only critical security patches (with override capability)
   - Explicit version pinning
   - Long-term support (LTS) mode

**Update Channel Configuration:**
```typescript
interface UpdateChannelConfig {
  instanceId: string;
  channel: 'auto-update' | 'manual-approval' | 'frozen';
  pinnedVersion?: string;
  maintenanceWindow?: {
    dayOfWeek: number; // 0-6 (Sunday-Saturday)
    startTime: string; // HH:MM
    duration: number; // minutes
    timezone: string;
  };
  autoApprovalRules?: {
    securityPatches: boolean;
    minorUpdates: boolean;
    patchUpdates: boolean;
  };
  notifications: {
    email: string[];
    slack?: string;
  };
}
```

#### 3.4.3. Upgrade Workflow

```
┌──────────────────────────────────────────────────────────┐
│  1. Version Release                                       │
│     - Publish new version                                 │
│     - Generate changelog                                  │
│     - Tag artifacts                                       │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  2. Instance Eligibility Check                            │
│     - Check update channel policy                         │
│     - Verify compatibility                                │
│     - Check dependencies                                  │
│     - Determine upgrade path                              │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  3. Approval (if required)                                │
│     - Send notification                                   │
│     - Wait for approval                                   │
│     - Validate rollback plan                              │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  4. Pre-Upgrade Preparation                               │
│     - Create backup                                       │
│     - Run pre-upgrade checks                              │
│     - Allocate resources                                  │
│     - Schedule maintenance window                         │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  5. Upgrade Execution                                     │
│     - Put instance in maintenance mode                    │
│     - Pull new artifacts                                  │
│     - Run database migrations                             │
│     - Update configuration                                │
│     - Deploy new version                                  │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  6. Post-Upgrade Validation                               │
│     - Run health checks                                   │
│     - Run smoke tests                                     │
│     - Verify data integrity                               │
│     - Monitor error rates                                 │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  7. Completion or Rollback                                │
│     - If successful: Mark upgrade complete                │
│     - If failed: Execute rollback                         │
│     - Send notification                                   │
│     - Update audit logs                                   │
└──────────────────────────────────────────────────────────┘
```

#### 3.4.4. Rollback Mechanism

**Rollback Capabilities:**
- Automatic rollback on critical failures
- Manual rollback trigger
- Database migration rollback
- Configuration rollback
- Artifact version rollback

**Rollback Procedure:**
```typescript
interface RollbackProcedure {
  instanceId: string;
  fromVersion: string;
  toVersion: string;
  reason: string;
  steps: RollbackStep[];
  estimatedDuration: number; // minutes
  dataLossRisk: 'none' | 'low' | 'medium' | 'high';
}

interface RollbackStep {
  order: number;
  action: string;
  description: string;
  reversible: boolean;
  estimatedDuration: number; // seconds
}
```

---

## 4. Data Models

### 4.1. Database Schema

**Core Tables:**

```sql
-- Platform Configuration
CREATE TABLE platform_config (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  key VARCHAR(255) NOT NULL UNIQUE,
  value JSONB NOT NULL,
  description TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_by VARCHAR(255) NOT NULL
);

-- Partners
CREATE TABLE partners (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  status VARCHAR(50) NOT NULL DEFAULT 'active',
  configuration JSONB NOT NULL DEFAULT '{}',
  metadata JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  created_by VARCHAR(255) NOT NULL
);

-- Tenants
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  partner_id UUID REFERENCES partners(id),
  name VARCHAR(255) NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'active',
  configuration JSONB NOT NULL DEFAULT '{}',
  metadata JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  created_by VARCHAR(255) NOT NULL
);

-- Instances
CREATE TABLE instances (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  type VARCHAR(50) NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'provisioning',
  version VARCHAR(50) NOT NULL,
  update_channel VARCHAR(50) NOT NULL DEFAULT 'manual-approval',
  configuration JSONB NOT NULL DEFAULT '{}',
  resources JSONB NOT NULL DEFAULT '{}',
  metadata JSONB NOT NULL DEFAULT '{}',
  health JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  created_by VARCHAR(255) NOT NULL
);

-- Versions
CREATE TABLE versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  number VARCHAR(50) NOT NULL,
  type VARCHAR(50) NOT NULL,
  release_date TIMESTAMP NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'draft',
  changelog TEXT,
  breaking BOOLEAN NOT NULL DEFAULT FALSE,
  security_patch BOOLEAN NOT NULL DEFAULT FALSE,
  dependencies JSONB NOT NULL DEFAULT '{}',
  artifacts JSONB NOT NULL DEFAULT '{}',
  metadata JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Jobs
CREATE TABLE jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type VARCHAR(255) NOT NULL,
  priority INTEGER NOT NULL DEFAULT 5,
  payload JSONB NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
  attempts INTEGER NOT NULL DEFAULT 0,
  max_attempts INTEGER NOT NULL DEFAULT 3,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  failed_at TIMESTAMP,
  error TEXT,
  result JSONB,
  metadata JSONB NOT NULL DEFAULT '{}'
);

-- Schedules
CREATE TABLE schedules (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  cron_expression VARCHAR(255) NOT NULL,
  timezone VARCHAR(100) NOT NULL DEFAULT 'UTC',
  job_type VARCHAR(255) NOT NULL,
  job_payload JSONB NOT NULL,
  enabled BOOLEAN NOT NULL DEFAULT TRUE,
  last_run_at TIMESTAMP,
  next_run_at TIMESTAMP NOT NULL,
  metadata JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Audit Logs
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  timestamp TIMESTAMP NOT NULL DEFAULT NOW(),
  actor JSONB NOT NULL,
  action VARCHAR(255) NOT NULL,
  resource JSONB NOT NULL,
  changes JSONB,
  result VARCHAR(50) NOT NULL,
  error TEXT,
  metadata JSONB NOT NULL DEFAULT '{}'
);

-- Indexes
CREATE INDEX idx_instances_status ON instances(status);
CREATE INDEX idx_instances_type ON instances(type);
CREATE INDEX idx_jobs_status ON jobs(status);
CREATE INDEX idx_jobs_type ON jobs(type);
CREATE INDEX idx_jobs_created_at ON jobs(created_at);
CREATE INDEX idx_schedules_enabled ON schedules(enabled);
CREATE INDEX idx_schedules_next_run_at ON schedules(next_run_at);
CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_audit_logs_actor ON audit_logs USING GIN(actor);
```

---

## 5. API Specifications

### 5.1. Authentication & Authorization

**Authentication Method:** JWT (JSON Web Tokens)

**Authorization Levels:**
- **Super Admin** - Full platform access
- **Partner Admin** - Partner-scoped access
- **Tenant Admin** - Tenant-scoped access
- **System** - Internal service-to-service

**JWT Payload:**
```typescript
interface JWTPayload {
  sub: string; // User ID
  email: string;
  role: 'super-admin' | 'partner-admin' | 'tenant-admin' | 'system';
  scope: string[]; // Permissions
  partnerId?: string;
  tenantId?: string;
  exp: number; // Expiration timestamp
  iat: number; // Issued at timestamp
}
```

### 5.2. Error Handling

**Standard Error Response:**
```typescript
interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: Record<string, any>;
    requestId: string;
    timestamp: string;
  };
}
```

**Error Codes:**
- `UNAUTHORIZED` - Authentication required
- `FORBIDDEN` - Insufficient permissions
- `NOT_FOUND` - Resource not found
- `VALIDATION_ERROR` - Invalid input
- `CONFLICT` - Resource conflict
- `INTERNAL_ERROR` - Internal server error
- `SERVICE_UNAVAILABLE` - Service temporarily unavailable

### 5.3. Rate Limiting

**Rate Limits:**
- Super Admin API: 1000 requests/minute
- Partner API: 500 requests/minute
- Tenant API: 200 requests/minute

**Rate Limit Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1643723400
```

---

## 6. Security Architecture

### 6.1. Security Layers

1. **Network Security**
   - VPC isolation
   - Security groups
   - Network ACLs
   - DDoS protection

2. **Application Security**
   - Input validation
   - Output encoding
   - CSRF protection
   - SQL injection prevention
   - XSS prevention

3. **Authentication & Authorization**
   - Multi-factor authentication (MFA)
   - JWT-based authentication
   - Role-based access control (RBAC)
   - IP whitelisting
   - Session management

4. **Data Security**
   - Encryption at rest (AES-256)
   - Encryption in transit (TLS 1.3)
   - Database encryption
   - Secrets management (HashiCorp Vault)

5. **Audit & Compliance**
   - Comprehensive audit logging
   - Log immutability
   - Compliance reporting
   - Incident response procedures

### 6.2. Secrets Management

**Secrets Storage:** HashiCorp Vault

**Secret Types:**
- Database credentials
- API keys
- Encryption keys
- SSL certificates
- Service tokens

**Secret Rotation:**
- Automatic rotation every 90 days
- Manual rotation on demand
- Zero-downtime rotation
- Audit trail for all rotations

---

## 7. Monitoring & Observability

### 7.1. Metrics

**System Metrics:**
- CPU usage
- Memory usage
- Disk usage
- Network I/O
- Database connections

**Application Metrics:**
- Request rate
- Error rate
- Response time (p50, p95, p99)
- Job queue depth
- Job processing rate
- Job failure rate

**Business Metrics:**
- Active instances
- Provisioning rate
- Upgrade success rate
- Audit log volume

### 7.2. Logging

**Log Levels:**
- ERROR - Error events
- WARN - Warning events
- INFO - Informational events
- DEBUG - Debug events

**Log Format:**
```json
{
  "timestamp": "2026-01-30T12:34:56.789Z",
  "level": "INFO",
  "service": "instance-orchestration",
  "message": "Instance provisioned successfully",
  "context": {
    "instanceId": "inst_123",
    "version": "1.0.0",
    "duration": 45000
  },
  "requestId": "req_456",
  "userId": "user_789"
}
```

### 7.3. Alerting

**Alert Channels:**
- Email
- Slack
- PagerDuty
- SMS

**Alert Conditions:**
- High error rate (> 5%)
- High latency (p95 > 1000ms)
- Job queue depth (> 10000)
- Instance health check failure
- Database connection failure
- Disk usage (> 80%)

---

## 8. Operational Procedures

### 8.1. Instance Provisioning Runbook

**Prerequisites:**
- Valid partner/tenant credentials
- Resource availability confirmed
- Configuration validated

**Steps:**
1. Validate provisioning request
2. Allocate infrastructure resources
3. Provision database
4. Deploy application containers
5. Run database migrations
6. Configure environment
7. Enable suites and capabilities
8. Run health checks
9. Activate instance
10. Send notification

**Estimated Duration:** 5-10 minutes

**Rollback Procedure:** Terminate instance and release resources

### 8.2. Upgrade Management Runbook

**Prerequisites:**
- Backup completed
- Maintenance window scheduled
- Rollback plan prepared

**Steps:**
1. Create instance backup
2. Put instance in maintenance mode
3. Pull new artifacts
4. Run database migrations
5. Update configuration
6. Deploy new version
7. Run health checks
8. Run smoke tests
9. Exit maintenance mode
10. Monitor for issues

**Estimated Duration:** 15-30 minutes

**Rollback Procedure:** Execute rollback to previous version

### 8.3. Incident Response Runbook

**Severity Levels:**
- **P0 (Critical)** - Platform-wide outage
- **P1 (High)** - Multiple instances affected
- **P2 (Medium)** - Single instance affected
- **P3 (Low)** - Minor issue, no user impact

**Response Procedures:**
1. Detect and acknowledge incident
2. Assess severity and impact
3. Assemble response team
4. Investigate root cause
5. Implement fix or mitigation
6. Verify resolution
7. Communicate status
8. Document incident
9. Conduct post-mortem

---

## 9. Testing Strategy

### 9.1. Unit Tests

**Coverage Target:** 80%

**Test Framework:** Jest

**Test Categories:**
- Service logic
- Data validation
- Error handling
- Utility functions

### 9.2. Integration Tests

**Test Scenarios:**
- API endpoint functionality
- Database operations
- Job queue operations
- Scheduler operations
- Authentication & authorization

### 9.3. End-to-End Tests

**Test Scenarios:**
- Instance provisioning workflow
- Upgrade workflow
- Rollback workflow
- Audit logging
- Health monitoring

### 9.4. Performance Tests

**Test Scenarios:**
- API load testing (1000 req/s)
- Job queue throughput (10000 jobs/min)
- Database query performance
- Concurrent instance provisioning

---

## 10. Deployment Strategy

### 10.1. Deployment Environments

1. **Development** - Local development environment
2. **Staging** - Pre-production environment
3. **Production** - Live production environment

### 10.2. Deployment Process

**CI/CD Pipeline:**
```
┌──────────────────────────────────────────────────────────┐
│  1. Code Commit                                           │
│     - Push to Git repository                              │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  2. Automated Tests                                       │
│     - Unit tests                                          │
│     - Integration tests                                   │
│     - Linting & formatting                                │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  3. Build Artifacts                                       │
│     - Build Docker images                                 │
│     - Tag with version                                    │
│     - Push to container registry                          │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  4. Deploy to Staging                                     │
│     - Pull latest images                                  │
│     - Run database migrations                             │
│     - Deploy services                                     │
│     - Run smoke tests                                     │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  5. Manual Approval                                       │
│     - Review staging results                              │
│     - Approve production deployment                       │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│  6. Deploy to Production                                  │
│     - Blue-green deployment                               │
│     - Run database migrations                             │
│     - Deploy services                                     │
│     - Run health checks                                   │
│     - Switch traffic                                      │
└──────────────────────────────────────────────────────────┘
```

### 10.3. Rollback Strategy

**Automated Rollback Triggers:**
- Health check failure rate > 10%
- Error rate > 5%
- Response time p95 > 2000ms

**Manual Rollback:**
- One-click rollback button
- Rollback to previous version
- Automatic traffic switch

---

## 11. Architecture Decision Records (ADRs)

### ADR-001: Choice of Job Queue Technology

**Status:** Accepted

**Context:** Need a reliable, scalable job queue system for stateful compute.

**Decision:** Use BullMQ with Redis.

**Rationale:**
- Battle-tested in production
- Excellent TypeScript support
- Built-in retry logic and delayed jobs
- Good monitoring and observability
- Active community and maintenance

**Consequences:**
- Redis becomes a critical dependency
- Need to manage Redis cluster for high availability
- Memory usage scales with queue depth

### ADR-002: Database Choice for Control Plane

**Status:** Accepted

**Context:** Need a database for the Super Admin control plane.

**Decision:** Use PostgreSQL.

**Rationale:**
- ACID compliance for critical data
- Excellent JSON support for flexible schemas
- Mature ecosystem and tooling
- Strong security features
- Proven scalability

**Consequences:**
- Need to manage database backups
- Need to plan for database scaling
- Need to implement connection pooling

### ADR-003: Audit Log Immutability

**Status:** Accepted

**Context:** Audit logs must be tamper-proof for compliance.

**Decision:** Use append-only table with database-level write restrictions.

**Rationale:**
- Simple to implement
- Leverages database guarantees
- No additional infrastructure required
- Easy to query and export

**Consequences:**
- Audit log table will grow indefinitely
- Need to implement archival strategy
- Need to monitor storage usage

### ADR-004: Update Channel Policy Design

**Status:** Accepted

**Context:** Need to support different update preferences for different clients.

**Decision:** Implement three update channels: auto-update, manual-approval, frozen.

**Rationale:**
- Balances automation with control
- Supports different risk tolerances
- Enables long-term support (LTS) model
- Allows emergency security patches

**Consequences:**
- Need to maintain multiple version branches
- Increased testing complexity
- Need to communicate version support lifecycle

### ADR-005: Instance Isolation Model

**Status:** Accepted

**Context:** Need to ensure strict tenant isolation (INV-002).

**Decision:** Each instance gets dedicated database and isolated network.

**Rationale:**
- Strongest isolation guarantee
- Simplifies data residency compliance
- Enables instance-level scaling
- Reduces blast radius of failures

**Consequences:**
- Higher infrastructure costs
- More complex orchestration
- Need to manage more databases

---

## 12. Future Enhancements

### 12.1. Planned Improvements

1. **Multi-Region Support**
   - Deploy instances across multiple regions
   - Automatic failover
   - Data replication

2. **Advanced Monitoring**
   - Distributed tracing
   - Application performance monitoring (APM)
   - Real-time dashboards

3. **Self-Service Portal**
   - Partner self-service instance provisioning
   - Usage analytics
   - Billing integration

4. **Automated Scaling**
   - Auto-scaling based on load
   - Predictive scaling
   - Cost optimization

5. **Disaster Recovery**
   - Automated backups
   - Point-in-time recovery
   - Cross-region replication

### 12.2. Research Areas

1. **Kubernetes Migration**
   - Evaluate Kubernetes for instance orchestration
   - Compare with current Docker-based approach
   - Plan migration strategy

2. **Event-Driven Architecture**
   - Evaluate event streaming platforms
   - Design event-driven workflows
   - Plan integration with PF-2 (Realtime)

3. **AI-Powered Operations**
   - Anomaly detection
   - Predictive maintenance
   - Automated incident response

---

## 13. Compliance & Governance

### 13.1. Data Residency Compliance

**Supported Modes:**
- Single-Country (Nigeria-only)
- Regional (West Africa)
- Hybrid (Primary + Backup regions)
- Fully Sovereign (Client-owned infrastructure)

**Implementation:**
- Data classification at creation
- Configurable residency rules
- Cross-border transfer logging
- Audit trail for compliance

### 13.2. Security Compliance

**Standards:**
- ISO 27001
- SOC 2 Type II
- GDPR (for EU clients)
- NDPR (Nigeria Data Protection Regulation)

**Controls:**
- Access control
- Encryption
- Audit logging
- Incident response
- Business continuity

---

## 14. Glossary

| Term | Definition |
|------|------------|
| **Instance** | A deployed, isolated platform environment for a partner or tenant |
| **Super Admin** | Platform-level administrator with cross-tenant access |
| **Partner** | Organization that deploys WebWaka for their clients |
| **Tenant** | End customer of a partner |
| **Update Channel** | Policy that determines how and when updates are applied |
| **Stateful Compute** | Long-running background jobs that maintain state |
| **Job Queue** | System for managing asynchronous job execution |
| **Scheduler** | System for executing jobs on a schedule |
| **Audit Log** | Immutable record of all administrative actions |
| **Control Plane** | Administrative interface for platform management |

---

## 15. References

### 15.1. Related Documents

- [PF-1: Foundational Extensions](../phases/PF-1_FOUNDATIONAL_EXTENSIONS.md) - Phase definition
- [WebWaka Master Control Board](../governance/WEBWAKA_MASTER_CONTROL_BOARD.md) - Governance document
- [Capability Dependency Maps](./ARCH_00_CAPABILITY_DEPENDENCY_MAPS.md) - Architecture overview

### 15.2. External Resources

- [BullMQ Documentation](https://docs.bullmq.io/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

---

## 16. Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-30 | Manus AI | Initial architecture document |

---

**End of Document**
