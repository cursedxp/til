# Git Merging & Rebasing Guide

Understanding how to combine branches effectively.

---

## Understanding Merge vs Rebase

**Concept**: Both merge and rebase combine changes from different branches, but they do it differently.

### Merge: Combines histories by creating a new "merge commit"

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

### Rebase: Replays commits on top of another branch

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

### Golden Rule

**Never rebase commits you've pushed to a shared branch**

Rebasing changes commit SHAs, which confuses collaborators and can cause conflicts.

---

## Merging a Feature Branch

### Basic Merge Workflow

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

---

## Handling Merge Conflicts

When Git can't automatically merge, you'll see:
```bash
$ git merge feature/new-feature
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and then commit the result.
```

### Fix conflicts step by step:

**1. Open conflicted files** - Git marks conflicts like this:
```javascript
function greet() {
<<<<<<< HEAD (current change - main branch)
  console.log("Hello from main");
=======
  console.log("Hello from feature");
>>>>>>> feature/new-feature (incoming change)
}
```

**2. Edit the file** - Remove markers and keep what you want:
```javascript
function greet() {
  console.log("Hello from feature");  // Decided to keep feature version
}
```

**3. Stage resolved files**:
```bash
git add file.txt
```

**4. Complete the merge**:
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

## Rebasing Your Branch

### Use case
Keep your feature branch up-to-date with main while maintaining clean history.

```bash
# Step 1: Switch to your feature branch
git checkout feature/new-feature

# Step 2: Rebase on main
git rebase main

# Git replays each of your commits on top of main
# If successful:
# Successfully rebased and updated refs/heads/feature/new-feature
```

### What happens during rebase:

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

## Handling Rebase Conflicts

Conflicts are resolved one commit at a time:

```bash
$ git rebase main
# ... replaying commits ...
CONFLICT (content): Merge conflict in file.txt
# Fix the conflict, then continue
```

**Steps**:

1. **Fix the conflict** (same as merge - edit the file)
2. **Stage the fixed file**:
```bash
git add file.txt
```
3. **Continue the rebase**:
```bash
git rebase --continue
```

This might cause more conflicts for next commits. Repeat until done.

### Rebase commands:
```bash
git rebase --continue  # After fixing conflict, continue
git rebase --skip      # Skip current commit (rarely used)
git rebase --abort     # Cancel rebase, go back to before
```

---

## Interactive Rebase (Advanced)

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

### Common uses:
- **Combine tiny commits**: Change `pick` to `squash`
- **Fix commit messages**: Change `pick` to `reword`
- **Remove commits**: Change `pick` to `drop`

### Example: Squashing commits

```
Before:
pick a3f5b89 Add login form
pick b4e6c90 Fix typo in login
pick c5f7d01 Fix another typo

Change to:
pick a3f5b89 Add login form
squash b4e6c90 Fix typo in login
squash c5f7d01 Fix another typo

Result: One clean commit "Add login form"
```

---

## When to Use Each?

### Use Merge when:
- On shared/public branches (main, develop)
- Want to preserve exact history
- Multiple people work on same branch
- Following team conventions

### Use Rebase when:
- On your own feature branch
- Want clean, linear history
- Before creating a pull request
- Updating feature branch with latest main

---

## Typical Workflow

```bash
# 1. Create feature branch
git checkout -b feature/login
# ... make commits ...

# 2. Main got updated, sync your branch (rebase)
git fetch origin
git rebase origin/main

# 3. If conflicts, resolve them
git add .
git rebase --continue

# 4. Push your clean branch
git push origin feature/login
# If you've already pushed before rebasing:
git push --force-with-lease

# 5. Later: Merge into main via Pull Request (merge)
# GitHub/GitLab handles the merge
```

---

## Merge Types

### Fast-Forward Merge (default when possible)
```
Before:
main:     A---B
               \
feature:        C---D

After:
main:     A---B---C---D  (just moved pointer)
```

No merge commit created, just moves the branch pointer.

### No-Fast-Forward Merge
```bash
git merge --no-ff feature/login
```

```
Before:
main:     A---B
               \
feature:        C---D

After:
main:     A---B---------M (merge commit created)
               \       /
feature:        C---D
```

Always creates a merge commit, preserves feature branch history.

### Squash Merge
```bash
git merge --squash feature/login
git commit -m "Add login feature"
```

Combines all feature commits into one on main. Cleaner but loses detailed history.

---

## Cherry-Picking (Bonus)

Apply a specific commit from one branch to another:

```bash
# Get the commit SHA from another branch
git log other-branch --oneline

# Apply that commit to current branch
git cherry-pick abc1234
```

**Use case**: You need one specific fix from a feature branch but don't want to merge everything.

---

## Conflict Resolution Tips

### See which files have conflicts:
```bash
git status
# Unmerged paths:
#   both modified:   file.txt
```

### Use merge tools:
```bash
# VS Code
code .

# Or built-in merge tool
git mergetool
```

### Undo conflict resolution:
```bash
# During merge
git merge --abort

# During rebase
git rebase --abort
```

### Accept all changes from one side:
```bash
# Keep ours (current branch)
git checkout --ours file.txt

# Keep theirs (incoming branch)
git checkout --theirs file.txt

# Don't forget to stage after
git add file.txt
```

---

## Best Practices

1. **Pull before push**: Always sync with remote before pushing
2. **Rebase often**: Keep feature branch updated with main
3. **Don't rebase public history**: Only rebase your own unpushed commits
4. **Test after merge/rebase**: Make sure code still works
5. **Communicate force pushes**: Tell team if you force push shared branch
6. **Use --force-with-lease**: Safer than --force

---

## Navigation

- **Previous**: [Common Scenarios](git-scenarios.md)
- **Next**: [Workflows & Best Practices](git-workflows.md)
- **See Also**: [Git Basics](git-basics.md)