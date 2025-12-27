# /project — Project Lifecycle Manager

Track projects: 📚 Learn (study) | 🌱 Incubate (develop)

> **IMPORTANT**: Use subagent (haiku) for `learn`, `incubate`, `create`, `find`, `sync`.
> But **`list` is instant** — run bash directly, no subagent needed!

## Usage

```
/project                  # list all tracked projects (INSTANT - no subagent)
/project learn [input]    # add to learn (url, slug, or search)
/project incubate [input] # add to incubate (url, slug, or search)
/project create [name]    # create NEW GitHub repo + incubate
/project find [query]     # search for project, offer to register slug
/project sync             # find untracked ghq repos
```

## Input Resolution (Organic Slug Growth)

When input is not a URL:

1. Check `ψ/memory/slugs.yaml` for slug
2. If not found → **search** all locations:
   - ghq repos (`~/Code/github.com/`)
   - ψ/learn/repo/
   - ψ/incubate/repo/
3. **Show results** to user:
   ```
   Found matches for "mem":
   1. ~/Code/github.com/thedotmack/claude-mem
   2. (no other matches)

   Select (1-N) or 0 to cancel:
   ```
4. User confirms → add to `ψ/memory/slugs.yaml`
5. Proceed with action

## Slug Registry

File: `ψ/memory/slugs.yaml`

```yaml
# Grows organically from usage
# Format: slug: full-path

mem: ~/Code/github.com/thedotmack/claude-mem
headline: ~/Code/github.com/laris-co/the-headline
```

---

## Actions

### list (default)

> **NO SUBAGENT** — direct bash, instant response

```bash
# COPY-PASTE THIS EXACTLY — do NOT wrap in a for loop!
echo "## 📚 Learn" && ls -la ψ/learn/repo/github.com/*/ 2>/dev/null | grep "^l" | awk '{print "  ✅", $9, "→", $11}' && echo "" && echo "## 🌱 Incubate" && ls -la ψ/incubate/repo/github.com/*/ 2>/dev/null | grep "^l" | awk '{print "  ✅", $9, "→", $11}'
```

⚠️ **Common mistake**: Do NOT use `for d in .../*; do ls "$d"; done` — the glob expansion in `ls -la .../*/` handles multiple orgs correctly.

Output:
```
## 📚 Learn
✅ claude-mem → /Users/nat/Code/github.com/thedotmack/claude-mem
✅ dev-browser → /Users/nat/Code/github.com/SawyerHood/dev-browser

## 🌱 Incubate
✅ arthur → /Users/nat/Code/github.com/laris-co/arthur
```

### find [query]

Search for a project and optionally register a slug.

```
/project find mem
→ Search all locations
→ Show matches
→ "Register as slug? Enter short name (or skip):"
→ User: "mem"
→ Added to slugs.yaml
```

### learn [input]

Add project to learn tracking.

**Input types**:
- URL: `https://github.com/owner/repo` → clone via ghq + symlink
- Slug: `claude-mem` → lookup in registry → symlink
- Search: `some-name` → search → confirm → symlink

**Steps**:
1. Resolve input (URL / slug / search)
2. If new repo → `ghq get -u [url]`
3. Create symlink: `ψ/learn/repo/github.com/[org]/[repo]`
4. **Auto-register slug** (repo basename → path)

```bash
# Auto-register after symlink
REPO_NAME=$(basename "$REPO_PATH")
echo "$REPO_NAME: $SYMLINK_PATH" >> ψ/memory/slugs.yaml
```

### incubate [input]

Same as learn, but symlinks to `ψ/incubate/repo/`.

**Decision tree based on GROWTH POTENTIAL, not just existence:**

```
/project incubate [name]

1. Search ALL locations:
   - slugs.yaml
   - ghq repos (~/Code/github.com/)
   - GitHub (laris-co org)
   - Local folders (ψ/active/*, ψ/incubate/repo/*)

2. If found in ghq/GitHub:
   → Already a repo, just create symlink

3. If found LOCALLY (ψ/active/, ψ/incubate/repo/ as folder):
   → Show context:
     "Found [name] in [location]"
     "Files: [count], Created: [date]"

   → Ask the RIGHT question:
     "Will this grow beyond Nat-s-Agents? [Y/n]"

   → If Y: Create repo + migrate files
   → If N: Keep in place, register slug only

4. If NOT found anywhere:
   → "Create new repo laris-co/[name]? [Y/n]"
```

**The key insight**: Incubation = growth potential, not current size.
- 2-file demo that will evolve → create repo
- 50-file tool that's done → keep in place

See: `ψ/memory/learnings/2025-12-22_project-incubation-growth-pattern.md`

### create [name]

Create a NEW GitHub repo and set up for incubation.

**Flow**:
```
/project create arthur

1. Confirm with user:
   → Name: laris-co/arthur
   → Visibility: private (default) or public
   → [Create] [Cancel]

2. Create GitHub repo:
   gh repo create laris-co/[name] --private --clone=false

3. Clone via ghq:
   ghq get laris-co/[name]

4. Create symlink:
   mkdir -p ψ/incubate/repo/github.com/laris-co
   ln -s ~/Code/github.com/laris-co/[name] ψ/incubate/repo/github.com/laris-co/[name]

5. Register slug:
   echo "[name]: ~/Code/github.com/laris-co/[name]" >> ψ/memory/slugs.yaml

6. Check for existing local files:
   - If ψ/incubate/repo/[name] exists (non-symlink folder)
   - Or ψ/active/[name]-* exists
   → Offer to move files to new repo
   → git add, commit, push
```

**With existing files**:
```bash
# If local files exist, move them
if [ -d "ψ/incubate/repo/arthur" ] && [ ! -L "ψ/incubate/repo/arthur" ]; then
  # Move contents to ghq repo
  cp -r ψ/incubate/repo/arthur/* ~/Code/github.com/laris-co/arthur/
  rm -rf ψ/incubate/repo/arthur
  # Now create symlink
  ln -s ~/Code/github.com/laris-co/arthur ψ/incubate/repo/github.com/laris-co/arthur
  # Commit initial files
  cd ~/Code/github.com/laris-co/arthur
  git add -A
  git commit -m "Initial commit: migrate from Nat-s-Agents incubate"
  git push -u origin main
fi
```

**Output**:
```
✅ Created: laris-co/arthur (private)
✅ Cloned via ghq: ~/Code/github.com/laris-co/arthur
✅ Symlinked: ψ/incubate/repo/github.com/laris-co/arthur
✅ Registered slug: arthur
✅ Migrated 5 files from local folder
✅ Pushed initial commit

Ready to develop!
```

### sync

Find ghq repos not yet tracked in learn or incubate.

```
## Untracked repos
antigravity → ~/Code/github.com/laris-co/antigravity
esphome-fw → ~/Code/github.com/laris-co/esphome-fw

Add to learn/incubate? (or skip)
```

---

## Directory Structure

```
ψ/
├── learn/
│   ├── repo/                    # Symlinks to ghq repos
│   │   └── github.com/owner/name → ~/Code/github.com/owner/name
│   └── [name]/                  # Documentation (from /learn command)
│       ├── ARCHITECTURE.md
│       ├── CODE-SNIPPETS.md
│       └── QUICK-REFERENCE.md
│
├── incubate/
│   └── repo/                    # Symlinks to ghq repos
│       └── github.com/owner/name → ~/Code/github.com/owner/name
│
└── memory/
    └── slugs.yaml               # Slug registry (organic growth)
```

---

## Examples

```bash
# By URL (new repo)
/project learn https://github.com/jqlang/jq
→ clones + symlinks + offers slug registration

# By slug (registered)
/project incubate headline
→ looks up "headline" → symlinks to incubate

# By search (not registered)
/project learn claude-mem
→ "Found: ~/Code/.../claude-mem. Register as slug?"
→ User: "mem"
→ symlinks + registers

# Find and register
/project find maw
→ "Found: multi-agent-workflow-kit"
→ "Register as 'maw'?" → Yes

# Create NEW repo (name doesn't exist anywhere)
/project incubate arthur
→ "arthur" not found anywhere
→ "Create new repo laris-co/arthur? [Y/n]"
→ User: Y
→ Creates GitHub repo + ghq + symlink + slug

# Explicit create
/project create my-new-tool
→ Creates laris-co/my-new-tool (private)
→ ghq get → symlink → register

# Create with existing local files
/project create arthur
→ Found: ψ/incubate/repo/arthur (local folder)
→ Creates repo, migrates files, pushes initial commit
```

---

## Related Commands

| Command | Purpose |
|---------|---------|
| `/project learn` | Clone/symlink for study |
| `/project incubate` | Clone/symlink for development |
| `/project create` | Create NEW GitHub repo + incubate |
| `/learn [slug]` | Deep dive → create documentation |
| `/trace [slug]` | Find project across history |

**Workflow**:
```
# Existing repo
/project learn [url]  →  clone + symlink
/learn [slug]         →  explore + create docs

# New project
/project create [name]  →  create repo + ghq + symlink
/project incubate [name] → if not found, offers to create
```

---

## Notes

- **ghq** = single source of truth for cloned repos
- **symlinks** = views organized by purpose (learn/incubate)
- **slugs.yaml** = grows organically from actual usage
- Both learn/ and incubate/ are gitignored
- **create** = always creates under `laris-co/` org (default private)
- **incubate** = asks "will this grow?" not just "does this exist?"
- **Migration** = if local files exist, moves them to new repo automatically

## Philosophy

> **Incubation = Growth Potential**
>
> The question isn't "does this exist elsewhere?"
> The question is "will this grow beyond its current home?"
>
> A 2-file demo that will evolve → create repo
> A 50-file tool that's done → keep in place

See: `ψ/memory/learnings/2025-12-22_project-incubation-growth-pattern.md`
