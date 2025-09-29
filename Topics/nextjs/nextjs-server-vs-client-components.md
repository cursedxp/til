# Next.js Server vs Client Components

## Overview
Next.js App Router introduces Server Components by default, with the option to explicitly create Client Components. Understanding when to use each is crucial for performance and functionality.

## Default Behavior
```typescript
// This is a Server Component by default
export default function Page() {
  return <div>Server Component</div>;
}

// This becomes a Client Component
"use client"
export default function ClientPage() {
  return <div>Client Component</div>;
}
```

## Server Components

### What They Are
- **Default in App Router**: All components are Server Components unless marked otherwise
- **Run on the server**: Execute during build time or request time
- **No JavaScript sent**: Zero JavaScript bundle for these components
- **Can be async**: Can use async/await directly

### Capabilities
```typescript
// ✅ Server Components CAN:
export default async function ServerComponent() {
  // Direct database access
  const posts = await db.posts.findMany();

  // Environment variables
  const apiKey = process.env.SECRET_API_KEY;

  // File system access
  const data = fs.readFileSync('data.json');

  // Server-only libraries
  const processed = await heavyServerProcessing(data);

  return <div>{posts.length} posts</div>;
}
```

### Limitations
```typescript
// ❌ Server Components CANNOT:
export default function ServerComponent() {
  // No browser APIs
  const width = window.innerWidth; // Error!

  // No event handlers
  return <button onClick={() => {}}> // Error!

  // No React hooks
  const [state, setState] = useState(); // Error!

  // No browser-only libraries
  import confetti from 'canvas-confetti'; // Error!
}
```

## Client Components

### What They Are
- **Explicitly marked**: Must include `"use client"` directive
- **Run in browser**: Execute after hydration
- **Include JavaScript**: Add to the client bundle
- **Interactive**: Can handle events and use hooks

### When to Use
```typescript
"use client"
import { useState, useEffect } from 'react';

// ✅ Client Components for:
export default function ClientComponent() {
  // Interactive state
  const [count, setCount] = useState(0);

  // Browser APIs
  const [width, setWidth] = useState(window.innerWidth);

  // Event handlers
  const handleClick = () => setCount(c => c + 1);

  // React hooks
  useEffect(() => {
    setWidth(window.innerWidth);
  }, []);

  return (
    <div>
      <button onClick={handleClick}>Count: {count}</button>
      <p>Width: {width}px</p>
    </div>
  );
}
```

## The Boundary Rules

### Client Boundary
```typescript
// Parent: Server Component
import ClientCounter from './ClientCounter';

export default function ServerPage() {
  const data = await fetchData(); // Server-side

  return (
    <div>
      <h1>Server: {data.title}</h1>
      <ClientCounter /> {/* Client boundary starts here */}
    </div>
  );
}

// Child: Client Component
"use client"
export default function ClientCounter() {
  const [count, setCount] = useState(0); // Client-side
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

### Passing Data Across Boundary
```typescript
// ✅ CORRECT: Pass serializable data
export default async function ServerPage() {
  const posts = await fetchPosts();

  return <ClientComponent posts={posts} />; // Serializable data
}

// ❌ WRONG: Pass functions or non-serializable data
export default async function ServerPage() {
  const callback = () => console.log('hello');

  return <ClientComponent onCallback={callback} />; // Error!
}
```

## Common Patterns

### Pattern 1: Server Page with Client Interactions
```typescript
// app/blog/[slug]/page.tsx - Server Component
import { getPost } from '@/lib/api';
import LikeButton from './LikeButton';
import ShareButton from './ShareButton';

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug); // Server-side data

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>

      {/* Client components for interactivity */}
      <LikeButton postId={post.id} initialLikes={post.likes} />
      <ShareButton url={post.url} title={post.title} />
    </article>
  );
}

// LikeButton.tsx - Client Component
"use client"
export default function LikeButton({ postId, initialLikes }) {
  const [likes, setLikes] = useState(initialLikes);

  const handleLike = async () => {
    // Client-side interaction
    const newLikes = await updateLikes(postId);
    setLikes(newLikes);
  };

  return <button onClick={handleLike}>👍 {likes}</button>;
}
```

### Pattern 2: Mixed Dashboard
```typescript
// app/dashboard/page.tsx - Server Component
export default async function Dashboard() {
  const [user, stats, notifications] = await Promise.all([
    getUser(),
    getStats(),
    getNotifications()
  ]);

  return (
    <div>
      {/* Server-rendered sections */}
      <UserInfo user={user} />
      <StatsCards stats={stats} />

      {/* Client-interactive sections */}
      <NotificationCenter notifications={notifications} />
      <RealTimeChart />
    </div>
  );
}
```

### Pattern 3: Progressive Enhancement
```typescript
// Search.tsx - Starts as Server Component
export default function Search({ initialResults }) {
  return (
    <div>
      <SearchResults results={initialResults} /> {/* Server-rendered */}
      <SearchWidget /> {/* Client component for real-time search */}
    </div>
  );
}

// SearchWidget.tsx - Client Component for enhancement
"use client"
export default function SearchWidget() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  // Real-time search functionality
  useEffect(() => {
    if (query) searchAPI(query).then(setResults);
  }, [query]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {results.map(result => <div key={result.id}>{result.title}</div>)}
    </div>
  );
}
```

## Error Scenarios and Solutions

### Error 1: useEffect in Server Component
```typescript
// ❌ WRONG
export default function ServerComponent() {
  useEffect(() => {
    // Error: useEffect only works in Client Components
  }, []);
}

// ✅ SOLUTION 1: Make it a Client Component
"use client"
export default function ClientComponent() {
  useEffect(() => {
    // Works!
  }, []);
}

// ✅ SOLUTION 2: Use Server Component features
export default async function ServerComponent() {
  const data = await fetchData(); // Server-side equivalent
  return <div>{data}</div>;
}
```

### Error 2: Event Handlers in Server Component
```typescript
// ❌ WRONG
export default function ServerComponent() {
  return <button onClick={() => alert('Hi')}>Click</button>; // Error!
}

// ✅ SOLUTION: Extract interactive part
export default function ServerComponent() {
  return (
    <div>
      <h1>Server Content</h1>
      <InteractiveButton /> {/* Client component */}
    </div>
  );
}

"use client"
function InteractiveButton() {
  return <button onClick={() => alert('Hi')}>Click</button>;
}
```

### Error 3: Browser APIs in Server Component
```typescript
// ❌ WRONG
export default function ServerComponent() {
  const width = window.innerWidth; // Error: window not defined
}

// ✅ SOLUTION: Use Client Component
"use client"
export default function ClientComponent() {
  const [width, setWidth] = useState(0);

  useEffect(() => {
    setWidth(window.innerWidth);
  }, []);

  return <div>Width: {width}</div>;
}
```

## Performance Implications

### Server Components Benefits
- **Zero JavaScript**: No client bundle impact
- **Faster initial load**: Pre-rendered HTML
- **Better SEO**: Content available immediately
- **Reduced hydration**: Less work for the browser

### Client Components Costs
- **Larger bundles**: More JavaScript to download
- **Hydration overhead**: React needs to "hydrate" the HTML
- **Runtime execution**: Code runs in browser

## Best Practices

### 1. Server Components by Default
```typescript
// Start with Server Components
export default async function Page() {
  const data = await fetchData();
  return <PageContent data={data} />;
}
```

### 2. Client Components for Interactivity
```typescript
// Only use Client Components when needed
"use client"
export default function InteractiveWidget() {
  const [state, setState] = useState();
  return <button onClick={() => setState(!state)}>Toggle</button>;
}
```

### 3. Minimal Client Boundaries
```typescript
// ✅ GOOD: Small client boundary
export default async function Page() {
  const data = await fetchData();

  return (
    <div>
      <h1>{data.title}</h1> {/* Server */}
      <p>{data.content}</p> {/* Server */}
      <LikeButton /> {/* Small client component */}
    </div>
  );
}

// ❌ BAD: Large client boundary
"use client"
export default function Page() {
  // Entire page is now client-side
  return <div>...</div>;
}
```

### 4. Data Fetching Strategy
```typescript
// ✅ Server: Fetch data server-side
export default async function Page() {
  const data = await fetchData(); // No loading states needed
  return <Content data={data} />;
}

// ✅ Client: For dynamic/user-specific data
"use client"
export default function UserDashboard() {
  const { data, loading } = useSWR('/api/user', fetcher);
  if (loading) return <Loading />;
  return <Dashboard data={data} />;
}
```

## Summary

### Use Server Components for:
- Static content
- Data fetching
- SEO-important content
- Initial page renders
- Heavy computations

### Use Client Components for:
- Interactive elements
- Browser APIs
- Real-time updates
- Form handling
- State management

### Key Rule
**Push Client Components as low as possible in the component tree** to maximize Server Component benefits while keeping interactivity where needed.