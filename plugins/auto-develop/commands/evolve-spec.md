---
description: Evolve an auto-develop capability spec by analyzing results and discovering new capabilities
---

Analyze the latest auto-develop results to discover new capabilities that are now possible or valuable, then update the capability specification accordingly.

The auto-develop skill to analyze: $ARGUMENTS

---

## Overview

After running an auto-develop cycle, the capability specification may have room to grow — new capabilities that are now possible because of what was built, natural follow-ons, or capabilities discovered during implementation. This skill:
1. Reads the latest auto-develop results and the current spec
2. Analyzes what was built and what new possibilities emerged
3. Identifies new capabilities that are now valuable or feasible
4. Proposes additions and gets user approval
5. Updates the capability spec
6. Validates the updated spec consistency

---

## Phase 1: Load Context

1. **Identify the feature**: Parse `$ARGUMENTS` to determine the auto-develop skill name. Expected format: a feature name like `search`, `auth`, `export`, etc.

2. **Load the current spec**: Read `.claude/commands/spec-{feature}.md` (or the appropriate spec file — check `.claude/commands/` for the matching spec).

3. **Load the results log**: Read `.debug/auto-develop-{feature}/results.tsv` to understand the implementation trajectory — what was attempted, accepted, rejected, and in what order.

4. **Load the regression suite**: Read `.debug/auto-develop-{feature}/regression.json` to see all currently-tracked capabilities and baseline checks.

5. **Load implementation logs**: Read the latest iteration's plan and verification files from `.debug/auto-develop-{feature}/logs/` and `.debug/auto-develop-{feature}/snapshots/` to understand what was actually built and any issues encountered.

6. **Load the current code**: Read the files that were modified during auto-develop (from git log or the implementation plans) to see the current state of the feature.

If any of these files don't exist, report what's missing and stop.

---

## Phase 2: Analyze What Was Built

For each implemented capability (`[x]` status), examine:

### 2a. Implementation Patterns
- What patterns were established during implementation?
- What infrastructure was created that could be reused?
- What abstractions emerged that could be extended?

### 2b. New Extension Points
- Did implementations create new hooks, config options, or interfaces?
- Are there new code paths that could be extended?
- Were any utility functions created that enable further capabilities?

### 2c. TODOs and Notes Added During Implementation
- Search for TODOs, FIXMEs, HACKs, or "future work" comments added during auto-develop
- Check implementation plans for noted limitations or future improvements
- Look for commented-out code or placeholder implementations

### 2d. Failed Capability Analysis
- For `[!]` (attempted-failed) capabilities, understand why they failed
- Could they succeed now with different infrastructure in place?
- Are there simpler versions of failed capabilities that would work?

### 2e. User-Facing Gaps
- With the new capabilities in place, what would a user expect next?
- Are there incomplete workflows (feature starts something but doesn't finish it)?
- Are there asymmetries (can create but not delete, can read but not write)?

---

## Phase 3: Gap Analysis

Based on the analysis, identify new capabilities across three categories:

### 3a. Natural Follow-Ons
Capabilities that logically follow from what was built:
- If search was added, filtering and sorting are natural next steps
- If create was added, update and delete follow
- If basic version works, advanced options follow

### 3b. Infrastructure-Enabled
Capabilities that are now feasible because of infrastructure built during implementation:
- New abstractions that can be extended
- Shared utilities that enable new features
- Patterns that can be replicated

### 3c. Retry Candidates
Previously-failed capabilities that might succeed now:
- Failed due to missing infrastructure that now exists
- Can be decomposed into simpler sub-capabilities
- Different approach possible with current codebase state

For each candidate, assess:
- **Value**: How much does this improve the feature for users?
- **Feasibility**: How likely is implementation to succeed given current state?
- **Risk**: Could this break existing capabilities?
- **Dependencies**: What must exist first?

---

## Phase 4: Present Proposals

Present the analysis to the user in this format:

```
## Spec Evolution Analysis: {feature}

### Current State
- Capabilities implemented: {count} / {total}
- Capabilities failed: {count}
- Capabilities remaining: {count}
- Regression suite size: {count} checks

### What Was Built (Summary)
{Brief summary of implemented capabilities and patterns established}

### Proposed New Capabilities

#### High Priority (recommend adding)

**Tier {N}: {tier name}**
| ID | Capability | Acceptance Criteria | Dependencies | Reasoning |
|---|---|---|---|---|
| CAP-{NNN} | {name} | {criteria} | CAP-{dep} | {natural follow-on / infrastructure-enabled / retry candidate} |
| ... | | | | |

#### Medium Priority (consider adding)
...

#### Retry Candidates (previously failed, may now succeed)
| Original ID | Capability | Previous Failure Reason | Why It Might Work Now |
|---|---|---|---|
| CAP-{NNN} | {name} | {reason} | {new infrastructure / different approach} |
| ... | | | |

### Iteration Budget
- Current max iterations: {N}
- Iterations used so far: {N}
- Iterations remaining: {N}
- **Recommended new budget**: {N} — {reasoning: e.g., "adding 5 capabilities, suggest increasing by 5-7 iterations"}

### Recommended Changes Summary
- Add {N} new capabilities
- Retry {N} previously-failed capabilities
- Net change: {current count} → {new count} capabilities
- Iteration budget: {current} → {recommended new budget}
```

Wait for user feedback. The user may:
- Approve all additions
- Approve some and reject others
- Modify proposed capabilities
- Add capabilities not proposed
- Re-tier or re-prioritize
- Adjust dependencies
- Adjust the iteration budget

---

## Phase 5: Apply Changes

After user approval:

1. **Read the current spec file** (`.claude/commands/spec-{feature}.md`)

2. **Apply approved changes**:
   - Add new capabilities to the appropriate tier sections
   - Assign new CAP-{NNN} IDs (continue from the highest existing ID)
   - Set status to `[ ]` pending for new capabilities
   - For retry candidates: reset status from `[!]` back to `[ ]`
   - Update dependency references
   - Update the Summary section counts
   - Update the Dependency Graph

3. **Verify dependency consistency**:
   - No circular dependencies
   - All referenced dependencies exist
   - No dependency on `[!]` capabilities (unless being retried)
   - Tier ordering respected (Tier 1 capabilities don't depend on Tier 2+)

4. **Update the auto-develop skill** (`.claude/commands/auto-develop-{feature}.md`):
   - Update the capability count reference in the description
   - Update max iterations if the user approved a new budget

---

## Phase 6: Validation

Verify the updated spec is consistent and well-formed:

1. **Parse check**: All capabilities have required fields (ID, status, priority, dependencies, acceptance criteria, verification)
2. **Dependency check**: DAG is valid — no cycles, no missing references, tier ordering respected
3. **Status check**: Only `[ ]`, `[x]`, and `[!]` statuses present
4. **ID check**: All IDs are unique, sequentially assigned within tiers
5. **Regression check**: All `[x]` capabilities have corresponding entries in regression.json

Report:
```
## Spec Evolution Complete: {feature}

### Changes Applied
- Added: CAP-{ids} — {names}
- Reset for retry: CAP-{ids} — {names}
- Total capabilities: {old} → {new}

### Validation
- Dependency graph: Valid / {issues}
- All fields present: Yes / {missing}
- Regression suite consistent: Yes / {issues}

### Next Steps
Run `/auto-develop-{feature}` to implement the new capabilities.
The loop will pick up where it left off, selecting the next eligible capability.
```

---

## Important Notes

- **Never auto-approve spec changes** — always present to the user first. The spec is the most important artifact; changes affect all future development iterations.
- **Maintain verifiability** — every acceptance criterion must be objectively testable, not subjective.
- **Preserve existing IDs** — never change the ID of an existing capability. Add new ones with new IDs.
- **Don't over-expand** — a spec with 50 capabilities is worse than one with 20 well-chosen ones. Each capability should represent meaningful, distinct functionality.
- **Evidence is mandatory** — every proposed new capability must cite specific code, patterns, or implementation artifacts that justify its addition.
- **Respect the dependency graph** — new capabilities should integrate cleanly into the existing dependency structure, not create a tangled mess.
