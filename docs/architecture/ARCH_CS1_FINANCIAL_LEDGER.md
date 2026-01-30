# ARCH_CS1_FINANCIAL_LEDGER.md

**Architecture Document: CS-1 Financial Ledger Service**

**Version:** 1.0  
**Date:** January 30, 2026  
**Author:** Manus AI  
**Status:** Implementation Complete

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Overview](#system-overview)
3. [Architectural Principles](#architectural-principles)
4. [Data Model](#data-model)
5. [Double-Entry Accounting Engine](#double-entry-accounting-engine)
6. [API Design](#api-design)
7. [Security Architecture](#security-architecture)
8. [Storage Strategy](#storage-strategy)
9. [Query and Reporting](#query-and-reporting)
10. [Integration Points](#integration-points)
11. [Deployment Architecture](#deployment-architecture)
12. [Testing Strategy](#testing-strategy)

---

## 1. Executive Summary

The Financial Ledger Service provides a centralized, immutable, append-only ledger that serves as the single source of truth for all financial transactions across the WebWaka platform. The service implements double-entry accounting principles, supports multiple actor types, enforces strict tenant isolation, and provides comprehensive query and reporting capabilities.

The architecture emphasizes financial correctness, auditability, immutability, and compliance with accounting standards while maintaining high performance and scalability. The ledger is designed to support complex multi-level revenue flows including the Multi-Level Affiliate System while remaining agnostic to specific pricing or commission calculation logic.

---

## 2. System Overview

### 2.1 Core Responsibilities

The Financial Ledger Service is responsible for recording all financial transactions across the platform, maintaining account balances, ensuring double-entry accounting integrity, providing transaction history and reporting capabilities, and supporting audit and compliance requirements.

### 2.2 Actor Types

The service supports six distinct actor types within the WebWaka ecosystem. The **Platform** represents the WebWaka platform itself, which earns platform fees and pays operational expenses. **Partners** are organizations that deploy WebWaka instances and earn revenue from their clients. **Clients** are businesses using the platform through partners, paying subscription fees and earning revenue from their operations. **Merchants/Vendors** are businesses selling products or services through client instances. **Agents** are individuals in the MLAS hierarchy earning commissions from referrals. **End Users** are consumers making purchases or payments through the platform.

### 2.3 Transaction Types

The ledger supports comprehensive transaction types covering all financial events. **Sale** transactions represent revenue from product or service sales. **Refund** transactions reverse previous sale transactions. **Commission** transactions record affiliate or agent commission earnings. **Payout** transactions represent money transfers to external accounts. **Fee** transactions capture platform, payment gateway, or service fees. **Adjustment** transactions handle corrections, write-offs, or manual adjustments.

### 2.4 Key Features

The service provides an immutable append-only ledger where all transactions are permanent and corrections are made via adjustment transactions. The double-entry accounting engine ensures every transaction has balanced debits and credits with automatic validation. Multi-actor support enables tracking of complex revenue flows across all actor types. Strict tenant isolation guarantees complete data separation with row-level security. The comprehensive audit trail maintains complete transaction history with actor tracking and metadata. Query and reporting APIs provide flexible querying with aggregation, filtering, and export capabilities.

---

## 3. Architectural Principles

### 3.1 Immutability

All ledger entries are append-only and immutable. Once a transaction is recorded, it cannot be modified or deleted. Corrections are made through adjustment transactions that create new entries. This ensures complete auditability and prevents retroactive tampering with financial records.

### 3.2 Double-Entry Accounting

The service implements standard double-entry bookkeeping principles where every transaction affects at least two accounts with equal and opposite entries. The sum of all debits must always equal the sum of all credits. This fundamental principle ensures financial integrity and enables standard accounting practices.

### 3.3 Tenant Isolation (INV-002)

Ledger entries are strictly isolated by tenant identifier. All queries include tenant context and row-level security enforcement prevents cross-tenant data access. This architectural separation ensures compliance with the platform's strict tenant isolation invariant.

### 3.4 Audited Access (INV-003)

All access to ledger data is logged with actor identification, timestamp, and operation details. Super Admin access to tenant financial data is explicitly audited with justification and approval tracking. This ensures accountability and supports compliance requirements.

### 3.5 MLAS Foundation (INV-006)

The ledger provides the foundational data for MLAS commission calculations while remaining agnostic to specific commission logic. Transaction records include all necessary metadata for attribution tracking without embedding commission calculation rules. This separation maintains the principle that MLAS is infrastructure, not policy.

---

## 4. Data Model

### 4.1 Core Entities

#### Accounts Table

The accounts table defines the chart of accounts for the platform. Each account has a unique identifier, tenant identifier for isolation, account code following standard accounting conventions, descriptive account name, account type (Asset, Liability, Equity, Revenue, Expense), currency code, current balance maintained through triggers, parent account identifier for hierarchical organization, and active status flag.

#### Ledger Entries Table

The ledger_entries table stores all financial transactions in an immutable, append-only structure. Each entry includes a unique identifier, tenant identifier, transaction identifier linking related entries, entry timestamp with microsecond precision, account identifier, actor type and identifier, transaction type, amount with precision to handle fractional currency units, currency code, debit or credit indicator, reference to external transaction, description and metadata, and created timestamp.

#### Transactions Table

The transactions table groups related ledger entries into logical transactions. Each transaction record contains a unique identifier, tenant identifier, transaction type, transaction timestamp, total amount, currency code, status (pending, completed, failed, reversed), reference to external system, description, metadata in JSONB format, actor information, and audit timestamps.

#### Account Balances Table

The account_balances table maintains current and historical balances for efficient querying. Records include account identifier, tenant identifier, balance amount, currency code, effective date, balance type (current, historical), and audit timestamps.

### 4.2 Entity Relationships

Transactions have a one-to-many relationship with ledger entries, where each transaction creates multiple balanced entries. Accounts have a hierarchical relationship with parent accounts for organizational structure. Ledger entries reference accounts and transactions through foreign keys. Account balances are derived from ledger entries but cached for performance.

### 4.3 Data Integrity Constraints

The data model enforces several critical integrity constraints. Tenant isolation is enforced through row-level security policies on all tables. Double-entry validation ensures the sum of debits equals the sum of credits for each transaction. Immutability is guaranteed through database triggers preventing updates or deletes on ledger entries. Currency consistency requires all entries in a transaction to use the same currency. Balance reconciliation is validated through periodic checks comparing account balances to ledger entry sums.

---

## 5. Double-Entry Accounting Engine

### 5.1 Core Principles

The double-entry accounting engine is the heart of the financial ledger service. Every financial transaction creates at least two ledger entries with equal and opposite effects. The fundamental equation Assets = Liabilities + Equity must always hold true. Debits increase assets and expenses while decreasing liabilities, equity, and revenue. Credits increase liabilities, equity, and revenue while decreasing assets and expenses.

### 5.2 Transaction Recording Process

When a transaction is recorded, the system follows a strict validation and recording process. First, transaction validation ensures all required fields are present, amounts are positive, currency codes are valid, and actor types are recognized. Next, account validation verifies that all referenced accounts exist, belong to the correct tenant, are active, and have compatible currency settings.

The double-entry generation phase creates balanced ledger entries based on transaction type. For example, a sale transaction debits Cash (Asset) and credits Revenue (Revenue). A commission transaction debits Commission Expense (Expense) and credits Commission Payable (Liability). Balance validation ensures the sum of debits equals the sum of credits before committing.

Finally, atomic commit writes all entries in a single database transaction, updates account balances through triggers, records audit trail, and returns transaction confirmation.

### 5.3 Account Types and Normal Balances

The system supports five standard account types with their normal balances. **Asset** accounts have a normal debit balance and include cash, accounts receivable, and inventory. **Liability** accounts have a normal credit balance and include accounts payable, commissions payable, and loans. **Equity** accounts have a normal credit balance and include owner's equity and retained earnings. **Revenue** accounts have a normal credit balance and include sales revenue, commission revenue, and service fees. **Expense** accounts have a normal debit balance and include cost of goods sold, commission expense, and operating expenses.

### 5.4 Transaction Templates

The system provides predefined transaction templates for common operations. A **Sale Transaction** debits Cash/Accounts Receivable and credits Sales Revenue. A **Refund Transaction** debits Sales Returns and credits Cash/Accounts Receivable. A **Commission Transaction** debits Commission Expense and credits Commission Payable. A **Payout Transaction** debits Commission Payable and credits Cash. A **Platform Fee Transaction** debits Fee Expense and credits Platform Revenue. An **Adjustment Transaction** uses appropriate accounts based on the nature of the correction.

---

## 6. API Design

### 6.1 Transaction Recording API

#### POST /api/v1/ledger/transactions

This endpoint records a new financial transaction with automatic double-entry generation. The request body includes tenant identifier, transaction type, amount, currency, actor information, reference to external transaction, description, and metadata. The response returns the created transaction with all generated ledger entries, updated account balances, and transaction confirmation.

Authentication requires JWT token with appropriate permissions. Authorization enforces tenant-level access control. Validation ensures double-entry balance, currency consistency, and account existence. Idempotency is supported through transaction reference to prevent duplicate recording.

#### POST /api/v1/ledger/transactions/batch

This endpoint records multiple transactions in a single atomic operation. The request accepts an array of transaction objects. The response returns all created transactions or rolls back entirely on any failure. This endpoint is critical for high-volume operations and ensures atomicity across multiple related transactions.

### 6.2 Query API

#### GET /api/v1/ledger/transactions

This endpoint retrieves transaction history with filtering and pagination. Query parameters include tenant identifier, date range filters, transaction type filter, actor type and identifier filters, amount range filters, pagination parameters, and sort order. The response returns an array of transactions with embedded ledger entries, pagination metadata, and summary statistics.

#### GET /api/v1/ledger/transactions/:id

This endpoint retrieves a specific transaction by identifier. The response includes complete transaction details, all related ledger entries, account information, and audit trail.

#### GET /api/v1/ledger/accounts/:id/balance

This endpoint retrieves current balance for a specific account. Query parameters include effective date for historical balance and currency for multi-currency accounts. The response returns current balance, currency, last transaction date, and balance history.

#### GET /api/v1/ledger/accounts/:id/entries

This endpoint retrieves all ledger entries for a specific account. Query parameters include date range, pagination, and sort order. The response returns an array of ledger entries with transaction context and running balance.

### 6.3 Reporting API

#### GET /api/v1/ledger/reports/balance-sheet

This endpoint generates a balance sheet report for a specific date. Query parameters include tenant identifier, effective date, and currency. The response returns assets, liabilities, equity, and verification that Assets = Liabilities + Equity.

#### GET /api/v1/ledger/reports/income-statement

This endpoint generates an income statement for a date range. Query parameters include tenant identifier, start and end dates, and currency. The response returns revenue, expenses, and net income.

#### GET /api/v1/ledger/reports/transaction-summary

This endpoint generates transaction summary statistics. Query parameters include tenant identifier, date range, group by dimensions, and filters. The response returns aggregated transaction counts, amounts, and breakdowns by type, actor, or other dimensions.

#### POST /api/v1/ledger/reports/export

This endpoint exports ledger data in various formats (CSV, JSON, Excel). The request body specifies report type, filters, format, and delivery method. The response returns a download link or initiates asynchronous export with status tracking.

### 6.4 Admin API

#### GET /api/v1/ledger/admin/audit-log

This endpoint retrieves audit log for Super Admin access. Query parameters include date range, actor filter, and operation type. The response returns complete audit trail with justification and approval tracking. This endpoint is restricted to Super Admin users and all access is logged per INV-003.

#### POST /api/v1/ledger/admin/reconcile

This endpoint triggers balance reconciliation across all accounts. The request body specifies tenant identifier and reconciliation date. The response returns reconciliation status, discrepancies found, and corrective actions taken.

---

## 7. Security Architecture

### 7.1 Authentication

All API endpoints require JWT token authentication. Tokens include tenant identifier, user identifier, role, and permissions. Token expiration is enforced with refresh token support. Multi-factor authentication is required for Super Admin operations.

### 7.2 Authorization

Role-based access control defines permissions for different user types. **Tenant Admin** can read and write ledger entries for their tenant. **Tenant User** can read ledger entries for their tenant. **Super Admin** can read all ledger data with explicit audit logging. **System Service** can write ledger entries for automated processes.

Tenant isolation is enforced at the database level through row-level security policies. All queries automatically filter by tenant context. Cross-tenant queries are explicitly prohibited and logged as security violations.

### 7.3 Audit Logging

All ledger operations are logged with actor identification, timestamp, operation type, affected resources, and operation result. Super Admin access to tenant data requires justification and approval tracking. Audit logs are immutable and retained for seven years. Suspicious access patterns trigger alerts for security review.

### 7.4 Data Encryption

Data at rest is encrypted using AES-256 encryption. Data in transit uses TLS 1.3 for all API communications. Sensitive fields such as account numbers and amounts are encrypted with additional field-level encryption. Encryption keys are managed through a key management service with rotation policies.

---

## 8. Storage Strategy

### 8.1 Database Selection

PostgreSQL 15+ is selected as the primary database for its robust support for ACID transactions, row-level security, JSONB for flexible metadata, excellent performance for financial data, and mature ecosystem with proven reliability.

### 8.2 Schema Design

The schema is optimized for append-only operations with minimal indexes on write-heavy tables. Partitioning is implemented by tenant and date range for large-scale deployments. Materialized views cache complex aggregations for reporting performance. Database triggers maintain account balances and enforce immutability constraints.

### 8.3 Immutability Enforcement

Database triggers prevent UPDATE and DELETE operations on ledger_entries table. Application-level checks provide additional validation before database operations. Audit logs record all attempted modifications. Backup and recovery procedures preserve complete transaction history.

### 8.4 Performance Optimization

Indexes are strategically placed on tenant_id, account_id, transaction_id, created_at, and transaction_type. Query optimization uses EXPLAIN ANALYZE to identify slow queries. Connection pooling manages database connections efficiently. Read replicas handle reporting and analytics queries to reduce load on primary database.

### 8.5 Scalability

Horizontal partitioning by tenant enables distribution across multiple database instances. Time-based partitioning archives historical data to separate storage. Caching layers reduce database load for frequently accessed data. Asynchronous processing handles high-volume batch operations.

---

## 9. Query and Reporting

### 9.1 Query Patterns

Common query patterns include transaction history by date range, account balance at specific date, transactions by actor, transactions by type, and aggregated summaries by various dimensions. The system optimizes these patterns through appropriate indexing and materialized views.

### 9.2 Reporting Capabilities

The reporting system generates standard financial reports including balance sheets, income statements, cash flow statements, and trial balances. Custom reports support flexible filtering, grouping, and aggregation. Export formats include CSV, JSON, Excel, and PDF. Scheduled reports can be configured for automatic generation and delivery.

### 9.3 Real-Time vs. Batch

Real-time queries provide up-to-the-second account balances and transaction status. Batch reports run during off-peak hours for complex aggregations. Materialized views refresh periodically to balance freshness and performance. Users can choose between real-time and cached data based on their needs.

---

## 10. Integration Points

### 10.1 Payment Gateways

The ledger integrates with payment gateways through webhook callbacks. When a payment is processed, the gateway sends a webhook notification. The ledger service validates the webhook signature, records the transaction with payment gateway reference, updates account balances, and sends confirmation to the originating system.

### 10.2 MLAS Commission System (CB-1)

The ledger provides transaction data for MLAS commission calculations. Commission calculations query the ledger for qualifying transactions. Calculated commissions are recorded back to the ledger as commission transactions. The ledger maintains the audit trail for commission attribution without implementing commission logic.

### 10.3 Pricing Engine (CS-4)

The pricing engine determines transaction amounts based on pricing rules. The ledger records transactions with amounts provided by the pricing engine. The ledger does not implement pricing logic but maintains pricing metadata in transaction records for audit purposes.

### 10.4 Reporting and Analytics (CB-2)

The reporting and analytics capability queries the ledger for financial data. The ledger provides optimized query APIs and materialized views for analytics. Complex aggregations can be performed on read replicas to avoid impacting transactional performance.

### 10.5 Notification Service (CS-2)

The ledger emits events for significant financial transactions. The notification service subscribes to these events and sends notifications to relevant actors. Event types include transaction recorded, payout completed, balance threshold reached, and reconciliation discrepancy detected.

---

## 11. Deployment Architecture

### 11.1 Service Components

The deployment includes an API server handling HTTP requests and business logic, a database cluster with primary and read replicas, a cache layer using Redis for frequently accessed data, a message queue for asynchronous processing, and a monitoring and alerting system.

### 11.2 High Availability

The system achieves high availability through database replication with automatic failover, load balancing across multiple API server instances, health checks and automatic recovery, and disaster recovery with point-in-time recovery capability.

### 11.3 Monitoring

Comprehensive monitoring tracks API response times and error rates, database performance metrics, transaction volume and patterns, account balance reconciliation status, and security events and audit log anomalies. Alerts are configured for critical issues requiring immediate attention.

### 11.4 Backup and Recovery

The backup strategy includes continuous database replication, daily full backups with seven-year retention, hourly incremental backups, and transaction log archival for point-in-time recovery. Recovery procedures are tested regularly to ensure reliability.

---

## 12. Testing Strategy

### 12.1 Unit Tests

Unit tests cover double-entry accounting logic, transaction validation rules, account balance calculations, currency conversion logic, and error handling and edge cases. Tests use mocked dependencies to isolate component behavior.

### 12.2 Integration Tests

Integration tests verify end-to-end transaction recording flows, API endpoint functionality, database transaction integrity, tenant isolation enforcement, and integration with external services. Tests use a dedicated test database with realistic data.

### 12.3 Performance Tests

Performance tests measure transaction recording throughput, query response times under load, concurrent transaction handling, database scalability limits, and cache effectiveness. Load testing simulates production traffic patterns.

### 12.4 Security Tests

Security tests validate authentication and authorization, tenant isolation enforcement, SQL injection prevention, API rate limiting, and audit logging completeness. Penetration testing is performed regularly by security specialists.

### 12.5 Compliance Tests

Compliance tests ensure double-entry balance validation, immutability enforcement, audit trail completeness, data retention policies, and regulatory reporting requirements. Tests verify adherence to accounting standards and financial regulations.

---

## Appendices

### Appendix A: Transaction Flow Diagrams

Detailed flow diagrams illustrate the complete lifecycle of various transaction types from initiation through validation, recording, balance updates, and confirmation.

### Appendix B: Database Schema

Complete database schema definitions include all tables, columns, indexes, constraints, triggers, and row-level security policies.

### Appendix C: API Reference

Comprehensive API reference documentation covers all endpoints with request/response schemas, authentication requirements, error codes, and usage examples.

### Appendix D: Accounting Standards

Reference documentation on double-entry accounting principles, chart of accounts structure, and compliance with generally accepted accounting principles.

### Appendix E: Glossary

Definitions of key terms including ledger, transaction, account, debit, credit, balance, tenant, actor, and immutability.

---

**Document Version History**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | January 30, 2026 | Manus AI | Initial architecture document |

**End of Document**
