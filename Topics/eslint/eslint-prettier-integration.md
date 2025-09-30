# ESLint & Prettier Integration

## Overview

Professional developers create **consistent, automated workflows** that work across teams and environments. This guide covers setting up ESLint and Prettier to work together seamlessly.

## Understanding the Tools

### Prettier - Code Formatter
- **Purpose**: Automatic, opinionated code formatting
- **Focus**: How code looks (spacing, quotes, semicolons)
- **Operation**: Rewrites code to match style rules

### ESLint - Code Quality Tool
- **Purpose**: Find bugs and enforce code quality rules
- **Focus**: How code works (logic errors, best practices)
- **Operation**: Analyzes code and reports issues

## Package Installation

### Core Dependencies
```bash
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

### What Each Package Does

**`prettier`**
- The actual code formatter
- Ensures consistency across the team
- Everyone uses the same version

**`eslint-config-prettier`**
- Disables ESLint rules that conflict with Prettier
- Prevents rule fights between tools
- Essential for integration

**`eslint-plugin-prettier`**
- Shows Prettier issues as ESLint errors
- Integrates Prettier into ESLint workflow
- Unified error reporting

## Configuration Files

### 1. Prettier Configuration (`.prettierrc`)
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "tabWidth": 2,
  "printWidth": 80
}
```

**Configuration Explained:**
- `semi: true` - Always use semicolons
- `trailingComma: "es5"` - Add trailing commas where valid in ES5
- `singleQuote: false` - Use double quotes
- `tabWidth: 2` - 2 spaces for indentation
- `printWidth: 80` - Wrap lines at 80 characters

### 2. VS Code Workspace Settings (`.vscode/settings.json`)
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

**What This Enables:**
- Auto-format files when saving
- Use Prettier (not VS Code's built-in formatter)
- Auto-fix ESLint errors on save

### 3. NPM Scripts (`package.json`)
```json
{
  "scripts": {
    "lint:fix": "eslint . --fix",
    "format": "prettier --write ."
  }
}
```

**Usage:**
```bash
# Fix all ESLint errors automatically
npm run lint:fix

# Format all files with Prettier
npm run format
```

### 4. ESLint Integration (`eslint.config.mjs`)
```javascript
import { FlatCompat } from "@eslint/eslintrc";

const compat = new FlatCompat({
  baseDirectory: import.meta.dirname,
});

export default [
  ...compat.extends("next/core-web-vitals", "next/typescript", "prettier")
];
```

**Key Points:**
- Modern "flat config" format
- Includes Prettier config at the end
- Proper array syntax (commas between strings)

## Testing the Setup

### Quick Test Process
1. Open any `.tsx` or `.ts` file
2. Add extra spaces, remove semicolons, mess up formatting
3. Save the file (`Cmd+S` / `Ctrl+S`)
4. **Should auto-format instantly!**

### Expected Behavior
- ✅ Prettier fixes spacing, quotes, semicolons
- ✅ ESLint shows errors for actual code problems
- ✅ No conflicts between the two tools
- ✅ Consistent formatting across all files

## Professional Workflow Benefits

### 1. Code Quality Automation
Professional teams don't rely on manual formatting:
- **Prettier**: Automatic formatting on save
- **ESLint**: Catch bugs and enforce rules
- **Git hooks**: Prevent bad code from being committed
- **CI/CD**: Verify quality before deployment

### 2. Team Consistency
The goal: **predictable, maintainable code** that any team member can understand and modify

### 3. Time Savings
- No manual formatting
- No style debates
- Automated quality checks
- Faster code reviews

## Common Issues and Solutions

### Issue 1: ESLint and Prettier Fighting
**Symptom**: Code keeps reformatting back and forth

**Solution**: Ensure `eslint-config-prettier` is installed and added to ESLint config

```javascript
// Must have "prettier" at the END of extends array
...compat.extends("next/core-web-vitals", "next/typescript", "prettier")
```

### Issue 2: Format on Save Not Working
**Check:**
1. Prettier extension installed in VS Code?
2. `.vscode/settings.json` configured correctly?
3. Prettier installed as npm dependency?

**Solution**: Install both the extension AND npm package

### Issue 3: ESLint Config Syntax Error
**Common mistake:**
```javascript
// Wrong (missing comma):
...compat.extends("next/core-web-vitals", "next/typescript,prettier")

// Right (proper array):
...compat.extends("next/core-web-vitals", "next/typescript", "prettier")
```

## Alternative Approaches

### Separate Prettier and ESLint
Some teams prefer running Prettier separately:
```json
{
  "scripts": {
    "lint": "eslint .",
    "format": "prettier --write .",
    "check": "npm run lint && npm run format"
  }
}
```

### Pre-commit Hooks with Husky
Automate quality checks before commits:
```bash
npm install --save-dev husky lint-staged

# Configure to run Prettier and ESLint before commits
```

## Key Takeaways

1. **Tools work together as a system**, not in isolation
2. **Configuration matters** as much as code
3. **Small setup time** saves hours of manual work later
4. **Understanding the "why"** prevents future problems
5. **Use both NPM package and VS Code extension** for best experience

## Why This Matters

- **Debugging skills**: Learning to trace problems to root cause
- **Environment setup**: Foundation for all future development
- **Team collaboration**: Tools that scale beyond personal projects
- **Professional practices**: How real software teams work

## Related Topics
- [NPM Dependencies vs VS Code Extensions](../development-tools/npm-dependencies-vs-vscode-extensions.md)
- [ESLint Configuration Syntax Errors](./eslint-config-syntax-errors.md)
- [Conventional Commits](../git/conventional-commits.md)