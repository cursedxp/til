# React cache() Function

## Overview
The `cache()` function from React is a **request-level memoization** utility designed specifically for Server Components. It prevents duplicate function calls with the same arguments during a single request lifecycle.

## What It Does

```typescript
import { cache } from 'react';

// Without cache - multiple database calls
async function getUser(id: string) {
  console.log(`Fetching user ${id}`);
  return database.user.findById(id);
}

// With cache - single database call per request
const getUserCached = cache(async (id: string) => {
  console.log(`Fetching user ${id}`);
  return database.user.findById(id);
});
```

## How It Works

### Request-Level Deduplication
```typescript
// During a single page request
export default async function Page() {
  const user1 = await getUserCached('123'); // Executes database query
  const user2 = await getUserCached('123'); // Returns cached result
  const user3 = await getUserCached('123'); // Returns cached result

  const user4 = await getUserCached('456'); // Different argument, executes query

  console.log(user1 === user2); // true - same object reference
}
```

### Execution Timeline
```
Request 1 (10:00:00):
- getUserCached('123') → Database call → Cache result
- getUserCached('123') → Return from cache
- getUserCached('456') → Database call → Cache result

Request 2 (10:00:01):
- getUserCached('123') → Database call → Cache result (fresh request)
- getUserCached('123') → Return from cache
```

## Key Characteristics

### 1. Per-Request Scope
```typescript
const getPost = cache(async (slug: string) => {
  return database.posts.findBySlug(slug);
});

// Request A
await getPost('react-tutorial'); // Database call
await getPost('react-tutorial'); // Cached

// Request B (different user/browser)
await getPost('react-tutorial'); // Database call (new request)
await getPost('react-tutorial'); // Cached within this request
```

### 2. Argument-Based Caching
```typescript
const fetchData = cache(async (id: string, type: string) => {
  return database.getData(id, type);
});

// Different combinations create different cache entries
await fetchData('1', 'post');  // Cache key: ['1', 'post']
await fetchData('1', 'user');  // Cache key: ['1', 'user'] - different
await fetchData('2', 'post');  // Cache key: ['2', 'post'] - different
await fetchData('1', 'post');  // Uses cached result from first call
```

### 3. Object Reference Equality
```typescript
const getUser = cache(async (id: string) => {
  return { id, name: 'John', createdAt: new Date() };
});

const user1 = await getUser('123');
const user2 = await getUser('123');

console.log(user1 === user2); // true - exact same object
console.log(user1.createdAt === user2.createdAt); // true - same Date object
```

## Real-World Examples

### Example 1: Blog Post Service
```typescript
// services/postService.ts
import { cache } from 'react';

class PostService {
  // Cache post fetching
  getPost = cache(async (slug: string) => {
    console.log(`Fetching post: ${slug}`);
    return database.posts.findBySlug(slug);
  });

  // This method might call getPost internally
  getRelatedPosts = cache(async (slug: string) => {
    const post = await this.getPost(slug); // Uses cache if already called
    if (!post) return [];

    return database.posts.findRelated(post.tags);
  });

  // This also needs the post data
  getPostWithAuthor = cache(async (slug: string) => {
    const post = await this.getPost(slug); // Uses cache if already called
    if (!post) return null;

    const author = await this.getAuthor(post.authorId);
    return { ...post, author };
  });

  getAuthor = cache(async (authorId: string) => {
    return database.authors.findById(authorId);
  });
}

export const postService = new PostService();
```

### Example 2: Component Tree Optimization
```typescript
// app/blog/[slug]/page.tsx
export default async function BlogPost({ params }) {
  // All these might call getPost internally, but only 1 database call
  const post = await postService.getPost(params.slug);
  const related = await postService.getRelatedPosts(params.slug);
  const author = await postService.getPostWithAuthor(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <AuthorInfo author={author} />
      <RelatedPosts posts={related} />
    </article>
  );
}
```

### Example 3: Configuration Loading
```typescript
const getConfig = cache(async () => {
  console.log('Loading configuration');
  return {
    apiUrl: process.env.API_URL,
    features: await database.features.getEnabled(),
    settings: await database.settings.getAll()
  };
});

// Multiple components can call this without redundant work
export async function Header() {
  const config = await getConfig(); // First call
  return <nav>{config.features.navigation && <Navigation />}</nav>;
}

export async function Footer() {
  const config = await getConfig(); // Uses cached result
  return <footer>{config.settings.showFooter && <FooterContent />}</footer>;
}
```

## When to Use cache()

### ✅ Excellent Use Cases

1. **Database Queries**
```typescript
const getUser = cache(async (id: string) => {
  return prisma.user.findUnique({ where: { id } });
});
```

2. **External API Calls**
```typescript
const fetchWeather = cache(async (city: string) => {
  const response = await fetch(`https://api.weather.com/v1/weather?q=${city}`);
  return response.json();
});
```

3. **File System Operations**
```typescript
const readConfig = cache(async (path: string) => {
  const content = await fs.readFile(path, 'utf-8');
  return JSON.parse(content);
});
```

4. **Expensive Computations**
```typescript
const calculateStats = cache(async (data: number[]) => {
  // Expensive calculation
  return {
    mean: data.reduce((a, b) => a + b) / data.length,
    median: calculateMedian(data),
    mode: calculateMode(data)
  };
});
```

### ❌ Don't Use For

1. **Simple Calculations**
```typescript
// Unnecessary overhead
const add = cache((a: number, b: number) => a + b);
```

2. **Random/Time-Based Values**
```typescript
// Breaks the purpose of randomness
const getRandom = cache(() => Math.random());

// Time becomes stale
const getCurrentTime = cache(() => new Date());
```

3. **User-Specific Data in Global Context**
```typescript
// Could cache wrong user's data
const getCurrentUser = cache(() => getCurrentSession());
```

## Performance Benefits

### Before cache()
```typescript
// Without cache - Multiple database calls
export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);           // DB call #1
  const author = await getAuthor(post.authorId);     // DB call #2
  const relatedPosts = await getRelatedPosts(post);  // Might call getPost again - DB call #3
  const comments = await getComments(post.id);       // DB call #4

  return <BlogLayout post={post} author={author} related={relatedPosts} />;
}

// If getRelatedPosts internally calls getPost again:
// Total: 4 database calls (including duplicate)
```

### After cache()
```typescript
// With cache - Optimized calls
const getPost = cache(async (slug: string) => { /* ... */ });
const getAuthor = cache(async (id: string) => { /* ... */ });
const getRelatedPosts = cache(async (post) => { /* ... */ });
const getComments = cache(async (postId: string) => { /* ... */ });

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);           // DB call #1
  const author = await getAuthor(post.authorId);     // DB call #2
  const relatedPosts = await getRelatedPosts(post);  // Uses cached post - no extra DB call
  const comments = await getComments(post.id);       // DB call #3

  return <BlogLayout post={post} author={author} related={relatedPosts} />;
}

// Total: 3 database calls (duplicate eliminated)
```

## Comparison with Other Caching

| Caching Type | Scope | Duration | Use Case |
|--------------|-------|----------|----------|
| `cache()` | Per request | Single request | Deduplication |
| `unstable_cache()` | Cross-request | Multiple requests | Data caching |
| `useMemo()` | Component | Component lifecycle | Client optimization |
| `React.memo()` | Component | Props unchanged | Render optimization |

### Example Comparison
```typescript
// cache() - Request-level deduplication
const getPost = cache(async (slug) => fetchPost(slug));

// unstable_cache() - Persistent caching
import { unstable_cache } from 'next/cache';
const getCachedPost = unstable_cache(
  async (slug) => fetchPost(slug),
  ['posts'],
  { revalidate: 3600 } // Cache for 1 hour
);

// useMemo() - Component-level (client)
function Component({ data }) {
  const processed = useMemo(() => processData(data), [data]);
  return <div>{processed}</div>;
}
```

## Best Practices

### 1. Wrap Functions, Not Values
```typescript
// ✅ Good - Wrap the function
const getUser = cache(async (id: string) => {
  return await fetchUser(id);
});

// ❌ Bad - Don't cache values directly
const user = cache(await fetchUser('123')); // Error!
```

### 2. Use Consistent Arguments
```typescript
// ✅ Good - Consistent argument types
const getPost = cache(async (slug: string) => {
  return await fetchPost(slug);
});

// ❌ Problematic - Inconsistent arguments
const getPost = cache(async (identifier: string | number) => {
  // cache('123') and cache(123) are different cache entries
});
```

### 3. Consider Argument Serialization
```typescript
// Objects as arguments use reference equality
const getPostsByFilter = cache(async (filter: { tag: string; author: string }) => {
  return await fetchPosts(filter);
});

// These create different cache entries even with same values:
await getPostsByFilter({ tag: 'react', author: 'john' }); // Cache entry 1
await getPostsByFilter({ tag: 'react', author: 'john' }); // Cache entry 2 (different object)

// Better approach:
const getPostsByFilter = cache(async (tag: string, author: string) => {
  return await fetchPosts({ tag, author });
});
```

### 4. Use with TypeScript
```typescript
interface Post {
  id: string;
  title: string;
  content: string;
}

const getPost = cache(async (slug: string): Promise<Post | null> => {
  const response = await fetch(`/api/posts/${slug}`);
  if (!response.ok) return null;
  return response.json();
});
```

## Common Patterns

### Pattern 1: Service Class with Cache
```typescript
class ApiService {
  getUser = cache(async (id: string) => {
    return this.fetch(`/users/${id}`);
  });

  getPost = cache(async (slug: string) => {
    return this.fetch(`/posts/${slug}`);
  });

  private async fetch(endpoint: string) {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    if (!response.ok) throw new Error(`API Error: ${response.status}`);
    return response.json();
  }
}
```

### Pattern 2: Utility Functions
```typescript
// lib/cache.ts
export const cachedFetch = cache(async (url: string) => {
  const response = await fetch(url);
  return response.json();
});

export const cachedDbQuery = cache(async (query: string, params: any[]) => {
  return database.query(query, params);
});
```

## Summary

### Key Benefits:
- **Eliminates duplicate requests** within single request
- **Improves performance** by reducing redundant operations
- **Maintains object reference equality** for cached results
- **Zero configuration** - just wrap your function

### Remember:
- **Per-request only** - not a persistent cache
- **Argument-based** - same arguments = same cached result
- **Server Components only** - not for client-side use
- **Automatic cleanup** - cache cleared after request completes

The `cache()` function is a powerful tool for optimizing Server Components by ensuring expensive operations only happen once per request, even when called from multiple places in your component tree.