---
sidebar_position: 7
---

# Routing

Angular Router (`@angular/router`) is the official library for managing navigation in Angular applications and a core part of the framework. It is included by default in all projects created by Angular CLI.

Source: [angular.dev/guide/routing](https://angular.dev/guide/routing)

## Why Routing in a SPA?

A single-page application (SPA) differs from traditional web pages in that the browser only makes a request to a web server for the first page (`index.html`). After that, a client-side router takes over, controlling which content displays based on the URL. When a user navigates to a different URL, the router updates the page's content in place without triggering a full-page reload.

## How Angular Manages Routing

Routing in Angular is comprised of three primary parts:

1. **Routes** define which component displays when a user visits a specific URL.
2. **Outlets** are placeholders in your templates that dynamically load and render components based on the active route.
3. **Links** provide a way for users to navigate between different routes without triggering a full page reload.

## Defining Routes

Routes are defined as an array of `Route` objects, typically in an `app.routes.ts` file:

```typescript
import { Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: '**', component: NotFoundComponent },
];
```

### Route Configuration Properties

| Property | Description |
|---|---|
| `path` | URL path segment that triggers this route |
| `component` | Component to render for this route |
| `redirectTo` | URL to redirect to |
| `pathMatch` | How to match the path (`'full'` or `'prefix'`) |
| `children` | Nested child routes |
| `loadComponent` | Lazy-loaded standalone component |
| `loadChildren` | Lazy-loaded child routes |
| `canActivate` | Guards that check access before activation |
| `resolve` | Data resolvers that pre-fetch data |
| `title` | Static or resolved page title |

### Providing Routes

Routes are provided in your application configuration:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
```

Source: [angular.dev/guide/routing/define-routes](https://angular.dev/guide/routing/define-routes)

## Router Outlets

The `<router-outlet>` directive marks where the router should render the matched component:

```html
<nav>
  <a routerLink="/">Home</a>
  <a routerLink="/about">About</a>
</nav>

<router-outlet />
```

### Named Outlets

You can have multiple outlets with names for advanced layouts:

```html
<router-outlet />
<router-outlet name="sidebar" />
```

Source: [angular.dev/guide/routing/show-routes-with-outlets](https://angular.dev/guide/routing/show-routes-with-outlets)

## Navigation

### RouterLink

Use `routerLink` for declarative navigation in templates:

```html
<a routerLink="/users">Users</a>
<a [routerLink]="['/users', userId]">User Detail</a>
<a routerLink="/about" routerLinkActive="active">About</a>
```

`routerLinkActive` adds a CSS class when the link's route is active.

### Programmatic Navigation

Use the `Router` service for navigation from TypeScript:

```typescript
import { Router } from '@angular/router';

@Component({ ... })
export class SearchComponent {
  private router = inject(Router);

  goToUser(id: number) {
    this.router.navigate(['/users', id]);
  }

  goToSearchResults(query: string) {
    this.router.navigate(['/search'], { queryParams: { q: query } });
  }
}
```

Source: [angular.dev/guide/routing/navigate-to-routes](https://angular.dev/guide/routing/navigate-to-routes)

## Reading Route State

### Route Parameters

```typescript
import { Component, input } from '@angular/core';

@Component({ ... })
export class UserDetailComponent {
  // With `withComponentInputBinding()`, route params are bound as inputs
  id = input.required<string>();
}
```

Enable route parameter binding in your app config:

```typescript
provideRouter(routes, withComponentInputBinding())
```

### ActivatedRoute

For more control, use `ActivatedRoute`:

```typescript
import { ActivatedRoute } from '@angular/router';

@Component({ ... })
export class UserDetailComponent {
  private route = inject(ActivatedRoute);

  constructor() {
    // Snapshot for one-time read
    const id = this.route.snapshot.paramMap.get('id');

    // Observable for reacting to changes
    this.route.paramMap.subscribe(params => {
      console.log('User ID:', params.get('id'));
    });
  }
}
```

Source: [angular.dev/guide/routing/read-route-state](https://angular.dev/guide/routing/read-route-state)

## Route Guards

Guards control access to routes. Angular supports functional guards:

### canActivate

Prevents navigation to a route:

```typescript
import { inject } from '@angular/core';
import { Router } from '@angular/router';

export const authGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }
  return router.parseUrl('/login');
};

// Usage in routes
{ path: 'admin', component: AdminComponent, canActivate: [authGuard] }
```

### canDeactivate

Prevents leaving a route (e.g., unsaved changes):

```typescript
export const unsavedChangesGuard = (component: EditComponent) => {
  return component.hasUnsavedChanges() ? confirm('Discard changes?') : true;
};
```

### Other Guard Types

| Guard | Purpose |
|---|---|
| `canActivate` | Controls access to a route |
| `canActivateChild` | Controls access to child routes |
| `canDeactivate` | Controls leaving a route |
| `canMatch` | Controls whether a route can even be considered |
| `resolve` | Pre-fetches data before route activation |

Source: [angular.dev/guide/routing/route-guards](https://angular.dev/guide/routing/route-guards)

## Data Resolvers

Resolvers fetch data before a route is activated:

```typescript
export const userResolver = (route: ActivatedRouteSnapshot) => {
  const userService = inject(UserService);
  return userService.getUser(route.paramMap.get('id')!);
};

// Usage in routes
{
  path: 'users/:id',
  component: UserDetailComponent,
  resolve: { user: userResolver },
}
```

Source: [angular.dev/guide/routing/data-resolvers](https://angular.dev/guide/routing/data-resolvers)

## Lazy-Loaded Routes

Lazy loading defers the download of route components until navigation, reducing the initial bundle size:

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent),
  },
  {
    path: 'settings',
    loadChildren: () => import('./settings/settings.routes').then(m => m.settingsRoutes),
  },
];
```

Source: [angular.dev/guide/routing/define-routes](https://angular.dev/guide/routing/define-routes)

## Redirects and Wildcards

```typescript
export const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'legacy-page', redirectTo: '/home' },
  { path: '**', component: PageNotFoundComponent }, // Wildcard — must be last
];
```

The wildcard route (`**`) matches any URL that doesn't match a previous route. It must be the last route in the array.

## Rendering Strategies

Angular 19 introduced server route configuration that controls how routes are rendered:

```typescript
import { RenderMode, ServerRoute } from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  { path: '', renderMode: RenderMode.Prerender },
  { path: 'dashboard', renderMode: RenderMode.Client },
  { path: 'api/**', renderMode: RenderMode.Server },
];
```

| Mode | Description |
|---|---|
| `RenderMode.Prerender` | Static HTML generated at build time |
| `RenderMode.Server` | HTML generated on each request |
| `RenderMode.Client` | Client-side rendering only |

Source: [angular.dev/guide/routing/rendering-strategies](https://angular.dev/guide/routing/rendering-strategies)
