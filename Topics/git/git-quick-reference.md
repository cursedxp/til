# Git Quick Reference

Fast lookup for common Git commands.

---

## Essential Commands

### Setup & Configuration

```bash
# Set identity
git config --global user.name "Your Name"
git config --global user.email "email@example.com"

# Set editor
git config --global core.editor "nano"

# View config
git config --list
```

---

## Basic Workflow

```bash
# Check status
git status

# Stage files
git add filename.txt          # Specific file
git add .                     # All files
git add *.js                  # Pattern

# Commit
git commit -m "message"       # With message
git commit                    # Open editor
git commit --amend            # Modify last commit

# Push
git push                      # To tracked branch
git push -u origin branch     # Set upstream
git push --force-with-lease   # Force push (safer)

# Pull
git pull                      # Fetch and merge
git pull --rebase             # Fetch and rebase
```

---

## Viewing History

```bash
# Log
git log                       # Full log
git log --oneline             # Compact
git log --graph --all         # Visual graph
git log -5                    # Last 5 commits
git log --author="Name"       # By author
git log --since="2 weeks"     # By date

# Diff
git diff                      # Unstaged changes
git diff --staged             # Staged changes
git diff HEAD                 # All changes
git diff branch1..branch2     # Compare branches

# Show commit
git show <commit-sha>         # Show commit details
git show HEAD~2               # Show 2 commits back
```

---

## Branches

```bash
# List
git branch                    # Local branches
git branch -r                 # Remote branches
git branch -a                 # All branches
git branch -v                 # With last commit

# Create & Switch
git checkout -b feature       # Create and switch
git switch -c feature         # Same (newer syntax)
git branch feature            # Create only
git checkout feature          # Switch only
git switch feature            # Switch (newer)

# Delete
git branch -d feature         # Safe delete
git branch -D feature         # Force delete
git push origin --delete feature  # Delete remote

# Merge
git merge feature             # Merge feature into current
git merge --no-ff feature     # Always create merge commit
git merge --squash feature    # Squash commits
```

---

## Undoing Changes

```bash
# Discard changes
git restore filename          # Discard unstaged changes
git restore .                 # Discard all unstaged
git checkout -- filename      # Old syntax

# Unstage
git restore --staged filename # Unstage file
git reset HEAD filename       # Old syntax

# Undo commits
git reset --soft HEAD~1       # Undo commit, keep staged
git reset HEAD~1              # Undo commit, keep unstaged
git reset --hard HEAD~1       # Undo commit, discard changes

# Revert (safe for public history)
git revert <commit-sha>       # Create new commit that undoes
```

---

## Stashing

```bash
# Save changes
git stash                     # Stash tracked files
git stash -u                  # Include untracked
git stash save "message"      # With description

# List & Apply
git stash list                # Show all stashes
git stash show                # Show latest stash
git stash apply               # Apply latest, keep stash
git stash pop                 # Apply and remove
git stash apply stash@{2}     # Apply specific stash

# Manage
git stash drop                # Delete latest
git stash clear               # Delete all
git stash branch new-branch   # Create branch from stash
```

---

## Remote Operations

```bash
# Remotes
git remote -v                 # List remotes
git remote add origin <url>   # Add remote
git remote set-url origin <url>  # Change URL
git remote remove origin      # Remove remote

# Fetch
git fetch                     # Fetch all remotes
git fetch origin              # Fetch specific remote
git fetch --prune             # Remove deleted branches

# Clone
git clone <url>               # Clone repository
git clone <url> folder-name   # Clone to specific folder
```

---

## Rebasing

```bash
# Basic rebase
git rebase main               # Rebase current on main
git rebase --continue         # After fixing conflicts
git rebase --abort            # Cancel rebase

# Interactive rebase
git rebase -i HEAD~3          # Last 3 commits
git rebase -i <commit-sha>    # Since specific commit
```

---

## Tags

```bash
# Create
git tag v1.0.0                # Lightweight tag
git tag -a v1.0.0 -m "msg"    # Annotated tag

# List & Show
git tag                       # List all tags
git tag -l "v1.*"             # List pattern
git show v1.0.0               # Show tag details

# Push
git push origin v1.0.0        # Push specific tag
git push --tags               # Push all tags

# Delete
git tag -d v1.0.0             # Delete local
git push origin --delete v1.0.0  # Delete remote
```

---

## Cherry-picking

```bash
# Apply specific commit
git cherry-pick <commit-sha>  # Apply one commit
git cherry-pick A B C         # Apply multiple
git cherry-pick --continue    # After fixing conflicts
git cherry-pick --abort       # Cancel operation
```

---

## Searching

```bash
# In files
git grep "search term"        # Search in working directory
git grep "term" branch-name   # Search in specific branch

# In commits
git log -S "code"             # Find commits with code
git log --grep="bug"          # Search commit messages

# Who changed what
git blame filename            # Show who changed each line
git blame -L 10,20 file       # Specific lines
```

---

## Cleaning

```bash
# Preview
git clean -n                  # Dry run (see what would be deleted)

# Clean
git clean -f                  # Remove untracked files
git clean -fd                 # Remove untracked files & dirs
git clean -fdx                # Include ignored files

# Maintenance
git gc                        # Garbage collection
git prune                     # Remove unreachable objects
```

---

## Submodules

```bash
# Add
git submodule add <url> path  # Add submodule

# Update
git submodule update --init   # Initialize and update
git submodule update --remote # Update to latest

# Clone with submodules
git clone --recursive <url>   # Clone with submodules
```

---

## Aliases (Shortcuts)

```bash
# Set up useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --oneline --graph --all'

# Now you can use:
git st                        # Instead of git status
git co main                   # Instead of git checkout main
git visual                    # Pretty log graph
```

---

## Emergency Commands

```bash
# Reflog (recover "lost" commits)
git reflog                    # Show HEAD history
git reset --hard <sha>        # Go back to any state

# Abort operations
git merge --abort             # Cancel merge
git rebase --abort            # Cancel rebase
git cherry-pick --abort       # Cancel cherry-pick

# Force operations (use carefully!)
git push --force-with-lease   # Safer force push
git reset --hard origin/main  # Match remote exactly
```

---

## Inspection Commands

```bash
# File status
git status                    # Working directory status
git status -s                 # Short format

# What changed
git diff                      # Unstaged changes
git diff --staged             # Staged changes
git diff HEAD                 # All local changes
git diff <commit1> <commit2>  # Between commits

# Commit info
git show <commit>             # Show commit details
git show --stat <commit>      # Show files changed

# Branch info
git branch -v                 # Branches with last commit
git branch --merged           # Merged branches
git branch --no-merged        # Unmerged branches
```

---

## Useful Flags

### Common flags across commands:

```bash
-v, --verbose                 # More detailed output
-q, --quiet                   # Suppress output
-f, --force                   # Force operation
-n, --dry-run                 # Preview without executing
-a, --all                     # All items
-p, --patch                   # Interactive mode
-u, --set-upstream            # Set tracking branch
```

---

## Special References

```bash
HEAD                          # Current commit
HEAD~1, HEAD~2                # 1, 2 commits back
HEAD^                         # First parent
HEAD^^                        # Grandparent
@                             # Shortcut for HEAD
main@{yesterday}              # Branch state yesterday
main@{2.hours.ago}            # 2 hours ago
@{-1}                         # Previous branch
```

---

## Quick Workflows

### Start new feature
```bash
git checkout main
git pull
git checkout -b feature/new-feature
# ... work ...
git add .
git commit -m "feat: add new feature"
git push -u origin feature/new-feature
```

### Update feature branch with main
```bash
git checkout feature/my-feature
git fetch origin
git rebase origin/main
git push --force-with-lease
```

### Undo last commit (keep changes)
```bash
git reset --soft HEAD~1
# Edit files
git add .
git commit -m "Better commit message"
```

### Quick fix on main
```bash
git checkout main
git pull
# Fix issue
git add .
git commit -m "fix: quick hotfix"
git push
```

---

## Cheat Sheet by Scenario

| I want to... | Command |
|--------------|---------|
| Start new repo | `git init` |
| Clone repo | `git clone <url>` |
| See changes | `git status` |
| Stage file | `git add filename` |
| Commit | `git commit -m "message"` |
| Push | `git push` |
| Pull | `git pull` |
| Create branch | `git checkout -b branch-name` |
| Switch branch | `git checkout branch-name` |
| Merge branch | `git merge branch-name` |
| Discard changes | `git restore filename` |
| Undo commit | `git reset HEAD~1` |
| Stash changes | `git stash` |
| Apply stash | `git stash pop` |
| View history | `git log --oneline` |
| See who changed line | `git blame filename` |

---

## Navigation

- **Detailed Guides**:
  - [Git Basics](git-basics.md)
  - [Common Scenarios](git-scenarios.md)
  - [Merge & Rebase](git-merge-rebase.md)
  - [Workflows](git-workflows.md)
  - [Troubleshooting](git-troubleshooting.md)
  - [Editors Guide](git-editors.md)
- **Full Guide**: [Git Commands Guide](git-commands-guide.md)