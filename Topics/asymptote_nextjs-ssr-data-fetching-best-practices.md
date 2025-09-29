# Next.js SSR Data Fetching Best Practices

## Overview
Modern Next.js App Router provides powerful data fetching capabilities directly in Server Components. This guide covers professional patterns and best practices for organizing data fetching logic.

## Professional Project Structure

### Standard Industry Structure
```
app/
├── blog/
│   └── [slug]/
│       └── page.tsx        # Clean page component
├── lib/
│   ├── api/               # API/data fetching functions
│   │   ├── posts.ts
│   │   └── users.ts
│   ├── db/               # Database operations
│   │   ├── posts.ts
│   │   └── schema.ts
│   └── utils/            # Utility functions
│       └── date.ts
├── services/             # Business logic layer
│   ├── postService.ts
│   └── userService.ts
├── repositories/         # Data access layer (optional)
│   └── postRepository.ts
└── types/               # TypeScript definitions
    └── blog.ts
```

## Why NOT Keep Functions in Page Files

### ❌ Bad Practice: Logic in Page
```typescript
// app/blog/[slug]/page.tsx - DON'T DO THIS
export default async function BlogPost({ params }) {
  // ❌ Business logic mixed with presentation
  const post = mockBlogPosts.find(p => p.slug === params.slug);

  if (!post) {
    // ❌ Error handling in component
    throw new Error('Post not found');
  }

  // ❌ Complex logic in component
  const relatedPosts = mockBlogPosts
    .filter(p => p.slug !== post.slug)
    .filter(p => p.tags.some(tag => post.tags.includes(tag)))
    .slice(0, 3);

  return <article>{post.title}</article>;
}
```

### ✅ Good Practice: Separated Concerns
```typescript
// lib/api/posts.ts - Data layer
export async function getPostBySlug(slug: string): Promise<BlogPost | null> {
  return mockBlogPosts.find(p => p.slug === slug) || null;
}

export async function getRelatedPosts(slug: string, limit = 3): Promise<BlogPost[]> {
  const post = await getPostBySlug(slug);
  if (!post) return [];

  return mockBlogPosts
    .filter(p => p.slug !== slug &&
      p.tags.some(tag => post.tags.includes(tag))
    )
    .slice(0, limit);
}

// app/blog/[slug]/page.tsx - Clean page component
import { getPostBySlug, getRelatedPosts } from '@/lib/api/posts';
import { notFound } from 'next/navigation';

export default async function BlogPost({ params }) {
  const post = await getPostBySlug(params.slug);

  if (!post) {
    notFound(); // Next.js built-in 404
  }

  const relatedPosts = await getRelatedPosts(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <RelatedPosts posts={relatedPosts} />
    </article>
  );
}
```

## Data Fetching Patterns

### Pattern 1: Simple API Layer
```typescript
// lib/api/posts.ts
import { cache } from 'react';
import { BlogPost } from '@/types/blog';

export const getPost = cache(async (slug: string): Promise<BlogPost | null> => {
  // In real app: database query or external API
  const response = await fetch(`${process.env.API_URL}/posts/${slug}`);

  if (!response.ok) {
    if (response.status === 404) return null;
    throw new Error('Failed to fetch post');
  }

  return response.json();
});

export const getPosts = cache(async (): Promise<BlogPost[]> => {
  const response = await fetch(`${process.env.API_URL}/posts`);

  if (!response.ok) {
    throw new Error('Failed to fetch posts');
  }

  return response.json();
});

export const getPostsByTag = cache(async (tag: string): Promise<BlogPost[]> => {
  const posts = await getPosts();
  return posts.filter(post => post.tags.includes(tag));
});
```

### Pattern 2: Service Layer (Enterprise)
```typescript
// services/postService.ts
import { cache } from 'react';
import * as postApi from '@/lib/api/posts';
import { BlogPost } from '@/types/blog';

class PostService {
  getPost = cache(async (slug: string): Promise<BlogPost | null> => {
    try {
      return await postApi.getPost(slug);
    } catch (error) {
      console.error(`Failed to get post ${slug}:`, error);
      return null;
    }
  });

  getPostsWithMetrics = cache(async (): Promise<Array<BlogPost & { readTime: string }>> => {
    const posts = await postApi.getPosts();

    return posts.map(post => ({
      ...post,
      readTime: this.calculateReadTime(post.content)
    }));
  });

  getRelatedPosts = cache(async (slug: string, limit = 3): Promise<BlogPost[]> => {
    const post = await this.getPost(slug);
    if (!post) return [];

    const allPosts = await postApi.getPosts();

    return allPosts
      .filter(p => p.slug !== slug)
      .filter(p => this.hasCommonTags(p.tags, post.tags))
      .sort((a, b) => this.calculateSimilarity(b, post) - this.calculateSimilarity(a, post))
      .slice(0, limit);
  });

  private calculateReadTime(content: string): string {
    const wordsPerMinute = 200;
    const wordCount = content.split(' ').length;
    const minutes = Math.ceil(wordCount / wordsPerMinute);
    return `${minutes} min read`;
  }

  private hasCommonTags(tags1: string[], tags2: string[]): boolean {
    return tags1.some(tag => tags2.includes(tag));
  }

  private calculateSimilarity(post1: BlogPost, post2: BlogPost): number {
    const commonTags = post1.tags.filter(tag => post2.tags.includes(tag));
    return commonTags.length;
  }
}

export const postService = new PostService();
```

### Pattern 3: Repository Pattern (Clean Architecture)
```typescript
// repositories/postRepository.ts
import { BlogPost } from '@/types/blog';

export interface PostRepository {
  findBySlug(slug: string): Promise<BlogPost | null>;
  findAll(): Promise<BlogPost[]>;
  findByAuthor(authorId: string): Promise<BlogPost[]>;
  findByTag(tag: string): Promise<BlogPost[]>;
  findPublished(): Promise<BlogPost[]>;
}

// repositories/implementations/apiPostRepository.ts
export class ApiPostRepository implements PostRepository {
  constructor(private baseUrl: string) {}

  async findBySlug(slug: string): Promise<BlogPost | null> {
    const response = await fetch(`${this.baseUrl}/posts/${slug}`);
    return response.ok ? response.json() : null;
  }

  async findAll(): Promise<BlogPost[]> {
    const response = await fetch(`${this.baseUrl}/posts`);
    if (!response.ok) throw new Error('Failed to fetch posts');
    return response.json();
  }

  async findByAuthor(authorId: string): Promise<BlogPost[]> {
    const response = await fetch(`${this.baseUrl}/posts?author=${authorId}`);
    if (!response.ok) throw new Error('Failed to fetch posts by author');
    return response.json();
  }

  async findByTag(tag: string): Promise<BlogPost[]> {
    const posts = await this.findAll();
    return posts.filter(post => post.tags.includes(tag));
  }

  async findPublished(): Promise<BlogPost[]> {
    const posts = await this.findAll();
    return posts.filter(post => post.published);
  }
}

// lib/repositories.ts
import { ApiPostRepository } from '@/repositories/implementations/apiPostRepository';

export const postRepository = new ApiPostRepository(process.env.API_URL!);
```

## Database Integration Examples

### With Prisma (Most Popular)
```typescript
// lib/db/posts.ts
import { cache } from 'react';
import { prisma } from '@/lib/prisma';

export const getPost = cache(async (slug: string) => {
  return prisma.post.findUnique({
    where: { slug },
    include: {
      author: {
        select: { name: true, avatar: true }
      },
      tags: true,
      _count: {
        select: { comments: true, likes: true }
      }
    }
  });
});

export const getPosts = cache(async (take = 10, skip = 0) => {
  return prisma.post.findMany({
    take,
    skip,
    where: { published: true },
    include: {
      author: { select: { name: true } },
      tags: true,
      _count: { select: { comments: true, likes: true } }
    },
    orderBy: { createdAt: 'desc' }
  });
});
```

### With Drizzle (Growing Popularity)
```typescript
// lib/db/posts.ts
import { cache } from 'react';
import { db } from '@/lib/drizzle';
import { posts, users, tags, eq, and, desc } from '@/lib/schema';

export const getPost = cache(async (slug: string) => {
  return db.query.posts.findFirst({
    where: eq(posts.slug, slug),
    with: {
      author: {
        columns: { name: true, avatar: true }
      },
      tags: true
    }
  });
});

export const getPosts = cache(async (limit = 10) => {
  return db.query.posts.findMany({
    where: eq(posts.published, true),
    limit,
    orderBy: [desc(posts.createdAt)],
    with: {
      author: { columns: { name: true } },
      tags: true
    }
  });
});
```

## Performance Optimizations

### 1. Parallel Data Fetching
```typescript
// ✅ GOOD: Parallel fetching
export default async function BlogPost({ params }) {
  const [post, relatedPosts, author] = await Promise.all([
    getPost(params.slug),
    getRelatedPosts(params.slug),
    getAuthor(params.slug)
  ]);

  return <BlogContent post={post} related={relatedPosts} author={author} />;
}

// ❌ BAD: Sequential fetching
export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);      // Wait
  const related = await getRelatedPosts(params.slug); // Wait
  const author = await getAuthor(params.slug);        // Wait

  return <BlogContent post={post} related={related} author={author} />;
}
```

### 2. Conditional Fetching
```typescript
export default async function BlogPost({ params, searchParams }) {
  const post = await getPost(params.slug);

  if (!post) {
    notFound();
  }

  // Only fetch related posts if needed
  const promises = [post];

  if (searchParams.showRelated !== 'false') {
    promises.push(getRelatedPosts(params.slug));
  }

  if (searchParams.showComments === 'true') {
    promises.push(getComments(post.id));
  }

  const [, relatedPosts, comments] = await Promise.all(promises);

  return (
    <BlogContent
      post={post}
      relatedPosts={relatedPosts}
      comments={comments}
    />
  );
}
```

### 3. React Cache for Request Deduplication
```typescript
// lib/api/posts.ts
import { cache } from 'react';

// Multiple components can call this, but it only executes once per request
export const getPost = cache(async (slug: string) => {
  console.log(`Fetching post: ${slug}`); // Only logs once per request
  return fetchPostFromAPI(slug);
});

// In your page, these all use the same cached result
export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);        // Executes
  const related = await getRelatedPosts(params.slug); // Might call getPost again
  const author = await getAuthorByPost(params.slug);  // Might call getPost again

  // But getPost only hits the API once!
}
```

### 4. Next.js Caching with unstable_cache
```typescript
// lib/api/posts.ts
import { unstable_cache } from 'next/cache';

export const getPost = unstable_cache(
  async (slug: string) => {
    return fetchPostFromAPI(slug);
  },
  ['posts'], // Cache key prefix
  {
    revalidate: 3600, // Cache for 1 hour
    tags: ['posts']   // Cache tags for invalidation
  }
);

// Invalidate cache when needed
import { revalidateTag } from 'next/cache';

export async function updatePost(slug: string, data: any) {
  await updatePostInAPI(slug, data);
  revalidateTag('posts'); // Invalidate all post caches
}
```

## Error Handling Strategies

### 1. Graceful Degradation
```typescript
// services/postService.ts
export class PostService {
  getPostWithFallback = cache(async (slug: string): Promise<BlogPost | null> => {
    try {
      return await this.getPost(slug);
    } catch (error) {
      console.error(`Failed to fetch post ${slug}:`, error);

      // Fallback to cache or static content
      return this.getCachedPost(slug) || this.getStaticPost(slug);
    }
  });

  getPostsWithPartialFailure = cache(async (): Promise<BlogPost[]> => {
    try {
      return await this.getAllPosts();
    } catch (error) {
      console.error('Failed to fetch all posts:', error);

      // Return cached posts or empty array
      return this.getCachedPosts() || [];
    }
  });
}
```

### 2. Custom Error Pages
```typescript
// app/blog/[slug]/page.tsx
import { getPost } from '@/services/postService';
import { notFound } from 'next/navigation';

export default async function BlogPost({ params }) {
  try {
    const post = await getPost(params.slug);

    if (!post) {
      notFound(); // Triggers app/not-found.tsx
    }

    return <BlogContent post={post} />;
  } catch (error) {
    // Triggers app/error.tsx
    throw new Error('Failed to load blog post');
  }
}
```

## Testing Data Fetching

### Unit Testing Data Functions
```typescript
// __tests__/lib/api/posts.test.ts
import { getPost, getPosts } from '@/lib/api/posts';

// Mock fetch
global.fetch = jest.fn();

describe('Posts API', () => {
  beforeEach(() => {
    (fetch as jest.Mock).mockClear();
  });

  it('should fetch a single post', async () => {
    const mockPost = { id: 1, slug: 'test', title: 'Test Post' };
    (fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => mockPost
    });

    const result = await getPost('test');

    expect(fetch).toHaveBeenCalledWith(
      expect.stringContaining('/posts/test')
    );
    expect(result).toEqual(mockPost);
  });

  it('should return null for 404', async () => {
    (fetch as jest.Mock).mockResolvedValueOnce({
      ok: false,
      status: 404
    });

    const result = await getPost('nonexistent');
    expect(result).toBeNull();
  });
});
```

## Summary of Best Practices

### ✅ DO:
- Separate data fetching into dedicated files (`lib/api/`, `services/`)
- Use TypeScript for type safety
- Implement proper error handling
- Use React `cache()` for request deduplication
- Fetch data in parallel when possible
- Use proper Next.js error boundaries

### ❌ DON'T:
- Put business logic in page components
- Ignore error handling
- Fetch data sequentially when parallel is possible
- Forget to use caching mechanisms
- Mix data fetching with presentation logic

### File Organization:
- **Simple projects**: `lib/api/` folder
- **Medium projects**: `services/` layer
- **Enterprise projects**: Repository pattern with interfaces
- **Always**: Keep page components clean and focused on rendering