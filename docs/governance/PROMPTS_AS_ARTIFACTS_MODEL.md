# Prompts-as-Artifacts (PaA) Model

**Version:** 1.0 (Ratified)  
**Date:** January 30, 2026  
**Author:** Manus AI  
**Status:** 🔒 **Ratified by Founder**

> **This document is subordinate to the Master Control Board.**

---

## 1. Purpose

This document defines the **Prompts-as-Artifacts (PaA) Model**, the official documentation and prompt architecture for the WebWaka platform. The PaA model treats execution prompts not as disposable messages, but as permanent, version-controlled artifacts that live directly within the `webwaka-governance` repository. This transforms the documentation from a passive library into an active, self-navigating system for assigning and managing work, structurally eliminating ambiguity and context drift.

---

## 2. Core Principle

> **From this point forward, execution prompts are not messages. They are governed artifacts. If it isn't documented, it didn't happen.**

This principle operationalizes the foundational governance rule:

> **If it is not visible in the Master Control Board, it does not exist.**

---

## 3. Canonical Documentation Hierarchy

The foundation of the PaA model is a strict, predictable, and numbered hierarchy within the `webwaka-governance` repository. This ensures any agent can navigate the entire context in a logical order.

```
/docs
├── governance/
│   ├── WEBWAKA_MASTER_CONTROL_BOARD.md (00 - Supreme)
│   └── PROMPTS_AS_ARTIFACTS_MODEL.md (This document)
├── planning/
│   ├── 01_PLATFORM_IMPLEMENTATION_FRAMEWORK.md
│   └── 02_WEBWAKA_MASTER_IMPLEMENTATION_PLAN.md
├── architecture/
│   ├── ARCH_00_CAPABILITY_DEPENDENCY_MAPS.md
│   └── ...
├── decisions/
│   ├── DECISION_001_...
│   └── ...
├── phases/
│   ├── PF-1_FOUNDATIONAL_EXTENSIONS.md
│   ├── SC-1_COMMERCE_SUITE_V1.md
│   └── ...
└── reports/
    └── ...
```

**Hierarchy Rules:**

*   **The Master Control Board is the absolute root (00).** All other documents are subordinate to it.
*   **Numbered Prefixes:** Core documents are numbered to enforce a clear reading order for any onboarding agent.
*   **Thematic Directories:** Documents are organized by their function (governance, planning, architecture, decisions, phases), making the system self-discoverable.

---

## 4. The Embedded Execution Prompt

The core artifact of the PaA model is the **Execution Prompt**, a standardized block of Markdown embedded directly within the relevant governance document (e.g., a phase definition).

### 4.1. Mandatory Structure

Every execution prompt **must** include the following sections:

```markdown
#### 🚀 EXECUTION PROMPT: [Clear, One-Sentence Title]

**Objective:** [Clear, one-sentence description of what must be accomplished]

**Scope:**
- [Explicit list of what is in scope]

**Non-Goals:**
- [Explicit list of what is NOT in scope]

**Canonical Governance:**
- **Master Control Board:** [Link to the specific section in the MCB]
- **Dependencies:** [Links to all dependency definitions]

**Deliverables:**
1. [Specific, verifiable deliverable 1]
2. [Specific, verifiable deliverable 2]
...

**Required Documentation Outputs:**
- [List of all documentation artifacts that must be created]

**Output:** [Instruction on where to document technical details and how to link back to this prompt]
```

### 4.2. Binding Constraints

1.  **Execution prompts are authoritative.** An agent must not act outside what is defined in the embedded prompt. Chat-based instructions are non-binding unless reflected back into the document.
2.  **Each execution prompt must explicitly declare** scope, non-goals, dependencies (with links), expected artifacts, required documentation outputs, and a Control Board reference.
3.  **Prompt closure is mandatory.** Execution is not complete until:
    *   Artifacts are committed to the appropriate repository.
    *   Documentation is written.
    *   The originating prompt is updated with backlinks to the outputs.

---

## 5. The Self-Navigating, Closed-Loop Workflow

This architecture creates a fully traceable and verifiable workflow:

1.  **Assign Task:** An agent is assigned a task by being given a single URL pointing directly to the **Execution Prompt** block within a phase document.
2.  **Navigate & Read:** The agent follows the links within the prompt to the Master Control Board and other canonical documents, guaranteeing it has the full, correct context.
3.  **Execute:** The agent performs the work as specified in the prompt.
4.  **Document & Link Back:** The agent creates the required output documentation and then **edits the original phase document** to link the new architecture document back to the prompt. This closes the loop, creating a permanent, bi-directional link between the request and the result.

---

## 6. Governance Rules (Binding)

### 6.1. No Orphan Prompts

Every execution prompt must map to:

*   A Control Board item
*   A phase
*   An owner

### 6.2. No Silent Execution

If work is done and not traceable to a prompt, it is invalid.

### 6.3. Agent-Agnostic by Design

Prompts must assume:

*   Manus, Replit, Emergent, Lovable, or future agents
*   Zero prior conversational context
*   The document itself must fully onboard the agent

---

## 7. Phase Document Requirements

All phase documents must contain:

1.  **An Execution Prompt section** (as defined in Section 4)
2.  **A Status & History section** tracking the current state and all major updates

---

## 8. How PaA Eliminates Ambiguity & Ensures Scalability

| Problem | How the PaA Model Solves It |
| :--- | :--- |
| **Context Drift** | The prompt is a permanent, version-controlled artifact. It can never drift from the canonical governance because **it is part of the canonical governance**. |
| **Ambiguity** | The prompt explicitly links to the single source of truth (the Master Control Board), eliminating any ambiguity about scope, dependencies, or constraints. |
| **Lost Knowledge** | All execution history is captured in the Git history of the governance documents themselves. We can see not just what was built, but the **exact, version-controlled prompt** that was used to build it. |
| **Scalability** | As new phases and suites are added, new documents with embedded prompts are created. The system scales linearly and remains organized due to the canonical hierarchy. |

---

## 9. Ratification Status

**Status:** 🔒 **Ratified by Founder on January 30, 2026**

This model is now the official and mandatory approach for all future execution on the WebWaka platform.

---

**End of Document**
