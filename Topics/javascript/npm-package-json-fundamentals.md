# npm package.json Fundamentals

## The Problem (Before package.json)

In early JavaScript, there was no standard way to:
- Share code between projects
- Track what libraries a project needs
- Know which version of a library works

Developers would manually download files and hope they work together.

## The Solution: npm + package.json (2010)

Node.js created npm with a simple idea:

> "What if every project had ONE file that lists everything it needs?"

That file is `package.json`. Now anyone can:
1. Clone your project
2. Run `npm install`
3. Get exact same dependencies

## What package.json Is

Your project's **identity card + shopping list**.

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

## Key Properties

### Identity Fields
| Field | Purpose |
|-------|---------|
| `name` | Package name (used for imports) |
| `version` | Semantic version (1.0.0) |
| `description` | What the package does |
| `private` | If `true`, can't be published to npm |

### Entry Points
| Field | Purpose |
|-------|---------|
| `main` | Default file when someone imports your package |
| `types` | TypeScript definitions location |
| `exports` | Multiple entry points (modern way) |

### Dependencies
| Field | Purpose |
|-------|---------|
| `dependencies` | Required to run (shipped to production) |
| `devDependencies` | Only for development (testing, building) |
| `peerDependencies` | "You must also install this" (for libraries) |

### Commands
| Field | Purpose |
|-------|---------|
| `scripts` | Custom commands (`npm run build`, `npm run dev`) |

### Metadata
| Field | Purpose |
|-------|---------|
| `author` | Who made it |
| `license` | MIT, ISC, etc. |
| `repository` | Link to source code |

## Scripts Property

Scripts are **nicknames for terminal commands**.

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "test": "vitest"
  }
}
```

Instead of typing `next dev`, you type `npm run dev`.

### How Scripts Run

```
1. You type: npm run dev
        ↓
2. npm reads package.json
        ↓
3. npm finds "dev": "next dev"
        ↓
4. npm runs "next dev" in terminal
```

### Common Scripts
| Script | Purpose |
|--------|---------|
| `dev` | Start development server |
| `build` | Create production build |
| `start` | Run production server |
| `test` | Run tests |
| `lint` | Check code quality |

## How npm Finds Commands

When you run `next dev`, npm looks in `node_modules/.bin/`:

```
node_modules/
├── .bin/
│   ├── next       ← executable
│   ├── eslint     ← executable
│   └── vitest     ← executable
└── next/
    └── (actual Next.js code)
```

npm automatically adds `node_modules/.bin/` to PATH when running scripts.

## Library vs CLI Packages

### Library Package (no executable)
Just code you import.

```json
{
  "name": "lodash",
  "main": "./index.js"
}
```

Usage: `import { map } from "lodash"`

### CLI Package (has executable)
Has commands you run in terminal.

```json
{
  "name": "eslint",
  "bin": {
    "eslint": "./bin/eslint.js"
  }
}
```

Usage: `eslint src/`

### Comparison
| Type | Has `bin`? | Has `main`? | Used via |
|------|-----------|-------------|----------|
| Library | No | Yes | `import` |
| CLI | Yes | No | terminal command |
| Both | Yes | Yes | both ways |

## Creating a CLI Package

### Two Requirements

1. **Shebang line** in your .js file:
```javascript
#!/usr/bin/env node

console.log("Hello from CLI!");
```

2. **bin field** in package.json:
```json
{
  "name": "my-cli",
  "bin": {
    "mycli": "./cli.js"
  }
}
```

### What Each Part Does
| Part | Purpose |
|------|---------|
| `#!/usr/bin/env node` | Tells computer: "Run this with Node" |
| `"bin": {}` | Tells npm: "Create a command called..." |

### Folder Structure
```
my-cli/
├── package.json
└── cli.js
```

Just 2 files needed.

## Key Takeaways

1. **package.json** = project's identity card + dependency list
2. **scripts** = saved terminal commands with nicknames
3. **npm run X** = npm reads package.json, finds X, runs the command
4. **node_modules/.bin/** = where executable commands live
5. **Library packages** have `main`, used via `import`
6. **CLI packages** have `bin`, used via terminal
7. **Creating CLI** needs shebang (`#!/usr/bin/env node`) + `bin` field
