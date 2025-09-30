# ESLint Configuration Syntax Errors

## Common Configuration Mistakes

### The Flat Config Format

Modern ESLint uses a "flat config" format (`eslint.config.mjs`) instead of the legacy `.eslintrc` format. This new format uses JavaScript/TypeScript directly, which means you need to follow proper JavaScript syntax.

## Most Common Error: Array Syntax

### The Problem

```javascript
// ❌ Wrong (missing comma, creating one string):
...compat.extends("next/core-web-vitals", "next/typescript,prettier")

// ✅ Right (proper array with separate strings):
...compat.extends("next/core-web-vitals", "next/typescript", "prettier")
```

### What Happened

In the wrong version:
- `"next/typescript,prettier"` is treated as **one string**
- ESLint looks for a config called "next/typescript,prettier"
- Config doesn't exist → Error

In the correct version:
- Three separate strings in array
- ESLint loads each config: "next/core-web-vitals", "next/typescript", "prettier"
- All configs found → Works!

## Error Messages to Watch For

### Cannot Find Module Error
```
Error: Cannot find module 'next/typescript,prettier'
```

**Diagnosis**: Check for missing commas in extends array

### Config Not Found
```
ESLint couldn't find the config "..." to extend from
```

**Diagnosis**: Typo in config name or missing comma in array

### Unexpected Token
```
SyntaxError: Unexpected token
```

**Diagnosis**: Invalid JavaScript syntax in config file

## Configuration File Examples

### Correct Flat Config Format

```javascript
import { FlatCompat } from "@eslint/eslintrc";

const compat = new FlatCompat({
  baseDirectory: import.meta.dirname,
});

export default [
  // Spread operator to merge configs
  ...compat.extends(
    "next/core-web-vitals",
    "next/typescript",
    "prettier"  // Note: prettier should be last to override conflicting rules
  )
];
```

### Adding Custom Rules

```javascript
export default [
  ...compat.extends("next/core-web-vitals", "next/typescript", "prettier"),
  {
    rules: {
      "no-console": "warn",
      "prefer-const": "error",
      "@typescript-eslint/no-unused-vars": ["error", {
        "argsIgnorePattern": "^_"
      }]
    }
  }
];
```

### Multiple Config Objects

```javascript
export default [
  // Base configs
  ...compat.extends("next/core-web-vitals", "next/typescript"),

  // Prettier integration (separate for clarity)
  ...compat.extends("prettier"),

  // Custom rules
  {
    rules: {
      "no-console": "warn"
    }
  },

  // Files to ignore
  {
    ignores: ["dist", "build", "node_modules"]
  }
];
```

## Debugging Steps

### Step 1: Check Syntax
```bash
# Run ESLint to see error messages
npm run lint
```

### Step 2: Verify Config File
```javascript
// Look for these common mistakes:
// 1. Missing commas between array items
// 2. Missing quotes around strings
// 3. Typos in config names
// 4. Missing spread operator (...)
```

### Step 3: Validate JavaScript
```bash
# Try running the config file directly
node eslint.config.mjs
```

### Step 4: Check Dependencies
```bash
# Ensure all referenced configs are installed
npm list eslint-config-prettier
npm list @eslint/eslintrc
```

## Key Lessons

### 1. JavaScript Arrays Need Proper Syntax
- Separate items with commas
- Each string in quotes
- No trailing commas before closing bracket (in most cases)

### 2. Small Typos Cause Big Errors
- Missing comma makes two items into one
- ESLint gives confusing error messages
- Always check syntax first

### 3. Read Error Messages Carefully
- "Cannot find module X,Y" → Missing comma
- "Unexpected token" → Syntax error
- "Config not found" → Typo or missing dependency

### 4. Order Matters
```javascript
// Prettier should be LAST to override conflicting rules
...compat.extends(
  "next/core-web-vitals",
  "next/typescript",
  "prettier"  // Last!
)
```

## Prevention Strategies

### 1. Use IDE Extensions
- ESLint extension shows errors in real-time
- Syntax highlighting helps spot mistakes
- Auto-completion prevents typos

### 2. Test After Changes
```bash
# Run lint after editing config
npm run lint
```

### 3. Format Config Files
```bash
# Use Prettier on config files too
npx prettier --write eslint.config.mjs
```

### 4. Version Control
```bash
# Commit working configs
git add eslint.config.mjs
git commit -m "chore: update ESLint configuration"
```

## Understanding Flat Config vs Legacy

### Legacy Format (`.eslintrc.json`)
```json
{
  "extends": [
    "next/core-web-vitals",
    "next/typescript",
    "prettier"
  ]
}
```

### Flat Config Format (`eslint.config.mjs`)
```javascript
export default [
  ...compat.extends(
    "next/core-web-vitals",
    "next/typescript",
    "prettier"
  )
];
```

**Key Differences:**
- Flat config uses JavaScript/TypeScript
- More flexible and powerful
- Requires proper JS syntax
- New default in ESLint v9+

## Resources

### Official Documentation
- [ESLint Flat Config](https://eslint.org/docs/latest/use/configure/configuration-files)
- [ESLint Migration Guide](https://eslint.org/docs/latest/use/configure/migration-guide)

### Helpful Tools
- ESLint VS Code extension
- ESLint config inspector: `npx eslint --inspect-config`

## Related Topics
- [ESLint & Prettier Integration](./eslint-prettier-integration.md)
- [NPM Dependencies vs VS Code Extensions](../development-tools/npm-dependencies-vs-vscode-extensions.md)
- [Troubleshooting Template](../troubleshooting/problem-solving-template.md)