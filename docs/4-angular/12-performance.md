---
sidebar_position: 12
---

# Performance

Angular includes many optimizations out of the box, but as applications grow, you may need to fine-tune both how quickly your app loads and how responsive it feels during use. These guides cover the tools and techniques Angular provides.

Source: [angular.dev/best-practices/performance](https://angular.dev/best-practices/performance)

## Loading Performance

Loading performance determines how quickly your application becomes visible and interactive. Slow loading directly impacts Core Web Vitals like LCP (Largest Contentful Paint) and TTFB (Time to First Byte).

| Technique | What It Does | When to Use |
|---|---|---|
| Lazy-loaded routes | Defers loading route components until navigation | Multiple routes where not all are needed on initial load |
| `@defer` | Splits components into separate bundles that load on demand | Components not visible on initial render, heavy libraries |
| Image optimization | Prioritizes LCP images, lazy loads others, generates responsive `srcset` | Any application that displays images |
| Server-side rendering | Renders pages on the server for faster first paint and better SEO | Content-heavy applications, pages needing search engine indexing |

### Lazy-Loaded Routes

Defer loading route components until the user navigates to them:

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () =>
      import('./admin/admin.component').then(m => m.AdminComponent),
  },
  {
    path: 'reports',
    loadChildren: () =>
      import('./reports/reports.routes').then(m => m.reportRoutes),
  },
];
```

### Deferred Loading with @defer

Split components into separate bundles that load on demand:

```html
@defer (on viewport) {
  <heavy-data-table [data]="tableData" />
} @placeholder {
  <p>Scroll to load table</p>
} @loading (minimum 300ms) {
  <spinner />
}
```

See the [Templates](./4-templates.md#deferred-loading-with-defer) chapter for full `@defer` documentation.

### Image Optimization with NgOptimizedImage

Angular provides `NgOptimizedImage` to automatically optimize image loading:

```typescript
import { NgOptimizedImage } from '@angular/common';

@Component({
  imports: [NgOptimizedImage],
  template: `
    <!-- LCP image — prioritized automatically -->
    <img ngSrc="/img/hero.jpg" width="1200" height="600" priority />

    <!-- Below-the-fold image — lazy loaded automatically -->
    <img ngSrc="/img/product.jpg" width="400" height="300" />
  `,
})
export class ProductPage {}
```

`NgOptimizedImage`:
- Automatically adds `loading="lazy"` to non-priority images
- Adds `fetchpriority="high"` to priority images
- Generates responsive `srcset` attributes when configured with a loader
- Enforces `width` and `height` attributes to prevent CLS (Cumulative Layout Shift)
- Warns about common image performance issues during development

Source: [angular.dev/best-practices/performance/image-optimization](https://angular.dev/best-practices/performance/image-optimization)

## Runtime Performance

Runtime performance determines how responsive your application feels after it loads. Angular's change detection system keeps the DOM in sync with your data, and optimizing how and when it runs is the primary lever for improving runtime performance.

| Technique | What It Does | When to Use |
|---|---|---|
| Zoneless change detection | Removes ZoneJS overhead; triggers CD only on signal/event changes | New apps (default in v21+), or existing apps ready to migrate |
| Slow computations | Identifies expensive template expressions and lifecycle hooks | Profiling reveals specific components causing slow CD cycles |
| OnPush change detection | Skips unchanged component trees | Applications needing finer control over change detection |
| Zone pollution | Prevents unnecessary CD caused by third-party libraries or timers | Zone-based apps with excessive CD cycles |

### Zoneless Change Detection

Zoneless change detection removes the ZoneJS dependency entirely. Angular triggers change detection only when signals change or when explicit events occur, resulting in less overhead and more predictable behavior.

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideZonelessChangeDetection } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZonelessChangeDetection(),
  ],
};
```

Zoneless is experimental in Angular 19 and becomes the default in Angular v21+. Key requirements:

- Use signals for state management (signals automatically notify Angular of changes)
- Avoid relying on ZoneJS to trigger change detection (e.g., `setTimeout`, `setInterval`, `Promise.then`)

Source: [angular.dev/guide/zoneless](https://angular.dev/guide/zoneless)

### OnPush Change Detection

`OnPush` components only check for changes when:

- An input reference changes
- An event handler in the component or its children fires
- A signal read in the template changes
- `ChangeDetectorRef.markForCheck()` is called
- An async pipe receives a new value

```typescript
@Component({
  selector: 'user-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<p>{{ user().name }}</p>`,
})
export class UserCard {
  user = input.required<User>();
}
```

Source: [angular.dev/best-practices/skipping-subtrees](https://angular.dev/best-practices/skipping-subtrees)

### Avoiding Slow Computations

Expensive computations in templates are re-evaluated on every change detection cycle. Use signals or `computed` to memoize results:

```typescript
// Avoid: recalculated every CD cycle
@Component({
  template: `<p>{{ getFilteredItems().length }} results</p>`,
})
export class BadExample {
  getFilteredItems() {
    return this.items.filter(item => item.active); // Runs every CD cycle
  }
}

// Prefer: memoized with computed signal
@Component({
  template: `<p>{{ filteredCount() }} results</p>`,
})
export class GoodExample {
  items = signal<Item[]>([]);
  filteredCount = computed(() => this.items().filter(item => item.active).length);
}
```

Source: [angular.dev/best-practices/slow-computations](https://angular.dev/best-practices/slow-computations)

### Zone Pollution

In zone-based applications, some third-party libraries trigger unnecessary change detection by running code inside Angular's zone. Run such code outside the zone:

```typescript
import { NgZone, inject } from '@angular/core';

@Component({ ... })
export class MapComponent {
  private ngZone = inject(NgZone);

  initMap() {
    this.ngZone.runOutsideAngular(() => {
      // Third-party library code that triggers many events
      // (e.g., Google Maps, D3 animations)
      // This won't trigger Angular change detection
      initializeThirdPartyMap();
    });
  }
}
```

Source: [angular.dev/best-practices/zone-pollution](https://angular.dev/best-practices/zone-pollution)

## Measuring Performance

Identifying what to optimize is just as important as knowing how to optimize it.

### Chrome DevTools Angular Track

Angular integrates with Chrome DevTools Performance panel. Record a performance trace and look for the Angular-specific track that shows:

- Component rendering durations
- Change detection cycles
- Lifecycle hook execution times

Color-coded flame charts help identify which components take the longest to render.

Source: [angular.dev/best-practices/profiling-with-chrome-devtools](https://angular.dev/best-practices/profiling-with-chrome-devtools)

### Angular DevTools

The Angular DevTools browser extension provides:

- **Component tree inspector** — Explore the component hierarchy, inspect inputs/outputs, and view dependency injection
- **Profiler** — Visualize change detection cycles and identify slow components
- **DI tree view** — Inspect the dependency injection hierarchy

Install from the [Chrome Web Store](https://chrome.google.com/webstore/detail/angular-devtools/ienfalfjdbdpebioblfackkekamfmbnh) or [Firefox Add-ons](https://addons.mozilla.org/en-US/firefox/addon/angular-devtools/).

## What to Optimize First

If you are unsure where to start, profile your application first using the Chrome DevTools Angular track to identify specific bottlenecks.

As a general starting point:

- **Slow interactions after load** — Check whether zoneless change detection is enabled, look for slow computations in templates or lifecycle hooks, and consider `OnPush` to reduce unnecessary change detection.
- **Slow initial load** — Use `@defer` to split large components out of the main bundle, `NgOptimizedImage` to prioritize above-the-fold images, and server-side rendering to deliver content faster.
