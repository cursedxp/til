# Async/Await in Next.js Server Components

## Overview
Next.js App Router Server Components have a unique superpower: they can be async functions that await data directly. This eliminates the need for useEffect, loading states, and complex data fetching patterns.

## Basic Concept

### Traditional Client-Side Pattern (Old Way)
```typescript
'use client'
import { useState, useEffect } from 'react';

export default function BlogPost({ params }) {
  const [post, setPost] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchPost() {
      try {
        const response = await fetch(`/api/posts/${params.slug}`);
        const data = await response.json();
        setPost(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchPost();
  }, [params.slug]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!post) return <div>Post not found</div>;

  return <article>{post.title}</article>;
}
```

### Modern Server Component Pattern (New Way)
```typescript
// No 'use client' - this is a Server Component
import { getPostBySlug } from '@/lib/api/posts';
import { notFound } from 'next/navigation';

export default async function BlogPost({ params }) {
  // ✅ Direct await - no hooks needed!
  const post = await getPostBySlug(params.slug);

  if (!post) {
    notFound(); // Built-in Next.js 404
  }

  return <article>{post.title}</article>;
}
```

## How Async Server Components Work

### Execution Flow
```
1. User navigates to /blog/my-post
2. Next.js calls BlogPost({ params: { slug: 'my-post' } })
3. Component function runs on server
4. await getPostBySlug('my-post') executes
5. Database/API call completes
6. Component returns JSX with data
7. Server sends complete HTML to browser
8. No loading states needed!
```

### Component Lifecycle
```typescript
export default async function Page() {
  console.log('1. Component starts executing');

  const data1 = await fetchData1(); // Waits for completion
  console.log('2. First data fetched');

  const data2 = await fetchData2(); // Then waits for this
  console.log('3. Second data fetched');

  console.log('4. Rendering JSX');
  return <div>{data1.title} - {data2.description}</div>;
}

// Output:
// 1. Component starts executing
// 2. First data fetched
// 3. Second data fetched
// 4. Rendering JSX
```

## Async Component Patterns

### Pattern 1: Single Data Fetch
```typescript
import { getPost } from '@/services/postService';

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

### Pattern 2: Multiple Sequential Fetches
```typescript
export default async function BlogPost({ params }) {
  // Sequential - each waits for the previous
  const post = await getPost(params.slug);
  const author = await getAuthor(post.authorId); // Needs post data first
  const comments = await getComments(post.id);   // Needs post data first

  return (
    <article>
      <h1>{post.title}</h1>
      <p>By {author.name}</p>
      <CommentsList comments={comments} />
    </article>
  );
}
```

### Pattern 3: Parallel Data Fetching (Faster)
```typescript
export default async function BlogPost({ params }) {
  // ✅ Parallel - all fetch at the same time
  const [post, relatedPosts, popularPosts] = await Promise.all([
    getPost(params.slug),
    getRelatedPosts(params.slug),
    getPopularPosts()
  ]);

  return (
    <div>
      <article>{post.title}</article>
      <RelatedPosts posts={relatedPosts} />
      <PopularPosts posts={popularPosts} />
    </div>
  );
}
```

### Pattern 4: Conditional Data Fetching
```typescript
export default async function BlogPost({ params, searchParams }) {
  const post = await getPost(params.slug);

  if (!post) {
    notFound();
  }

  // Conditional fetching based on query params
  const [author, comments] = await Promise.all([
    getAuthor(post.authorId),
    searchParams.comments === 'true' ? getComments(post.id) : Promise.resolve([])
  ]);

  return (
    <article>
      <h1>{post.title}</h1>
      <AuthorInfo author={author} />
      {comments.length > 0 && <Comments comments={comments} />}
    </article>
  );
}
```

## Error Handling in Async Components

### Basic Error Handling
```typescript
export default async function BlogPost({ params }) {
  try {
    const post = await getPost(params.slug);

    if (!post) {
      notFound(); // Triggers app/not-found.tsx
    }

    return <article>{post.title}</article>;
  } catch (error) {
    // Triggers app/error.tsx
    throw new Error('Failed to load blog post');
  }
}
```

### Graceful Error Handling
```typescript
export default async function BlogPost({ params }) {
  const post = await getPost(params.slug).catch(() => null);

  if (!post) {
    return (
      <div>
        <h1>Post Not Available</h1>
        <p>The requested post could not be loaded.</p>
        <Link href="/blog">← Back to Blog</Link>
      </div>
    );
  }

  return <article>{post.title}</article>;
}
```

### Partial Failure Handling
```typescript
export default async function Dashboard() {
  // Some data is critical, some is optional
  const [userData, settingsData, analyticsData] = await Promise.allSettled([
    getUserData(),     // Critical
    getSettings(),     // Critical
    getAnalytics()     // Optional
  ]);

  // Check critical data
  if (userData.status === 'rejected' || settingsData.status === 'rejected') {
    throw new Error('Failed to load critical data');
  }

  return (
    <div>
      <UserProfile user={userData.value} />
      <Settings settings={settingsData.value} />

      {/* Optional data - show if available */}
      {analyticsData.status === 'fulfilled' ? (
        <Analytics data={analyticsData.value} />
      ) : (
        <div>Analytics temporarily unavailable</div>
      )}
    </div>
  );
}
```

## Real-World Examples

### Example 1: Blog Post with Full Data
```typescript
// app/blog/[slug]/page.tsx
import { getPost, getRelatedPosts, getAuthor } from '@/services/postService';
import { notFound } from 'next/navigation';

export default async function BlogPost({ params }) {
  // Parallel fetch for better performance
  const [post, relatedPosts] = await Promise.all([
    getPost(params.slug),
    getRelatedPosts(params.slug, 3)
  ]);

  if (!post) {
    notFound();
  }

  // Sequential fetch that depends on post data
  const author = await getAuthor(post.authorId);

  return (
    <div className="max-w-4xl mx-auto">
      <article>
        <header>
          <h1 className="text-4xl font-bold">{post.title}</h1>
          <div className="flex items-center gap-4 mt-4">
            <img src={author.avatar} alt={author.name} className="w-12 h-12 rounded-full" />
            <div>
              <p className="font-semibold">{author.name}</p>
              <time>{new Date(post.publishedAt).toLocaleDateString()}</time>
            </div>
          </div>
        </header>

        <div className="prose max-w-none mt-8">
          {post.content}
        </div>
      </article>

      <aside className="mt-12">
        <h2 className="text-2xl font-bold mb-4">Related Posts</h2>
        <div className="grid gap-4">
          {relatedPosts.map(relatedPost => (
            <PostCard key={relatedPost.id} post={relatedPost} />
          ))}
        </div>
      </aside>
    </div>
  );
}
```

### Example 2: Dashboard with Multiple Data Sources
```typescript
// app/dashboard/page.tsx
import { getUser, getStats, getNotifications, getRecentActivity } from '@/services/dashboardService';

export default async function Dashboard() {
  // Critical data - must succeed
  const user = await getUser();

  if (!user) {
    redirect('/login');
  }

  // Parallel fetch for dashboard widgets
  const [stats, notifications, recentActivity] = await Promise.all([
    getStats(user.id).catch(() => null),           // Optional
    getNotifications(user.id).catch(() => []),      // Optional with fallback
    getRecentActivity(user.id).catch(() => [])      // Optional with fallback
  ]);

  return (
    <div className="dashboard">
      <header className="mb-8">
        <h1>Welcome back, {user.name}!</h1>
      </header>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        {/* Stats Widget */}
        <div className="stats-widget">
          {stats ? (
            <StatsCard stats={stats} />
          ) : (
            <div>Stats unavailable</div>
          )}
        </div>

        {/* Notifications Widget */}
        <div className="notifications-widget">
          <NotificationsList notifications={notifications} />
        </div>

        {/* Activity Widget */}
        <div className="activity-widget">
          <ActivityFeed activities={recentActivity} />
        </div>
      </div>
    </div>
  );
}
```

### Example 3: E-commerce Product Page
```typescript
// app/products/[id]/page.tsx
import { getProduct, getReviews, getRelatedProducts } from '@/services/productService';

export default async function ProductPage({ params }) {
  const product = await getProduct(params.id);

  if (!product) {
    notFound();
  }

  // Fetch related data in parallel
  const [reviews, relatedProducts] = await Promise.all([
    getReviews(product.id),
    getRelatedProducts(product.categoryId, 4)
  ]);

  return (
    <div className="product-page">
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
        {/* Product Images */}
        <div>
          <ProductImageGallery images={product.images} />
        </div>

        {/* Product Info */}
        <div>
          <h1 className="text-3xl font-bold">{product.name}</h1>
          <p className="text-2xl font-semibold text-green-600">${product.price}</p>
          <p className="mt-4">{product.description}</p>

          {/* Add to Cart Button - This would be a Client Component */}
          <AddToCartButton productId={product.id} />
        </div>
      </div>

      {/* Reviews Section */}
      <section className="mt-12">
        <h2 className="text-2xl font-bold mb-6">Customer Reviews</h2>
        <ReviewsList reviews={reviews} />
      </section>

      {/* Related Products */}
      <section className="mt-12">
        <h2 className="text-2xl font-bold mb-6">Related Products</h2>
        <ProductGrid products={relatedProducts} />
      </section>
    </div>
  );
}
```

## Performance Considerations

### Loading States vs Server Rendering
```typescript
// ❌ Client Component - Shows loading state
'use client'
export default function ClientBlogPost({ params }) {
  const [post, setPost] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchPost(params.slug).then(setPost).finally(() => setLoading(false));
  }, [params.slug]);

  if (loading) return <PostSkeleton />; // User sees loading
  return <PostContent post={post} />;
}

// ✅ Server Component - No loading state needed
export default async function ServerBlogPost({ params }) {
  const post = await getPost(params.slug); // Happens on server
  return <PostContent post={post} />; // User sees content immediately
}
```

### Parallel vs Sequential Performance
```typescript
// ❌ Slow - Sequential (3 seconds total)
export default async function SlowPage() {
  const data1 = await fetch1(); // 1 second
  const data2 = await fetch2(); // 1 second
  const data3 = await fetch3(); // 1 second
  return <Content data1={data1} data2={data2} data3={data3} />;
}

// ✅ Fast - Parallel (1 second total)
export default async function FastPage() {
  const [data1, data2, data3] = await Promise.all([
    fetch1(), // All execute simultaneously
    fetch2(),
    fetch3()
  ]);
  return <Content data1={data1} data2={data2} data3={data3} />;
}
```

### Streaming with Suspense
```typescript
// app/blog/[slug]/page.tsx
import { Suspense } from 'react';

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug); // Critical data

  return (
    <div>
      <article>
        <h1>{post.title}</h1>
        <p>{post.content}</p>
      </article>

      {/* Non-critical data can be streamed */}
      <Suspense fallback={<CommentsSkeleton />}>
        <CommentsSection postId={post.id} />
      </Suspense>

      <Suspense fallback={<RelatedPostsSkeleton />}>
        <RelatedPosts slug={params.slug} />
      </Suspense>
    </div>
  );
}

// Separate async component for streaming
async function CommentsSection({ postId }) {
  const comments = await getComments(postId); // This can take time
  return <CommentsList comments={comments} />;
}
```

## When NOT to Use Async Components

### Client-Side Interactions
```typescript
// ❌ Wrong - Can't use async for client interactions
'use client'
export default async function ClientComponent() { // Error!
  const handleClick = async () => {
    // Client-side async operations go in event handlers
    const response = await fetch('/api/update');
  };

  return <button onClick={handleClick}>Update</button>;
}

// ✅ Correct - Async in event handlers
'use client'
export default function ClientComponent() {
  const handleClick = async () => { // Async event handler is fine
    const response = await fetch('/api/update');
  };

  return <button onClick={handleClick}>Update</button>;
}
```

### Real-Time Data
```typescript
// ❌ Wrong - Async components render once
export default async function LiveData() {
  const data = await fetchLiveData(); // Only fetches once at render
  return <div>{data.price}</div>; // Price won't update
}

// ✅ Correct - Use client component for real-time updates
'use client'
export default function LiveData() {
  const [data, setData] = useState(null);

  useEffect(() => {
    const ws = new WebSocket('/api/live-data');
    ws.onmessage = (event) => setData(JSON.parse(event.data));
    return () => ws.close();
  }, []);

  return <div>{data?.price}</div>; // Updates in real-time
}
```

## Best Practices

### 1. Parallel Fetching for Independent Data
```typescript
// ✅ Good - Independent data fetched in parallel
const [user, posts, settings] = await Promise.all([
  getUser(userId),
  getPosts(),
  getSettings()
]);
```

### 2. Sequential Fetching for Dependent Data
```typescript
// ✅ Good - Dependent data fetched sequentially
const user = await getUser(userId);
const userPosts = await getUserPosts(user.id); // Needs user data
```

### 3. Error Boundaries for Graceful Failures
```typescript
// ✅ Good - Handle errors gracefully
try {
  const criticalData = await getCriticalData();
  const optionalData = await getOptionalData().catch(() => null);

  return <Page critical={criticalData} optional={optionalData} />;
} catch (error) {
  throw new Error('Failed to load page');
}
```

### 4. Use TypeScript for Better Safety
```typescript
interface Post {
  id: string;
  title: string;
  content: string;
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post: Post | null = await getPost(params.slug);

  if (!post) {
    notFound();
  }

  return <article>{post.title}</article>;
}
```

## Summary

### Key Benefits:
- **No loading states** - Data ready when component renders
- **Better SEO** - Content available immediately
- **Simpler code** - No useEffect or state management
- **Server-side errors** - Caught and handled on server

### Key Patterns:
- **Single fetch**: `await getData()`
- **Parallel fetch**: `await Promise.all([getData1(), getData2()])`
- **Sequential fetch**: `await getData1(); await getData2(data1)`
- **Error handling**: Try/catch or `.catch()` for graceful failures

### Remember:
- **Server Components only** - Client components can't be async
- **Renders once** - Not suitable for real-time updates
- **Error boundaries** - Use Next.js error.tsx for error handling
- **Performance** - Prefer parallel fetching when possible

Async Server Components are one of the most powerful features of modern Next.js, enabling you to write simpler, more performant applications with server-side data fetching.