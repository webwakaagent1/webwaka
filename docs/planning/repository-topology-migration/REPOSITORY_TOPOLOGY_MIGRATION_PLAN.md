# WebWaka Repository Topology Migration Plan

**Status:** 🟡 **PLANNING ONLY - NOT EXECUTED**  
**Version:** 1.0 (Final Draft)  
**Date:** January 30, 2026  
**Author:** Manus AI  
**Authority:** Pending Founder Ratification

---

## 1. Executive Summary

This document presents a comprehensive, decision-grade plan for migrating the WebWaka platform from its temporary single-repository topology (INV-012) to a permanent, scalable, and governance-preserving multi-repository architecture. This migration is the mandatory next step following the successful completion of all Wave 1, 2, and 3 phases.

The proposed architecture consists of **six primary repositories**, each aligned with a specific layer of the platform, designed to strengthen parallel execution, enforce clear boundaries, and maintain the supremacy of the Master Control Board.

**This is a planning document only. Execution requires explicit Founder approval.**

---

## 2. Target Multi-Repository Architecture

### 2.1. Proposed Repository Structure

The target topology consists of six repositories, creating a clear separation of concerns that mirrors the platform's layered architecture:

| Repository | Purpose | Contains |
| :--- | :--- | :--- |
| `webwaka-governance` | **Control Plane** | MCB, Invariants, PaA Model, Planning Docs, ADRs, Verification Reports |
| `webwaka-platform-foundation` | **Foundation Layer** | PF Phase Implementations, Shared SDKs, Platform Utilities |
| `webwaka-core-services` | **Core Services Layer** | CS Phase Implementations, Service-Specific Docs & Tests |
| `webwaka-capabilities` | **Business Capabilities Layer** | CB Phase Implementations, Capability-Specific Docs & Tests |
| `webwaka-infrastructure` | **Infrastructure & Deployment** | ID Phase Implementations, IaC, Deployment Manifests, Operational Tooling |
| `webwaka-suites` | **Business Suites Layer** | Future SC Phase Implementations, Suite-Specific Orchestration & UI |

### 2.2. Architectural Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    webwaka-governance                           │
│                   (Control Plane - Supreme)                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓ (governs all)
        ┌─────────────────────┴─────────────────────┐
        ↓                                           ↓
┌───────────────────────┐                 ┌───────────────────────┐
│ webwaka-infrastructure│                 │webwaka-platform-      │
│      (ID Phases)      │                 │   foundation          │
└───────────────────────┘                 │    (PF Phases)        │
        ↓ (deploys all)                   └───────────────────────┘
                                                    ↓ (foundation for)
                                          ┌───────────────────────┐
                                          │ webwaka-core-services │
                                          │     (CS Phases)       │
                                          └───────────────────────┘
                                                    ↓ (services for)
                                          ┌───────────────────────┐
                                          │ webwaka-capabilities  │
                                          │     (CB Phases)       │
                                          └───────────────────────┘
                                                    ↓ (capabilities for)
                                          ┌───────────────────────┐
                                          │   webwaka-suites      │
                                          │     (SC Phases)       │
                                          └───────────────────────┘
```

---

## 3. Governance Preservation & Invariant Mapping

All platform invariants are preserved, with INV-012 being formally retired and replaced. Governance is strengthened through structural enforcement.

| Invariant | Status | Post-Migration Enforcement Mechanism |
| :--- | :--- | :--- |
| INV-002: Tenant Isolation | ✅ **Preserved** | Cross-repository integration tests; shared utilities in `webwaka-platform-foundation`. |
| INV-004: Layered Dependency | ✅ **Strengthened** | **Structurally enforced** by repository boundaries and CI/CD validation. |
| INV-011: PaA Execution | ✅ **Preserved** | Prompts in `webwaka-governance` explicitly target implementation repositories. |
| INV-012: Single-Repository | 🔄 **RETIRED** | Replaced with **INV-012v2 (Multi-Repository Topology)**. **Founder decision required.** |
| **All Others** | ✅ **Preserved** | No changes to enforcement mechanisms. |

**Master Control Board Supremacy:** The MCB remains the supreme source of truth in the `webwaka-governance` repository, now tracking repository locations and cross-repository commit SHAs for enhanced traceability.

---

## 4. Zero-Break Migration Strategy

The migration will be executed in **six sequential phases** to ensure zero disruption and complete data integrity.

| Phase | Objective | Estimated Duration |
| :--- | :--- | :--- |
| **1. Governance Repo Creation** | Create `webwaka-governance` and migrate all governance docs with full history. | 3-5 days |
| **2. Foundation & Infra Migration** | Create and migrate `webwaka-platform-foundation` & `webwaka-infrastructure`. | 3-5 days |
| **3. Core Services Migration** | Create and migrate `webwaka-core-services`. | 3-5 days |
| **4. Capabilities Migration** | Create and migrate `webwaka-capabilities`. | 3-5 days |
| **5. Suites Repo Preparation** | Create placeholder `webwaka-suites` for future work. | 1-2 days |
| **6. Monorepo Archival** | Archive the original `webwaka` repository, making it read-only. | 1-2 days |

**Total Estimated Duration:** 4-6 weeks (inclusive of Founder approval gates between each phase).

**Historical Preservation:** The `git filter-repo` tool will be used to extract all subdirectories with their complete, unaltered Git history, ensuring 100% traceability.

---

## 5. Execution Model Impact

The core PaA model remains intact, with minor adjustments to accommodate the new structure.

| Aspect | Current (Monorepo) | Post-Migration (Multi-Repo) |
| :--- | :--- | :--- |
| **Repositories per Agent** | 1 | 2 (governance + target) |
| **Execution Prompt** | Implicit repository | **Explicit `Target Repository` field** |
| **Backlinks** | Relative path | `repository@commit-sha:path` |
| **Verification** | Single-repo check | **Cross-repo validation** |
| **Parallel Execution** | 2-3 phases (high conflict risk) | **4-6+ phases (low conflict risk)** |

This new model dramatically improves parallel execution capacity, a key requirement for scaling the WebWaka platform.

---

## 6. Risk Analysis & Mitigation

Fifteen primary risks have been identified across operational, governance, human/agent error, tooling, and migration-specific categories. The overall risk is assessed as **MEDIUM**, reducing to **LOW** with the proposed mitigations.

| Risk Category | Key Risk | Mitigation Strategy |
| :--- | :--- | :--- |
| **Tooling** | Git history corruption during migration. | Use `git filter-repo`, perform dry runs, and maintain full backups. |
| **Operational** | Agent confusion about target repository. | Mandate `Target Repository` in prompts and create comprehensive onboarding guides. |
| **Governance** | MCB desynchronization. | Implement automated validation and enforce strict verification checks. |
| **Migration** | Execution failure leaving an inconsistent state. | Use a phased approach with approval gates and clear rollback plans for each phase. |

---

## 7. Founder Decisions Required

This migration plan requires explicit Founder approval on the following points before execution can begin.

| Decision | Options | Recommendation | Rationale |
| :--- | :--- | :--- | :--- |
| **1. Approve Migration** | Proceed / Defer / Reject | **PROCEED** | The multi-repository model is critical for future scalability and parallel execution. The risks are manageable. |
| **2. Retire INV-012** | Approve / Reject | **APPROVE** | The single-repository invariant was explicitly temporary and has served its purpose. |
| **3. Ratify INV-012v2** | Approve / Reject | **APPROVE** | A new invariant is required to govern the multi-repository topology. |
| **4. Approve Timeline** | Approve 4-6 week plan / Request changes | **APPROVE** | The phased timeline with approval gates minimizes risk and ensures Founder oversight. |
| **5. Assign Executor** | Manus / Other | **Manus** | Manus possesses the complete context and technical capability to execute the migration reliably. |

---

**Next Steps:** Upon Founder approval, a formal execution prompt will be created to initiate Phase 1 of the migration.
