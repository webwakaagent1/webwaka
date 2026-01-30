# CS-4: Pricing & Billing Service

**Date:** January 30, 2026  
**Status:** 🟡 **Authorized for Execution (Wave 2)**  
**Assigned Platform:** Replit  
**Execution Wave:** Wave 2

---

## 1. Objective

Implement a flexible, data-driven pricing engine that supports multi-actor pricing authority, composable pricing models, a decoupled billing engine, deployment-aware pricing, and auditable overrides. The engine must be declarative, not hard-coded, and no suite may embed its own billing logic.

---

## 2. Execution Prompt (Implement Pricing & Billing Service - CS-4) (Prompt v2)

### 2.1. Execution Validity & Governance (MANDATORY)

> **Execution of this prompt is INVALID unless the following conditions are met:**
> 
> 1.  **Single Source of Truth:** All work must be committed and pushed to the canonical `webwaka` repository (`https://github.com/webwakaagent1/webwaka`) on the `main` branch.
> 2.  **Implementation Directory:** All implementation code must be placed in `/implementations/cs4-pricing-billing-service/`.
> 3.  **GitHub Persistence:** Work not pushed to GitHub is INVALID.
> 4.  **Control Board Synchronization:** The Master Control Board must be updated by the verifier upon completion.
> 5.  **Scope Discipline:** Execute only what is explicitly defined in this prompt. If something is unclear or blocked, STOP and document the blocker.
> 6.  **Documentation as Artifact:** All required documentation must be produced and committed.
> 7.  **Closure Rule:** This prompt is not closed until all artifacts are committed, all documentation is complete, and all completion evidence is provided.
> 8.  **Invariant Compliance:** This work is bound by all platform invariants, including INV-001 (Pricing Flexibility) and INV-012 (Single-Repository Topology).

### 2.2. Core Requirements

1.  **Multi-Actor Pricing Authority:** Implement the ability for Super Admin, Partners, Clients, Merchants/Vendors, Agents, and Staff to configure pricing independently for individuals, groups, segments, and contracts.
2.  **Composable Pricing Models:** Implement support for flat, usage-based, tiered, subscription, and revenue-share/commission-based pricing models, as well as hybrid combinations.
3.  **Decoupled Billing Engine:** Implement a declarative pricing rules engine that is separate from the billing execution logic.
4.  **Deployment-Aware Pricing:** Implement support for Shared SaaS, Partner-deployed/whitelabel, and Fully isolated/self-hosted instances, with the ability for each deployment to override or inherit pricing defaults.
5.  **Auditability & Override Safety:** Implement a versioned, auditable, and reversible pricing override mechanism.

### 2.3. Required Documentation

1.  **Architecture Document:** Create a detailed architecture document (`/docs/architecture/ARCH_CS4_PRICING_BILLING.md`) that explains the design of the pricing and billing service.
2.  **API Documentation:** Create comprehensive API documentation for the pricing and billing service.
3.  **Runbooks:** Create runbooks for:
    *   Configuring pricing models
    *   Managing billing cycles
    *   Troubleshooting common pricing and billing issues

### 2.4. Completion Evidence (MANDATORY)

Upon completion, you must provide:
1.  ✅ **Git commit SHA(s)**
2.  ✅ **Explicit list of files changed/added**
3.  ✅ **Links to all produced documentation**
4.  ✅ **Confirmation that all work is pushed to GitHub**
5.  ✅ **Clear statement of what was completed**

---

## 3. Status & History

*   **January 30, 2026:** Authorized for execution as part of Wave 2.
