# Git Workflows & Best Practices

Professional workflows and conventions for team collaboration.

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

```
fix: resolve navbar z-index issue on mobile

The navbar was appearing behind modal dialogs on mobile.
Increased z-index from 100 to 1000 to fix stacking context.

Fixes #456
```

**Bad**:
```
fixed stuff
```

```
updated code and did things
```

```
changes
```

### Tips

1. **Use imperative mood**: "Add feature" not "Added feature" or "Adding feature"
2. **Be specific**: "Fix navbar z-index on mobile" not "Fix bug"
3. **Explain why, not what**: Code shows what changed, commit explains why
4. **Reference issues**: "Closes #123" or "Fixes #456"
5. **Keep first line short**: 50 chars max (enforced by many tools)
6. **Separate subject from body**: Blank line between summary and description
7. **Wrap body at 72 chars**: For better readability in various tools

---

## Feature Branch Workflow

Most common for team projects.

### Overview

Everyone works on feature branches and merges to main via Pull Requests.

```
main (protected)
  ├── feature/user-login
  ├── feature/payment-integration
  └── bugfix/navbar-issue
```

### Workflow Steps

```bash
# 1. Start from updated main
git checkout main
git pull

# 2. Create feature branch
git checkout -b feature/user-profile

# 3. Make commits
git add .
git commit -m "feat: add user profile page"

# More commits...
git commit -m "test: add profile page tests"

# 4. Keep branch updated with main
git fetch origin
git rebase origin/main

# 5. Push to remote
git push -u origin feature/user-profile

# 6. Create Pull Request on GitHub/GitLab
# (via web interface)

# 7. After PR approved and merged, cleanup
git checkout main
git pull
git branch -d feature/user-profile
```

### Benefits

- ✅ Main branch always stable
- ✅ Code review before merge
- ✅ Easy to collaborate
- ✅ Clear feature isolation

---

## Gitflow Workflow

For projects with scheduled releases and multiple environments.

### Branch Structure

```
main          - Production code (tagged with versions)
develop       - Integration branch
feature/*     - New features (branch from develop)
release/*     - Preparing releases (branch from develop)
hotfix/*      - Urgent production fixes (branch from main)
```

### Visual

```
main:       v1.0 -------- hotfix/bug ------ v1.1 -------- v2.0
                \                            /            /
develop:         \---- feature/A ---- release/2.0 -------
                  \                   /
feature/A:         \-----------------
```

### Workflow

**Starting a feature**:
```bash
git checkout develop
git pull
git checkout -b feature/shopping-cart
# ... work ...
git push origin feature/shopping-cart
# Create PR to develop
```

**Creating a release**:
```bash
git checkout develop
git checkout -b release/1.2.0
# Bump version, update changelog, final testing
git checkout main
git merge release/1.2.0
git tag v1.2.0
git checkout develop
git merge release/1.2.0
git push --all && git push --tags
```

**Hotfix**:
```bash
git checkout main
git checkout -b hotfix/critical-bug
# ... fix ...
git checkout main
git merge hotfix/critical-bug
git tag v1.1.1
git checkout develop
git merge hotfix/critical-bug
git push --all && git push --tags
```

### When to use

- Scheduled releases (monthly, quarterly)
- Multiple versions in production
- Large teams
- Need strict control over what goes to production

---

## Trunk-Based Development

For continuous deployment and fast iteration.

### Overview

Everyone works on main (or very short-lived branches). Deploy multiple times per day.

```
main: A---B---C---D---E---F  (all production-ready)
```

### Workflow

```bash
# Option 1: Direct to main (small changes)
git checkout main
git pull
# ... make small change ...
git commit -m "fix: typo in button text"
git push

# Option 2: Short-lived branch (max 1 day)
git checkout -b quick-feature
# ... work for hours ...
git push
# Create PR, merge same day
```

### Rules

- Commits must be small and atomic
- All commits production-ready
- Feature flags for incomplete features
- Strong automated testing
- Fast code review (< 1 hour)

### When to use

- Continuous deployment
- Small, experienced team
- Strong automated testing
- Fast-moving projects

---

## Pull Request Best Practices

### Creating Good PRs

**1. Size**: Keep PRs small (< 400 lines changed)
```
Good: One feature, well-tested
Bad:  Multiple features + refactoring + bug fixes
```

**2. Description**:
```markdown
## Summary
Brief description of changes

## Changes Made
- Added user authentication
- Created login/signup pages
- Added JWT middleware

## Why
Users need to log in to access protected features

## Test Plan
- [ ] Manual testing on dev environment
- [ ] Unit tests pass
- [ ] E2E tests pass

## Screenshots
(if UI changes)

Closes #123
```

**3. Keep it focused**:
- One feature per PR
- Don't mix refactoring with features
- Don't mix bug fixes with features

### Reviewing PRs

**As a reviewer**:
1. Check code quality and style
2. Look for bugs and edge cases
3. Suggest improvements
4. Approve or request changes

```bash
# Check out PR locally to test
gh pr checkout 123
npm test
npm run dev
```

**As an author**:
1. Respond to all comments
2. Make requested changes
3. Thank reviewers
4. Don't take criticism personally

---

## Branch Protection Rules

Set these on main/develop branches:

### On GitHub:

Settings → Branches → Add branch protection rule

**Recommended settings**:
- ✅ Require pull request before merging
- ✅ Require approvals (1-2)
- ✅ Dismiss stale approvals when new commits pushed
- ✅ Require status checks to pass (tests, linting)
- ✅ Require branches to be up to date before merging
- ✅ Require linear history (no merge commits)
- ✅ Do not allow bypassing the above settings

---

## Common Team Conventions

### Branch Naming

```
feature/description      feature/user-authentication
bugfix/description       bugfix/navbar-z-index
hotfix/description       hotfix/security-vulnerability
chore/description        chore/update-dependencies
docs/description         docs/add-contributing-guide
test/description         test/add-integration-tests
refactor/description     refactor/simplify-auth-logic
```

### Commit Frequency

**Too few commits**:
```
❌ feat: entire authentication system (1000 lines)
```

**Too many commits**:
```
❌ add file
❌ fix typo
❌ fix another typo
❌ actually fix typo
```

**Good balance**:
```
✅ feat: add login endpoint
✅ feat: add signup endpoint
✅ test: add auth tests
✅ docs: update API documentation
```

### When to Squash

**Before merging PR** (via GitHub):
- Multiple WIP commits
- Many small fix commits
- Want clean main history

**Don't squash**:
- Meaningful separate commits
- Want to preserve history
- Multiple logical changes

---

## Collaboration Tips

### Before Starting Work

```bash
# Always start from latest main
git checkout main
git pull
git checkout -b feature/new-work
```

### During Work

```bash
# Commit often (locally)
git commit -m "WIP: working on feature"

# Keep branch updated
git fetch origin
git rebase origin/main

# Push regularly (backup)
git push
```

### Before Creating PR

```bash
# Clean up commits (if needed)
git rebase -i HEAD~5

# Make sure tests pass
npm test

# Make sure linting passes
npm run lint

# Update from main one last time
git fetch origin
git rebase origin/main
git push --force-with-lease
```

### After PR Merged

```bash
# Update local main
git checkout main
git pull

# Delete feature branch
git branch -d feature/completed-work

# Start new work
git checkout -b feature/next-task
```

---

## Tags and Releases

### Creating Tags

```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push tags
git push origin v1.0.0
# Or push all tags
git push --tags
```

### Semantic Versioning

```
v1.2.3
│ │ │
│ │ └─ PATCH: Bug fixes
│ └─── MINOR: New features (backwards compatible)
└───── MAJOR: Breaking changes

Examples:
v1.0.0 → v1.0.1  (bug fix)
v1.0.1 → v1.1.0  (new feature)
v1.1.0 → v2.0.0  (breaking change)
```

### Release Process

```bash
# 1. Update version in package.json
npm version minor  # or major, or patch

# 2. Update CHANGELOG.md

# 3. Commit and tag
git commit -am "chore: release v1.2.0"
git tag -a v1.2.0 -m "Release v1.2.0"

# 4. Push
git push && git push --tags

# 5. Create GitHub release
gh release create v1.2.0 --notes "Release notes here"
```

---

## Navigation

- **Previous**: [Merging & Rebasing](git-merge-rebase.md)
- **Next**: [Troubleshooting](git-troubleshooting.md)
- **See Also**: [Quick Reference](git-quick-reference.md)