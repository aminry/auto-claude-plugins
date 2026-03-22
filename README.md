# auto-claude-plugins

A marketplace of Claude Code plugins for automation, quality, and productivity. These plugins create autonomous iterative loops that analyze your codebase, interview you once, then run hands-off — making changes, evaluating results, and keeping only what works.

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add aminry/auto-claude-plugins

# List available plugins
/plugin list

# Install a plugin
/plugin install auto-improve@auto-claude-plugins
/plugin install auto-develop@auto-claude-plugins
```

## Available Plugins

| Plugin | Description | Version |
|--------|-------------|---------|
| [auto-improve](./.claude-plugin/plugins/auto-improve/) | Creates custom auto-improve loops for any feature — builds evaluation rubrics and generates autonomous iterative improvement skills | 1.0.1 |
| [auto-develop](./.claude-plugin/plugins/auto-develop/) | Creates custom auto-develop loops for any feature — builds capability specs and generates autonomous iterative development skills that add new functionality | 1.0.0 |

### auto-improve

Improves the **quality** of existing features. Builds an evaluation rubric (40-100 yes/no questions), then runs a loop that scores output, identifies the highest-impact failure cluster, makes a single fix, and keeps only changes that improve scores.

```
/auto-improve:create <feature>        # Build rubric + improvement loop
/auto-improve:evolve-rubric <feature> # Expand rubric after running cycles
```

```
Initialize → Generate & Evaluate → Analyze Failures → Implement Fix → Compare & Decide → Loop
```

### auto-develop

Adds **new functionality** to existing features. Builds a capability specification (10-20 tiered capabilities with dependencies and acceptance criteria), then runs a loop that selects the next eligible capability, plans the implementation, writes code, verifies acceptance criteria, and runs regression checks.

```
/auto-develop:create <feature>       # Build spec + development loop
/auto-develop:evolve-spec <feature>  # Expand spec after running cycles
```

```
Initialize → Select Capability → Plan → Implement → Verify → Regression Check → Accept/Reject → Loop
```

### When to use which

| Situation | Plugin |
|-----------|--------|
| Feature exists but output quality is poor | **auto-improve** |
| Feature exists but is missing functionality | **auto-develop** |
| Want to hit a quality score target | **auto-improve** |
| Want to build out a feature roadmap | **auto-develop** |
| Fixing bugs, formatting, accuracy | **auto-improve** |
| Adding endpoints, options, capabilities | **auto-develop** |

Both plugins auto-detect your tech stack (Node.js, Python, Go, Rust, Ruby, Java/Kotlin) and app type (web backend, frontend, CLI, data pipeline, ML/AI, library, mobile).

## For Teams

Add to your project's `.claude/settings.json` to auto-enable for all team members:

```json
{
  "extraKnownMarketplaces": {
    "auto-claude-plugins": {
      "source": {
        "source": "github",
        "repo": "aminry/auto-claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "auto-improve@auto-claude-plugins": true,
    "auto-develop@auto-claude-plugins": true
  }
}
```

## Repository Structure

```
auto-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json                ← Marketplace catalog
├── plugins/
│   ├── README.md                       ← Plugin development guide
│   ├── auto-improve/                   ← Auto-improve factory plugin
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── commands/
│   │   │   ├── create.md
│   │   │   └── evolve-rubric.md
│   │   └── README.md
│   └── auto-develop/                   ← Auto-develop factory plugin
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/
│       │   ├── create.md
│       │   └── evolve-spec.md
│       └── README.md
├── README.md
└── LICENSE
```

## Inspiration

These plugins were inspired by [autoresearch](https://github.com/karpathy/autoresearch) by Andrej Karpathy — the idea of using automated iterative loops with evaluation functions to continuously improve and expand software.

## Contributing

See [Plugin Development Guide](./.claude-plugin/plugins/README.md) for instructions on adding new plugins.

## License

MIT
