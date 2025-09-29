# Next.js Hydration Errors

## Overview
Hydration errors occur when the server-rendered HTML doesn't match what React renders on the client side during hydration.

## What is Hydration?
- **Server-Side**: Next.js generates static HTML
- **Client-Side**: React "hydrates" the HTML by attaching event listeners
- **Error**: Occurs when server and client HTML differ

## Common Causes

### 1. Dynamic Content
```tsx
// ❌ Problem - Different values on server/client
function BadComponent() {
  return <div>{Date.now()}</div>  // Changes every render
}

// ✅ Solution - Client-only rendering
function GoodComponent() {
  const [timestamp, setTimestamp] = useState<number>()

  useEffect(() => {
    setTimestamp(Date.now())
  }, [])

  return <div>{timestamp}</div>
}
```

### 2. Browser-Only APIs
```tsx
// ❌ Problem - window not available on server
function BadComponent() {
  return <div>{window.innerWidth}</div>
}

// ✅ Solution - Check if browser
function GoodComponent() {
  const [width, setWidth] = useState(0)

  useEffect(() => {
    setWidth(window.innerWidth)
  }, [])

  return <div>{width}</div>
}
```

### 3. Random Values
```tsx
// ❌ Problem - Random values differ
function BadComponent() {
  return <div id={Math.random()}></div>
}

// ✅ Solution - Use useId or stable IDs
import { useId } from 'react'

function GoodComponent() {
  const id = useId()
  return <div id={id}></div>
}
```

### 4. Third-Party Libraries
```tsx
// ❌ Problem - Library assumes browser environment
import SomeBrowserLibrary from 'browser-lib'

// ✅ Solution - Dynamic import with no SSR
import dynamic from 'next/dynamic'

const DynamicComponent = dynamic(
  () => import('./ClientOnlyComponent'),
  { ssr: false }
)
```

## Image and Link Related Issues

### Problem: Missing Dimensions
```tsx
// ❌ Can cause hydration errors
<Image src="/image.jpg" alt="" />

// ✅ Always specify dimensions
<Image
  src="/image.jpg"
  alt=""
  width={300}
  height={200}
/>
```

### Problem: Dynamic Image URLs
```tsx
// ❌ If imageUrl changes between server/client
<Image src={post?.imageUrl} alt="" fill />

// ✅ Ensure consistent data
<Image
  src={post?.imageUrl || '/placeholder.jpg'}
  alt=""
  fill
/>
```

## Solutions and Patterns

### 1. Client-Only Rendering
```tsx
import { useState, useEffect } from 'react'

function ClientOnlyComponent() {
  const [mounted, setMounted] = useState(false)

  useEffect(() => {
    setMounted(true)
  }, [])

  if (!mounted) {
    return null  // Return nothing on server
  }

  return <div>Client only content</div>
}
```

### 2. Dynamic Imports
```tsx
import dynamic from 'next/dynamic'

// Component won't render on server
const NoSSRComponent = dynamic(
  () => import('./ClientComponent'),
  { ssr: false }
)

// With loading state
const ComponentWithLoading = dynamic(
  () => import('./ClientComponent'),
  {
    ssr: false,
    loading: () => <div>Loading...</div>
  }
)
```

### 3. Suppressions (Use Sparingly)
```tsx
// Only use when you're sure it's safe
<div suppressHydrationWarning>
  {typeof window !== 'undefined' && window.innerWidth}
</div>
```

### 4. Consistent Data
```tsx
// Ensure same data on server and client
export async function getServerSideProps() {
  return {
    props: {
      timestamp: Date.now()  // Fixed at request time
    }
  }
}
```

## Debugging Tips

### 1. Check Console
```
Hydration failed because the server rendered HTML
didn't match the client.
```

### 2. Compare Server vs Client HTML
- View page source (server HTML)
- Inspect element (client HTML after hydration)
- Look for differences

### 3. Use React DevTools
- Enable "Highlight updates when components render"
- Look for unexpected re-renders

### 4. Gradual Isolation
```tsx
// Temporarily disable SSR to isolate issue
const TempNoSSR = dynamic(() => import('./ProblemComponent'), {
  ssr: false
})
```

## Prevention Strategies

### 1. Avoid Dynamic Values in JSX
- Don't use `Date.now()`, `Math.random()` in render
- Don't access `window` or `localStorage` directly

### 2. Use Proper Hooks
- `useEffect` for client-side effects
- `useState` for client-side state
- `useId` for stable IDs

### 3. Test SSR
```bash
npm run build
npm run start  # Test production build
```

### 4. Handle Loading States
```tsx
function Component() {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetchData().then(setData).finally(() => setLoading(false))
  }, [])

  if (loading) return <div>Loading...</div>
  return <div>{data}</div>
}
```

## Related Concepts
- Server-Side Rendering (SSR)
- Client-Side Rendering (CSR)
- React hydration process
- Next.js rendering methods