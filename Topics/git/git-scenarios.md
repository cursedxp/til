# Git Common Scenarios & Solutions

Practical solutions for everyday Git tasks.

---

## 1. Fixing Commit Messages

### Scenario: You just committed with the wrong message

**Concept**: `git commit --amend` replaces your last commit with a new one. It's like using an "undo" button on the most recent commit.

**Before pushing:**
```bash
# Option 1: Open editor to change message
git commit --amend
# Opens your default editor with the current message
# Edit it, save, and close

# Option 2: Change message directly (faster)
git commit --amend -m "Correct commit message"
```

**What happens**: Git creates a new commit with:
- Same changes as before
- Same parent commit
- New commit message
- **Different SHA** (technically a new commit)

**After pushing:**
```bash
# Step 1: Amend the commit
git commit --amend -m "Correct commit message"

# Step 2: Force push (use with caution!)
git push --force
# OR safer option (fails if someone else pushed)
git push --force-with-lease
```

**Why force push is needed**: Your local commit has a different SHA than the remote commit. Git sees them as different commits, so you need to "force" the remote to accept your version.

**force vs force-with-lease**:
- `--force`: Overwrites remote no matter what (dangerous!)
- `--force-with-lease`: Fails if someone else pushed (safer)

⚠️ **Warning**: Never force push to shared branches like `main` or `develop` without team coordination.

**When to use each**:
- **Own feature branch**: Safe to force push
- **Shared branch**: Don't force push, or coordinate with team
- **Main/master**: Almost never force push

---

## 2. Undoing Changes

### Scenario: You want to undo changes in a file (not committed yet)

**Concept**: Discard uncommitted changes in your working directory.

```bash
# Discard changes in a specific file
git checkout -- filename.txt

# Or using newer syntax (Git 2.23+)
git restore filename.txt

# Discard all changes
git checkout -- .
git restore .
```

**What happens**: The file reverts to how it was in the last commit. Your edits are **permanently lost**.

⚠️ **Warning**: This cannot be undone! Make sure you really want to discard the changes.

---

### Scenario: You want to unstage a file

**Concept**: Move files from the staging area back to working directory (keep your changes, just "un-add" them).

```bash
# Unstage a specific file (old way)
git reset HEAD filename.txt

# Or using newer syntax (Git 2.23+)
git restore --staged filename.txt
```

**Visual representation**:
```
Before:
Working Dir: file.txt (modified)
Staging:     file.txt (ready to commit) ← We want to remove this
Committed:   file.txt (old version)

After unstaging:
Working Dir: file.txt (modified) ← Changes still here!
Staging:     (empty)
Committed:   file.txt (old version)
```

Your changes are safe, just not staged anymore.

---

### Scenario: You want to undo the last commit (keep changes)

**Concept**: The `git reset` command moves the HEAD pointer backward. Three modes control what happens to your changes:

**1. Soft Reset** (keep changes staged)
```bash
git reset --soft HEAD~1
```
**Result**:
- Commit is gone
- Changes stay in staging area
- Ready to commit again with new message

**Use case**: You want to re-commit with different files or message.

---

**2. Mixed Reset** (keep changes unstaged) - **DEFAULT**
```bash
git reset HEAD~1
# OR explicitly
git reset --mixed HEAD~1
```
**Result**:
- Commit is gone
- Changes stay in working directory (unstaged)
- Need to `git add` again

**Use case**: You want to reorganize what goes in the commit.

---

**3. Hard Reset** (discard changes completely)
```bash
# ⚠️ DANGEROUS: This deletes changes permanently
git reset --hard HEAD~1
```
**Result**:
- Commit is gone
- Changes are deleted
- Working directory matches previous commit

**Use case**: You want to completely throw away the last commit.

⚠️ **Warning**: This is permanent! Use only if you're absolutely sure.

---

**Visual comparison**:
```
Original state:
HEAD → Commit C (we want to undo this)
       └─ Changes: file.txt modified

After --soft:
HEAD → Commit B
Staging: file.txt (ready to commit)

After --mixed (default):
HEAD → Commit B
Working Dir: file.txt (modified, not staged)

After --hard:
HEAD → Commit B
Everything clean (changes lost!)
```

**Quick decision guide**:
- Want to fix commit message? → `--soft` then commit again
- Want to reorganize commits? → `--mixed` (default)
- Made a total mistake? → `--hard` (be careful!)

---

### Pro Tip: Recovering from mistakes

If you accidentally did `git reset --hard`, you might still recover:
```bash
# Find your lost commit
git reflog

# Look for the commit SHA, then:
git reset --hard <SHA>
```

The `reflog` keeps a history of where HEAD has been, even "deleted" commits stay around for ~30 days.

---

## 3. Working with Branches

**Concept**: Branches let you work on different features simultaneously without interfering with each other. Think of them as parallel universes for your code.

### Scenario: Create and switch to a new branch

```bash
# Create and switch in one command (old way)
git checkout -b feature/new-feature

# Or using newer syntax (Git 2.23+, clearer)
git switch -c feature/new-feature

# Two-step approach (create, then switch)
git branch feature/new-feature
git checkout feature/new-feature
```

**What happens**:
1. Git creates a new branch pointing to your current commit
2. HEAD moves to the new branch
3. You can now make commits without affecting other branches

**Branch naming conventions**:
```bash
feature/user-login      # New features
bugfix/fix-navbar       # Bug fixes
hotfix/security-patch   # Urgent production fixes
chore/update-deps       # Maintenance tasks
```

---

### Scenario: Switch between branches

```bash
# Old way
git checkout main

# New way (clearer purpose)
git switch main

# See all branches
git branch          # Local branches
git branch -a       # All branches (including remote)
```

**What happens when switching**:
1. Git saves your current branch state
2. Updates your working directory to match the target branch
3. HEAD points to the new branch

⚠️ **Important**: You must commit or stash changes before switching, or Git will refuse (to prevent losing work).

```bash
# If you have uncommitted changes and try to switch:
$ git switch main
error: Your local changes would be overwritten by checkout.
Please commit or stash them.

# Solution 1: Commit your work
git add .
git commit -m "WIP: in progress"

# Solution 2: Stash your work (temporary save)
git stash
git switch main
# Later, come back and restore:
git switch feature/new-feature
git stash pop
```

---

### Scenario: Delete a branch

**Concept**: Clean up branches you no longer need.

```bash
# Delete local branch (safe - must be merged)
git branch -d feature/old-feature

# Force delete local branch (dangerous - even if not merged)
git branch -D feature/old-feature

# Delete remote branch
git push origin --delete feature/old-feature
```

**Safe vs Force delete**:
- `-d` (safe): Prevents deletion if branch has unmerged commits
- `-D` (force): Deletes even if you haven't merged (lose work!)

**Common workflow**:
```bash
# After merging your PR on GitHub:

# 1. Switch to main and update
git checkout main
git pull

# 2. Delete local branch (safe)
git branch -d feature/completed-feature

# 3. Delete remote branch (if not auto-deleted)
git push origin --delete feature/completed-feature

# 4. Clean up tracking branches
git fetch --prune  # Remove references to deleted remote branches
```

---

### Scenario: See what branches exist

```bash
# List local branches (* shows current)
git branch
# * main
#   feature/login
#   bugfix/navbar

# List remote branches
git branch -r
#   origin/main
#   origin/feature/payments

# List all branches (local + remote)
git branch -a

# See last commit on each branch
git branch -v
# * main    a3f5b89 Add user authentication
#   feature abc1234 Work in progress

# See which branches are merged into current branch
git branch --merged
# See which are not merged yet
git branch --no-merged
```

---

## 4. Stashing Work

**Concept**: Stash is like a temporary clipboard for uncommitted changes. Use it when you need to switch tasks but aren't ready to commit.

### Scenario: You need to switch branches but have uncommitted changes

**The problem**:
```bash
# You're working on feature A
# Suddenly need to fix urgent bug on main
# But you have uncommitted changes...

$ git switch main
error: Your local changes would be overwritten
```

**The solution - Stash**:

```bash
# Save changes temporarily
git stash
# or more descriptive:
git stash save "WIP: working on login feature"

# Now you can switch branches
git switch main
# ... fix the bug, commit, etc ...

# Come back to your feature
git switch feature/login

# Restore your changes
git stash pop  # Apply and delete stash
# OR
git stash apply  # Apply but keep stash (safer)
```

---

### Stash Commands Reference

```bash
# List all stashes
git stash list
# stash@{0}: WIP on feature/login: working on form
# stash@{1}: WIP on main: quick fix
# stash@{2}: On bugfix: navbar layout

# Apply most recent stash (keep stash)
git stash apply

# Apply and remove most recent stash
git stash pop

# Apply specific stash
git stash apply stash@{2}

# See what's in a stash (without applying)
git stash show
git stash show -p  # Show full diff

# Delete a specific stash
git stash drop stash@{2}

# Clear all stashes
git stash clear
```

---

### What Gets Stashed?

```bash
# By default: stashes tracked files (staged + unstaged)
git stash

# Include untracked files (new files not yet added)
git stash -u
# OR
git stash --include-untracked

# Include everything (even .gitignore files)
git stash -a
# OR
git stash --all
```

**Visual example**:
```
Before stash:
Working Dir: file1.txt (modified), file2.txt (new)
Staging:     file3.txt (modified)

After git stash:
Working Dir: (clean)
Staging:     (clean)
Stash stack: [file1.txt, file3.txt]  ← file2.txt NOT included (untracked)

After git stash -u:
Working Dir: (clean)
Staging:     (clean)
Stash stack: [file1.txt, file2.txt, file3.txt]  ← Everything included!
```

---

### Real-World Stash Workflow

**Scenario 1: Quick context switch**
```bash
# Working on feature
git stash
git switch main
git pull
# ... do urgent work ...
git switch feature/mywork
git stash pop
```

**Scenario 2: Stashing specific files**
```bash
# You have changes in 5 files, only want to stash 2
git stash push file1.js file2.js
```

**Scenario 3: Creating a branch from stash**
```bash
# Made changes on wrong branch, but already stashed
git stash
git switch correct-branch
git stash pop

# OR create new branch directly from stash
git stash branch new-branch-name stash@{0}
```

---

### Stash vs Commit

**When to stash**:
- Quick context switch (coming back soon)
- Not ready to commit (work incomplete)
- Experimental changes you might discard

**When to commit**:
- Logical checkpoint reached
- Want permanent record
- Ready to share work with others

**Pro tip**: Don't use stash as long-term storage. If work is valuable, commit it (even as "WIP" commit).

---

## Navigation

- **Previous**: [Git Basics](git-basics.md)
- **Next**: [Merging & Rebasing](git-merge-rebase.md)
- **See Also**: [Quick Reference](git-quick-reference.md)