# Sol's Claude Code Plugins

Personal plugin marketplace for Claude Code extensions.

## Installation

Add this marketplace:

```
/plugin marketplace add SolAstrius/claude-plugins
```

Then install plugins:

```
/plugin install example-tools@sol-plugins
```

## Available Plugins

| Plugin | Description |
|--------|-------------|
| example-tools | Example plugin with a simple greeting skill |

## Creating New Plugins

1. Create a directory under `plugins/`
2. Add `.claude-plugin/plugin.json` manifest
3. Add skills, commands, agents, or hooks
4. Update `marketplace.json` to list the new plugin
