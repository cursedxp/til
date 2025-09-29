# Troubleshooting Guide - Problems I've Solved

## 🛠️ How to Use This Guide
Document every problem you encounter and how you solved it. This becomes your personal debugging reference.

---

## ❌ Next.js Multiple Lockfiles Warning

### Problem
```
⚠ Warning: Next.js inferred your workspace root, but it may not be correct.
We detected multiple lockfiles and selected the directory of /Users/anilozsoy/package-lock.json as the root directory.
```

### Root Cause
- Accidental `package.json` files in home directory
- Usually caused by running `npm install` in wrong directory

### Diagnosis Steps
1. `ls -la ~/` to check home directory
2. `npm list -g --depth=0` to verify global packages
3. Identify which files shouldn't be there

### Solution
```bash
cd ~
rm package.json package-lock.json
rm -rf node_modules
```

### Prevention
- Always check `pwd` before npm commands
- Use `npm install -g` for CLI tools
- Keep project dependencies in project folders only

---

## ❌ ESLint Configuration Error

### Problem
ESLint config not loading, syntax errors in console

### Root Cause
Wrong syntax in `eslint.config.mjs` - missing quotes in extends array

### Diagnosis Steps
1. Check ESLint config file syntax
2. Look for array vs string formatting issues
3. Verify proper flat config format

### Solution
```javascript
// Wrong:
...compat.extends("next/core-web-vitals", "next/typescript,prettier")

// Right:
...compat.extends("next/core-web-vitals", "next/typescript", "prettier")
```

### Prevention
- Use proper JavaScript array syntax
- Test configurations after changes
- Read error messages carefully

---

## Template for Future Problems

## ❌ [Problem Title]

### Problem
[Description of what went wrong]

### Root Cause
[What actually caused the issue]

### Diagnosis Steps
1. [How I investigated]
2. [What I checked]
3. [How I found the cause]

### Solution
```[language]
[Code or commands that fixed it]
```

### Prevention
[How to avoid this in the future]

---