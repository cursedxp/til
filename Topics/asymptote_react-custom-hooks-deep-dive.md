# React Custom Hooks Deep Dive

## What Are Custom Hooks?

Custom hooks are JavaScript functions that:
- Start with "use" prefix (naming convention)
- Use other React hooks internally
- Extract and reuse stateful logic between components
- Follow the same rules as built-in React hooks

## Rules of Hooks

### Rule 1: Only Call at Top Level
```typescript
// ❌ WRONG - Conditional hook
function MyComponent() {
  if (condition) {
    useState(); // Never do this!
  }
}

// ✅ CORRECT - Hook at top level
function MyComponent() {
  const [value, setValue] = useState();
  if (condition) {
    // Use the value conditionally
  }
}
```

### Rule 2: Only Call from React Functions
- React function components
- Other custom hooks
- Never from regular JavaScript functions

## Render Cycle and useEffect Timing

### The Complete Flow
```
1. Component function runs (RENDER PHASE)
   ↓
2. React calculates virtual DOM differences
   ↓
3. React updates actual DOM (COMMIT PHASE)
   ↓
4. Browser paints the screen
   ↓
5. useEffect runs (AFTER PAINT)
```

### Visual Example
```typescript
function MyComponent() {
  console.log("1. Render phase - runs first");

  const [state, setState] = useState();

  useEffect(() => {
    console.log("3. Effect - runs AFTER render & DOM update");
  }, []);

  console.log("2. Still render phase");

  return <div>Content</div>;
}
```

## Key Concept: What Runs When

### Outside useEffect (Every Render)
```typescript
function useCustomHook(value) {
  // These run on EVERY render:
  console.log("Runs every render");
  const calculation = expensiveOperation(); // ⚠️ Performance issue!
  const searchResult = data.find(x => x.id === value); // ⚠️ Recalculated every time

  useEffect(() => {
    // This runs only when dependencies change
  }, [value]);
}
```

### Inside useEffect (Controlled Execution)
```typescript
function useCustomHook(value) {
  useEffect(() => {
    // Only runs when 'value' changes:
    console.log("Runs when dependency changes");
    const searchResult = data.find(x => x.id === value); // ✅ Only when needed
  }, [value]);
}
```

## Common Mistakes and Solutions

### Mistake 1: Function/Variable Outside useEffect
```typescript
// ❌ WRONG - Runs every render
export function usePost(slug: string) {
  const foundPost = mockBlogPosts.find(p => p.slug === slug); // Runs EVERY render

  useEffect(() => {
    setPost(foundPost);
  }, [slug]); // Missing foundPost dependency
}

// ✅ CORRECT - Runs only when needed
export function usePost(slug: string) {
  useEffect(() => {
    const foundPost = mockBlogPosts.find(p => p.slug === slug); // Only when slug changes
    setPost(foundPost);
  }, [slug]);
}
```

### Mistake 2: Setters in Dependencies
```typescript
// ❌ WRONG - Setters don't need to be dependencies
useEffect(() => {
  setPost(data);
  setLoading(false);
}, [slug, setPost, setLoading]); // Setters are stable, don't include them

// ✅ CORRECT - Only actual dependencies
useEffect(() => {
  setPost(data);
  setLoading(false);
}, [slug]); // Only values that trigger re-execution
```

### Mistake 3: Missing Error Reset
```typescript
// ❌ WRONG - Error persists even when successful
useEffect(() => {
  const foundPost = findPost(slug);
  if (!foundPost) {
    setError("Not found");
  }
  setPost(foundPost);
}, [slug]);

// ✅ CORRECT - Always reset error state
useEffect(() => {
  const foundPost = findPost(slug);
  if (foundPost) {
    setPost(foundPost);
    setError(null); // Clear previous errors
  } else {
    setPost(null);
    setError("Not found");
  }
}, [slug]);
```

## Building a Proper Custom Hook

### Step 1: Initial State
```typescript
function usePost(slug: string) {
  // Initialize state variables
  const [post, setPost] = useState<Post | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
```

### Step 2: Side Effects in useEffect
```typescript
  useEffect(() => {
    // Reset states at the beginning
    setLoading(true);
    setError(null);

    // Perform the operation
    const foundPost = mockBlogPosts.find(p => p.slug === slug);

    // Update states based on result
    if (foundPost) {
      setPost(foundPost);
    } else {
      setPost(null);
      setError(`Post "${slug}" not found`);
    }

    setLoading(false);
  }, [slug]); // Only depend on values that should trigger update
```

### Step 3: Return Values
```typescript
  return { post, loading, error }; // Return state and control functions
}
```

## Performance Optimization

### Problem: Expensive Calculations
```typescript
function useExpensiveHook(data: any[]) {
  // ⚠️ Runs on every render
  const processed = data.map(item => expensiveProcess(item));

  return processed;
}
```

### Solution 1: useMemo
```typescript
function useExpensiveHook(data: any[]) {
  // ✅ Only recalculates when data changes
  const processed = useMemo(() => {
    return data.map(item => expensiveProcess(item));
  }, [data]);

  return processed;
}
```

### Solution 2: Move to useEffect
```typescript
function useExpensiveHook(data: any[]) {
  const [processed, setProcessed] = useState([]);

  useEffect(() => {
    // Runs after render, doesn't block UI
    const result = data.map(item => expensiveProcess(item));
    setProcessed(result);
  }, [data]);

  return processed;
}
```

## Testing Custom Hooks

### Basic Test Structure
```typescript
import { renderHook, act } from '@testing-library/react';

test('usePost returns post data', async () => {
  const { result } = renderHook(() => usePost('my-slug'));

  // Initial state
  expect(result.current.loading).toBe(true);
  expect(result.current.post).toBe(null);

  // After effect runs
  await waitFor(() => {
    expect(result.current.loading).toBe(false);
  });

  expect(result.current.post?.slug).toBe('my-slug');
});
```

## Real-World Pattern: Data Fetching Hook

```typescript
export function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // Create abort controller for cleanup
    const abortController = new AbortController();

    async function fetchData() {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(url, {
          signal: abortController.signal
        });

        if (!response.ok) throw new Error('Failed to fetch');

        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();

    // Cleanup function
    return () => {
      abortController.abort();
    };
  }, [url]);

  return { data, loading, error };
}
```

## Key Takeaways

1. **Custom hooks** extract stateful logic for reuse
2. **useEffect runs AFTER** render completes
3. **Everything outside useEffect** runs on every render
4. **Dependencies** control when effects re-run
5. **Cleanup functions** prevent memory leaks
6. **State setters** are stable and don't need to be dependencies
7. **Always reset error states** when operations succeed
8. **Use useMemo/useCallback** for expensive operations

## Common Use Cases

- Data fetching
- Local storage sync
- Window/document event listeners
- Timers and intervals
- Form handling
- Animation state
- WebSocket connections
- Authentication state