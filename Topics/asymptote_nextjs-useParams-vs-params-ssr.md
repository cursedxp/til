# Next.js useParams vs params for SSR

## Overview
Understanding the difference between client-side `useParams()` hook and server-side `params` prop for accessing route parameters in Next.js.

## Key Differences

| Aspect | useParams() | params prop |
|--------|-------------|-------------|
| **Environment** | Client-side only | Server-side + Client-side |
| **SSR Support** | ❌ No | ✅ Yes |
| **Usage** | Hook in component | Function parameter |
| **When Available** | After hydration | During SSR + runtime |

## Server-Side Rendering (SSR) Approach

### Page Components (Recommended)
```tsx
// app/blog/[slug]/page.tsx
interface PageProps {
  params: { slug: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

// ✅ SSR-friendly - params available during server rendering
export default function BlogPost({ params }: PageProps) {
  const { slug } = params  // Available on server and client

  // Can fetch data during SSR
  const post = getPostBySlug(slug)

  return (
    <div>
      <h1>{post.title}</h1>
      <p>Slug: {slug}</p>
    </div>
  )
}
```

### With Data Fetching
```tsx
export default async function BlogPost({ params }: PageProps) {
  const { slug } = params

  // This runs on the server during SSR
  const post = await fetch(`/api/posts/${slug}`).then(r => r.json())

  return (
    <div>
      <h1>{post.title}</h1>
      <p>Content: {post.content}</p>
    </div>
  )
}
```

## Client-Side Approach

### Client Components
```tsx
'use client'
import { useParams } from 'next/navigation'

// ❌ Not SSR-friendly - only works after hydration
export default function ClientBlogPost() {
  const params = useParams()
  const slug = params.slug  // undefined during SSR

  // Need loading state for SSR
  if (!slug) return <div>Loading...</div>

  return <div>Slug: {slug}</div>
}
```

### When to Use useParams()
```tsx
'use client'
import { useParams } from 'next/navigation'

// ✅ Good use case - child component that needs params
export default function BlogSidebar() {
  const params = useParams()
  const slug = params.slug

  return (
    <aside>
      <p>Related to: {slug}</p>
    </aside>
  )
}
```

## Practical Examples

### Example 1: Blog Post Page
```tsx
// app/blog/[slug]/page.tsx
interface PageProps {
  params: { slug: string }
}

export default function BlogPost({ params }: PageProps) {
  const { slug } = params

  // Server-side data fetching
  const post = posts.find(p => p.slug === slug)

  if (!post) {
    return <div>Post not found</div>
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      <BlogSidebar />  {/* Child component uses useParams */}
    </article>
  )
}
```

### Example 2: Nested Components
```tsx
// Parent: Server Component
export default function BlogPost({ params }: PageProps) {
  return (
    <div>
      <BlogContent slug={params.slug} />  {/* Pass as prop */}
      <BlogComments />  {/* Uses useParams internally */}
    </div>
  )
}

// Child: Client Component
'use client'
function BlogComments() {
  const params = useParams()
  const slug = params.slug

  return <div>Comments for {slug}</div>
}
```

## Multiple Parameters

### File Structure
```
app/
├── blog/
│   └── [category]/
│       └── [slug]/
│           └── page.tsx
```

### Accessing Multiple Params
```tsx
interface PageProps {
  params: {
    category: string
    slug: string
  }
}

export default function CategoryPost({ params }: PageProps) {
  const { category, slug } = params

  return (
    <div>
      <p>Category: {category}</p>
      <p>Post: {slug}</p>
    </div>
  )
}

// Client-side equivalent
'use client'
function ClientComponent() {
  const params = useParams()
  const { category, slug } = params

  return <div>{category}/{slug}</div>
}
```

## Search Parameters

### Server-Side
```tsx
interface PageProps {
  params: { slug: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export default function BlogPost({ params, searchParams }: PageProps) {
  const { slug } = params
  const { tab, sort } = searchParams

  return (
    <div>
      <h1>Post: {slug}</h1>
      <p>Tab: {tab}</p>
      <p>Sort: {sort}</p>
    </div>
  )
}
```

### Client-Side
```tsx
'use client'
import { useParams, useSearchParams } from 'next/navigation'

export default function ClientComponent() {
  const params = useParams()
  const searchParams = useSearchParams()

  const slug = params.slug
  const tab = searchParams.get('tab')

  return <div>{slug} - {tab}</div>
}
```

## Best Practices

### 1. Prefer Server Components
```tsx
// ✅ Use params prop for main page components
export default function Page({ params }: PageProps) {
  // SSR-friendly, better performance
}
```

### 2. Use useParams in Client Components
```tsx
// ✅ Use useParams only in client components that need it
'use client'
export default function InteractiveWidget() {
  const params = useParams()
  // ... interactive logic
}
```

### 3. Pass Props When Possible
```tsx
// ✅ Pass params as props instead of using useParams
export default function ParentPage({ params }: PageProps) {
  return <ChildComponent slug={params.slug} />
}

function ChildComponent({ slug }: { slug: string }) {
  return <div>Slug: {slug}</div>
}
```

### 4. Handle Loading States
```tsx
'use client'
export default function ClientComponent() {
  const params = useParams()

  // Handle SSR case
  if (!params.slug) {
    return <div>Loading...</div>
  }

  return <div>Slug: {params.slug}</div>
}
```

## Performance Implications

- **SSR with params**: Faster initial page load, better SEO
- **Client-side with useParams**: Requires hydration, potential loading states
- **Mixed approach**: Use server components for main content, client components for interactivity

## Related Concepts
- Next.js App Router
- Server vs Client Components
- SSR vs CSR
- React Server Components