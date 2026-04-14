# Parallel Worktrees Pattern

Run multiple MARVIN sessions simultaneously on different workstreams using git worktrees.

## Why You'd Want This

Sometimes you need to work on multiple things at once:
- Main MARVIN session for daily operations (email, calendar, briefing)
- A focused session for building a new feature
- An experimental session for trying something that might not work

Git worktrees let you have multiple checkouts of the same repo, each on its own branch, without cloning multiple copies. Each worktree can run its own Claude Code session.

## How Git Worktrees Work

A worktree is a linked working directory that shares the same `.git` history as your main repo:

```
~/marvin/                          # Main worktree (main branch)
├── .git/                          # Shared git database
├── CLAUDE.md
├── state/
└── ...

~/marvin-feature-x/                # Worktree (feature/x branch)
├── .git -> ~/marvin/.git          # Links back to main
├── CLAUDE.md                      # Same file, different branch
├── state/
└── ...
```

Both directories share git history. Commits in one are visible in the other. But each has its own working directory, branch, and staged changes.

## Workflow

### Creating a Worktree

When starting focused work that needs isolation:

```bash
# From your main marvin directory
git worktree add ../marvin-<feature-name> -b feature/<feature-name>
```

**Naming conventions:**
- Directory: `../marvin-<short-name>` (sibling to main marvin folder)
- Branch: `feature/<descriptive-name>`

**Examples:**
```bash
git worktree add ../marvin-new-integration -b feature/slack-integration
git worktree add ../marvin-website -b feature/personal-site
git worktree add ../marvin-experiment -b feature/daemon-v2
```

### Running Parallel Sessions

```
Terminal 1: cd ~/marvin && claude          # Daily ops on main
Terminal 2: cd ~/marvin-website && claude   # Focused on website
```

Each session:
- Has its own working directory
- Can have its own Claude Code session with full MARVIN context
- Shares git history with the main repo
- Can be merged back when complete

### When to Suggest a Worktree

MARVIN should proactively suggest worktrees when the user starts work that is:
- A new feature or integration
- Experimental (might not be merged)
- A separate project needing isolation
- Work that would conflict with ongoing changes on main

**Suggested prompt:**
> "This sounds like a separate workstream. Want me to set up a git worktree so you can run another session on it?"

### Merging Back

When feature work is complete:

```bash
# From main marvin directory
cd ~/marvin
git checkout main
git merge feature/<feature-name>
git push

# Clean up
git worktree remove ../marvin-<feature-name>
git branch -d feature/<feature-name>
```

### Listing and Managing

```bash
# See all worktrees
git worktree list

# Remove a worktree (after merging or abandoning)
git worktree remove ../marvin-<feature-name>

# Clean up stale references
git worktree prune
```

## Key Decisions

| Decision | What we learned |
|----------|----------------|
| Sibling directories | `../marvin-*` keeps things organized |
| One branch per worktree | Git requires this; it's also good practice |
| Main for daily ops | Keep main stable for email/calendar/briefing |
| Feature branches for focused work | Isolation prevents accidental conflicts |
| Proactive suggestion | Users don't think to create worktrees; MARVIN should offer |

## Session Patterns

| Session | Branch | Purpose |
|---------|--------|---------|
| Main (`~/marvin`) | `main` | Daily operations, email, calendar, briefing |
| Feature (`~/marvin-*`) | `feature/*` | Focused development work |
| Experiment (`~/marvin-*`) | `experiment/*` | Exploratory work that may be discarded |

## Getting Started

1. No setup needed. Git worktrees are built into git.
2. Add a routing rule: when starting focused feature work, suggest a worktree
3. Use the naming conventions above for consistency
4. Merge or discard when done. Don't let worktrees accumulate.
5. Run `git worktree list` periodically to see what's active
