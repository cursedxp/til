# Next.js Metadata Guide

## What is Metadata?

Metadata is information about your web page that search engines, social media platforms, and browsers use to understand and display your content. In Next.js 13+, the App Router provides a built-in Metadata API to manage this information declaratively.

## Why Metadata Matters

- **SEO**: Helps search engines understand and rank your content
- **Social Sharing**: Controls how your pages look when shared on social media
- **User Experience**: Sets page titles, favicons, and viewport settings
- **Web Standards**: Defines character sets, language, and compatibility

## How to Use Metadata in Next.js

### 1. Static Metadata (Config-based)

Export a `metadata` object from any `page.tsx` or `layout.tsx`:

```tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'My Blog',
  description: 'A blog about web development',
};

export default function Page() {
  return <div>Content</div>;
}
```

### 2. Dynamic Metadata (Function-based)

Use `generateMetadata` when you need dynamic values (e.g., from URL params or database):

```tsx
import { Metadata } from 'next';

type Props = {
  params: { id: string };
};

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await fetchPost(params.id);

  return {
    title: post.title,
    description: post.excerpt,
  };
}

export default function PostPage({ params }: Props) {
  return <div>Post content</div>;
}
```

## Common Metadata Fields

### Basic Fields

```tsx
export const metadata: Metadata = {
  title: 'Page Title',
  description: 'Page description for search engines',
  keywords: ['nextjs', 'react', 'metadata'],
  authors: [{ name: 'John Doe' }],
  creator: 'John Doe',
  publisher: 'My Company',
};
```

### Open Graph (Social Media)

Controls how your page appears when shared on Facebook, LinkedIn, etc.:

```tsx
export const metadata: Metadata = {
  title: 'My Article',
  description: 'An interesting article',
  openGraph: {
    title: 'My Article',
    description: 'An interesting article',
    url: 'https://example.com/article',
    siteName: 'My Blog',
    images: [
      {
        url: 'https://example.com/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'Article preview image',
      },
    ],
    locale: 'en_US',
    type: 'article',
  },
};
```

### Twitter Card

Specific metadata for Twitter/X sharing:

```tsx
export const metadata: Metadata = {
  twitter: {
    card: 'summary_large_image',
    title: 'My Article',
    description: 'An interesting article',
    creator: '@myhandle',
    images: ['https://example.com/twitter-image.jpg'],
  },
};
```

### Viewport and Icons

```tsx
export const metadata: Metadata = {
  viewport: {
    width: 'device-width',
    initialScale: 1,
    maximumScale: 1,
  },
  icons: {
    icon: '/favicon.ico',
    apple: '/apple-icon.png',
  },
};
```

## Practical Examples

### Example 1: Blog Homepage

```tsx
// app/blog/page.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Tech Blog | Latest Web Development Articles',
  description: 'Explore tutorials and guides on Next.js, React, and modern web development.',
  keywords: ['web development', 'nextjs', 'react', 'tutorials'],
  openGraph: {
    title: 'Tech Blog',
    description: 'Latest web development articles and tutorials',
    url: 'https://myblog.com/blog',
    type: 'website',
    images: ['/blog-og-image.jpg'],
  },
};

export default function BlogPage() {
  return <div>Blog posts list</div>;
}
```

### Example 2: Dynamic Blog Post

```tsx
// app/blog/[slug]/page.tsx
import { Metadata } from 'next';

type Props = {
  params: { slug: string };
};

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.slug);

  return {
    title: `${post.title} | My Blog`,
    description: post.excerpt,
    authors: [{ name: post.author }],
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      publishedTime: post.publishedDate,
      authors: [post.author],
      images: [
        {
          url: post.coverImage,
          width: 1200,
          height: 630,
          alt: post.title,
        },
      ],
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  };
}

export default function PostPage({ params }: Props) {
  return <div>Post content</div>;
}
```

### Example 3: Category Page

```tsx
// app/blog/category/[category]/page.tsx
import { Metadata } from 'next';

type Props = {
  params: { category: string };
};

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const categoryName = params.category.replace('-', ' ');

  return {
    title: `${categoryName} Articles | My Blog`,
    description: `Browse all ${categoryName} articles and tutorials`,
    openGraph: {
      title: `${categoryName} Articles`,
      description: `Browse all ${categoryName} articles`,
      type: 'website',
    },
  };
}

export default function CategoryPage({ params }: Props) {
  return <div>Category posts</div>;
}
```

## Template Titles

Create consistent title patterns across pages:

```tsx
// app/layout.tsx
export const metadata: Metadata = {
  title: {
    template: '%s | My Blog',
    default: 'My Blog',
  },
  description: 'A blog about web development',
};
```

Now child pages can just provide the title:

```tsx
// app/blog/page.tsx
export const metadata: Metadata = {
  title: 'All Posts', // Becomes "All Posts | My Blog"
};
```

## Best Practices

1. **Always provide title and description** - Essential for SEO
2. **Use dynamic metadata for data-driven pages** - Blog posts, products, etc.
3. **Include Open Graph for social sharing** - Better appearance on social media
4. **Keep descriptions between 150-160 characters** - Optimal for search results
5. **Use absolute URLs for images** - Required for social media previews
6. **Set proper image dimensions** - 1200x630 for Open Graph
7. **Use template titles** - Maintains consistency across your site

## How Next.js Merges Metadata

Metadata is merged from root to leaf:

```
app/layout.tsx (root)
  ↓
app/blog/layout.tsx (inherits + overrides)
  ↓
app/blog/[slug]/page.tsx (inherits + overrides)
```

Each level can override or extend the parent's metadata.

## Testing Your Metadata

1. **View Source**: Right-click → View Page Source, look for `<meta>` tags
2. **Social Media Debuggers**:
   - Facebook: [Sharing Debugger](https://developers.facebook.com/tools/debug/)
   - Twitter: [Card Validator](https://cards-dev.twitter.com/validator)
   - LinkedIn: [Post Inspector](https://www.linkedin.com/post-inspector/)
3. **Google Search Console**: Check how Google sees your pages

## Resources

- [Next.js Metadata API Docs](https://nextjs.org/docs/app/api-reference/functions/generate-metadata)
- [Open Graph Protocol](https://ogp.me/)
- [Twitter Cards Guide](https://developer.twitter.com/en/docs/twitter-for-websites/cards/overview/abouts-cards)
