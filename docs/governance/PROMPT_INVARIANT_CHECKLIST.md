# PaA Prompt Invariant Checklist

**Version:** 1.0  
**Date:** January 30, 2026  
**Author:** Manus AI  
**Status:** 🟢 **Active & Enforced**

> **This document is subordinate to the Master Control Board. It defines the minimum requirements for any PaA execution prompt to be considered valid.**

---

## 1. Purpose

This checklist ensures that every Prompts-as-Artifacts (PaA) execution prompt is complete, unambiguous, and enforceable. It guarantees that all execution is traceable, verifiable, and aligned with the platform's governance model. If a prompt does not satisfy every item on this checklist, it is considered **invalid** and cannot be used to authorize work.

---

## 2. The Checklist

Every Prompts-as-Artifacts (PaA) execution prompt **MUST** satisfy **ALL** items below.

### A. Authority & Scope

| Check | Invariant | Description |
| :--- | :--- | :--- |
| ☐ | **MCB Reference** | Prompt MUST reference the specific Master Control Board item(s) it implements. |
| ☐ | **Phase ID & Scope** | Prompt MUST state the phase identifier and a clear, one-sentence scope. |
| ☐ | **In-Scope Definition** | Prompt MUST explicitly list what is IN SCOPE. |
| ☐ | **Out-of-Scope Definition** | Prompt MUST explicitly list what is OUT OF SCOPE / FORBIDDEN. |

### B. Execution Validity

| Check | Invariant | Description |
| :--- | :--- | :--- |
| ☐ | **GitHub as SoT** | Prompt MUST include an explicit statement that GitHub is the single source of truth. |
| ☐ | **Push is Mandatory** | Prompt MUST include an explicit requirement that all work must be committed and pushed. |
| ☐ | **Work Not Pushed is Invalid** | Prompt MUST include an explicit statement that work not pushed is INVALID. |
| ☐ | **Repo & Branch Specified** | Prompt MUST clearly specify the target repository and branch for the work. |

### C. Evidence & Traceability

| Check | Invariant | Description |
| :--- | :--- | :--- |
| ☐ | **Commit SHA Reporting** | Prompt MUST require the final commit SHA(s) to be reported. |
| ☐ | **File Manifest** | Prompt MUST require a list of files added, modified, or deleted. |
| ☐ | **Documentation Links** | Prompt MUST require links to all new or updated documentation. |
| ☐ | **MCB Sync Confirmation** | Prompt MUST require the executor to confirm the Master Control Board status has been updated. |

### D. Verification Hooks

| Check | Invariant | Description |
| :--- | :--- | :--- |
| ☐ | **Correctness Definition** | Prompt MUST define how correctness will be verified. |
| ☐ | **Verifier Evidence** | Prompt MUST specify what evidence the verifier will check. |
| ☐ | **Pass/Fail Conditions** | Prompt MUST state the pass/fail conditions clearly. |

### E. Failure & Block Handling

| Check | Invariant | Description |
| :--- | :--- | :--- |
| ☐ | **Blocker Protocol** | Prompt MUST define what the executor must do if blocked. |
| ☐ | **Blocker Documentation** | Prompt MUST require the documentation of all blockers. |
| ☐ | **No Silent Failure** | Prompt MUST forbid silent, partial, or undocumented completion. |

### F. Closure Requirements

| Check | Invariant | Description |
| :--- | :--- | :--- |
| ☐ | **Completion Report Format** | Prompt MUST define the exact format for the completion report. |
| ☐ | **MCB Update Pre-Submission** | Prompt MUST require the Control Board to be updated *before* submission. |
| ☐ | **Handover to Verifier** | Prompt MUST require an explicit handover to the designated verifier. |

### G. Governance Enforcement

| Check | Invariant | Description |
| :--- | :--- | :--- |
| ☐ | **INV-011 Reference** | Prompt MUST reference INV-011 (PaA Execution). |
| ☐ | **Deviation Policy** | Prompt MUST state that any deviation requires explicit Founder approval. |
| ☐ | **Undocumented Work Policy** | Prompt MUST state that undocumented work “does not exist.” |

---

## 3. Governance Statement (Binding)

> “Execution prompts are governed artifacts. If a prompt does not explicitly require GitHub persistence and Control Board synchronization, the execution is invalid by definition.”

This statement is enforced by the Master Control Board and applies to all execution across the WebWaka platform.

---

**End of Document**
