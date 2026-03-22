---
description: Create a custom auto-develop loop for any feature in your codebase
---

Create a custom auto-develop skill for a specific feature in the application. Analyzes the feature, interviews the user to discover and curate new capabilities, builds a capability specification, instruments code if needed, and generates a complete auto-develop skill that iteratively implements new functionality.

The feature to develop: $ARGUMENTS

---

## Overview

This skill creates two files:
1. **A capability specification** (`.claude/commands/spec-{feature}.md`) — the enumerated list of new capabilities to implement
2. **An auto-develop skill** (`.claude/commands/auto-develop-{feature}.md`) — the iterative development loop

It does this through a structured process that minimizes back-and-forth — all user input is gathered in a single round after initial analysis.

---

## Phase 0: App & Stack Detection

Before analyzing the feature, discover the project's technology stack and app type. This "stack profile" informs all later phases.

### 0a: Language & Build System

Scan the project root for manifest files and infer the stack:

| Manifest File | Language/Ecosystem |
|---|---|
| `package.json` | Node.js / TypeScript / JavaScript |
| `pyproject.toml`, `setup.py`, `requirements.txt` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `Gemfile` | Ruby |
| `build.gradle`, `pom.xml` | Java / Kotlin |
| `Makefile` (alone) | C / C++ / generic |
| `mix.exs` | Elixir |
| `Package.swift` | Swift |
| `*.csproj`, `*.sln` | C# / .NET |

Also detect:
- **Package manager**: npm/yarn/pnpm, pip/poetry/uv, cargo, go modules, bundler, maven/gradle, etc.
- **Framework**: Next.js, Django, Flask, FastAPI, Rails, Gin, Actix, Spring Boot, Phoenix, etc.
- **Test runner**: vitest/jest/mocha, pytest, go test, cargo test, rspec, junit, etc.
- **Linter/type checker**: eslint/tsc, ruff/mypy/pylint, go vet, cargo clippy, rubocop, etc.
- **Build command**: npm run build, python -m build, go build, cargo build, make, etc.

### 0b: App Type Classification

Based on the code structure, classify the app type:

| App Type | Signals |
|---|---|
| **Web backend / API** | Route handlers, controllers, middleware, OpenAPI specs |
| **Web frontend** | React/Vue/Svelte components, pages, CSS/styles, bundler config |
| **Full-stack web** | Both frontend components and backend routes |
| **CLI tool** | Argument parsers (argparse, clap, cobra, commander), main entry point, help text |
| **Data pipeline** | DAG definitions, ETL steps, schedulers, data transformers |
| **Mobile app** | Android/iOS project files, screens, navigation, platform-specific code |
| **ML/AI pipeline** | Model configs, prompts, training scripts, evaluation harnesses, notebooks |
| **Library / SDK** | Public API surface, no main entry point, type definitions, examples/ dir |
| **Monorepo** | Multiple packages/services — identify which sub-project the feature lives in |

### 0c: Execution & Test Infrastructure

Determine:
- **How does the app run?** Dev server, CLI invocation, test harness, container, notebook, etc.
- **How are tests run?** Test runner command, test directory structure, fixture patterns
- **CI/CD**: Check for `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`, etc.

### 0d: Store Stack Profile

Record the stack profile for use in later phases:

```
## Stack Profile

- **Language**: {language}
- **Framework**: {framework}
- **App type**: {type}
- **Package manager**: {pm}
- **Build command**: `{cmd}`
- **Test command**: `{cmd}`
- **Lint/check command**: `{cmd}`
- **Dev run command**: `{cmd}`
```

Proceed to Phase 1.

---

## Phase 1: Feature Analysis

Deeply analyze the feature specified in `$ARGUMENTS`. Your goal is to build a complete mental model of:

1. **Code discovery**: Search the codebase to find ALL files involved in this feature. What to look for depends on the app type detected in Phase 0:

   **Web backend / API**:
   - API routes, controllers, middleware, request/response handlers
   - Service layer, business logic, validators
   - Database models, queries, migrations

   **Web frontend**:
   - Components, pages, layouts
   - Hooks, state management, context providers
   - API client calls, data fetching

   **CLI tool**:
   - Command handlers, subcommand definitions
   - Argument parsers, flag definitions
   - Output formatters, help text generators

   **Data / AI pipeline**:
   - Pipeline steps, DAG definitions, orchestrators
   - Transformers, validators, data loaders
   - Prompt templates (if LLM-based), model configs
   - Output writers, result formatters

   **Mobile app**:
   - Screens, navigation definitions, view models
   - Platform-specific implementations
   - UI components, theme definitions

   **Library / SDK**:
   - Public API surface, exported functions/classes
   - Internal modules, type definitions
   - Example code, usage patterns

   For all types, also identify:
   - Core logic files (the main implementation)
   - Data models and schemas (DB tables, types, interfaces, protobuf, etc.)
   - Output format (what the feature produces — JSON, HTML, images, DB records, stdout, files)
   - Dependencies (other features/services it calls)

2. **Current feature surface**: Map exactly what the feature does today:
   - What inputs does it accept?
   - What outputs does it produce?
   - What configuration options exist?
   - What edge cases are handled vs not handled?

3. **Extension points**: Identify where new functionality could naturally be added:
   - Plugin/middleware hooks
   - Configuration options not yet implemented
   - TODOs, FIXMEs, and "future work" comments in the code
   - Error cases that are caught but not handled gracefully
   - Adjacent features in similar systems that this feature lacks

4. **Architecture constraints**: Understand boundaries for new development:
   - What patterns does the codebase use? (MVC, event-driven, functional, etc.)
   - What are the interface contracts? (API schemas, type definitions, protocols)
   - What dependencies exist? (external APIs, databases, services)
   - What are the performance constraints?

5. **Adjacent feature patterns**: Look at how other features in the same codebase are implemented:
   - What patterns do similar features follow?
   - What capabilities do adjacent features have that this one lacks?
   - What shared infrastructure exists that this feature could leverage?

6. **Existing verification infrastructure**: Check what already exists that can be used to verify new functionality:
   - Test commands, test suites, integration tests that exercise the feature
   - Test fixtures, test data, seed files, sample inputs
   - Scripts, Makefile targets, or CLI commands that can run/test the feature
   - CI pipelines that validate the feature
   - Jupyter notebooks or example scripts that demonstrate the feature
   - A way to run the feature in isolation (single item, dry-run mode, specific endpoint)

Present your findings as a structured summary:

```
## Feature Analysis: {feature name}

### Stack Profile
{from Phase 0}

### Files Involved
- **Entry point**: {path} — {what it does}
- **Core logic**: {path} — {what it does}
- **Config/Schema**: {path} — {what it does}
- ...

### Current Feature Surface
- Inputs: {what it accepts}
- Outputs: {what it produces}
- Configuration: {what's configurable}

### Extension Points
1. {extension point}: {why it's a natural place to add functionality}
2. ...

### Architecture Constraints
- Pattern: {architectural pattern used}
- Interfaces: {key contracts}
- Dependencies: {external dependencies}

### Adjacent Feature Patterns
- {feature}: has {capability} that {this feature} lacks
- ...

### Existing Verification Infrastructure
- Test commands: {what's available}
- Test data/fixtures: {what's available}
- Isolated execution: {how to run for a single item}
```

Then proceed to Phase 2.

---

## Phase 2: Capability Discovery Interview (Single Round)

Based on your analysis from Phases 0-1, propose candidate capabilities AND ask all configuration questions in a **single message**. For each proposed capability, provide your reasoning — the user curates the list. This avoids multiple back-and-forth rounds.

Present in this format:

---

**I've completed my analysis. Here are proposed capabilities for {feature} and the configuration questions I need answered. Review the capability list — accept, reject, re-tier, re-prioritize, add missing ones, or adjust as needed.**

### Proposed Capabilities

I've identified the following candidate capabilities across three tiers, based on natural extensions, adjacent feature parity, TODOs/FIXMEs, unused config options, and unhandled edge cases:

#### Tier 1: Foundation (must exist before Core capabilities)
| ID | Capability | Acceptance Criteria | Dependencies | Reasoning |
|---|---|---|---|---|
| CAP-001 | {name} | {observable, testable condition} | none | {why: TODO found, pattern from adjacent feature, natural extension, etc.} |
| CAP-002 | {name} | {observable, testable condition} | CAP-001 or none | {reasoning} |
| ... | | | | |

#### Tier 2: Core (the main new functionality)
| ID | Capability | Acceptance Criteria | Dependencies | Reasoning |
|---|---|---|---|---|
| CAP-003 | {name} | {observable, testable condition} | CAP-001 | {reasoning} |
| ... | | | | |

#### Tier 3: Enhancement (polish and advanced features)
| ID | Capability | Acceptance Criteria | Dependencies | Reasoning |
|---|---|---|---|---|
| CAP-00N | {name} | {observable, testable condition} | CAP-003 | {reasoning} |
| ... | | | | |

Note: IDs are assigned sequentially (CAP-001, CAP-002, CAP-003, ...) regardless of tier. The tier is metadata on the capability, not encoded in the ID. Acceptance criteria in this table are summaries — full multi-line criteria will be elaborated in the spec (Phase 4).

> **Your turn**: For each capability — keep ✓, reject ✗, or modify. Add any I missed. Adjust tiers, priorities, or dependencies as needed.

### Development Configuration

1. **What's the max iterations budget?**
   > *My recommendation: {10-15} iterations — {reasoning based on number of capabilities and complexity}*

2. **Which files should be modifiable during development?**
   > *My recommendation: {list files from analysis with brief reason for each}*

3. **What verification command checks code health after changes?**
   > *My recommendation: `{lint_command}` — {detected from stack profile}*

4. **How should we verify new functionality works?**
   > *My recommendation: {test command, manual check, or specific verification approach based on what the codebase supports}*

5. **Are there files that must NEVER be modified?**
   > *My recommendation: {list critical files — schemas, migrations, test infrastructure, config, etc.}*

6. **Any architectural constraints or patterns new code must follow?**
   > *My recommendation: Based on my analysis, new code should follow: {patterns identified in Phase 1}*

---

Wait for the user to respond to the capability list and all questions at once, then proceed with their curated capabilities and answers (using your recommendations for any they don't override).

---

## Phase 3: Instrumentation

Based on the stack profile (Phase 0), feature analysis (Phase 1), and user interview (Phase 2), set up the verification and execution infrastructure.

### 3a: Select Verification Strategy

Based on the app type, determine how to verify that new capabilities work:

| App Type | Verification Pattern |
|---|---|
| Data/AI pipeline | Run pipeline step with test input, check output contains expected behavior |
| Web backend API | Start server + curl endpoint, or run integration test targeting new functionality |
| Web frontend | Build + serve + test framework to verify new UI behavior, or run component test |
| CLI tool | Run CLI with test args, verify new output/behavior in stdout/stderr |
| Mobile app | Build + simulator + UI test framework |
| Library | Run test harness or example script exercising new API surface |
| ML model | Run inference with test data, verify new model behavior |

Use whatever the user confirmed in Phase 2, or adapt based on what the codebase supports. Record the **verification command** template.

### 3b: Select Execution Strategy

Determine how to run the feature to exercise specific capabilities:

| Output Type | Execution Pattern |
|---|---|
| DB records | Query/insert script to verify new data handling |
| API response | curl/httpie to verify new endpoints or response fields |
| File output | Run and check output files for new content/format |
| Stdout/logs | Capture and parse CLI output for new behavior |
| UI render | Test framework to verify new UI elements |
| Model artifacts | Load and inspect model output for new capabilities |

Create any verification scripts/commands if they don't exist:
- **Write them in the project's native language** (TypeScript for Node.js projects, Python for Python projects, etc.)
- Follow existing test/script conventions in the project
- Each script must be able to verify a specific capability's acceptance criteria

Record the **execution command** template.

### 3c: Identify Build/Check Command

Use the detected lint/check command from the stack profile, confirmed by the user:

| Stack | Verify Command |
|---|---|
| Node.js/TypeScript | `npm run lint` or `npx tsc --noEmit` |
| Python | `ruff check .` or `mypy .` or `pylint` |
| Go | `go vet ./...` |
| Rust | `cargo check` or `cargo clippy` |
| Ruby | `bundle exec rubocop` |
| Java/Kotlin | `./gradlew check` or `mvn verify` |
| C#/.NET | `dotnet build --no-restore` |
| Generic | `make check` or skip if none found |

Record the **check command**.

### 3d: Record Instrumentation

Record the finalized:
- Verification command (how to verify a capability works)
- Execution command (how to run the feature)
- Check command (lint/build health)

Proceed directly to Phase 4 — the user will review everything together after all files are generated.

---

## Phase 4: Build Capability Specification

Create the capability specification file at `.claude/commands/spec-{feature}.md`.

### Structure:

```markdown
---
description: Capability specification for {feature} development
---

Capability specification for {feature description}. This file tracks all planned capabilities, their implementation status, dependencies, and acceptance criteria.

---

## Capability Specification: {feature}

### Summary
- **Total capabilities**: {N}
- **Tier 1 (Foundation)**: {N}
- **Tier 2 (Core)**: {N}
- **Tier 3 (Enhancement)**: {N}

### Configuration
- **Check command**: `{check_command}`
- **Global verification command**: `{verification_command}`
- **Execution command**: `{execution_command}`
- **Max iterations**: {N}
- **Modifiable files**: {list}
- **Protected files (never modify)**: {list}

### Dependency Graph

Use this format — indented list with arrows showing dependencies:
\`\`\`
CAP-001 (Tier 1)
CAP-002 (Tier 1) → CAP-001
CAP-003 (Tier 2) → CAP-001, CAP-002
CAP-004 (Tier 2) → CAP-003
CAP-005 (Tier 3) → CAP-004
\`\`\`

---

## Tier 1: Foundation

### CAP-001: {Capability Name}
- **Status**: [ ] pending
- **Priority**: {1-N within tier}
- **Dependencies**: none
- **Acceptance Criteria**:
  1. {Observable, testable condition}
  2. {Observable, testable condition}
- **Verification**: {How to verify this capability exists and works}
- **Files likely involved**: {paths}

### CAP-002: {Capability Name}
...

---

## Tier 2: Core

### CAP-003: {Capability Name}
- **Status**: [ ] pending
- **Priority**: {1-N within tier}
- **Dependencies**: CAP-001
- **Acceptance Criteria**:
  1. {Observable, testable condition}
  2. {Observable, testable condition}
- **Verification**: {How to verify this capability exists and works}
- **Files likely involved**: {paths}

...

---

## Tier 3: Enhancement

### CAP-00N: {Capability Name}
- **Status**: [ ] pending
- **Priority**: {1-N within tier}
- **Dependencies**: CAP-003
- **Acceptance Criteria**:
  1. {Observable, testable condition}
  2. {Observable, testable condition}
- **Verification**: {How to verify this capability exists and works}
- **Files likely involved**: {paths}

...

---

## Regression Checklist

Baseline capabilities (existing functionality that must not break):

1. {Existing behavior}: {how to verify it still works}
2. {Existing behavior}: {how to verify it still works}
...

---

## Status Legend
- `[ ]` — pending (not yet attempted)
- `[x]` — implemented (accepted and committed)
- `[!]` — attempted-failed (tried but could not implement successfully)
```

### Spec Design Principles:
- **10-20 capabilities** depending on feature scope (user curated in Phase 2)
- Organize into **3 tiers**: Foundation → Core → Enhancement
- Each capability must have **objectively verifiable** acceptance criteria (not subjective)
- **Dependencies must form a DAG** — no circular dependencies
- Include **verification instructions** specific to the capability
- The **regression checklist** captures ALL existing functionality as baseline tests
- Seed regression checklist from the current feature surface analysis (Phase 1)

---

## Phase 5: Build Auto-Develop Skill

Create the auto-develop skill at `.claude/commands/auto-develop-{feature}.md`.

Follow the **7-phase pattern** for iterative capability development:

```markdown
---
description: Iteratively develop new capabilities for {feature}
---

Automatically develop new capabilities for the {feature} in an iterative loop. Uses the capability specification as a development roadmap, implementing one capability per iteration and keeping only changes that pass verification and regression checks.

**IMPORTANT — Hands-off operation**: This skill is designed to run without user interaction. Do NOT use the Agent tool for implementations or any sub-tasks — perform all work (planning, implementation, verification, regression checking) directly in the main conversation. Do NOT ask the user for confirmation before running commands. Execute every phase autonomously until a stop condition is met.

---

## Phase 1: Initialize

1. Create the output directory structure:
\`\`\`bash
mkdir -p .debug/auto-develop-{feature}/logs .debug/auto-develop-{feature}/snapshots
\`\`\`

2. **If `.debug/auto-develop-{feature}/regression.json` already exists**, load it and skip regression initialization. Otherwise, create it from the spec's Regression Checklist:
\`\`\`json
{
  "baseline": [
    {
      "name": "{existing behavior}",
      "verify": "{verification command or check}",
      "source": "baseline"
    }
  ],
  "capabilities": [],
  "createdAt": "..."
}
\`\`\`

3. Load the capability spec from `.claude/commands/spec-{feature}.md`. Parse all capabilities and their statuses.

4. **If `.debug/auto-develop-{feature}/results.tsv` already exists**, load it, count existing iterations, and set `N` to continue from the last iteration number + 1. Otherwise, initialize with header row:
\`\`\`
iteration\tcapability\tstatus\tverification\tregression\tcommit\ttimestamp
\`\`\`
   and set iteration counter: `N = 1`

---

## Phase 2: Select Next Capability

1. Read the current spec from `.claude/commands/spec-{feature}.md`
2. Collect all capabilities with status `[ ]` (pending)
3. Filter to those whose **all dependencies** have status `[x]` (implemented)
4. From eligible capabilities, select by:
   - Tier 1 before Tier 2 before Tier 3
   - Within a tier, lowest priority number first
5. If no eligible capabilities exist:
   - If all are `[x]` or `[!]` → **STOP: all capabilities processed**
   - If some are `[ ]` but blocked by `[!]` dependencies → **STOP: remaining capabilities blocked**
6. Announce: `"Iteration {N}: Implementing CAP-{id} — {name}"`

---

## Phase 3: Plan Implementation

Before writing any code, create an implementation plan:

1. Read the capability's acceptance criteria and verification instructions from the spec
2. Read ALL files listed in "Files likely involved" for this capability
3. Read adjacent feature implementations for patterns to follow
4. Read any relevant existing tests or examples

Produce a brief implementation plan and save to `.debug/auto-develop-{feature}/logs/iter{N}_plan.md`:

\`\`\`markdown
## Implementation Plan: CAP-{id} — {name}

### Acceptance Criteria
{list from spec}

### Approach
{2-5 sentences describing the implementation strategy}

### Files to Modify
- {path}: {what changes}

### Files to Create (if any)
- {path}: {what it does}

### Risks
- {potential issues and mitigations}
\`\`\`

This planning step is mandatory — it reduces implementation failures by forcing structured thinking before coding.

---

## Phase 4: Implement

Execute the implementation plan:

1. Make code changes following codebase patterns strictly:
   - Use the same code style, naming conventions, and patterns as existing code
   - Follow the architectural patterns identified in the spec
   - Import from the same locations as similar features
   - Handle errors the same way adjacent code does

2. Run the check command after changes:
\`\`\`bash
{check_command}
\`\`\`

3. If the check command fails:
   - Fix the issue
   - Re-run the check
   - **Maximum 3 fix attempts** — if still failing after 3 attempts, save a diff patch (`git diff HEAD > .debug/auto-develop-{feature}/logs/iter{N}_{cap_id}_diff.patch`), revert all changes (`git checkout -- . && git clean -fd --exclude=.debug/`), and mark the capability as `[!]` in the spec, then return to Phase 2

4. **NEVER modify**:
   - The capability spec (`.claude/commands/spec-{feature}.md`) — except status updates in Phase 7
   - The regression suite (`.debug/auto-develop-{feature}/regression.json`) — except additions in Phase 7
   - Test infrastructure, CI configuration, or build scripts
   - Files listed as "never modify" in the spec

   **Note**: Creating new test data, fixtures, or sample inputs for new capabilities IS allowed and often necessary. This is distinct from modifying existing test infrastructure.

---

## Phase 5: Verify Capability

Run the acceptance criteria checks for the specific capability:

1. Read the capability's acceptance criteria AND its **Verification** field from the spec
2. Execute the capability-specific verification command from the spec's **Verification** field. If the capability does not specify its own verification, fall back to the global verification command:
\`\`\`bash
{capability-specific verification from spec, or global verification_command as fallback}
\`\`\`
3. For each acceptance criterion, check whether the output/behavior satisfies it
4. Record results to `.debug/auto-develop-{feature}/snapshots/iter{N}_{cap_id}_verify.md`:

\`\`\`markdown
## Verification: CAP-{id} — {name}

### Criterion 1: {description}
- **Result**: PASS/FAIL
- **Evidence**: {what was observed}

### Criterion 2: {description}
- **Result**: PASS/FAIL
- **Evidence**: {what was observed}

### Overall: PASS/FAIL ({passed}/{total} criteria met)
\`\`\`

5. **Overall PASS** requires ALL acceptance criteria to pass

---

## Phase 6: Regression Check

Run ALL items in the regression suite to verify existing functionality is not broken:

1. Load `.debug/auto-develop-{feature}/regression.json`
2. For each item (both baseline and previously-accepted capabilities):
   - Run its verification command/check
   - Record PASS/FAIL with evidence
3. Save results to `.debug/auto-develop-{feature}/snapshots/iter{N}_regression.md`:

\`\`\`markdown
## Regression Check: Iteration {N} (after CAP-{id})

### Baseline
1. {name}: PASS/FAIL — {evidence}
2. ...

### Previously Accepted Capabilities
1. CAP-{id} — {name}: PASS/FAIL — {evidence}
2. ...

### Overall: PASS/FAIL ({passed}/{total} checks passed)
\`\`\`

4. **Overall PASS** requires ALL regression items to pass

---

## Phase 7: Accept/Reject

### If verification PASS AND regression PASS → ACCEPT

1. Stage only the files from the implementation plan (Files to Modify + Files to Create) and commit:
\`\`\`bash
git add {files from plan} && git commit -m "auto-develop({feature}): implement CAP-{id} — {name}"
\`\`\`
   Do NOT use `git add -A` — this risks staging unrelated files (.debug/ logs, editor temp files, etc.). Stage only the specific files that were modified or created for this capability.

2. Update the spec: change capability status from `[ ]` to `[x]`:
   - Read `.claude/commands/spec-{feature}.md`
   - Change `- **Status**: [ ] pending` to `- **Status**: [x] implemented`

3. Add the capability to the regression suite:
\`\`\`json
{
  "id": "CAP-{id}",
  "name": "{name}",
  "verify": "{verification command for this capability}",
  "criteria": ["{criterion 1}", "{criterion 2}"],
  "source": "auto-develop",
  "iteration": {N}
}
\`\`\`
   Append this to the `"capabilities"` array in `regression.json`.

4. Append to `results.tsv`:
\`\`\`
{N}\tCAP-{id}\tACCEPTED\tPASS\tPASS\t{commit_hash}\t{timestamp}
\`\`\`

### If verification FAIL → REJECT (capability not feasible)

1. Save a patch of the attempted changes for future reference:
\`\`\`bash
git diff HEAD > .debug/auto-develop-{feature}/logs/iter{N}_{cap_id}_diff.patch
\`\`\`

2. Revert all changes (both tracked and untracked files, excluding .debug/):
\`\`\`bash
git checkout -- . && git clean -fd --exclude=.debug/
\`\`\`

3. Update the spec: change capability status from `[ ]` to `[!]`:
   - Read `.claude/commands/spec-{feature}.md`
   - Change `- **Status**: [ ] pending` to `- **Status**: [!] attempted-failed`

4. Append to `results.tsv`:
\`\`\`
{N}\tCAP-{id}\tREJECTED-VERIFY\tFAIL\tN/A\tnone\t{timestamp}
\`\`\`

### If verification PASS but regression FAIL → REJECT (breaks existing code)

1. Save a patch of the attempted changes for future reference:
\`\`\`bash
git diff HEAD > .debug/auto-develop-{feature}/logs/iter{N}_{cap_id}_diff.patch
\`\`\`

2. Revert all changes (both tracked and untracked files, excluding .debug/):
\`\`\`bash
git checkout -- . && git clean -fd --exclude=.debug/
\`\`\`

3. Keep status as `[ ]` (will retry with different approach)

4. Check `results.tsv` for previous `REJECTED-REGRESSION` entries with this same capability ID. If one already exists, this is the second regression failure — change status to `[!]` (attempted-failed) instead of keeping `[ ]`.

5. Append to `results.tsv`:
\`\`\`
{N}\tCAP-{id}\tREJECTED-REGRESSION\tPASS\tFAIL\tnone\t{timestamp}
\`\`\`

### After accept or reject, increment iteration counter and return to Phase 2.

---

## Stop Conditions

Stop the loop if ANY of these conditions are met:

1. **All done**: All targeted-tier capabilities are `[x]` (implemented) or `[!]` (attempted-failed)
2. **Budget exhausted**: {max_iterations} iterations completed
3. **Stuck**: 2 consecutive rejections on **different** capabilities
4. **Blocked**: All remaining `[ ]` capabilities are blocked by `[!]` dependencies

On exit, print final summary:

\`\`\`
## Auto-Develop Complete: {feature}

### Results
- Iterations: {N}
- Capabilities implemented: {count} / {total}
- Capabilities failed: {count}
- Capabilities remaining: {count}

### Implemented Capabilities
| ID | Name | Iteration | Commit |
|---|---|---|---|
| CAP-{id} | {name} | {iter} | {hash} |
| ... | | | |

### Failed Capabilities
| ID | Name | Reason |
|---|---|---|
| CAP-{id} | {name} | {verification fail / regression fail x2} |
| ... | | |

### Remaining Capabilities
| ID | Name | Blocked By |
|---|---|---|
| CAP-{id} | {name} | {dependency that is [!]} |
| ... | | |

### Regression Suite
- Total checks: {N} (baseline: {N}, capabilities: {N})
- All passing: Yes/No
\`\`\`

---

## Important Notes
- Each iteration implements ONE capability — isolate changes to understand impact
- The capability spec is the source of truth — never invent capabilities not in the spec
- Plan before implementing — the planning phase is mandatory, not optional
- Binary accept/reject — no partial implementations
- New code must follow existing codebase patterns strictly
- The regression suite only grows — never remove items from it
- If blocked, report clearly and stop — don't force implementations
- As capabilities accumulate, regression checks grow. If regression checks become slow (15+ items), consider consolidating related checks into a single test suite command rather than running individual verification commands
- Creating new test data, fixtures, or sample inputs for new capabilities IS allowed — this is not the same as modifying test infrastructure
```

---

## Phase 6: Review & Confirmation

After generating both files, present a **single consolidated review** to the user and ask for feedback once:

```
## Auto-Develop Skill Created: {feature}

### Files Created
- `.claude/commands/spec-{feature}.md` — {N} capabilities across 3 tiers ({T1} foundation, {T2} core, {T3} enhancement)
- `.claude/commands/auto-develop-{feature}.md` — iterative development loop
- `{verification_script_path}` — capability verification (if created)

### Commands Configured
- **Execute**: `{execution_command}`
- **Verify**: `{verification_command}`
- **Check**: `{check_command}`

### Configuration
- Capabilities: {N} total ({T1} foundation, {T2} core, {T3} enhancement)
- Dependency graph: {brief summary}
- Max iterations: {N}
- Modifiable files: {list}
- Protected files: {list}
- Regression baseline: {N} existing behaviors tracked
```

Then ask **one question**:

> "Here's everything I've generated. Want me to change anything — capabilities, acceptance criteria, dependencies, tiers, commands, modifiable files, or anything else? If it looks good, I'll run a validation test next."

Apply any user feedback, then proceed to Phase 7.

---

## Phase 7: Validation Run

After user confirmation:

1. Run the check command to verify code health baseline
2. Run the verification command on one existing behavior to verify it produces checkable output
3. Verify all regression baseline items can be checked successfully
4. Verify the capability spec parses correctly (all statuses, dependencies, criteria readable)
5. Report any issues found and **fix them automatically**

If any step fails, diagnose the issue, fix it, and re-run until the full pipeline works end-to-end.

---

## Phase 8: Smoke Test — Run the Auto-Develop Skill

After validation passes, run the actual auto-develop skill for **1 iteration** to verify the full loop works:

1. Execute: `/auto-develop-{feature}` with max iterations set to 1 (override the configured max for this test run only)
2. Verify that:
   - Capability selection works (correct capability chosen based on tier + dependencies)
   - Implementation planning produces a coherent plan
   - Implementation follows codebase patterns
   - Check command passes after implementation
   - Verification correctly evaluates acceptance criteria
   - Regression check runs all baseline items
   - Accept/reject logic works correctly
   - Spec status is updated appropriately
   - Regression suite is updated on accept
   - Results are written to results.tsv
3. If the smoke test fails at any step, diagnose and fix the issue in the generated skill files, then re-run
4. Once the full loop completes successfully, report:

```
## Smoke Test Passed ✓

### Results
- Capability attempted: CAP-{id} — {name}
- Result: {ACCEPTED/REJECTED} — {reason}
- Regression suite: {N} items, all passing
- All artifacts written to `.debug/auto-develop-{feature}/`

### Ready to Use
Run `/auto-develop-{feature}` to start the full development loop.
```

If the smoke test reveals issues with the spec or skill that require user input, present the issues and ask for guidance before fixing.
