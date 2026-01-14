# Git Version Control

> **What it is:** Git is a distributed version control system that tracks changes to files. It allows multiple people to work on the same project, maintains history of all changes, and enables branching for parallel development.

## Git Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Git Workflow                                         │
│                                                                              │
│   WORKING         STAGING          LOCAL            REMOTE                  │
│   DIRECTORY       AREA             REPOSITORY       REPOSITORY              │
│   ┌─────────┐    ┌─────────┐      ┌─────────┐      ┌─────────┐             │
│   │ Your    │───►│ Ready   │─────►│ Committed│─────►│ GitHub  │             │
│   │ files   │add │ to      │commit│ history  │push  │ GitLab  │             │
│   └─────────┘    └─────────┘      └─────────┘      └─────────┘             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Initial Setup

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

## Basic Workflow

```bash
git status                        # Check status
git add .                         # Stage all changes
git commit -m "message"           # Commit
git push                          # Push to remote
git pull                          # Get latest changes
```

## Fetch vs Pull - Understanding the Difference

> **Key Concept:** `git fetch` is safe and read-only. `git pull` modifies your working directory.

### Visual Comparison

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│   git fetch                              git pull                        │
│   ─────────                              ────────                        │
│                                                                          │
│   Remote ──► Local Tracking              Remote ──► Local Tracking       │
│              (origin/main)                          (origin/main)        │
│                                                          │               │
│              Your branch                                 ▼               │
│              stays unchanged                        Your branch          │
│                                                     gets updated         │
│                                                                          │
│   = SAFE, just downloads                 = Downloads AND merges          │
│   = No changes to your work              = Can cause merge conflicts     │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### What Each Command Does

| Aspect | `git fetch` | `git pull` |
|--------|-------------|------------|
| **Downloads changes** | ✅ Yes | ✅ Yes |
| **Updates local tracking branches** | ✅ Yes (origin/main, etc.) | ✅ Yes |
| **Modifies your working branch** | ❌ No | ✅ Yes (merges into current) |
| **Can cause conflicts** | ❌ No | ✅ Yes |
| **Equivalent to** | Just downloading | `git fetch` + `git merge` |

### Git Pull = Git Fetch + Git Merge

```bash
# These two are equivalent:

# Option 1: git pull (one command)
git pull origin main

# Option 2: fetch + merge (two commands)
git fetch origin main
git merge origin/main
```

### When to Use `git fetch`

**Use fetch when you want to:**

1. **See what others have done before merging**
   ```bash
   git fetch origin
   git log HEAD..origin/main --oneline    # What's new on remote?
   git diff HEAD origin/main              # What are the actual changes?
   ```

2. **Check if you're behind before pushing**
   ```bash
   git fetch origin
   git status                             # Shows "Your branch is behind..."
   ```

3. **Update tracking branches without touching your work**
   ```bash
   git fetch --all                        # Update all remote tracking branches
   # Your current work is completely untouched
   ```

4. **Before rebasing onto latest remote**
   ```bash
   git fetch origin
   git rebase origin/main                 # Rebase your work onto latest
   ```

5. **Working in a team with frequent changes**
   ```bash
   # Someone says "I just pushed feature X"
   git fetch origin
   git log origin/feature-branch --oneline  # See their commits
   ```

### When to Use `git pull`

**Use pull when you:**

1. **Want to quickly update and you're okay with auto-merge**
   ```bash
   git pull                               # Update current branch
   ```

2. **Are on main/master and just want the latest**
   ```bash
   git checkout main
   git pull                               # Get all new commits
   ```

3. **Haven't made any local changes that could conflict**
   ```bash
   git status                             # Working tree clean
   git pull                               # Safe to pull
   ```

4. **Want to pull with rebase instead of merge**
   ```bash
   git pull --rebase                      # Fetch + rebase (cleaner history)
   ```

### Common Scenarios

#### Scenario 1: Check Before Merge (Recommended for Teams)

```bash
# 1. Download latest changes (safe)
git fetch origin

# 2. See what's new
git log HEAD..origin/main --oneline
# a1b2c3d Add new feature
# e4f5g6h Fix bug in login

# 3. Look at the actual code changes
git diff HEAD origin/main

# 4. If everything looks good, merge
git merge origin/main
```

#### Scenario 2: Quick Update (Solo or Trusted Team)

```bash
# Just update everything in one command
git pull
```

#### Scenario 3: You Have Local Commits to Push

```bash
# 1. Check what's on remote
git fetch origin

# 2. See if remote has new commits
git log origin/main --oneline -5

# 3. If remote is ahead, you have options:
git merge origin/main             # Option A: Merge (creates merge commit)
git rebase origin/main            # Option B: Rebase (linear history)

# 4. Then push your work
git push
```

#### Scenario 4: Your Pull Has Conflicts

```bash
git pull
# CONFLICT (content): Merge conflict in file.txt

# 1. Open conflicted files, look for:
# <<<<<<< HEAD
# your changes
# =======
# their changes
# >>>>>>> origin/main

# 2. Edit to resolve, then:
git add file.txt
git commit -m "Resolve merge conflict"
```

### Best Practices

```bash
# Set pull to rebase by default (cleaner history)
git config --global pull.rebase true

# Always fetch before starting new work
git fetch origin
git checkout -b my-feature origin/main

# Use --dry-run to see what would happen
git fetch --dry-run

# Fetch all remotes and prune deleted branches
git fetch --all --prune
```

### Quick Decision Tree

```
Need remote changes?
        │
        ▼
Do you want to review first?
        │
   ┌────┴────┐
   │         │
  YES       NO
   │         │
   ▼         ▼
git fetch   git pull
   │         
   ▼         
Review with:
  git log HEAD..origin/main
  git diff HEAD origin/main
   │
   ▼
Ready to merge?
   │
   ▼
git merge origin/main
  (or git rebase origin/main)
```

## Branches

```bash
# List branches
git branch                        # Local
git branch -a                     # All (including remote)

# Create and switch
git checkout -b feature/new-thing
git switch -c feature/new-thing   # Modern

# Switch branch
git checkout main
git switch main                   # Modern

# Delete branch
git branch -d branch-name         # Safe delete
git branch -D branch-name         # Force delete
git push origin --delete branch   # Delete remote
```

## Merging

```bash
git checkout main
git merge feature/branch

# If conflicts:
# 1. Edit conflicted files
# 2. git add resolved-file
# 3. git commit
```

## Stashing

```bash
git stash                         # Save work temporarily
git stash list                    # List stashes
git stash pop                     # Restore and remove
git stash apply                   # Restore and keep
```

## Viewing History

```bash
git log --oneline                 # Compact history
git log --graph --all --oneline   # Visual branch graph
git diff                          # View changes
git blame file                    # Who changed each line
```

## Undoing Changes

```bash
# Discard uncommitted changes
git checkout -- file
git restore file

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Revert a pushed commit (safe)
git revert abc123
```

## Remote Repositories

```bash
git remote -v                     # View remotes
git remote add origin URL         # Add remote
git push -u origin main           # First push
git fetch --all                   # Fetch all remotes
```

## Tags

```bash
git tag v1.0.0                    # Create tag
git tag -a v1.0.0 -m "Release"    # Annotated tag
git push origin --tags            # Push all tags
```

## Quick Reference

```bash
# Daily workflow
git pull                          # Get latest
git add .                         # Stage
git commit -m "message"           # Commit
git push                          # Push

# Branches
git checkout -b name              # Create & switch
git merge branch                  # Merge
git branch -d name                # Delete
```

