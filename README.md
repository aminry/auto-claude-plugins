# auto-claude-plugins

A marketplace of Claude Code plugins for automation, quality, and productivity.

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add aminry/auto-claude-plugins

# List available plugins
/plugin list

# Install a plugin
/plugin install auto-improve@auto-claude-plugins
```

## Available Plugins

| Plugin | Description | Version |
|--------|-------------|---------|
| [auto-improve](./.claude-plugin/plugins/auto-improve/) | Creates custom auto-improve loops for any feature — builds evaluation rubrics and generates autonomous iterative improvement skills | 1.0.1 |

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
    "auto-improve@auto-claude-plugins": true
  }
}
```

## Repository Structure

```
auto-claude-plugins/
├── .claude-plugin/
│   ├── marketplace.json                ← Marketplace catalog
│   └── plugins/
│       ├── README.md                   ← Plugin development guide
│       └── auto-improve/              ← Auto-improve factory plugin
│           ├── .claude-plugin/
│           │   └── plugin.json
│           ├── commands/
│           │   ├── create.md
│           │   └── evolve-rubric.md
│           └── README.md
├── README.md
└── LICENSE
```

## Contributing

See [Plugin Development Guide](./.claude-plugin/plugins/README.md) for instructions on adding new plugins.

## License

MIT
