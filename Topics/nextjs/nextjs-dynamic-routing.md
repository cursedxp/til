# Next.js Dynamic Routing

## Overview
Dynamic routing in Next.js allows you to create pages that respond to dynamic URLs using file-based routing with square brackets.

## Key Concepts

### File Structure
```
app/
├── blog/
│   └── [slug]/
│       └── page.tsx
```

### URL Mapping
- `[slug]` → Captures any value after `/blog/`
- `/blog/react-tutorial` → `slug = "react-tutorial"`
- `/blog/my-first-post` → `slug = "my-first-post"`

## Implementation Steps

### 1. Create Dynamic Route File
```tsx
// app/blog/[slug]/page.tsx
interface PageProps {
  params: { slug: string }
}

export default function BlogPost({ params }: PageProps) {
  const { slug } = params
  return <h1>Post: {slug}</h1>
}
```

### 2. Create Navigation Links
```tsx
// Main page
import Link from 'next/link'

<Link href={`/blog/${post.slug}`}>
  {post.title}
</Link>
```

### 3. Data Fetching
```tsx
export default function BlogPost({ params }: PageProps) {
  // Find post by slug from your data source
  const post = posts.find(p => p.slug === params.slug)

  return (
    <div>
      <h1>{post?.title}</h1>
      <p>{post?.content}</p>
    </div>
  )
}
```

## Benefits

- **SEO-friendly URLs**: `/blog/react-tutorial` vs `/blog?id=123`
- **Clean architecture**: One file handles all blog posts
- **Automatic routing**: No manual route configuration needed
- **Type safety**: TypeScript support for params

## Best Practices

1. Use descriptive slugs (kebab-case)
2. Validate slug parameters
3. Handle 404 cases for invalid slugs
4. Consider SEO implications
5. Use static generation when possible

## Related Concepts
- Static Site Generation (SSG)
- Server-Side Rendering (SSR)
- File-based routing
- Next.js App Router