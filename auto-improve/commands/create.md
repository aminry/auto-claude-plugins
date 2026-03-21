---
description: Create a custom auto-improve loop for any feature in your codebase
---

Create a custom auto-improve skill for a specific feature in the application. Analyzes the feature, interviews the user to understand quality goals, builds an evaluation rubric, instruments code if needed, and generates a complete auto-improve skill.

The feature to analyze: $ARGUMENTS

---

## Overview

This skill creates two files:
1. **An evaluation rubric** (`.claude/commands/evaluate-{feature}.md`) — the fitness function
2. **An auto-improve skill** (`.claude/commands/auto-improve-{feature}.md`) — the iterative improvement loop

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

2. **Data flow mapping**: Trace the complete path:
   - What inputs does the feature receive? (DB records, user input, API data, files, CLI args, other pipeline outputs)
   - What transformations happen? (LLM calls, data processing, validation, filtering)
   - What outputs does it produce? (DB writes, files, API responses, stdout, rendered UI)
   - What side effects does it have?

3. **Quality dimensions**: From the code, identify what aspects of quality matter:
   - Correctness (does it produce factually accurate output?)
   - Completeness (does it cover all expected cases?)
   - Consistency (is output format/style uniform?)
   - Relevance (is output appropriate for the context?)
   - Performance characteristics (speed, token usage, API costs)

4. **Existing extraction / test capability**: Check what already exists:
   - Scripts, Makefile targets, or CLI commands that can extract this feature's output
   - Integration tests, API tests, component tests that exercise the feature
   - Test fixtures, test data, seed files, sample inputs
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
- **Prompts/Config**: {path} — {what it does}
- ...

### Data Flow
Input: {description} → Processing: {description} → Output: {description}

### Quality Dimensions Identified
1. {dimension}: {why it matters}
2. ...

### Existing Infrastructure
- Extraction method: {exists/needs creation — describe what's available}
- Isolated execution: {how to run for a single item}
- Test data: {what's available}
```

Then proceed to Phase 2.

---

## Phase 2: User Interview (Single Round)

Based on your analysis from Phases 0-1, ask ALL questions in a **single message**. For each question, provide your **recommended answer** based on your analysis — the user can accept, modify, or override any recommendation. This avoids multiple back-and-forth rounds.

Present the questions in this format:

---

**I've completed my analysis. Here are all the questions I need answered to build your auto-improve skill. I've included my recommended answer for each based on what I found — just confirm, adjust, or override as needed.**

### Quality Goals

1. **What does good output look like for this feature?**
   > *My recommendation: {describe ideal output based on your code analysis, existing tests, and observed patterns}*

2. **What are the most common problems with current output?**
   > *My recommendation: Based on my analysis, likely pain points are: {list issues you identified from code patterns, TODOs, error handling gaps, etc.}*

3. **Are there hard requirements — things that must ALWAYS or NEVER happen?**
   > *My recommendation: {list constraints you inferred from validation logic, error checks, type constraints, etc.}*

### Evaluation Approach

4. **How should we test this? What items should we use as test cases?**
   > *My recommendation: {N} test items — {describe specific items or selection criteria based on existing test data, fixtures, or DB contents you found}*

5. **How do we run this feature in isolation?**
   > *My recommendation: `{command}` — {explain why, based on existing scripts, CLI entry points, test commands, or API endpoints you found}*

6. **How do we capture the output?**
   > *My recommendation: {extraction method} — {explain based on how the feature produces output: stdout, files, DB, API response, etc.}*

7. **What verification command checks code health after changes?**
   > *My recommendation: `{lint_command}` — {detected from stack profile}*

8. **Which quality dimensions matter most?**
   > *My recommendation: I identified these dimensions: [{list}]. I'd prioritize: {top 3-4 with reasoning}. Any I missed?*

9. **What score threshold should we target?**
   > *My recommendation: 90% — this is the sweet spot for most auto-improve loops (high enough to drive real improvement, achievable within iteration budget)*

### Improvement Scope

10. **Which files should be modifiable during auto-improvement?**
    > *My recommendation: {list files from analysis with brief reason for each}*

11. **What's the max iterations budget?**
    > *My recommendation: {5-8} iterations — {reasoning based on feature complexity}*

12. **Any known failure patterns to include as hints?**
    > *My recommendation: {list patterns you spotted from code analysis, or "None identified — we'll discover them during the first run"}*

---

Wait for the user to respond to all questions at once, then proceed with their answers (using your recommendations for any they don't override).

---

## Phase 3: Instrumentation

Based on the stack profile (Phase 0), feature analysis (Phase 1), and user interview (Phase 2), set up the extraction and execution infrastructure.

### 3a: Select Execution Strategy

Based on the app type, determine how to run the feature for a single test item:

| App Type | Execution Pattern |
|---|---|
| Data/AI pipeline | Run pipeline step: `{runner} {step} <id> [--force]` |
| Web backend API | Start server + curl/httpie endpoint, or run integration test |
| Web frontend | Build + serve + Playwright/Puppeteer capture, or run component test |
| CLI tool | Run CLI with test args, capture stdout/stderr |
| Mobile app | Build + simulator + UI test framework |
| Library | Run test harness or example script |
| ML model | Run training/inference script with test data |

Use whatever the user confirmed in Phase 2, or adapt based on what the codebase supports. Record the **execution command** template.

### 3b: Select Extraction Strategy

Based on the feature's output type, determine how to capture results for evaluation:

| Output Type | Extraction Pattern |
|---|---|
| DB records | Query script (SQL, ORM, or language-native — write in the project's language) |
| API response | curl/httpie capture to JSON |
| File output | Read output files directly |
| Stdout/logs | Capture and parse CLI output |
| UI render | Screenshot + DOM extraction via test framework |
| Model artifacts | Load and inspect model output files |

Create the extraction script/command if one doesn't exist:
- **Write it in the project's native language** (TypeScript for Node.js projects, Python for Python projects, Go for Go projects, shell script if simplest, etc.)
- If the project already has similar extraction patterns (existing scripts, test utilities, CLI commands), follow those conventions
- The script/command must:
  - Accept a test item identifier as argument (whatever the natural unit is — ID, filename, endpoint, etc.)
  - Output clean JSON to stdout (suitable for evaluation)
  - Handle edge cases (missing data, empty results)

Record the **extraction command** template.

### 3c: Identify Verification Command

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

Record the **verify command**.

### 3d: Record Instrumentation

Record the finalized:
- Execution command (how to run the feature for one item)
- Extraction script/command (and its location, if created)
- Verification command

Proceed directly to Phase 4 — the user will review everything together after all files are generated.

---

## Phase 4: Build Evaluation Rubric

Create the evaluation rubric file at `.claude/commands/evaluate-{feature}.md`.

### Structure (follow the exact pattern of existing evaluate-*.md files if any exist, otherwise use this template):

```markdown
Evaluate the quality of {feature description} using a structured rubric.

First, extract the data by running:

\`\`\`bash
{extraction_command} $ARGUMENTS
\`\`\`

If the command fails, report the error and stop.

Parse the JSON output, then evaluate against ALL {N} questions below. For each question, examine the actual data carefully and provide evidence for your answer.

---

## Evaluation Rubric

### A. {Category Name} ({count} questions)
1. {Yes/No question with clear pass/fail criteria and evidence requirement}
2. ...

### B. {Category Name} ({count} questions)
...

---

## Scoring

- Each Yes/No question: Yes = 1 point, No = 0 points
- Multiple choice questions: specify point values
- N/A questions: excluded from total
- **Overall score** = (points earned / applicable points) x 100
- Grade: A (90-100), B (80-89), C (70-79), D (60-69), F (<60)

## Output Format

Use this exact format:

\`\`\`
## {Feature} Evaluation: "{title}"
### ID: {id}

### A. {Category} ({points}/{applicable})
  1. ✅/❌ {question}: {answer} ({evidence})
  ...

---
### Overall Score: {earned}/{applicable} = {percentage}/100 (Grade: {letter})

### Summary
- **Strengths**: [2-3 top strengths]
- **Issues found**: [list all failures with question numbers]
- **Recommendations**: [2-3 actionable improvements]
\`\`\`

Be thorough and evidence-based. Quote specific data when flagging issues. Do not be lenient — apply the rubric strictly.
```

### Rubric Design Principles:
- **40-100 questions** depending on feature complexity
- Group into **5-10 categories** (A through J) by quality dimension
- Each question must be **objectively answerable** from the extracted data (no subjective judgment)
- Include **evidence requirements** ("Quote any found", "State the count", "List examples")
- Mark questions that may be **N/A** in certain contexts
- Include both **structural checks** (does X exist?) and **quality checks** (is X good?)
- Weight categories by importance (more questions = more weight)

### App-Type-Specific Quality Dimensions to Consider:

**Web backend / API**:
- Response schema correctness, status codes, error handling
- Pagination, rate limiting, authentication
- Latency, payload size

**Web frontend / UI**:
- Accessibility (ARIA, keyboard nav, screen reader)
- Responsive layout, loading states, error states
- Visual consistency, design system compliance

**Data pipeline**:
- Data completeness, schema conformance, row counts
- Deduplication, freshness, null/missing value rates
- Transformation accuracy

**ML / AI pipeline**:
- Accuracy metrics, bias checks, confidence calibration
- Edge case coverage, prompt quality
- Output format consistency

**CLI tool**:
- Exit codes, help text, error messages
- Output formatting, machine-parseable output
- Flag handling, input validation

**Library / SDK**:
- API surface correctness, type safety
- Documentation accuracy, example correctness
- Backward compatibility

### General Quality Checks to Always Consider:
- Completeness: Are all expected fields populated?
- Correctness: Are values factually accurate and consistent?
- Relevance: Is the output appropriate for the input context?
- Formatting: Are formats consistent and clean?
- No artifacts: No debug text, placeholders, instruction remnants?
- No duplication: Are outputs unique and not repetitive?
- Domain-appropriate: Does the output demonstrate domain knowledge?

---

## Phase 5: Build Auto-Improve Skill

Create the auto-improve skill at `.claude/commands/auto-improve-{feature}.md`.

Follow the **exact 6-phase pattern** used by existing auto-improve skills. The structure must be:

```markdown
Automatically improve the {feature} pipeline in an iterative loop. Uses the {N}-question evaluation rubric as a fitness function, making one targeted fix per iteration and keeping only changes that improve scores.

**IMPORTANT — Hands-off operation**: This skill is designed to run without user interaction. Do NOT use the Agent tool for evaluations or any sub-tasks — perform all work (generation, extraction, evaluation, scoring, code changes) directly in the main conversation. Do NOT ask the user for confirmation before running commands. Execute every phase autonomously until a stop condition is met.

---

## Phase 1: Initialize

1. Create the output directory structure:
\`\`\`bash
mkdir -p .debug/auto-improve-{feature}/logs .debug/auto-improve-{feature}/evaluations .debug/auto-improve-{feature}/extractions
\`\`\`

2. **If `.debug/auto-improve-{feature}/test-set.json` already exists**, load it and skip to Phase 2. Otherwise, continue with test set selection.

3. {Query/discovery instructions for finding test items — adapted from user interview and app type}

4. Select **{N} test items** that meet ALL criteria:
   - {Diversity criteria from interview}
   - {Size/complexity criteria}
   - {Domain spread criteria}

5. Write to `.debug/auto-improve-{feature}/test-set.json`:
\`\`\`json
{
  "{items}": [
    { "id": ..., ... }
  ],
  "selectedAt": "..."
}
\`\`\`

6. Initialize `.debug/auto-improve-{feature}/results.tsv` with header row.

---

## Phase 2: Generate & Evaluate

For each test item:

### Step 2a: Run the feature
\`\`\`bash
{execution_command} 2>&1 | tee .debug/auto-improve-{feature}/logs/iter<N>_<item>.log
\`\`\`

### Step 2b: Extract results
\`\`\`bash
{extraction_command} > .debug/auto-improve-{feature}/extractions/iter<N>_<item>.json
\`\`\`

### Step 2c: Evaluate inline
Read the extracted JSON. Apply the {N}-question rubric from `.claude/commands/evaluate-{feature}.md`. Output compact summary.

### Step 2d: Save evaluation
Save to `.debug/auto-improve-{feature}/evaluations/iter<N>_<item>.md`

### Step 2e: Record scores
Append to `.debug/auto-improve-{feature}/results.tsv`

---

## Phase 3: Analyze & Prioritize

1. Aggregate failures across all test items
2. Classify by fixability:
   - Code bug (1.0)
   - Prompt issue (0.7)
   - Config/data issue (0.5)
3. Rank: `impact = frequency × (points_at_stake / total_points) × fixability`
4. Select the single highest-impact failure cluster

---

## Phase 4: Implement Fix

Read relevant source files, make a **single focused change**.

**Allowed files** (priority order):
{List from user interview, ordered by likelihood of containing fixes}

**NEVER modify**: {schema, test infra, evaluation rubric, etc.}

After changes, verify: `{verify_command}`

### Known fix targets
{Table of known failures from interview and analysis}

---

## Phase 5: Compare & Decide

Re-generate and re-evaluate all test items, then:
- `mean_before` vs `mean_after`
- `min_before` vs `min_after`
- **ACCEPT** if: `mean_after > mean_before` AND `min_after >= min_before - {floor_tolerance}`
- **REJECT** if: mean didn't improve OR floor dropped too much
- On accept: git commit with descriptive message
- On reject: `git checkout -- <files>` to revert

---

## Phase 6: Loop Control

Stop if:
- Mean score ≥ {target}% (target reached!)
- {max_iterations} iterations completed
- 2 consecutive rejections on **different** failure clusters

On exit, print final summary with score progression table, commits, remaining failures.

---

## Important Notes
- Each iteration runs {N} test items — be deliberate about which fix to try
- The evaluation rubric is the source of truth — apply strictly
- One fix at a time — isolate changes to understand impact
- Binary keep/discard — no partial reverts
- Fixes should be generalizable, not specific to a single test item
```

---

## Phase 6: Review & Confirmation

After generating both files, present a **single consolidated review** to the user and ask for feedback once:

```
## Auto-Improve Skill Created: {feature}

### Files Created
- `.claude/commands/evaluate-{feature}.md` — {N}-question rubric ({list categories with counts})
- `.claude/commands/auto-improve-{feature}.md` — iterative improvement loop
- `{extraction_script_path}` — data extraction (if created)

### Commands Configured
- **Execute**: `{execution_command}`
- **Extract**: `{extraction_command}`
- **Verify**: `{verify_command}`

### Configuration
- Test items: {N} items, {diversity criteria}
- Target score: {X}%
- Max iterations: {N}
- Modifiable files: {list}
- Floor tolerance: {value}
```

Then ask **one question**:

> "Here's everything I've generated. Want me to change anything — rubric questions, modifiable files, iteration count, commands, or anything else? If it looks good, I'll run a validation test next."

Apply any user feedback, then proceed to Phase 7.

---

## Phase 7: Validation Run

After user confirmation:

1. Run the execution command on one real item to verify it works
2. Run the extraction command on that item to verify it produces parseable JSON
3. Run the evaluation rubric on that extracted data to verify all questions are answerable
4. Run the verification command to confirm it passes
5. Report any issues found and **fix them automatically**

If any step fails, diagnose the issue, fix it, and re-run until the full pipeline works end-to-end.

---

## Phase 8: Smoke Test — Run the Auto-Improve Skill

After validation passes, run the actual auto-improve skill for **1 iteration** to verify the full loop works:

1. Execute: `/auto-improve-{feature}` with max iterations set to 1 (override the configured max for this test run only)
2. Verify that:
   - Test set selection works (items are found and written to test-set.json)
   - Feature execution completes for all test items
   - Extraction produces valid JSON for all test items
   - Evaluation scores all test items without errors
   - The analyze & prioritize phase identifies failures (if any exist)
   - If a fix is attempted: the compare & decide phase correctly accepts or rejects it
   - Results are written to results.tsv
3. If the smoke test fails at any step, diagnose and fix the issue in the generated skill files, then re-run
4. Once the full loop completes successfully, report:

```
## Smoke Test Passed ✓

### Results
- Test items processed: {N}/{N}
- Initial mean score: {X}%
- Iterations completed: 1
- Fix attempted: {Yes/No} — {accepted/rejected/no fix needed}
- All artifacts written to `.debug/auto-improve-{feature}/`

### Ready to Use
Run `/auto-improve-{feature}` to start the full improvement loop.
```

If the smoke test reveals issues with the rubric or skill that require user input, present the issues and ask for guidance before fixing.
