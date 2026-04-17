# Git Cheatsheet — paperclipAI Agents

## Setup (one-time per task)

```bash
git config --global user.email "agent@paperclip.ai"
git config --global user.name "paperclipAI Agent"
git config --global credential.helper store
```

## Clone

```bash
# Basic clone
git clone "https://oauth2:${GITHUB_TOKEN}@github.com/${ORG}/${REPO}.git" .

# Specific branch only
git clone --branch main --depth 1 \
  "https://oauth2:${GITHUB_TOKEN}@github.com/${ORG}/${REPO}.git" .
```

## Status & Inspection

```bash
git status                  # What's changed?
git log --oneline -10       # Last 10 commits
git diff                    # Unstaged diff
git diff --staged           # Staged diff
git branch -a               # All branches (local + remote)
```

## Branching

```bash
git checkout -b feat/issue-42-my-feature   # Create and switch
git checkout main                           # Switch to main
git branch -d old-branch                   # Delete local branch
```

## Staging & Committing

```bash
git add -A                          # Stage everything
git add src/                        # Stage directory
git add -p                          # Interactive staging
git commit -m "fix: message (#42)"  # Commit
git commit --amend --no-edit        # Amend last commit (before push only)
```

## Remote Operations

```bash
# Set authenticated remote URL
git remote set-url origin \
  "https://oauth2:${GITHUB_TOKEN}@github.com/${ORG}/${REPO}.git"

# Push new branch
git push -u origin feat/issue-42-my-feature

# Pull latest from main
git fetch origin
git rebase origin/main

# Force push (use carefully, only on feature branches)
git push --force-with-lease origin feat/issue-42-my-feature
```

## Merging / Rebasing

```bash
git fetch origin main
git rebase origin/main          # Rebase current branch on top of main
git rebase --continue           # After resolving conflicts
git rebase --abort              # Cancel rebase
```

## Resolving Conflicts

```bash
git status                      # See conflicted files
# Edit files to resolve
git add <resolved-file>
git rebase --continue           # Or: git merge --continue
```

## Stash

```bash
git stash                       # Stash uncommitted changes
git stash pop                   # Reapply stash
git stash list                  # List stashes
```

## Tags

```bash
git tag v1.2.3                          # Lightweight tag
git tag -a v1.2.3 -m "Release v1.2.3"  # Annotated tag
git push origin v1.2.3                  # Push tag
```

## Undo Operations

```bash
git restore <file>              # Discard unstaged changes in file
git restore --staged <file>     # Unstage file
git revert HEAD                 # Revert last commit (safe, creates new commit)
git reset --soft HEAD~1         # Undo last commit, keep changes staged
```

## Common Patterns for Agents

### Check if on correct branch before making changes
```bash
CURRENT=$(git rev-parse --abbrev-ref HEAD)
if [ "$CURRENT" = "main" ] || [ "$CURRENT" = "master" ]; then
  echo "ERROR: Cannot commit directly to $CURRENT"
  exit 1
fi
```

### Check for uncommitted changes before switching branches
```bash
if ! git diff --quiet || ! git diff --staged --quiet; then
  git stash
fi
```

### Get current commit SHA (useful for API operations)
```bash
git rev-parse HEAD
git rev-parse origin/main
```
