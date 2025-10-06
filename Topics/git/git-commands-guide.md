# Git Commands & Real-Life Scenarios

## Table of Contents
1. [Understanding Git Basics](#understanding-git-basics)
2. [Common Scenarios & Solutions](#common-scenarios--solutions)
3. [Quick Reference Table](#quick-reference-table)
4. [Best Practices](#best-practices)
5. [Emergency Commands](#emergency-commands)

---

## Understanding Git Basics

### What is Git?
Git is a **distributed version control system** that tracks changes in your code over time. Think of it as a time machine for your project that lets you:
- Save snapshots (commits) of your work
- Work on different features simultaneously (branches)
- Collaborate with others without conflicts
- Undo mistakes and go back in time

### Key Concepts

#### 1. Repository (Repo)
A folder that Git is tracking. Contains all your files plus a hidden `.git` folder with the entire history.

```
my-project/
├── .git/          # Git's database (don't touch!)
├── src/           # Your code
├── README.md
└── package.json
```

#### 2. Working Directory, Staging Area, and Repository

Git has three main areas where your files can be:

```
Working Directory  →  Staging Area  →  Local Repository  →  Remote Repository
(your files)          (git add)        (git commit)         (git push)
```

**Working Directory**: Your actual files where you make changes
**Staging Area (Index)**: Files you've marked to be included in the next commit
**Local Repository**: Committed snapshots stored in `.git/`
**Remote Repository**: Server copy (like GitHub, GitLab)

**Example workflow:**
```bash
# 1. Edit file
echo "Hello" > file.txt

# 2. Stage it (prepare for commit)
git add file.txt

# 3. Commit it (save snapshot locally)
git commit -m "Add greeting"

# 4. Push it (send to remote server)
git push
```

#### 3. Commits
A **commit** is a snapshot of your project at a specific point in time. Each commit has:
- **SHA hash**: Unique ID (e.g., `a3f5b89`)
- **Author**: Who made the change
- **Timestamp**: When it was made
- **Message**: What changed
- **Parent**: Previous commit(s)

Think of commits as save points in a video game.

#### 4. Branches
A **branch** is an independent line of development. The default branch is usually `main` or `master`.

```
main:     A---B---C---D
                   \
feature:            E---F
```

- `main` is the stable production code
- `feature` is where you develop new features
- Eventually `feature` gets merged back into `main`

**Why use branches?**
- Work on features without breaking main code
- Multiple people can work simultaneously
- Easy to experiment and throw away failed attempts

#### 5. HEAD
`HEAD` is a pointer to your current position in the Git history. Usually points to the latest commit on your current branch.

```bash
HEAD         # Current commit
HEAD~1       # One commit back
HEAD~2       # Two commits back
HEAD^        # Parent commit (same as HEAD~1)
```

#### 6. Remote
A **remote** is a version of your repository hosted on a server (GitHub, GitLab, etc.). Common remotes:
- `origin`: Default name for the main remote
- `upstream`: Often used for the original repo when you fork

```bash
git remote -v  # See your remotes
# origin  https://github.com/user/repo.git (fetch)
# origin  https://github.com/user/repo.git (push)
```

#### 7. Merge vs Rebase

**Merge**: Combines branches by creating a new "merge commit"
```
main:     A---B---C-------F
                   \     /
feature:            D---E
```

**Rebase**: Replays your commits on top of another branch (cleaner history)
```
main:     A---B---C
                   \
feature:            D'---E'
```

Merge preserves history exactly as it happened. Rebase creates a linear history (cleaner, but rewrites history).

#### 8. Fast-Forward
When Git can simply move the branch pointer forward without creating a merge commit.

```
Before:
main:     A---B
               \
feature:        C---D

After merge:
main:     A---B---C---D
```

#### 9. Detached HEAD State
When HEAD points directly to a commit instead of a branch. Happens when you checkout a specific commit.

```bash
git checkout a3f5b89  # Now in detached HEAD state
# To fix: git checkout main
```

#### 10. Git Editors

Git uses a text editor for commit messages when you don't use `-m`:

**Vim** (default on many systems):
- Hard to use for beginners
- Commands: `i` (insert), `Esc` (exit insert), `:wq` (save & quit), `:q!` (quit without saving)

**Nano** (beginner-friendly):
- Shows commands at bottom
- `Ctrl+O` to save, `Ctrl+X` to exit

**VSCode** (recommended):
- Familiar interface
- Set it: `git config --global core.editor "code --wait"`

---

## Common Scenarios & Solutions

### 1. Fixing Commit Messages

#### Scenario: You just committed with the wrong message

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

### 2. Undoing Changes

#### Scenario: You want to undo changes in a file (not committed yet)

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

#### Scenario: You want to unstage a file

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

#### Scenario: You want to undo the last commit (keep changes)

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

#### Pro Tip: Recovering from mistakes

If you accidentally did `git reset --hard`, you might still recover:
```bash
# Find your lost commit
git reflog

# Look for the commit SHA, then:
git reset --hard <SHA>
```

The `reflog` keeps a history of where HEAD has been, even "deleted" commits stay around for ~30 days.

---

### 3. Working with Branches

**Concept**: Branches let you work on different features simultaneously without interfering with each other. Think of them as parallel universes for your code.

#### Scenario: Create and switch to a new branch

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

#### Scenario: Switch between branches

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

#### Scenario: Delete a branch

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

#### Scenario: See what branches exist

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

### 4. Viewing History & Changes

#### Scenario: See commit history
```bash
# Full history
git log

# Compact history
git log --oneline

# With graph
git log --oneline --graph --all

# Last 5 commits
git log -5
```

#### Scenario: See what changed in a file
```bash
# Show changes not yet staged
git diff

# Show changes staged for commit
git diff --staged

# Show changes in a specific file
git diff filename.txt

# Compare two branches
git diff main..feature/new-feature
```

#### Scenario: Find who changed a line
```bash
git blame filename.txt

# Show specific line range
git blame -L 10,20 filename.txt
```

---

### 5. Stashing Work

**Concept**: Stash is like a temporary clipboard for uncommitted changes. Use it when you need to switch tasks but aren't ready to commit.

#### Scenario: You need to switch branches but have uncommitted changes

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

#### Stash Commands Reference

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

#### What Gets Stashed?

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

#### Real-World Stash Workflow

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

#### Stash vs Commit

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

### 6. Syncing with Remote

#### Scenario: Get latest changes from remote
```bash
# Fetch changes (doesn't merge)
git fetch

# Fetch and merge
git pull

# Pull with rebase (cleaner history)
git pull --rebase
```

#### Scenario: Push your changes
```bash
# Push to remote
git push

# Push and set upstream
git push -u origin feature/new-feature

# Force push (use carefully!)
git push --force-with-lease
```

---

### 7. Merging & Rebasing

**Concept**: Both merge and rebase combine changes from different branches, but they do it differently.

---

#### Understanding Merge vs Rebase

**Merge**: Combines histories by creating a new "merge commit"
```
Before:
main:     A---B---C
               \
feature:        D---E

After merge:
main:     A---B---C-------F (merge commit)
               \         /
feature:        D---E---
```
✅ Preserves complete history
✅ Safe for shared branches
❌ Creates extra merge commits (messy history)

**Rebase**: Replays commits on top of another branch
```
Before:
main:     A---B---C
               \
feature:        D---E

After rebase:
main:     A---B---C
                   \
feature:            D'---E' (new commits!)
```
✅ Clean, linear history
✅ Easier to understand
❌ Rewrites history (don't use on shared branches!)
❌ Can be confusing for beginners

**Golden Rule**:
- **Never rebase commits you've pushed to a shared branch**
- Rebasing changes commit SHAs, confusing collaborators

---

#### Scenario: Merge a feature branch into main

```bash
# Step 1: Switch to main
git checkout main

# Step 2: Update main to latest
git pull

# Step 3: Merge feature branch
git merge feature/new-feature

# If no conflicts (fast-forward possible):
# Updating abc1234..def5678
# Fast-forward
#  file.txt | 2 +-
#  1 file changed

# If conflicts occur, Git pauses and tells you
```

**Handling Merge Conflicts**:

When Git can't automatically merge, you'll see:
```bash
$ git merge feature/new-feature
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and then commit the result.
```

**Fix conflicts step by step**:

1. **Open conflicted files** - Git marks conflicts like this:
```javascript
function greet() {
<<<<<<< HEAD (current change - main branch)
  console.log("Hello from main");
=======
  console.log("Hello from feature");
>>>>>>> feature/new-feature (incoming change)
}
```

2. **Edit the file** - Remove markers and keep what you want:
```javascript
function greet() {
  console.log("Hello from feature");  // Decided to keep feature version
}
```

3. **Stage resolved files**:
```bash
git add file.txt
```

4. **Complete the merge**:
```bash
git commit  # Opens editor with default merge message
# OR
git commit -m "Merge feature/new-feature into main"
```

**Abort merge** if you made a mistake:
```bash
git merge --abort  # Go back to before merge
```

---

#### Scenario: Rebase your branch on main

**Use case**: Keep your feature branch up-to-date with main while maintaining clean history.

```bash
# Step 1: Switch to your feature branch
git checkout feature/new-feature

# Step 2: Rebase on main
git rebase main

# Git replays each of your commits on top of main
# If successful:
# Successfully rebased and updated refs/heads/feature/new-feature
```

**What happens during rebase**:
```
Your commits: D, E, F
Main has new commits: X, Y

Rebase process:
1. Git "removes" D, E, F temporarily
2. Updates branch to include X, Y from main
3. Replays D, E, F one by one on top
4. Each replay might have conflicts

Result: A---B---X---Y---D'---E'---F'
```

---

**Handling Rebase Conflicts**:

Conflicts are resolved one commit at a time:

```bash
$ git rebase main
# ... replaying commits ...
CONFLICT (content): Merge conflict in file.txt
# Fix the conflict, then continue
```

1. **Fix the conflict** (same as merge)
2. **Stage the fixed file**:
```bash
git add file.txt
```
3. **Continue the rebase**:
```bash
git rebase --continue
```

This might cause more conflicts for next commits. Repeat until done.

**Rebase commands**:
```bash
git rebase --continue  # After fixing conflict, continue
git rebase --skip      # Skip current commit (rarely used)
git rebase --abort     # Cancel rebase, go back to before
```

---

#### Interactive Rebase (Advanced)

Clean up commits before pushing:
```bash
git rebase -i HEAD~3  # Interactively rebase last 3 commits
```

Opens editor showing:
```
pick a3f5b89 Add feature A
pick b4e6c90 Fix typo
pick c5f7d01 Add feature B

# Commands:
# p, pick = use commit
# r, reword = change commit message
# e, edit = edit commit
# s, squash = combine with previous commit
# f, fixup = like squash but discard message
# d, drop = remove commit
```

**Common uses**:
- Combine tiny commits: Change `pick` to `squash`
- Fix commit messages: Change `pick` to `reword`
- Remove commits: Change `pick` to `drop`

---

#### When to Use Each?

**Use Merge when**:
- On shared/public branches (main, develop)
- Want to preserve exact history
- Multiple people work on same branch
- Following team conventions

**Use Rebase when**:
- On your own feature branch
- Want clean, linear history
- Before creating a pull request
- Updating feature branch with latest main

**Typical workflow**:
```bash
# Work on feature
git checkout -b feature/login
# ... make commits ...

# Main got updated, sync your branch (rebase)
git fetch origin
git rebase origin/main

# Push your clean branch
git push origin feature/login

# Later: Merge into main via Pull Request (merge)
# GitHub/GitLab handles the merge
```

---

### 8. Cherry-picking

#### Scenario: Apply a specific commit to your branch
```bash
# Get the commit SHA from another branch
git log other-branch --oneline

# Apply that commit to current branch
git cherry-pick abc1234
```

---

### 9. Cleaning Up

#### Scenario: Remove untracked files
```bash
# See what would be deleted (dry run)
git clean -n

# Delete untracked files
git clean -f

# Delete untracked files and directories
git clean -fd

# Include ignored files too
git clean -fdx
```

---

### 10. Configuration

#### Scenario: Set up your identity
```bash
# Set name and email
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "code --wait"

# View all settings
git config --list
```

---

## Quick Reference Table

| Task | Command |
|------|---------|
| Check status | `git status` |
| Stage file | `git add filename` |
| Stage all | `git add .` |
| Commit | `git commit -m "message"` |
| Amend last commit | `git commit --amend` |
| Push | `git push` |
| Pull | `git pull` |
| Clone repo | `git clone <url>` |
| Create branch | `git checkout -b branch-name` |
| Switch branch | `git checkout branch-name` |
| Merge branch | `git merge branch-name` |
| Delete branch | `git branch -d branch-name` |
| View log | `git log --oneline` |
| Stash changes | `git stash` |
| Apply stash | `git stash pop` |
| Discard changes | `git restore filename` |
| Unstage file | `git restore --staged filename` |

---

## Best Practices

1. **Commit Often**: Make small, focused commits
2. **Write Clear Messages**: Use descriptive commit messages
3. **Pull Before Push**: Always pull latest changes before pushing
4. **Use Branches**: Never work directly on main/master
5. **Review Before Commit**: Check `git status` and `git diff` before committing
6. **Don't Force Push**: Avoid `--force` on shared branches
7. **Keep History Clean**: Use rebase for feature branches, merge for integration

---

## Emergency Commands

### "Oh no, I deleted everything!"
```bash
# If not committed, you might be out of luck
# But try:
git reflog  # Find the commit SHA before disaster
git reset --hard <SHA>
```

### "I committed to the wrong branch!"
```bash
# On wrong branch:
git log  # Copy the commit SHA

# Switch to correct branch:
git checkout correct-branch
git cherry-pick <SHA>

# Go back to wrong branch and remove commit:
git checkout wrong-branch
git reset --hard HEAD~1
```

### "I need to undo a pushed commit!"
```bash
# Create a new commit that reverses changes
git revert <commit-SHA>
git push
```

---

## Commit Message Best Practices

Good commit messages make your project history readable and useful.

### Format

```
<type>: <short summary> (50 chars max)

<optional longer description>
<why this change was made>
<what problems it solves>

<optional footer - references, breaking changes>
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style (formatting, no logic change)
- `refactor`: Code restructuring (no feature/bug change)
- `test`: Adding/updating tests
- `chore`: Build tasks, configs, dependencies

### Examples

**Good**:
```
feat: add user authentication with JWT

Implements login and signup endpoints using JWT tokens.
Adds middleware for route protection.
Includes unit tests for auth service.

Closes #123
```

**Bad**:
```
fixed stuff
```

```
updated code and did things
```

### Tips

1. **Use imperative mood**: "Add feature" not "Added feature"
2. **Be specific**: "Fix navbar z-index on mobile" not "Fix bug"
3. **Explain why, not what**: Code shows what changed, commit explains why
4. **Reference issues**: "Closes #123" or "Fixes #456"
5. **Keep first line short**: 50 chars max (enforced by many tools)

---

## Git Workflows

### Feature Branch Workflow

Most common for team projects:

```bash
# 1. Start from updated main
git checkout main
git pull

# 2. Create feature branch
git checkout -b feature/user-profile

# 3. Make commits
git add .
git commit -m "feat: add user profile page"

# 4. Push to remote
git push -u origin feature/user-profile

# 5. Create Pull Request on GitHub/GitLab

# 6. After PR merged, cleanup
git checkout main
git pull
git branch -d feature/user-profile
```

### Gitflow Workflow

For projects with scheduled releases:

```
main      - Production code
develop   - Integration branch
feature/* - New features (branch from develop)
release/* - Preparing releases (branch from develop)
hotfix/*  - Urgent production fixes (branch from main)
```

### Trunk-Based Development

For continuous deployment:

```bash
# Everyone works on main with short-lived branches
git checkout -b quick-feature
# ... work for hours/1 day max ...
git push
# Create PR, merge same day
```

---

## Troubleshooting Common Issues

### "fatal: not a git repository"
```bash
# You're not in a git project
# Either initialize one:
git init

# Or clone an existing one:
git clone <url>
```

### "Your branch is ahead of 'origin/main' by X commits"
```bash
# You have local commits not pushed yet
git push
```

### "Your branch is behind 'origin/main' by X commits"
```bash
# Remote has commits you don't have
git pull
```

### "Your branch has diverged from 'origin/main'"
```bash
# Both local and remote have different commits
# Option 1: Merge remote changes
git pull

# Option 2: Rebase your commits on top of remote
git pull --rebase
```

### "Permission denied (publickey)"
```bash
# SSH key not set up
# Generate SSH key:
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add to GitHub/GitLab:
cat ~/.ssh/id_ed25519.pub
# Copy output and add to GitHub Settings → SSH Keys
```

### Large file error on push
```bash
# File too big for GitHub (>100MB)
# Remove from history:
git rm --cached large-file.zip
git commit --amend
# Then push
```

### Accidentally committed sensitive data
```bash
# IMPORTANT: Removing from history doesn't remove from GitHub!
# If already pushed, consider credentials compromised

# Remove from latest commit:
git rm --cached .env
git commit --amend

# Remove from entire history (dangerous):
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch .env" \
  --prune-empty --tag-name-filter cat -- --all

git push --force
```

### Restore deleted file
```bash
# Find commit where file existed
git log -- path/to/deleted-file.txt

# Restore from that commit
git checkout <commit-sha> -- path/to/deleted-file.txt
```

### See what changed in a commit
```bash
git show <commit-sha>

# Or compare with previous commit
git diff <commit-sha>^ <commit-sha>
```

---

## Additional Resources

- [Official Git Documentation](https://git-scm.com/doc)
- [GitHub Git Cheat Sheet](https://training.github.com/downloads/github-git-cheat-sheet/)
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)
- [Learn Git Branching](https://learngitbranching.js.org/) - Interactive visual tutorial
- [Oh Shit, Git!?!](https://ohshitgit.com/) - Quick fixes for common mistakes
- [Git Flight Rules](https://github.com/k88hudson/git-flight-rules) - What to do when things go wrong