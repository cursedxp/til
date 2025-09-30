# Troubleshooting Guide Template

## How to Use This Template

Document every problem you encounter and how you solved it. This becomes your personal debugging reference. Create individual files for each major problem category, using this template as a guide.

---

## Template for Documenting Problems

## ❌ [Problem Title]

### Problem
[Description of what went wrong - error messages, unexpected behavior, etc.]

### Root Cause
[What actually caused the issue - be specific about the underlying reason]

### Diagnosis Steps
1. [How I investigated - commands run, logs checked]
2. [What I checked - configurations, files, dependencies]
3. [How I found the cause - breakthrough moment, key insight]

### Solution
```[language]
[Code or commands that fixed it]
```

### Prevention
[How to avoid this in the future - best practices, checks to run]

---

## Tips for Good Documentation

1. **Be specific**: Include exact error messages and file paths
2. **Show your work**: Document the investigation process, not just the solution
3. **Think future-you**: Will you understand this in 6 months?
4. **Include context**: What were you trying to do when this happened?
5. **Add timestamps**: When did this happen? (for version-specific issues)

## Categories to Consider

- **Configuration Issues**: Wrong settings, syntax errors
- **Dependency Conflicts**: Version mismatches, missing packages
- **Environment Problems**: Path issues, permissions, system-specific bugs
- **Logic Errors**: Code bugs, algorithmic mistakes
- **Performance Issues**: Slow queries, memory leaks
- **Build/Deploy Issues**: CI/CD failures, compilation errors

## Related Resources
- [Next.js Multiple Lockfiles Fix](../nextjs/nextjs-multiple-lockfiles-fix.md)
- [ESLint Configuration Errors](../eslint/eslint-config-syntax-errors.md)