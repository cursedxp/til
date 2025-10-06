# Next.js 15: Dynamic Route Params are Now Promises

## The Breaking Change

In **Next.js 15**, dynamic route parameters (`params`) are now asynchronous and return a **Promise**.

## Error You'll See

```
Error: Route "/blog/categories/[category]" used `params.category`.
`params` should be awaited before using its properties.
```

## Old Way (Next.js 14 and earlier) ❌

```typescript
// app/blog/[slug]/page.tsx
interface BlogPageProps {
  params: { slug: string }; // Synchronous
}

export default async function BlogPage({ params }: BlogPageProps) {
  const { slug } = params; // Direct access
  const post = await getPost(slug);

  return <div>{post.title}</div>;
}
```

## New Way (Next.js 15+) ✅

```typescript
// app/blog/[slug]/page.tsx
interface BlogPageProps {
  params: Promise<{ slug: string }>; // Now a Promise!
}

export default async function BlogPage({ params }: BlogPageProps) {
  const { slug } = await params; // Must await first!
  const post = await getPost(slug);

  return <div>{post.title}</div>;
}
```

## Real Examples

### Dynamic Category Page

```typescript
// app/blog/categories/[category]/page.tsx

// ❌ OLD (Next.js 14)
interface CategoryPageProps {
  params: { category: string };
}

export default async function CategoryPage({ params }: CategoryPageProps) {
  const posts = await getPostsByCategory(params.category);
  return <div>...</div>;
}

// ✅ NEW (Next.js 15)
interface CategoryPageProps {
  params: Promise<{ category: string }>;
}

export default async function CategoryPage({ params }: CategoryPageProps) {
  const { category } = await params; // Await params first
  const posts = await getPostsByCategory(category);
  return <div>...</div>;
}
```

### Dynamic Product Page

```typescript
// app/products/[id]/page.tsx

// ✅ Next.js 15
interface ProductPageProps {
  params: Promise<{ id: string }>;
}

export default async function ProductPage({ params }: ProductPageProps) {
  const { id } = await params;
  const product = await getProduct(id);

  return <div>{product.name}</div>;
}
```

## Multiple Dynamic Segments

```typescript
// app/blog/[category]/[slug]/page.tsx

interface BlogPostProps {
  params: Promise<{
    category: string;
    slug: string;
  }>;
}

export default async function BlogPost({ params }: BlogPostProps) {
  const { category, slug } = await params;

  const post = await getPostBySlug(slug);
  const relatedPosts = await getPostsByCategory(category);

  return <div>...</div>;
}
```

## Common Mistake

```typescript
// ❌ WRONG - Trying to access params.slug without await
export default async function Page({
  params
}: {
  params: Promise<{ slug: string }>
}) {
  const post = await getPost(params.slug); // ERROR!
  // params is a Promise, not an object
  // You can't access .slug directly
}

// ✅ CORRECT - Await params first
export default async function Page({
  params
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params; // Unwrap the Promise
  const post = await getPost(slug); // Now it works!
}
```

## Why This Change?

Next.js made this change to support:
1. **Streaming** - Better performance with React Server Components
2. **Partial Prerendering (PPR)** - New rendering strategy
3. **Consistency** - All dynamic APIs (params, searchParams) are now async

## Migration Checklist

For all dynamic routes (`[param]`, `[...slug]`, etc.):

- [ ] Update `params` type from `{ param: string }` to `Promise<{ param: string }>`
- [ ] Add `await` before accessing `params` properties
- [ ] Update `const { param } = params` to `const { param } = await params`
- [ ] Test all dynamic routes to ensure they work

## searchParams Also Affected

```typescript
// searchParams is also a Promise now!

interface PageProps {
  searchParams: Promise<{ query?: string }>;
}

export default async function SearchPage({ searchParams }: PageProps) {
  const { query } = await searchParams; // Must await!

  const results = await search(query);
  return <div>...</div>;
}
```

## Summary

- **Next.js 15**: `params` and `searchParams` are now **Promises**
- **Action Required**: Add `await` before accessing their properties
- **Type Change**: `{ slug: string }` → `Promise<{ slug: string }>`
- **Pattern**: `const { param } = await params;`

Learn more: https://nextjs.org/docs/messages/sync-dynamic-apis
