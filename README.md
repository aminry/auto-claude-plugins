# auto-claude-plugins

A marketplace of Claude Code plugins for automation, quality, and productivity.

## Plugins

| Plugin | Description | Commands |
|--------|-------------|----------|
| [auto-improve](./auto-improve/) | Creates custom auto-improve loops for any feature — builds evaluation rubrics and generates autonomous iterative improvement skills | `/auto-improve:create`, `/auto-improve:evolve-rubric` |

## Installation

Install a specific plugin by adding it to your project's `.claude/settings.json`:

```json
{
  "plugins": [
    "aminry/auto-claude-plugins/auto-improve"
  ]
}
```

Or clone locally and point to the plugin path:

```json
{
  "plugins": [
    "/path/to/auto-claude-plugins/auto-improve"
  ]
}
```

## Repository Structure

```
auto-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json       # Marketplace manifest
├── auto-improve/              # Auto-improve factory plugin
│   ├── .claude-plugin/
│   │   └── plugin.json        # Plugin manifest
│   ├── commands/
│   │   ├── create.md
│   │   └── evolve-rubric.md
│   └── README.md
├── README.md
└── LICENSE
```

## Adding a New Plugin

1. Create a new directory at the repo root with your plugin name
2. Add `.claude-plugin/plugin.json` with the required manifest fields
3. Add your `commands/`, `agents/`, `skills/`, or `hooks/` directories
4. Add a `README.md` documenting the plugin
5. Update this README's plugin table

### Plugin Manifest Template

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "What this plugin does",
  "keywords": ["relevant", "keywords"],
  "strict": true,
  "commands": ["./commands/my-command.md"]
}
```

## License

MIT
