# EXECUTION PROMPT: Repository Topology Migration - Phase 1

**Phase:** Phase 1 - Governance Repository Creation & Initialization  
**Status:** 🟡 **READY FOR EXECUTION**  
**Version:** 1.0  
**Date:** January 30, 2026  
**Authority:** Founder Approved  
**Executor:** Manus

---

## 1. CONTEXT

This is the formal execution prompt for **Phase 1** of the Repository Topology Migration, as approved by the Founder on January 30, 2026. This phase creates the `webwaka-governance` repository and migrates all governance, planning, and architecture documents while preserving complete Git history.

**Planning Documents:** `/docs/planning/repository-topology-migration/`  
**Master Control Board Item:** REPO-MIG-1  
**Migration Strategy:** `/docs/planning/repository-topology-migration/MIGRATION_STRATEGY.md`

---

## 2. EXECUTION PROMPT: Phase 1 - Governance Repository Creation (v1)

**Target Repository:** `webwaka-governance` (to be created)  
**Source Repository:** `webwaka` (current monorepo)

### Objective

Create the `webwaka-governance` repository and migrate all governance, planning, and architecture documents from the current `webwaka` monorepo while preserving complete Git history. This repository will serve as the supreme control plane for all WebWaka repositories.

### Pre-Execution Checklist

Before beginning execution, verify:

- [ ] All Wave 3 phases are marked as 🟢 Complete in the MCB
- [ ] No active execution is in progress (check MCB for any 🟡 "In Progress" phases)
- [ ] Full backup of `webwaka` repository has been created
- [ ] GitHub organization `webwakaagent1` has capacity for new repositories
- [ ] Founder approval has been received (confirmed in MCB)

### Deliverables

1. **New Repository:** `webwaka-governance` created at `https://github.com/webwakaagent1/webwaka-governance`
2. **Migrated Content:** All content from `/docs/` directory with full Git history
3. **Updated Governance Documents:**
   - Master Control Board with repository topology section
   - PaA Model with cross-repository execution instructions
   - Prompt Invariant Checklist with repository field requirements
4. **Migration Tracking:** `MIGRATION_STATUS.md` document tracking all migration phases
5. **Repository README:** Clear explanation of governance repository purpose and structure

### Execution Steps

#### Step 1.1: Create New Repository

```bash
# Create new repository on GitHub
gh repo create webwakaagent1/webwaka-governance --public \
  --description "WebWaka Platform Governance & Control Plane - Supreme Source of Truth"
```

#### Step 1.2: Clone and Prepare Source Repository

```bash
# Clone current repository with full history
cd /home/ubuntu
git clone https://github.com/webwakaagent1/webwaka.git webwaka-migration-governance
cd webwaka-migration-governance
```

#### Step 1.3: Extract Governance Subdirectory with History

```bash
# Install git-filter-repo if not already installed
sudo pip3 install git-filter-repo

# Extract docs/ directory with full history
git filter-repo --path docs/ --path README.md --path LICENSE --force

# Verify history preservation
git log --all --oneline | head -20
```

#### Step 1.4: Push to Governance Repository

```bash
# Add governance repository as remote
git remote add governance https://github.com/webwakaagent1/webwaka-governance.git

# Push with full history
git push governance main
```

#### Step 1.5: Update Governance Documents

Create or update the following documents in the new `webwaka-governance` repository:

**1. Update Master Control Board (`docs/governance/WEBWAKA_MASTER_CONTROL_BOARD.md`):**
- Add repository topology section explaining the six-repository model
- Update REPO-MIG-1 status to reflect Phase 1 completion
- Add repository URLs for all six repositories (even if not yet created)

**2. Update PaA Model (`docs/governance/PROMPTS_AS_ARTIFACTS_MODEL.md`):**
- Add "Target Repository" as mandatory field in execution prompt template
- Update workflow section to include cross-repository execution steps
- Update backlink format to require `repository@commit-sha:path` format

**3. Update Prompt Invariant Checklist (`docs/governance/PROMPT_INVARIANT_CHECKLIST.md`):**
- Add checklist item: "Target repository explicitly specified"
- Add checklist item: "Implementation path includes repository prefix"
- Add checklist item: "Backlinks use repository@commit-sha:path format"

**4. Create Migration Status Document (`docs/planning/repository-topology-migration/MIGRATION_STATUS.md`):**

```markdown
# Repository Topology Migration Status

**Status:** 🟡 **IN PROGRESS**  
**Current Phase:** Phase 1 - Governance Repository Creation  
**Last Updated:** [DATE]

## Migration Progress

| Phase | Status | Start Date | Completion Date | Commit SHA |
| :--- | :--- | :--- | :--- | :--- |
| Phase 1: Governance | 🟢 Complete | [DATE] | [DATE] | governance@[SHA] |
| Phase 2: Foundation & Infra | ⚪ Not Started | - | - | - |
| Phase 3: Core Services | ⚪ Not Started | - | - | - |
| Phase 4: Capabilities | ⚪ Not Started | - | - | - |
| Phase 5: Suites | ⚪ Not Started | - | - | - |
| Phase 6: Archival | ⚪ Not Started | - | - | - |

## Repository URLs

| Repository | URL | Status |
| :--- | :--- | :--- |
| webwaka-governance | https://github.com/webwakaagent1/webwaka-governance | 🟢 Created |
| webwaka-platform-foundation | https://github.com/webwakaagent1/webwaka-platform-foundation | ⚪ Not Created |
| webwaka-core-services | https://github.com/webwakaagent1/webwaka-core-services | ⚪ Not Created |
| webwaka-capabilities | https://github.com/webwakaagent1/webwaka-capabilities | ⚪ Not Created |
| webwaka-infrastructure | https://github.com/webwakaagent1/webwaka-infrastructure | ⚪ Not Created |
| webwaka-suites | https://github.com/webwakaagent1/webwaka-suites | ⚪ Not Created |
```

**5. Create Governance Repository README (`README.md`):**

```markdown
# WebWaka Governance Repository

**Supreme Source of Truth for WebWaka Platform**

This repository serves as the control plane for all WebWaka platform development. It contains the Master Control Board, platform invariants, PaA Model, planning documents, architecture documents, and all governance mechanisms.

## Repository Topology

WebWaka uses a multi-repository architecture (INV-012v2):

- **webwaka-governance** (this repo) - Control plane and supreme source of truth
- **webwaka-platform-foundation** - Foundation layer (PF phases)
- **webwaka-core-services** - Core services layer (CS phases)
- **webwaka-capabilities** - Business capabilities layer (CB phases)
- **webwaka-infrastructure** - Infrastructure & deployment (ID phases)
- **webwaka-suites** - Business suites layer (SC phases)

## Key Documents

- **Master Control Board:** `docs/governance/WEBWAKA_MASTER_CONTROL_BOARD.md`
- **Platform Invariants:** See Master Control Board Section 1
- **PaA Model:** `docs/governance/PROMPTS_AS_ARTIFACTS_MODEL.md`
- **Architecture Documents:** `docs/architecture/`
- **Planning Documents:** `docs/planning/`

## Governance Supremacy

All execution must be authorized by an execution prompt in this repository. All implementation repositories are subordinate to this governance repository. The Master Control Board is the single source of truth for platform state.
```

#### Step 1.6: Verification

Verify the following before marking Phase 1 complete:

- [ ] `webwaka-governance` repository is publicly accessible
- [ ] All governance documents are present in the new repository
- [ ] Git history is preserved (verify with `git log`)
- [ ] All commit SHAs match the original repository
- [ ] All relative links within governance documents still work
- [ ] README clearly explains repository purpose
- [ ] Migration status document is created and accurate
- [ ] Updated governance documents are committed

### Completion Criteria

Phase 1 is complete when:

1. ✅ `webwaka-governance` repository is operational
2. ✅ All governance documents migrated with full Git history
3. ✅ Updated governance documents (MCB, PaA Model, Checklist) committed
4. ✅ Migration status document created
5. ✅ Repository README created
6. ✅ All verification checks passed
7. ✅ This prompt updated with backlinks (see Section 3)
8. ✅ Founder approval requested for Phase 2

### Risk Mitigation

- **Backup:** Full backup of `webwaka` repository exists before starting
- **Rollback Plan:** If issues arise, delete `webwaka-governance` and retry
- **History Validation:** Verify commit SHAs match original repository
- **Link Validation:** Test all internal links after migration

---

## 3. EXECUTION BACKLINKS (To be updated upon completion)

**Status:** ⚪ **NOT EXECUTED**

Upon completion, update this section with:

- [ ] Governance repository URL: `https://github.com/webwakaagent1/webwaka-governance`
- [ ] Final commit SHA: `webwaka-governance@[SHA]`
- [ ] Completion date: `[DATE]`
- [ ] Verification report: Link to verification document
- [ ] Founder approval: Requested / Approved

---

## 4. PROMPT INVARIANT COMPLIANCE

This prompt complies with the following invariants:

- ✅ **INV-011 (PaA Execution):** This is a formal execution prompt in a canonical governance document
- ✅ **INV-012v2 (Multi-Repository Topology):** This prompt creates the first repository in the new topology
- ✅ **Prompt Invariant Checklist:** All mandatory fields present

---

**Status:** 🟡 **READY FOR EXECUTION**  
**Awaiting Execution by Manus**
