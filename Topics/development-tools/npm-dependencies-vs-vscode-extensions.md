# NPM Dependencies vs VS Code Extensions

## The Common Confusion

Many developers wonder: "I have the Prettier extension in VS Code, why do I need `npm install prettier`?"

## The Key Difference

They serve **different purposes**:

### VS Code Extensions (Personal Tools)
- Work only on YOUR computer
- Give you live feedback while coding in the IDE
- Won't help teammates or CI/CD pipelines
- IDE-specific functionality

### NPM Dependencies (Project Requirements)
- Work for EVERYONE who clones the project
- Ensure same formatting/linting rules across team
- Enable automation (git hooks, CI/CD)
- Version-controlled and documented in package.json

## The Integration Pattern

```
VS Code Extension → Uses → Project's NPM Package
                           ↓
                    Same rules for everyone!
```

### How It Works

1. **NPM Package**: Defines the rules and tool version
   ```bash
   npm install --save-dev prettier
   ```

2. **VS Code Extension**: Provides IDE integration
   - Runs the project's NPM package
   - Uses the project's configuration
   - Gives real-time feedback in editor

3. **Result**: Everyone on the team uses the same tool version with the same rules

## Real-World Examples

### Example 1: Prettier

**NPM Package** (`npm install --save-dev prettier`):
- Defines formatting rules in `.prettierrc`
- Ensures consistent formatting for all team members
- Can be run in CI/CD: `npm run format`

**VS Code Extension** (esbenp.prettier-vscode):
- Formats code on save
- Shows formatting preview
- Integrates with VS Code's format command

### Example 2: ESLint

**NPM Package** (`npm install --save-dev eslint`):
- Defines code quality rules
- Version-controlled configuration
- Runs in pre-commit hooks and CI/CD

**VS Code Extension** (dbaeumer.vscode-eslint):
- Shows errors inline while typing
- Auto-fixes issues on save
- Provides quick fixes

## Best Practice: Use Both

For professional development:

1. ✅ **Install NPM package** (required)
   - Ensures team consistency
   - Enables automation
   - Version-controlled

2. ✅ **Install VS Code extension** (recommended)
   - Better developer experience
   - Immediate feedback
   - IDE integration

## Configuration Example

### package.json
```json
{
  "devDependencies": {
    "prettier": "^3.0.0",
    "eslint": "^8.50.0"
  },
  "scripts": {
    "format": "prettier --write .",
    "lint": "eslint ."
  }
}
```

### .vscode/settings.json
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

## When to Use Each

### NPM Package ONLY
- CLI tools you run manually
- CI/CD pipeline requirements
- Git hooks
- Build processes

### Both NPM Package AND Extension
- Code formatters (Prettier)
- Linters (ESLint)
- Testing frameworks (Jest)
- Any tool that benefits from IDE integration

### Extension ONLY (Rare)
- Pure IDE features (themes, icons)
- Tools without NPM equivalents
- Personal productivity tools

## Common Mistakes

### ❌ Extension Only
```
Problem: Team members have different versions and rules
Result: Inconsistent code formatting across team
```

### ❌ NPM Package Without Extension
```
Problem: Manual formatting required
Result: Slower development, inconsistent local experience
```

### ✅ Both Together
```
Benefit: Consistent rules + Great developer experience
Result: Professional, scalable development workflow
```

## Key Takeaway

**NPM packages** define the standards for the project.
**VS Code extensions** make those standards easy to follow.

Use both for professional development that scales beyond your personal setup.

## Related Topics
- [ESLint & Prettier Integration](../eslint/eslint-prettier-integration.md)
- [Port Management Commands](./port-management-commands.md)