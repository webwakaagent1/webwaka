# CB-1: MLAS Capability

**Status:** 🟡 **Authorized for Execution (Wave 3)**

---

## 1. Phase Objective

Build MLAS as a configurable revenue-flow infrastructure supporting attribution tracking, commission calculation, payout routing, auditability, dispute resolution hooks, multi-level revenue trees, and support for Platform-First, Partner-Owned, Client-Owned, Zero-Platform-Cut, and Hybrid models.

---

## 2. Execution Prompt (Implement MLAS Capability - CB-1, Prompt v2)

### 2.1. Scope of Work

You are to implement the MLAS Capability as defined in the objective above. This includes:

*   **Attribution Tracking:** A service that can track referrals and attribute sales to the correct affiliates.
*   **Commission Calculation:** A flexible engine that can calculate commissions based on configurable rules.
*   **Payout Routing:** A system that can route payouts to the correct recipients.
*   **Auditability:** A comprehensive logging and auditing system for all MLAS transactions.
*   **Dispute Resolution Hooks:** A set of APIs and webhooks to enable dispute resolution workflows.
*   **Multi-Level Revenue Trees:** Support for complex, multi-level affiliate structures.

### 2.2. Deliverables

*   **Code:** All implementation code for the MLAS Capability.
*   **Documentation:**
    *   Architecture Decision Records (ADRs) for all significant design choices.
    *   API documentation for all new services.
    *   Runbooks for operating and maintaining the new services.
*   **Tests:** Unit, integration, and end-to-end tests for all new functionality.

### 2.3. Mandatory Invariants

*   **INV-011 (PaA Execution):** All work must be traceable to this prompt.
*   **INV-012 (Single-Repository Topology):** All work must be committed to the `webwaka` repository in the `/implementations/cb1-mlas-capability/` directory.

### 2.4. Completion Requirements

*   **Git Commit SHA(s):** Provide the SHA of the final commit(s).
*   **Explicit List of Files Changed/Added:** A clear list of all files that were created or modified.
*   **Links to all Produced Documentation:** Direct links to all ADRs, API docs, and runbooks.
*   **Confirmation of GitHub Push:** Explicit confirmation that all work has been pushed to GitHub.
*   **Clear Statement of Completion:** A clear statement of what was completed, what was not required, and any blockers or follow-ups.

---


