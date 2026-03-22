# auto-develop

A Claude Code plugin that creates custom auto-develop loops for any feature in any codebase. It analyzes your feature, interviews you to discover and curate new capabilities, builds a capability specification, instruments code for verification, and generates a complete auto-develop skill — all tailored to your project's tech stack.

## What it does

This plugin provides two slash commands:

### `/auto-develop:create <feature>`

Walks you through an 8-phase process to generate:

1. **A capability specification** (`.claude/commands/spec-{feature}.md`) — an enumerated, tiered list of new capabilities with acceptance criteria, dependencies, and status tracking
2. **An auto-develop skill** (`.claude/commands/auto-develop-{feature}.md`) — an autonomous iterative loop that selects the next eligible capability, plans the implementation, writes code, verifies acceptance criteria, runs regression checks, and keeps only changes that pass both

The process:

| Phase | What happens |
|-------|-------------|
| 0. Stack Detection | Scans your project for language, framework, build system, test runner, linter |
| 1. Feature Analysis | Maps all files, current surface, extension points, architecture constraints, adjacent patterns |
| 2. Capability Discovery | Proposes 10-20 capabilities across 3 tiers — you curate: accept, reject, re-tier, add missing |
| 3. Instrumentation | Creates verification scripts and execution commands in your project's native language |
| 4. Build Spec | Generates the capability specification with dependency graph and regression baseline |
| 5. Build Auto-Develop | Generates the iterative development skill |
| 6. Review & Confirmation | Presents everything for a single round of feedback |
| 7. Validation | Verifies the full pipeline works end-to-end |
| 8. Smoke Test | Runs the auto-develop skill for 1 iteration to confirm the loop works |

### `/auto-develop:evolve-spec <feature>`

After running an auto-develop cycle, the spec may have room to grow. This command:

1. Reads the latest auto-develop results and implementation logs
2. Analyzes what was built and what new possibilities emerged
3. Identifies natural follow-ons, infrastructure-enabled capabilities, and retry candidates
4. Proposes additions and gets your approval
5. Updates the spec

## Stack support

The plugin auto-detects your project's tech stack and adapts accordingly:

| Stack | Execution | Verification | Check |
|-------|-----------|--------------|-------|
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
    "aminry/auto-develop"
  ]
}
```

Or clone locally and point to the path:

```json
{
  "plugins": [
    "/path/to/auto-develop"
  ]
}
```

## Usage

```
# Create an auto-develop loop for a feature
/auto-develop:create user-auth

# After running auto-develop, evolve the spec
/auto-develop:evolve-spec user-auth

# Run the generated auto-develop skill
/auto-develop-user-auth
```

## How auto-develop loops work

Once `/auto-develop:create` generates your skill, running `/auto-develop-{feature}` starts an autonomous loop:

```
Initialize → Select Capability → Plan → Implement → Verify → Regression Check → Accept/Reject → Loop
                                                                                       ↑            |
                                                                                       └────────────┘
                                                                                        (until all done
                                                                                         or max iterations)
```

Each iteration:
- Selects the next eligible capability (by tier, then priority, respecting dependencies)
- Plans the implementation before writing code
- Implements following codebase patterns strictly
- Verifies all acceptance criteria pass
- Runs the full regression suite (baseline + all previously-accepted capabilities)
- Keeps the change only if both verification and regression pass (git commit), otherwise reverts

## auto-develop vs auto-improve

| Aspect | auto-improve | auto-develop |
|--------|-------------|-------------|
| Purpose | Improve quality of existing features | Add new functionality to features |
| Source of truth | Evaluation rubric (yes/no questions) | Capability spec (enumerated capabilities) |
| Loop action | Fix one failure cluster | Implement one capability |
| Selection | Highest-impact failure | Highest-priority eligible capability |
| Evaluation | Score N items against universal rubric | Verify specific acceptance criteria |
| Regression | Mean/floor score comparison | Explicit regression suite that grows |
| Planning | None (fixes are small) | Required (implementations are complex) |
| Accept condition | Mean improves AND floor holds | Verification passes AND regressions pass |

## Inspiration

This plugin was inspired by [autoresearch](https://github.com/karpathy/autoresearch) by Andrej Karpathy — the idea of using automated iterative loops to continuously expand functionality, and by the auto-improve plugin's approach to structured autonomous development.

## License

MIT
