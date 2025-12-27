# /learn - Deep Dive Learning Pattern

Explore a codebase with 3 parallel Haiku agents → create organized documentation.

## Usage

```
/learn [slug]                # use slug from ψ/memory/slugs.yaml
/learn [repo-path]           # path to repo (symlink or actual)
/learn [repo-name]           # shortcut: finds in ψ/learn/repo/
```

## Examples

```
/learn mem                   # slug → claude-mem
/learn headline              # slug → the-headline
/learn claude-mem            # name → finds in ψ/learn/repo/
/learn ~/Code/github.com/anthropics/claude-code  # full path
```

## Slug Registry

Slugs are **auto-registered by `/project learn`** (not manual).

**Flow**:
```
/project learn https://github.com/owner/repo
    → clone + symlink + auto-register "repo" slug

/learn repo
    → instant lookup → proceed
```

**If slug not found**:
1. Fast search (ψ/learn/repo/, ghq)
2. If found → offer to register
3. If not found → suggest `/project learn [url]` first

```yaml
# ψ/memory/slugs.yaml (auto-populated)
claude-mem: ψ/learn/repo/github.com/thedotmack/claude-mem
jq: ψ/learn/repo/github.com/jqlang/jq
```

## Output Structure

```
ψ/learn/
├── repo/                    # Cloned repos (nested, via ghq)
│   └── github.com/
│       └── owner/
│           └── name/        # The actual code
│
└── [repo-name]/             # Documentation (flat, timestamped)
    ├── [REPO-NAME].md       # Hub + timeline (uppercase, clear in graph)
    ├── YYYY-MM-DD_ARCHITECTURE.md
    ├── YYYY-MM-DD_CODE-SNIPPETS.md
    └── YYYY-MM-DD_QUICK-REFERENCE.md
```

**Key insights**:
- Repos nested, docs flat
- **Always timestamped** - never overwrite
- **[REPO-NAME].md** - hub file (uppercase = clear in Obsidian graph)
- Re-learning adds new dated versions, compares to previous

---

## Step 0: Timestamp + Fast Resolve

```bash
date "+🕐 %H:%M (%A %d %B %Y)"
```

### Fast Path (no subagent)

```bash
# 1. Full path? Use directly
[ -d "$INPUT" ] && REPO_PATH="$INPUT"

# 2. Slug lookup (instant grep)
grep "^$INPUT:" ψ/memory/slugs.yaml | cut -d: -f2 | xargs

# 3. Fuzzy search (split by - and chain greps)
# "claude-mem" → grep -i claude | grep -i mem
# "headline" → grep -i headline
find ψ/learn/repo -maxdepth 4 -type l 2>/dev/null | grep -i word1 | grep -i word2 | head -1
```

If any of above succeeds → **proceed directly to Step 1 (3 agents)**

### Slow Path (if fast path fails)

If not found:
1. **Search** ghq repos for partial match
2. **If found** → offer to register slug
3. **If not found** → suggest: "Run `/project learn [url]` first"

```
subagent_type: context-finder
model: haiku
prompt: |
  Search for "$INPUT" in ghq repos:
  ghq list -p | grep -i "$INPUT"

  Return matches with full paths.
```

---

## Step 1: Launch 3 Haiku Agents (PARALLEL)

Extract `REPO_NAME` from path (just the basename).
Get `TODAY` = current date in YYYY-MM-DD format.

**IMPORTANT**: First create the output directory:
```bash
mkdir -p ψ/learn/[REPO_NAME]
```

**Strategy**: Agents explore and **return content** → Main agent writes files using `cat > file << 'EOF'`

⚠️ **Why not have agents write?** Explore agents may have read-only access. Main agent always has write access.

### Agent 1: Architecture Explorer

```
subagent_type: Explore
model: haiku
prompt: |
  Explore [REPO_PATH] and document its architecture.

  Focus on:
  1. Directory structure and organization
  2. Entry points and main files
  3. Core abstractions and patterns
  4. Dependencies and how they're used
  5. Data flow between components

  Return your findings in this markdown format (I will save it):
  ---
  date: [TODAY]
  repo: [REPO_NAME]
  type: architecture
  ---

  # [REPO_NAME] Architecture
  **Explored**: [TODAY]
  ## Overview
  ## Directory Structure
  ## Core Components
  ## Design Patterns
  ## Dependencies
```

### Agent 2: Code Snippets Collector

```
subagent_type: Explore
model: haiku
prompt: |
  Explore [REPO_PATH] and extract key code snippets.

  Focus on:
  1. Main entry point code
  2. Core algorithm implementations
  3. Interesting patterns or techniques
  4. Configuration examples
  5. API usage examples

  Return your findings in this markdown format (I will save it):
  ---
  date: [TODAY]
  repo: [REPO_NAME]
  type: code-snippets
  ---

  # [REPO_NAME] Code Snippets
  **Explored**: [TODAY]
  ## Entry Point
  ## Core Implementation
  ## Patterns & Techniques
  ## Configuration
```

### Agent 3: Quick Reference Builder

```
subagent_type: Explore
model: haiku
prompt: |
  Explore [REPO_PATH] and create a quick reference guide.

  Focus on:
  1. What this project does (purpose)
  2. Installation/setup steps
  3. Key features and capabilities
  4. Common usage patterns
  5. CLI commands or API methods
  6. Configuration options

  Return your findings in this markdown format (I will save it):
  ---
  date: [TODAY]
  repo: [REPO_NAME]
  type: quick-reference
  ---

  # [REPO_NAME] Quick Reference
  **Explored**: [TODAY]
  ## What It Does
  ## Installation
  ## Key Features
  ## Usage
  ## CLI / API
  ## Configuration
```

### Step 1.5: Main Agent Writes Files

After agents return, main agent writes using heredoc:

```bash
# Write each file from agent output
cat > ψ/learn/[REPO_NAME]/[TODAY]_ARCHITECTURE.md << 'EOF'
[Agent 1 output here]
EOF

cat > ψ/learn/[REPO_NAME]/[TODAY]_CODE-SNIPPETS.md << 'EOF'
[Agent 2 output here]
EOF

cat > ψ/learn/[REPO_NAME]/[TODAY]_QUICK-REFERENCE.md << 'EOF'
[Agent 3 output here]
EOF
```

---

## Step 2: Create/Update Hub File ([REPO-NAME].md)

After 3 agents complete, main agent creates or updates the hub file.
Convert repo name to uppercase (e.g., `claude-mem` → `CLAUDE-MEM.md`).

**If hub exists** (re-learning):
1. Read previous version
2. Compare new docs to old docs
3. Extract discoveries (what's new/changed)
4. **Append** to timeline (don't replace)
5. Update "Latest Exploration" section

**If new** (first time):
1. Create fresh hub file

```markdown
# [REPO_NAME] Learning Index

> Navigation hub for Obsidian graph

## Latest Exploration

**Date**: [TODAY] [HH:MM]

**New Discoveries** (if re-learning):
- [What's new since last exploration]
- [What changed]
- [New patterns found]

**Compare to**: [[prev-date_ARCHITECTURE|Previous exploration]]

**Files**:
- [[YYYY-MM-DD_ARCHITECTURE|Architecture]]
- [[YYYY-MM-DD_CODE-SNIPPETS|Code Snippets]]
- [[YYYY-MM-DD_QUICK-REFERENCE|Quick Reference]]

## Timeline

### YYYY-MM-DD HH:MM (Re-explored)
- **New**: [new features/patterns discovered]
- **Changed**: [what evolved since last time]
- **Insight**: [key learning]
- Files: [[date_ARCHITECTURE]], [[date_CODE-SNIPPETS]], [[date_QUICK-REFERENCE]]

### YYYY-MM-DD HH:MM (First exploration)
- Initial discovery
- Core: [main architecture/pattern]
- Files: [[date_ARCHITECTURE]], [[date_CODE-SNIPPETS]], [[date_QUICK-REFERENCE]]

## All Versions

| Date | Architecture | Code | Quick Ref | Discoveries |
|------|-------------|------|-----------|-------------|
| YYYY-MM-DD | [[link]] | [[link]] | [[link]] | [summary] |
| YYYY-MM-DD | [[link]] | [[link]] | [[link]] | [summary] |

## Repository

**Source**: `ψ/learn/repo/github.com/[org]/[repo]`
**Slug**: `[slug]`

---
*Auto-generated by `/learn` command*
```

## Step 3: Main Agent Review

After hub file created:

1. **Verify docs**: Check all 4 files exist (hub + 3 docs)
2. **Quality check**: Skim each doc
3. **If re-learning**: Compare to previous versions, note changes
4. **Present summary**:

```markdown
## 📚 Learning Complete: [REPO_NAME]

**Date**: [TODAY]

### Created Documentation
| File | Description |
|------|-------------|
| [REPO-NAME].md | Hub + timeline (Obsidian graph node) |
| [TODAY]_ARCHITECTURE.md | [1-line summary] |
| [TODAY]_CODE-SNIPPETS.md | [1-line summary] |
| [TODAY]_QUICK-REFERENCE.md | [1-line summary] |

### Key Insights
[2-3 interesting things learned]

### New Discoveries (if re-learning)
[What changed from last exploration]

### Location
ψ/learn/[REPO_NAME]/
```

---

## Path Resolution Logic

```bash
# Input: claude-mem
# Step 1: Check if exists as-is
[ -d "claude-mem" ] && echo "claude-mem"

# Step 2: Search in ψ/learn/repo/
find ψ/learn/repo -type d -name "claude-mem" 2>/dev/null | head -1

# Step 3: Search in ghq
ghq list -p | grep -i "claude-mem" | head -1
```

---

## Compare

| Command | Purpose |
|---------|---------|
| `/project learn [url]` | Clone repo + symlink to ψ/learn/repo/ |
| `/learn [path]` | Explore repo + create documentation |
| `/trace [name]` | Find project across git history + repos |

**Workflow**:
```
/project learn [url]  →  clones to ψ/learn/repo/
/learn [repo-name]    →  explores + creates docs in ψ/learn/[name]/
```

---

## Notes

- **3 agents in parallel** = fast exploration
- **Haiku for exploration** = cost effective
- **Main reviews** = quality gate
- **Flat doc structure** = easy to find, organized by repo name
- **Nested repo structure** = follows ghq convention
