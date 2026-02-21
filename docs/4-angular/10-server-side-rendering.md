---
sidebar_position: 10
---

# Server-Side and Hybrid Rendering

By default, Angular renders applications in the browser (client-side rendering, or CSR). Angular also supports **hybrid rendering**, which combines server-side rendering (SSR), pre-rendering (static site generation), and CSR to optimize performance and SEO.

Source: [angular.dev/guide/ssr](https://angular.dev/guide/ssr)

## Why Use SSR?

- **Faster first paint** — Users see content immediately instead of waiting for JavaScript to download, parse, and execute.
- **Better SEO** — Search engine crawlers receive fully rendered HTML.
- **Improved Core Web Vitals** — Faster LCP (Largest Contentful Paint) and FCP (First Contentful Paint).

## Getting Started

### New Project with SSR

```bash
ng new my-app --ssr
```

### Adding SSR to an Existing Project

```bash
ng add @angular/ssr
```

This command adds `@angular/ssr`, configures the server entry point, and updates the build configuration.

## Render Modes

Angular supports three rendering strategies that can be configured per route:

| Mode | Description | When to Use |
|---|---|---|
| **Prerender** (SSG) | HTML generated at build time | Static content, blog posts, marketing pages |
| **Server** (SSR) | HTML generated on each request | Dynamic content, personalized pages, API-dependent content |
| **Client** (CSR) | Standard client-side rendering | Highly interactive pages, authenticated dashboards |

### Default Behavior

By default, Angular prerenders the entire application at build time. You can configure individual routes:

```typescript
import { RenderMode, ServerRoute } from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  // Static pages — prerendered at build time
  { path: '', renderMode: RenderMode.Prerender },
  { path: 'about', renderMode: RenderMode.Prerender },

  // Dynamic pages — rendered on each request
  { path: 'users/:id', renderMode: RenderMode.Server },
  { path: 'dashboard', renderMode: RenderMode.Server },

  // Client-only pages
  { path: 'admin/**', renderMode: RenderMode.Client },
];
```

### Disabling Prerendering

To disable prerendering and enable full SSR, update your `angular.json`:

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "options": {
            "outputMode": "server"
          }
        }
      }
    }
  }
}
```

Set `outputMode` to `"static"` to only prerender without SSR capabilities.

## Server Route Configuration

Server routes are defined in a `app.routes.server.ts` file:

```typescript
import { RenderMode, ServerRoute } from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'products/:id',
    renderMode: RenderMode.Prerender,
    async getPrerenderParams() {
      // Provide parameter values for prerendering
      const products = await fetchProductIds();
      return products.map(id => ({ id: String(id) }));
    },
  },
  {
    path: '**',
    renderMode: RenderMode.Server,
  },
];
```

### Route Parameter Resolution During Prerendering

For routes with parameters (like `/products/:id`), use `getPrerenderParams` to provide the parameter values at build time:

```typescript
{
  path: 'blog/:slug',
  renderMode: RenderMode.Prerender,
  async getPrerenderParams() {
    const posts = await fetch('https://api.example.com/posts')
      .then(res => res.json());
    return posts.map((post: any) => ({ slug: post.slug }));
  },
}
```

## Hydration

Hydration is the process where Angular takes the server-rendered HTML and attaches event listeners and Angular functionality to it on the client side, making it interactive. This avoids the flash of re-rendered content that would occur if the client had to re-render from scratch.

Angular's hydration:

- **Preserves the server-rendered DOM** instead of destroying and re-creating it.
- **Reuses existing DOM nodes** and attaches Angular's event handling and change detection.
- **Validates** that the server-rendered HTML matches what the client would render.

Hydration is enabled by default when using Angular SSR.

### Direct DOM Manipulation

If parts of your application directly manipulate the DOM (e.g., via `ElementRef.nativeElement`), hydration validation may fail. To skip hydration for specific components:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-chart',
  template: `<div id="chart-container"></div>`,
  host: { ngSkipHydration: 'true' },
})
export class ChartComponent {
  // This component uses a third-party charting library
  // that directly manipulates the DOM
}
```

Or in the template:

```html
<app-chart ngSkipHydration />
```

## Incremental Hydration

Incremental hydration defers the hydration of specific sections of the page until they are needed. This improves initial interactivity by reducing the amount of JavaScript that must execute on load.

Incremental hydration integrates with `@defer`:

```html
@defer (on viewport; hydrate on interaction) {
  <comments-section />
}
```

The `hydrate` trigger controls when the deferred block becomes interactive:

| Trigger | Description |
|---|---|
| `hydrate on interaction` | Hydrate when the user interacts with the block |
| `hydrate on hover` | Hydrate when the user hovers over the block |
| `hydrate on viewport` | Hydrate when the block enters the viewport |
| `hydrate on idle` | Hydrate when the browser is idle |
| `hydrate on timer(5s)` | Hydrate after a delay |
| `hydrate when condition` | Hydrate when a condition is true |
| `hydrate never` | Never hydrate on the client (server-render only) |

## Event Replay

Event replay is enabled by default in Angular 19. When a user interacts with a server-rendered page before hydration completes, Angular captures those events and replays them after hydration. This ensures that user actions (like button clicks) are not lost during the hydration window.

## Platform APIs

When code runs on the server, browser APIs like `window`, `document`, and `localStorage` are not available. Angular provides utilities to handle this:

```typescript
import { isPlatformBrowser, isPlatformServer } from '@angular/common';
import { PLATFORM_ID, inject } from '@angular/core';

@Component({ ... })
export class MyComponent {
  private platformId = inject(PLATFORM_ID);

  ngOnInit() {
    if (isPlatformBrowser(this.platformId)) {
      // Safe to use browser APIs
      window.addEventListener('resize', this.onResize);
    }
  }
}
```

### afterNextRender

Use `afterNextRender` or `afterRender` for code that should only run in the browser after the DOM is rendered:

```typescript
import { afterNextRender } from '@angular/core';

@Component({ ... })
export class MapComponent {
  constructor() {
    afterNextRender(() => {
      // This only runs in the browser, after the DOM is ready
      initializeMap(document.getElementById('map'));
    });
  }
}
```

## Build and Deployment

### Building for SSR

```bash
ng build
```

The build output includes both browser and server bundles.

### Serving

For development:

```bash
ng serve
```

For production, the server entry point is typically `server.ts`, which can be deployed to any Node.js hosting platform (Firebase, AWS Lambda, Vercel, etc.).
