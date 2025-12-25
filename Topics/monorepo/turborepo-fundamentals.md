# Turborepo Fundamentals

## What is a Monorepo?

A monorepo is a single repository containing multiple projects/packages that can share code and dependencies.

```
POLYREPO approach:                MONOREPO approach:
├── repo: my-web-app              └── repo: my-company
├── repo: my-admin-app                ├── apps/
├── repo: my-ui-library               │   ├── web/
└── repo: my-utils                    │   └── admin/
                                      └── packages/
                                          ├── ui/
                                          └── utils/
```

## When to Use a Monorepo

### Use It When:
- Multiple apps share code (web + mobile + admin)
- You want a shared UI component library
- You need shared TypeScript types/configs
- Your team works across related projects
- You want atomic changes across multiple packages

### Don't Use It When:
- You have a single standalone app
- Projects are completely unrelated
- Teams are isolated and don't share code

## Turborepo Project Structure

```
monorepo/
├── apps/
│   ├── web/              → Next.js app
│   └── docs/             → Documentation site
├── packages/
│   ├── ui/               → Shared React components
│   ├── utils/            → Shared utility functions
│   └── configs/          → Shared ESLint, TypeScript configs
├── turbo.json            → Turborepo configuration
├── pnpm-workspace.yaml   → Workspace definition (or package-lock.json for npm)
└── package.json          → Root package.json
```

## Shared Packages

### How They Work

Packages have a name in their `package.json`:
```json
{
  "name": "@repo/utils",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "types": "./src/index.ts"
}
```

Apps depend on them:
```json
{
  "dependencies": {
    "@repo/utils": "*",
    "@repo/ui": "*"
  }
}
```

Apps import from them:
```typescript
import { Button } from "@repo/ui/button";
import { formatDate } from "@repo/utils";
```

No npm publish needed - workspace linking handles it.

## Creating a Shared Package

### Step 1: Create folder structure
```bash
mkdir -p packages/utils/src
```

### Step 2: Create package.json
```json
{
  "name": "@repo/utils",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "types": "./src/index.ts"
}
```

### Step 3: Create the code
```typescript
// packages/utils/src/index.ts
export function formatDate(date: Date): string {
  return date.toLocaleDateString("en-US", {
    year: "numeric",
    month: "long",
    day: "numeric"
  });
}
```

### Step 4: Add as dependency in app
```json
{
  "dependencies": {
    "@repo/utils": "*"
  }
}
```

### Step 5: Install from root
```bash
npm install
```

## turbo.json Explained

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["$TURBO_DEFAULT$", ".env*"],
      "outputs": [".next/**", "!.next/cache/**"]
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### Key Concepts

| Field | Meaning |
|-------|---------|
| `dependsOn: ["^build"]` | Build dependencies first (^ = my dependencies) |
| `inputs` | Files that affect cache |
| `outputs` | What to cache (build artifacts) |
| `cache: false` | Never cache (for dev servers) |
| `persistent: true` | Keeps running (doesn't exit) |

### The Caret (^) Symbol

`"dependsOn": ["^build"]` means:
> "Before building ME, build all packages I depend on"

Example: `turbo build` on web app:
1. First builds `@repo/ui` (dependency)
2. Then builds `web`

## Turborepo Caching

### How It Works

1. First build → compiles everything, saves to cache
2. Second build (no changes) → uses cache → **FULL TURBO**
3. Change a file → only affected packages rebuild

### The Chef Analogy

- **First build**: Chef cooks from scratch, saves meal in fridge
- **No changes**: Chef grabs meal from fridge (instant)
- **File changed**: Chef detects new ingredient, cooks again

### Why It Matters

| Scenario | Without Cache | With Cache |
|----------|---------------|------------|
| Full rebuild | 10 min | 10 min |
| Change 1 file | 10 min | 30 sec |
| No changes | 10 min | instant |

## Common Commands

```bash
# Run all apps in dev mode
npx turbo dev

# Build all packages
npx turbo build

# Lint entire monorepo
npx turbo lint

# Run specific app only
npx turbo dev --filter=web

# Run command for specific package
npx turbo build --filter=@repo/ui
```

## Adding Dependencies

### To a specific package
```bash
npm install axios --workspace=@repo/utils
```

### As dev dependency
```bash
npm install vitest --workspace=@repo/utils --save-dev
```

## Package.json Fields for Workspace Packages

```json
{
  "name": "@repo/utils",
  "version": "1.0.0",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "scripts": {
    "build": "tsc",
    "lint": "eslint src/",
    "test": "vitest"
  },
  "exports": {
    ".": "./src/index.ts",
    "./date": "./src/date.ts"
  },
  "dependencies": {},
  "devDependencies": {}
}
```

### Field Reference

| Field | Purpose |
|-------|---------|
| `name` | How other packages import it |
| `private` | Prevents accidental npm publish |
| `main` | Default entry point |
| `exports` | Multiple entry points |
| `scripts` | Commands Turborepo can run |

## Scripts and Turborepo

When you run `npx turbo build`:

1. Turborepo checks every package
2. If package has `"build"` script → runs it
3. If no `"build"` script → skips it
4. Respects dependency order (`^build`)

## Key Takeaways

1. **Monorepo** = multiple projects in one repo sharing code
2. **Workspace packages** use `@repo/name` convention
3. **turbo.json** defines tasks and caching rules
4. **Caching** skips rebuilding unchanged code (FULL TURBO)
5. **^dependsOn** ensures correct build order
6. **Shared packages** = single source of truth
7. Use `--workspace=` flag to add deps to specific packages
