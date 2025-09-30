# Next.js Multiple Lockfiles Warning

## Problem

```
⚠ Warning: Next.js inferred your workspace root, but it may not be correct.
We detected multiple lockfiles and selected the directory of /Users/anilozsoy/package-lock.json as the root directory.
```

## Root Cause

- Accidental `package.json` files in home directory
- Usually caused by running `npm install` in wrong directory
- Next.js gets confused about which lockfile to use

## Diagnosis Steps

1. `ls -la ~/` to check home directory
2. `npm list -g --depth=0` to verify global packages
3. Identify which files shouldn't be there
4. Confirm project has its own package.json in correct location

## Solution

```bash
cd ~
rm package.json package-lock.json
rm -rf node_modules
```

## Prevention

- Always check `pwd` before npm commands
- Use `npm install -g` for CLI tools
- Keep project dependencies in project folders only
- Never run `npm install` in home directory unless intentional

## Related Topics
- [NPM Dependencies vs VS Code Extensions](../development-tools/npm-dependencies-vs-vscode-extensions.md)