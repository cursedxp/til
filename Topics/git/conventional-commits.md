# Conventional Commit Messages

## Overview

Conventional commits provide a standardized format for git commit messages, making project history clearer and enabling automated tooling for changelogs and semantic versioning.

## Format

```
<type>: <description>

[optional body]

[optional footer]
```

## Commit Types

### fix: Bug Fixes and Problem Solving
**Use when:** Solving problems, correcting errors, fixing bugs

**Examples:**
```bash
git commit -m "fix: resolve login validation error"
git commit -m "fix: correct ESLint configuration syntax"
git commit -m "fix: resolve multiple lockfiles warning"
git commit -m "fix: prevent memory leak in user session"
```

**When to use:**
- Fixing broken functionality
- Correcting syntax errors
- Resolving configuration issues
- Patching security vulnerabilities

---

### feat: New Features
**Use when:** Adding new functionality or capabilities

**Examples:**
```bash
git commit -m "feat: add user authentication"
git commit -m "feat: implement dark mode toggle"
git commit -m "feat: add export to PDF functionality"
git commit -m "feat: integrate payment gateway"
```

**When to use:**
- Adding new user-facing features
- Implementing new API endpoints
- Creating new components
- Adding new business logic

---

### chore: Maintenance Tasks
**Use when:** Routine maintenance, no bug fixes or features

**Examples:**
```bash
git commit -m "chore: update dependencies"
git commit -m "chore: add .gitignore entries"
git commit -m "chore: update Node.js version"
git commit -m "chore: clean up unused imports"
```

**When to use:**
- Dependency updates
- Build configuration changes
- Package manager updates
- Removing dead code
- Updating tooling configuration

---

### docs: Documentation Changes
**Use when:** Only documentation changes, no code modifications

**Examples:**
```bash
git commit -m "docs: update README with setup instructions"
git commit -m "docs: add API documentation"
git commit -m "docs: fix typos in contributing guide"
git commit -m "docs: add code examples to README"
```

**When to use:**
- README updates
- Comment additions/improvements
- Documentation site changes
- Inline code documentation

---

### style: Code Style Changes
**Use when:** Formatting changes only, no behavior change

**Examples:**
```bash
git commit -m "style: format code with prettier"
git commit -m "style: fix indentation"
git commit -m "style: update code to match style guide"
git commit -m "style: remove trailing whitespace"
```

**When to use:**
- Running code formatters
- Fixing indentation
- Removing whitespace
- Organizing imports
- **Important:** No logic changes!

---

### refactor: Code Restructuring
**Use when:** Improving code structure without changing behavior

**Examples:**
```bash
git commit -m "refactor: extract reusable components"
git commit -m "refactor: simplify authentication logic"
git commit -m "refactor: split monolithic function into modules"
git commit -m "refactor: rename variables for clarity"
```

**When to use:**
- Improving code organization
- Extracting functions/components
- Renaming for clarity
- Simplifying complex logic
- **Important:** Same behavior, better structure

---

### test: Test Changes
**Use when:** Adding or modifying tests

**Examples:**
```bash
git commit -m "test: add unit tests for auth service"
git commit -m "test: update snapshot tests"
git commit -m "test: add integration tests for API"
git commit -m "test: fix flaky test in user flow"
```

**When to use:**
- Adding new tests
- Updating existing tests
- Fixing test failures
- Improving test coverage

---

### perf: Performance Improvements
**Use when:** Improving performance without adding features

**Examples:**
```bash
git commit -m "perf: optimize database queries"
git commit -m "perf: reduce bundle size by lazy loading"
git commit -m "perf: add caching layer for API calls"
git commit -m "perf: memoize expensive calculations"
```

**When to use:**
- Optimization work
- Reducing load times
- Improving response times
- Reducing memory usage

---

### build: Build System Changes
**Use when:** Changes to build process or dependencies

**Examples:**
```bash
git commit -m "build: upgrade webpack to v5"
git commit -m "build: add production optimization flags"
git commit -m "build: configure code splitting"
git commit -m "build: update tsconfig for stricter checks"
```

**When to use:**
- Build tool updates
- Compiler configuration
- Build script changes
- CI/CD pipeline updates

---

### ci: Continuous Integration Changes
**Use when:** Changes to CI/CD configuration

**Examples:**
```bash
git commit -m "ci: add GitHub Actions workflow"
git commit -m "ci: update deployment pipeline"
git commit -m "ci: add code coverage reporting"
git commit -m "ci: configure automated tests"
```

**When to use:**
- GitHub Actions changes
- Jenkins configuration
- Travis CI updates
- Deployment automation

---

## Quick Decision Guide

**I'm fixing something broken** → `fix:`

**I'm adding something new** → `feat:`

**I'm doing housekeeping** → `chore:`

**I'm improving code structure** → `refactor:`

**I'm only changing formatting** → `style:`

**I'm only updating docs** → `docs:`

**I'm working on tests** → `test:`

**I'm optimizing performance** → `perf:`

## Advanced: Scope and Breaking Changes

### Adding Scope
```bash
git commit -m "feat(auth): add two-factor authentication"
git commit -m "fix(api): resolve timeout issue"
git commit -m "docs(readme): update installation steps"
```

### Breaking Changes
```bash
git commit -m "feat!: change API response format

BREAKING CHANGE: API now returns data in { data: {}, meta: {} } format"
```

## Real-World Examples

### Example 1: ESLint/Prettier Setup
```bash
# Could be either:
git commit -m "fix: resolve multiple lockfiles warning and correct ESLint config"
# OR
git commit -m "chore: setup ESLint and Prettier configuration"
```

### Example 2: Adding Dark Mode
```bash
git commit -m "feat: implement dark mode toggle

- Add theme context provider
- Create theme switcher component
- Update all components for theme support
- Persist theme preference in localStorage"
```

### Example 3: Refactoring
```bash
git commit -m "refactor: extract authentication logic into separate service

Moved auth logic from components into dedicated service for better
testability and reusability"
```

## Benefits

1. **Clear history**: Understand changes at a glance
2. **Automated changelogs**: Tools can generate release notes
3. **Semantic versioning**: Automatically determine version bumps
4. **Better collaboration**: Team understands change intent
5. **Searchable commits**: Easy to find specific types of changes

## Tools That Use Conventional Commits

- **semantic-release**: Automated version management
- **conventional-changelog**: Generate changelogs
- **commitlint**: Enforce commit message format
- **standard-version**: Automated versioning and changelog

## Related Topics
- [ESLint & Prettier Integration](../eslint/eslint-prettier-integration.md)
- [Git Workflow Best Practices](./git-workflow-best-practices.md) (future)