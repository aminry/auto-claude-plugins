# Auto Claude Plugins

This directory contains Claude Code plugins for automation, quality, and productivity.

## Using the Plugin Marketplace

This repository serves as a Claude Code plugin marketplace. Install plugins directly from this repo.

### Quick Start

```bash
# Add the marketplace
/plugin marketplace add aminry/auto-claude-plugins

# List available plugins
/plugin list

# Install a plugin
/plugin install auto-improve@auto-claude-plugins
```

### Available Plugins

| Plugin | Description |
|--------|-------------|
| `auto-improve` | Creates custom auto-improve loops for any feature — builds evaluation rubrics and generates autonomous iterative improvement skills |

## For Teams

To auto-enable plugins for everyone working in a project, add to `.claude/settings.json`:

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

## Creating New Plugins

To add a new plugin to the marketplace:

### 1. Create Plugin Directory

```bash
mkdir -p .claude-plugin/plugins/my-plugin/.claude-plugin
mkdir -p .claude-plugin/plugins/my-plugin/commands
```

### 2. Create Plugin Manifest

**File**: `.claude-plugin/plugins/my-plugin/.claude-plugin/plugin.json`

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "What the plugin does",
  "author": {
    "name": "Your Name"
  },
  "repository": "https://github.com/aminry/auto-claude-plugins",
  "license": "MIT",
  "keywords": ["relevant", "keywords"]
}
```

### 3. Add Components

Add your `commands/`, `agents/`, `skills/`, or `hooks/` directories inside the plugin directory.

### 4. Register in Marketplace

Add an entry to `.claude-plugin/marketplace.json`:

```json
{
  "plugins": [
    {
      "name": "my-plugin",
      "source": "./plugins/my-plugin",
      "description": "What it does",
      "version": "1.0.0",
      "author": {
        "name": "Your Name"
      }
    }
  ]
}
```

### 5. Test Locally

```bash
claude --plugin-dir .claude-plugin/plugins/my-plugin
```

## Plugin Structure

```
.claude-plugin/
├── marketplace.json                       ← Marketplace catalog
└── plugins/
    ├── README.md                          ← This file
    └── auto-improve/
        ├── .claude-plugin/
        │   └── plugin.json                ← Plugin manifest
        ├── commands/
        │   ├── create.md                  ← /auto-improve:create command
        │   └── evolve-rubric.md           ← /auto-improve:evolve-rubric command
        └── README.md                      ← Plugin documentation
```
