---
sidebar_position: 12
---

# Server Components and SSR

React Server Components (RSC) let you render components on the server, sending only the resulting HTML and minimal JavaScript to the client. Combined with server-side rendering (SSR) and streaming, they enable faster initial loads and smaller client bundles.

Source: [react.dev/reference/rsc/server-components](https://react.dev/reference/rsc/server-components)

## Server Components vs. Client Components

| Aspect | Server Components | Client Components |
|---|---|---|
| Rendering | Server only | Server (for SSR) + client |
| JavaScript sent to client | None | Component code is included in the bundle |
| Access to server resources | Direct (database, file system, env vars) | Through API calls |
| Interactivity | No (cannot use state, effects, or event handlers) | Full (hooks, event handlers) |
| Default in Next.js App Router | Yes (`'use server'` not needed) | Opt-in with `'use client'` |

### When to Use Each

| Use Server Components when | Use Client Components when |
|---|---|
| Fetching data | Adding interactivity (event handlers) |
| Accessing backend resources directly | Using state or effects |
| Keeping sensitive data on the server | Using browser-only APIs |
| Reducing client bundle size | Using custom hooks with state |

## Server Components in Practice

Server Components run once on the server per request and produce HTML. They can be `async` and can `await` data directly:

```tsx
// app/users/page.tsx (Server Component by default in Next.js)
async function UsersPage() {
  const users = await db.query('SELECT * FROM users');

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}

export default UsersPage;
```

No `useEffect`, no loading state, no API route needed. The data is fetched on the server and the HTML is sent to the client.

## Client Components

Add `'use client'` at the top of a file to make it a Client Component. Client Components can use hooks, state, and event handlers:

```tsx
'use client';

import { useState } from 'react';

export function LikeButton() {
  const [likes, setLikes] = useState(0);

  return (
    <button onClick={() => setLikes(likes + 1)}>
      {likes} Likes
    </button>
  );
}
```

### Composition Pattern

Server Components can import and render Client Components, but not the other way around. Pass server data to Client Components through props:

```tsx
// Server Component
import { LikeButton } from './LikeButton';

async function BlogPost({ id }) {
  const post = await getPost(id);

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      <LikeButton postId={post.id} initialLikes={post.likes} />
    </article>
  );
}
```

## Server Actions

Server Actions let Client Components call server-side functions directly, without creating API routes. Mark a function with `'use server'`:

```tsx
// app/actions.ts
'use server';

export async function createComment(formData: FormData) {
  const text = formData.get('text') as string;
  await db.insert('comments', { text, createdAt: new Date() });
  revalidatePath('/posts');
}
```

```tsx
// Client Component
'use client';

import { createComment } from './actions';

export function CommentForm() {
  return (
    <form action={createComment}>
      <textarea name="text" required />
      <button type="submit">Post Comment</button>
    </form>
  );
}
```

Source: [react.dev/reference/rsc/server-functions](https://react.dev/reference/rsc/server-functions)

## Server-Side Rendering (SSR)

SSR renders the initial HTML on the server for every request, improving Time to First Byte (TTFB) and enabling search engines to index your content.

### How SSR Works in React

1. **Server** renders the component tree to HTML and sends it to the browser.
2. **Browser** displays the HTML immediately (fast first paint).
3. **React hydration** attaches event handlers to the existing DOM, making the page interactive.

### Streaming SSR with Suspense

React can stream HTML as it renders, sending parts of the page to the browser before the entire page is ready:

```tsx
import { Suspense } from 'react';

function App() {
  return (
    <div>
      <Header />
      <Suspense fallback={<p>Loading sidebar...</p>}>
        <Sidebar />
      </Suspense>
      <Suspense fallback={<p>Loading content...</p>}>
        <MainContent />
      </Suspense>
    </div>
  );
}
```

Each `<Suspense>` boundary streams its HTML as soon as the content is ready, without waiting for the entire page.

## Static Site Generation (SSG)

Pages that do not change per request can be pre-rendered at build time:

```tsx
// Next.js: this page is statically generated at build time
async function AboutPage() {
  const content = await getAboutContent();
  return <div>{content}</div>;
}

export default AboutPage;
```

Next.js automatically detects whether a page can be statically generated based on whether it uses dynamic data.

## Incremental Static Regeneration (ISR)

ISR lets you update static pages after they have been built, without rebuilding the entire site:

```tsx
// Next.js: revalidate every 60 seconds
export const revalidate = 60;

async function ProductPage({ params }) {
  const product = await getProduct(params.id);
  return <div>{product.name}</div>;
}
```

## Caching

### Request Memoization

React automatically deduplicates identical `fetch` calls made during a single server render. If multiple components fetch the same URL, only one request is made:

```tsx
// Both of these produce only one HTTP request during SSR
async function Header() {
  const user = await fetch('/api/user').then((r) => r.json());
  return <h1>{user.name}</h1>;
}

async function Sidebar() {
  const user = await fetch('/api/user').then((r) => r.json());
  return <p>Welcome, {user.name}</p>;
}
```

### Data Cache (Next.js)

Next.js extends `fetch` with caching options:

```tsx
// Cached indefinitely (default for static pages)
const data = await fetch('https://api.example.com/data');

// Revalidate every 60 seconds
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 60 },
});

// Never cache
const data = await fetch('https://api.example.com/data', {
  cache: 'no-store',
});
```

## Choosing a Rendering Strategy

| Strategy | Best for |
|---|---|
| Server Components | Data-heavy pages, reducing client bundle size |
| Client Components | Interactive UI, forms, real-time features |
| SSR | Dynamic pages that need SEO and fast first paint |
| SSG | Content that rarely changes (marketing pages, docs) |
| ISR | Content that changes periodically (product pages, blog) |
