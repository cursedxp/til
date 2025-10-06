# Git Troubleshooting Guide

Common problems and their solutions.

---

## Error Messages

### "fatal: not a git repository"

**Problem**: You're not in a Git project directory.

**Solution**:
```bash
# Check if you're in the right folder
pwd

# Option 1: Navigate to your project
cd /path/to/your/project

# Option 2: Initialize a new repo
git init

# Option 3: Clone an existing repo
git clone <url>
```

---

### "Your branch is ahead of 'origin/main' by X commits"

**Problem**: You have local commits that aren't on the remote server.

**Solution**:
```bash
# Push your commits
git push

# If branch doesn't exist on remote yet
git push -u origin branch-name
```

---

### "Your branch is behind 'origin/main' by X commits"

**Problem**: The remote has commits you don't have locally.

**Solution**:
```bash
# Pull the latest changes
git pull

# Or fetch first to see what changed
git fetch
git log origin/main
git pull
```

---

### "Your branch has diverged from 'origin/main'"

**Problem**: Both local and remote have different commits.

```
Local:  A---B---C---X
Remote: A---B---C---Y
```

**Solution Option 1** (Merge - preserves both histories):
```bash
git pull
# Resolve any conflicts
git add .
git commit
git push
```

**Solution Option 2** (Rebase - linear history):
```bash
git pull --rebase
# Resolve any conflicts
git add .
git rebase --continue
git push
```

---

### "Permission denied (publickey)"

**Problem**: SSH authentication failed with GitHub/GitLab.

**Solution**:
```bash
# 1. Check if you have SSH keys
ls -la ~/.ssh
# Look for id_rsa.pub or id_ed25519.pub

# 2. If no keys, generate one
ssh-keygen -t ed25519 -C "your.email@example.com"
# Press Enter for all prompts

# 3. Copy your public key
cat ~/.ssh/id_ed25519.pub
# Copy the entire output

# 4. Add to GitHub
# Go to: GitHub → Settings → SSH and GPG keys → New SSH key
# Paste your key

# 5. Test connection
ssh -T git@github.com
# Should see: "Hi username! You've successfully authenticated..."
```

**Alternative**: Use HTTPS instead of SSH
```bash
# Change remote URL to HTTPS
git remote set-url origin https://github.com/username/repo.git
```

---

### "fatal: refusing to merge unrelated histories"

**Problem**: Trying to merge two repos with no common ancestor.

**Solution**:
```bash
# Allow unrelated histories (use carefully!)
git pull origin main --allow-unrelated-histories

# Resolve any conflicts
git add .
git commit
```

---

### "error: failed to push some refs"

**Problem**: Remote has changes you don't have, or you're trying to force push.

**Solution**:
```bash
# Pull first, then push
git pull
git push

# If you really need to force push (be careful!)
git push --force-with-lease
```

---

## File Issues

### Accidentally committed large file

**Problem**: GitHub rejects push because file > 100MB.

**Solution**:
```bash
# Remove from last commit
git rm --cached large-file.zip
git commit --amend --no-edit
git push

# If file is in history (use Git LFS or filter)
# Install git-lfs first
git lfs install
git lfs track "*.zip"
git add .gitattributes
git commit -m "chore: add Git LFS"
```

---

### Accidentally committed sensitive data (.env, passwords)

**Problem**: Committed `.env` file with API keys or passwords.

**⚠️ IMPORTANT**: Once pushed to GitHub, consider the data compromised. Change all credentials immediately!

**Solution**:
```bash
# Remove from latest commit (before push)
git rm --cached .env
git commit --amend --no-edit

# Add to .gitignore
echo ".env" >> .gitignore
git add .gitignore
git commit -m "chore: add .env to gitignore"

# If already pushed, remove from entire history
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch .env" \
  --prune-empty --tag-name-filter cat -- --all

git push --force

# Rotate all compromised credentials!
```

---

### Deleted file accidentally

**Problem**: Deleted important file and need to restore it.

**Solution**:
```bash
# If not yet committed, restore from last commit
git restore filename.txt

# If committed, find when it was deleted
git log --all --full-history -- path/to/file.txt

# Restore from before deletion
git checkout <commit-sha>^ -- path/to/file.txt

# Commit the restored file
git add path/to/file.txt
git commit -m "restore: bring back deleted file"
```

---

### Restore entire directory

```bash
# Restore directory from specific commit
git checkout <commit-sha> -- path/to/directory/

# Restore from a different branch
git checkout other-branch -- path/to/directory/
```

---

## Conflict Issues

### Merge conflicts

**Problem**: Git can't automatically merge changes.

**Solution**:
```bash
# 1. See conflicted files
git status

# 2. Open each file and look for markers
# <<<<<<< HEAD
# your changes
# =======
# their changes
# >>>>>>> branch-name

# 3. Edit file to resolve conflict
# Remove markers and choose what to keep

# 4. Stage resolved files
git add filename.txt

# 5. Complete merge
git commit

# Or abort if you want to start over
git merge --abort
```

---

### Rebase conflicts

**Problem**: Conflicts during rebase.

**Solution**:
```bash
# 1. Resolve conflict in file (same as merge)

# 2. Stage resolved file
git add filename.txt

# 3. Continue rebase
git rebase --continue

# If more conflicts, repeat steps 1-3

# Or abort entirely
git rebase --abort
```

---

## Branch Issues

### Can't delete branch

**Problem**: "error: The branch 'feature' is not fully merged"

**Solution**:
```bash
# Check if really unmerged
git branch --no-merged

# If safe to delete, force delete
git branch -D feature-branch

# If you want to merge first
git checkout main
git merge feature-branch
git branch -d feature-branch
```

---

### Accidentally working on wrong branch

**Problem**: Made commits on main instead of feature branch.

**Solution**:
```bash
# 1. Create new branch from current state
git branch feature/new-work

# 2. Reset main to before your commits
git checkout main
git reset --hard origin/main

# 3. Continue work on feature branch
git checkout feature/new-work
```

---

### Detached HEAD state

**Problem**: "You are in 'detached HEAD' state"

**Solution**:
```bash
# If you made commits you want to keep
git branch temp-branch
git checkout main
git merge temp-branch

# If you don't care about changes
git checkout main
```

---

## Commit Issues

### Undo last commit (keep changes)

```bash
# Undo commit, keep changes staged
git reset --soft HEAD~1

# Undo commit, keep changes unstaged
git reset HEAD~1

# Undo commit, discard changes (dangerous!)
git reset --hard HEAD~1
```

---

### Amend commit after pushing

**Problem**: Need to fix commit that's already pushed.

**Solution**:
```bash
# Make your changes
git add .
git commit --amend

# Force push (use with caution on shared branches!)
git push --force-with-lease
```

---

### Change author of last commit

```bash
git commit --amend --author="Name <email@example.com>"
git push --force-with-lease
```

---

### Committed to wrong branch

See "Accidentally working on wrong branch" above.

---

## Remote Issues

### Remote not found

**Problem**: "fatal: 'origin' does not appear to be a git repository"

**Solution**:
```bash
# Check remotes
git remote -v

# Add origin
git remote add origin https://github.com/user/repo.git

# Or update existing
git remote set-url origin https://github.com/user/repo.git
```

---

### Multiple remotes

```bash
# Add upstream (for forks)
git remote add upstream https://github.com/original/repo.git

# Fetch from upstream
git fetch upstream

# Merge upstream changes
git merge upstream/main
```

---

## Performance Issues

### Git is slow

**Problem**: Git commands taking too long.

**Solution**:
```bash
# Run garbage collection
git gc

# Clean up unnecessary files
git prune

# Check repo size
du -sh .git

# If too large, consider git-lfs for large files
```

---

### Repository too large

**Problem**: .git folder is huge.

**Solution**:
```bash
# Find large files in history
git rev-list --objects --all | \
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
  sed -n 's/^blob //p' | \
  sort --numeric-sort --key=2 | \
  tail -20

# Use Git LFS for large files
git lfs migrate import --include="*.zip,*.pdf"
```

---

## Recovery

### Recover deleted commits

```bash
# View reflog (history of HEAD)
git reflog

# Find your lost commit
# Look for the SHA before disaster

# Recover it
git cherry-pick <commit-sha>
# OR
git reset --hard <commit-sha>
```

---

### Recover deleted branch

```bash
# Find branch tip in reflog
git reflog | grep branch-name

# Recreate branch
git branch branch-name <commit-sha>
```

---

### "Oh no, I deleted everything!"

```bash
# If not committed yet, you're probably out of luck
# But try reflog
git reflog

# Find commit before disaster
git reset --hard <commit-sha>

# For future: commit often!
```

---

## Configuration Issues

### Check Git configuration

```bash
# Global config
git config --global --list

# Local config (repository)
git config --local --list

# System config
git config --system --list
```

---

### Reset specific config

```bash
# Remove specific setting
git config --global --unset user.name

# Edit config file directly
git config --global --edit
```

---

### Username/email wrong

```bash
# Set globally
git config --global user.name "Your Name"
git config --global user.email "email@example.com"

# Or just for current repo
git config user.name "Your Name"
git config user.email "email@example.com"
```

---

## Prevention Tips

1. **Commit often**: Small commits are easier to undo
2. **Branch always**: Don't work directly on main
3. **Pull before push**: Stay synced with remote
4. **Check status**: `git status` before committing
5. **Use .gitignore**: Don't commit sensitive files
6. **Test before push**: Make sure code works
7. **Read error messages**: Git usually tells you what to do

---

## Emergency Commands

### Nuclear option (start fresh)

```bash
# If everything is broken and you just want a clean slate:

# 1. Save your current work
cp -r myproject myproject-backup

# 2. Delete .git folder
rm -rf .git

# 3. Re-initialize
git init
git remote add origin <url>
git pull origin main
```

---

## Getting Help

```bash
# Help for a command
git help commit
git commit --help

# Quick reference
git <command> -h
```

---

## Navigation

- **Previous**: [Workflows](git-workflows.md)
- **See Also**: [Quick Reference](git-quick-reference.md)
- **Back to**: [Main Guide](git-commands-guide.md)