# auto-improve-factory

A Claude Code plugin that creates custom auto-improve loops for any feature in any codebase. It analyzes your feature, interviews you to understand quality goals, builds an evaluation rubric, instruments code for isolated execution, and generates a complete auto-improve skill — all tailored to your project's tech stack.

## What it does

This plugin provides two slash commands:

### `/auto-improve:create <feature>`

Walks you through an 8-phase process to generate:

1. **An evaluation rubric** (`.claude/commands/evaluate-{feature}.md`) — a structured scoring system with 40-100 yes/no questions grouped by quality dimension
2. **An auto-improve skill** (`.claude/commands/auto-improve-{feature}.md`) — an autonomous iterative loop that runs the feature, evaluates output against the rubric, identifies the highest-impact failure, makes a single fix, and keeps only changes that improve scores

The process:

| Phase | What happens |
|-------|-------------|
| 0. Stack Detection | Scans your project for language, framework, build system, test runner, linter |
| 1. Feature Analysis | Maps all files, data flow, quality dimensions, and existing test infrastructure |
| 2. User Interview | Single round of all questions with recommended answers — confirm, adjust, or override |
| 3. Instrumentation | Creates extraction scripts and execution commands in your project's native language |
| 4. Build Rubric | Generates the evaluation rubric with app-type-aware quality dimensions |
| 5. Build Auto-Improve | Generates the iterative improvement skill |
| 6. Review & Confirmation | Presents everything for a single round of feedback |
| 7. Validation | Verifies the full pipeline works end-to-end |
| 8. Smoke Test | Runs the auto-improve skill for 1 iteration to confirm the loop works |

### `/auto-improve:evolve-rubric <feature>`

After running an auto-improve cycle, the rubric may have blind spots. This command:

1. Reads the latest auto-improve results
2. Performs a rubric-independent quality analysis
3. Identifies gaps where real issues exist but no rubric question catches them
4. Proposes new questions and gets your approval
5. Updates the rubric

## Stack support

The plugin auto-detects your project's tech stack and adapts accordingly:

| Stack | Execution | Extraction | Verification |
|-------|-----------|------------|--------------|
| Node.js / TypeScript | `npm run` / `npx` | TypeScript scripts | `npm run lint` / `tsc --noEmit` |
| Python | `python` / `pytest` | Python scripts | `ruff check` / `mypy` |
| Go | `go run` / `go test` | Go scripts | `go vet ./...` |
| Rust | `cargo run` / `cargo test` | Rust scripts | `cargo check` / `cargo clippy` |
| Ruby | `bundle exec` / `rake` | Ruby scripts | `bundle exec rubocop` |
| Java / Kotlin | `gradle` / `mvn` | JVM scripts | `./gradlew check` / `mvn verify` |

App types: web backend, web frontend, full-stack, CLI, data pipeline, mobile, ML/AI pipeline, library.

## Installation

Add to your project's `.claude/settings.json`:

```json
{
  "plugins": [
    "aminry/auto-improve-factory"
  ]
}
```

Or clone locally and point to the path:

```json
{
  "plugins": [
    "/path/to/auto-improve-factory"
  ]
}
```

## Usage

```
# Create an auto-improve loop for a feature
/auto-improve:create product-search

# After running auto-improve, evolve the rubric
/auto-improve:evolve-rubric product-search

# Run the generated auto-improve skill
/auto-improve-product-search
```

## How auto-improve loops work

Once `/auto-improve:create` generates your skill, running `/auto-improve-{feature}` starts an autonomous loop:

```
Initialize → Generate & Evaluate → Analyze Failures → Implement Fix → Compare & Decide → Loop
                                                            ↑                    |
                                                            └────────────────────┘
                                                              (until target score
                                                               or max iterations)
```

Each iteration:
- Runs the feature on N test items
- Evaluates each against the rubric
- Aggregates failures and picks the highest-impact cluster
- Makes a single focused code change
- Re-evaluates to measure impact
- Keeps the change only if scores improve (git commit), otherwise reverts

## Inspiration

This plugin was inspired by [autoresearch](https://github.com/karpathy/autoresearch) by Andrej Karpathy — the idea of using automated iterative loops with evaluation rubrics to continuously improve output quality.

## License

MIT
