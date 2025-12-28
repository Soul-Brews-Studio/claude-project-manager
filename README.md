# claude-project-manager

Claude Code plugin for project lifecycle management.

## Features

| Command | Purpose |
|---------|---------|
| `/project` | List all tracked projects |
| `/project learn [url]` | Clone repo for study |
| `/project incubate [url]` | Clone repo for development |
| `/project create [name]` | Create NEW GitHub repo |
| `/project find [query]` | Search for projects |
| `/project sync` | Find untracked repos |

## Installation

### Option 1: Interactive UI (Recommended)

```bash
# First, add this repo as a marketplace
/plugin marketplace add Soul-Brews-Studio/claude-project-manager

# Then install via the plugin manager
/plugin
# Navigate to Discover tab → Select project-manager → Install
```

### Option 2: CLI Commands

```bash
# Add marketplace
/plugin marketplace add Soul-Brews-Studio/claude-project-manager

# Install plugin
/plugin install project-manager@claude-project-manager
```

### Option 3: Direct Settings

Add to `~/.claude/settings.json` (user) or `.claude/settings.json` (project):

```json
{
  "plugins": {
    "project-manager@claude-project-manager": {
      "enabled": true
    }
  },
  "pluginMarketplaces": [
    "Soul-Brews-Studio/claude-project-manager"
  ]
}
```

## How It Works

```
ghq (source of truth) → symlinks → ψ/learn/ or ψ/incubate/
```

- **ghq** manages all clones in `~/Code/github.com/`
- **symlinks** organize by purpose (learn vs incubate)
- **slugs.yaml** grows organically from usage

## Directory Structure

```
ψ/
├── learn/
│   └── <repo-name> → ~/Code/github.com/owner/repo
├── incubate/
│   └── <repo-name> → ~/Code/github.com/owner/repo
└── memory/
    └── slugs.yaml  # Slug registry
```

## Requirements

- [ghq](https://github.com/x-motemen/ghq) - Repository management
- [gh](https://cli.github.com/) - GitHub CLI (for `/project create`)

## Philosophy

> **Incubation = Growth Potential**
>
> The question isn't "does this exist?"
> The question is "will this grow beyond its current home?"

## Examples

```bash
# Study a repo
/project learn https://github.com/aiplanethub/openagi

# Start developing
/project incubate https://github.com/owner/repo

# Create new project
/project create my-new-tool

# Find where a project lives
/project find claude
```

## License

MIT

---

Made with Claude Code
