# WebWaka PF-1 Foundational Extensions

## Overview

WebWaka Platform Foundation Phase 1 - Foundational Extensions. This is a Node.js/TypeScript backend API that provides infrastructure for stateful compute, instance orchestration, and the Super Admin control plane.

## Project Structure

```
implementations/pf1-foundational-extensions/
├── src/
│   ├── config/         # Database and Redis configuration
│   ├── models/         # Data models (Instance, Job, AuditLog)
│   ├── services/       # Business logic services
│   ├── utils/          # Logger and utilities
│   └── index.ts        # Express API entry point
├── migrations/         # SQL migration files
├── tests/              # Unit and integration tests
└── package.json
```

## Technology Stack

- **Runtime**: Node.js 20
- **Language**: TypeScript 5.x
- **Framework**: Express.js 4.x
- **Database**: PostgreSQL (Replit Postgres)
- **Cache/Queue**: Redis (optional - not configured in Replit)
- **Job Queue**: BullMQ (requires Redis)

## Running the Application

The application runs on port 5000 with the following endpoints:

- `GET /` - API info
- `GET /health` - Health check
- `GET /api/v1/instances` - List instances
- `GET /api/v1/instances/:id` - Get instance by ID
- `GET /api/v1/audit-logs` - List audit logs

## Environment Variables

The application uses Replit's built-in environment variables:
- `DATABASE_URL` - PostgreSQL connection string (auto-configured)
- `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE` - Individual PG config

Optional Redis configuration:
- `REDIS_URL` or `REDIS_HOST`/`REDIS_PORT` - Redis connection (disabled by default)
- `REDIS_ENABLED=false` - Explicitly disable Redis

## Development

```bash
cd implementations/pf1-foundational-extensions
npm install
npm run dev
```

## Deployment

The application is configured for autoscale deployment:
- Build: `npm run build` (compiles TypeScript)
- Run: `node dist/index.js`
