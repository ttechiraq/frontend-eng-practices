---
sidebar_position: 6
---

# Dependency Injection

Dependency Injection (DI) is a design pattern used to organize and share code across an application by allowing you to "inject" features into different parts.

Source: [angular.dev/guide/di](https://angular.dev/guide/di)

## How DI Works in Angular

A dependency is any object, value, function, or service that a class needs to work but does not create itself. There are two ways code interacts with DI:

- Code can **inject**, or ask for, values as dependencies.
- Code can **provide**, or make available, values.

Common types of injected dependencies include:

- **Services**: Classes that provide common functionality, business logic, or state
- **Factories**: Functions that create objects or values based on runtime conditions
- **Configuration values**: Environment-specific constants, API URLs, feature flags

Angular components and directives automatically participate in DI, meaning they can inject dependencies and are available to be injected.

## What Are Services?

An Angular service is a TypeScript class decorated with `@Injectable`, which makes an instance of the class available to be injected as a dependency. Services are the most common way of sharing data and functionality across an application.

Common types of services:

- Utility functions (formatting, validation, calculations)
- Event handling and dispatch
- Logging and error handling
- Authentication and authorization
- State management
- Data clients (abstracting server requests)

```typescript
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class AnalyticsLogger {
  trackEvent(category: string, value: string) {
    console.log('Analytics event logged:', {
      category,
      value,
      timestamp: new Date().toISOString(),
    });
  }
}
```

The `providedIn: 'root'` option makes the service available throughout the entire application as a singleton. This is the recommended approach for most services.

## Injecting Dependencies with inject()

You can inject dependencies using Angular's `inject()` function:

```typescript
import { Component, inject } from '@angular/core';
import { Router } from '@angular/router';
import { AnalyticsLogger } from './analytics-logger';

@Component({
  selector: 'app-navbar',
  template: `<a href="#" (click)="navigateToDetail($event)">Detail Page</a>`,
})
export class Navbar {
  private router = inject(Router);
  private analytics = inject(AnalyticsLogger);

  navigateToDetail(event: Event) {
    event.preventDefault();
    this.analytics.trackEvent('navigation', '/details');
    this.router.navigate(['/details']);
  }
}
```

### Where Can inject() Be Used?

You can inject dependencies during construction of a component, directive, or service. The call to `inject` can appear in either the `constructor` or in a field initializer:

```typescript
@Component({ /* ... */ })
export class MyComponent {
  // In class field initializer
  private service = inject(MyService);

  // In constructor body
  private anotherService: AnotherService;
  constructor() {
    this.anotherService = inject(AnotherService);
  }
}
```

```typescript
// In a service
@Injectable({ providedIn: 'root' })
export class MyService {
  private http = inject(HttpClient);
}
```

```typescript
// In a route guard
export const authGuard = () => {
  const auth = inject(AuthService);
  return auth.isAuthenticated();
};
```

## Creating and Using Services

### Generating a Service with the CLI

```bash
ng generate service services/user
```

This creates `user.service.ts` with the `@Injectable` decorator.

### Service with HTTP Client

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }

  getUser(id: number): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }

  createUser(user: CreateUserDto): Observable<User> {
    return this.http.post<User>('/api/users', user);
  }
}
```

Source: [angular.dev/guide/di/creating-and-using-services](https://angular.dev/guide/di/creating-and-using-services)

## Defining Dependency Providers

### providedIn Options

| Option | Scope | Tree-shakable |
|---|---|---|
| `'root'` | Application-wide singleton | Yes |
| `'platform'` | Shared across multiple Angular apps on the same page | Yes |
| `'any'` | Unique instance per lazy-loaded module | Yes |

### Providing at the Component Level

```typescript
@Component({
  selector: 'user-list',
  providers: [UserService],
  template: `...`,
})
export class UserList {}
```

Component-level providers create a new instance for each component instance, scoped to that component and its children.

### Using InjectionTokens

For non-class dependencies (configuration values, factory functions), use `InjectionToken`:

```typescript
import { InjectionToken } from '@angular/core';

export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');

// Provide in application config
export const appConfig: ApplicationConfig = {
  providers: [
    { provide: API_BASE_URL, useValue: 'https://api.example.com' },
  ],
};

// Inject in a service
@Injectable({ providedIn: 'root' })
export class ApiService {
  private baseUrl = inject(API_BASE_URL);
}
```

### Provider Types

| Provider | Syntax | Use for |
|---|---|---|
| `useClass` | `{ provide: Logger, useClass: BetterLogger }` | Substitute a different class |
| `useValue` | `{ provide: API_URL, useValue: 'https://...' }` | Provide a constant value |
| `useFactory` | `{ provide: Logger, useFactory: loggerFactory }` | Create with a factory function |
| `useExisting` | `{ provide: Logger, useExisting: BetterLogger }` | Alias one token to another |

Source: [angular.dev/guide/di/defining-dependency-providers](https://angular.dev/guide/di/defining-dependency-providers)

## Hierarchical Injectors

Angular has a hierarchical injection system. There are two injector hierarchies:

1. **Environment injector hierarchy** — Configured via `providedIn` or the `providers` array in `ApplicationConfig`, routes, or `@NgModule`.
2. **Element injector hierarchy** — Created implicitly at each DOM element. A component's `providers` array configures the element injector.

When a component requests a dependency, Angular starts with the component's own injector and walks up the element injector tree, then the environment injector tree, until it finds a provider or throws an error.

```
Root Injector (ApplicationConfig providers)
  └── Route Injector (route providers)
       └── Component Injector (component providers)
            └── Child Component Injector
```

Source: [angular.dev/guide/di/hierarchical-dependency-injection](https://angular.dev/guide/di/hierarchical-dependency-injection)

## Injection Context

Angular uses the term "injection context" to describe any place where you can call `inject()`. While component, directive, and service construction is the most common, other injection contexts include:

- Field initializers of classes with `@Injectable`, `@Component`, `@Directive`, or `@Pipe`
- Factory functions used in `Provider` definitions
- The `runInInjectionContext` utility for running code in a specific injector's context

Source: [angular.dev/guide/di/dependency-injection-context](https://angular.dev/guide/di/dependency-injection-context)

## Lightweight Injection Tokens

When creating libraries, use lightweight injection tokens to optimize tree-shaking. Provide the token as an abstract class or `InjectionToken` so that unused implementations can be removed from production builds.

Source: [angular.dev/guide/di/lightweight-injection-tokens](https://angular.dev/guide/di/lightweight-injection-tokens)
