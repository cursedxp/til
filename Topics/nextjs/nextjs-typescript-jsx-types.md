# Next.js TypeScript JSX Types Error

## Error Message
```
JSX element implicitly has type 'any' because no interface 'JSX.IntrinsicElements' exists.
```

## Cause
This error occurs when TypeScript can't find the JSX type definitions. It typically means:
- The `next-env.d.ts` file is missing
- TypeScript isn't recognizing React's JSX types

## Solution

### 1. Install dependencies and run dev server
After cloning a Next.js repo, install dependencies and start the dev server:
```bash
npm install
npm run dev
```

This will:
- Install all required packages including TypeScript definitions
- Automatically generate the `next-env.d.ts` file

### 2. Restart TypeScript Server
In VSCode:
- Press `Cmd+Shift+P` (Mac) or `Ctrl+Shift+P` (Windows/Linux)
- Type "TypeScript: Restart TS Server"
- Press Enter

## Prevention
- Don't delete `next-env.d.ts` - it's auto-generated and required
- Always run `npm install` and `npm run dev` after cloning a Next.js repo
- Don't manually edit `next-env.d.ts`