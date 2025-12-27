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

Add to your Claude Code settings:

```json
{
  "plugins": [
    "Soul-Brews-Studio/claude-project-manager"
  ]
}
```

Or install via Claude Code CLI:

```bash
claude plugins add Soul-Brews-Studio/claude-project-manager
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
