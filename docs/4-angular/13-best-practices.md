---
sidebar_position: 13
---

# Best Practices

This chapter covers Angular's recommended practices for security, accessibility, coding style, error handling, and keeping your application up to date.

## Security

Angular provides built-in protections against common web vulnerabilities. Understanding these protections helps you avoid introducing security holes.

Source: [angular.dev/best-practices/security](https://angular.dev/best-practices/security)

### Cross-Site Scripting (XSS) Prevention

Angular treats all values as untrusted by default. When a value is inserted into the DOM through a template binding, Angular automatically sanitizes it to remove potentially dangerous content.

```typescript
// Angular sanitizes this automatically — script tags are stripped
@Component({
  template: `<div [innerHTML]="userContent"></div>`,
})
export class CommentComponent {
  userContent = '<p>Hello</p><script>alert("xss")</script>';
  // Rendered: <p>Hello</p> — script tag is removed
}
```

### Trusted Types

Angular supports [Trusted Types](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API), a browser API that helps prevent DOM-based XSS by requiring that values assigned to dangerous DOM sinks go through a policy.

Enable Trusted Types via Content Security Policy (CSP) headers:

```
Content-Security-Policy: trusted-types angular; require-trusted-types-for 'script'
```

### DomSanitizer

When you need to bypass Angular's built-in sanitization (e.g., for trusted HTML from a CMS), use `DomSanitizer`:

```typescript
import { DomSanitizer, SafeHtml } from '@angular/platform-browser';

@Component({ ... })
export class RichContentComponent {
  private sanitizer = inject(DomSanitizer);
  trustedHtml: SafeHtml;

  constructor() {
    // Only bypass sanitization for content you absolutely trust
    this.trustedHtml = this.sanitizer.bypassSecurityTrustHtml(trustedContent);
  }
}
```

:::warning[important]
Bypassing sanitization can expose your application to XSS attacks. Only use `bypassSecurityTrust*` methods for content from a trusted source that you control.
:::

### Security Best Practices

- Never use string concatenation to build templates or template URLs from user input.
- Avoid direct DOM manipulation via `ElementRef.nativeElement` — prefer Angular's template syntax.
- Do not disable Angular's built-in sanitization unless absolutely necessary.
- Keep Angular and its dependencies up to date to receive security patches.
- Use CSP (Content Security Policy) headers to add an additional layer of defense.

## Accessibility (a11y)

Angular provides tools and patterns to help you build accessible applications that comply with WCAG standards.

Source: [angular.dev/best-practices/a11y](https://angular.dev/best-practices/a11y)

### Angular CDK Accessibility Utilities

The Angular CDK (Component Dev Kit) provides several accessibility utilities:

- **`FocusTrap`** — Traps focus within a container (essential for modals/dialogs).
- **`LiveAnnouncer`** — Announces messages to screen readers via ARIA live regions.
- **`FocusMonitor`** — Monitors focus state and origin (keyboard, mouse, programmatic).
- **`HighContrastModeDetector`** — Detects if high contrast mode is active.

```typescript
import { LiveAnnouncer } from '@angular/cdk/a11y';

@Component({ ... })
export class NotificationComponent {
  private announcer = inject(LiveAnnouncer);

  notify(message: string) {
    this.announcer.announce(message, 'polite');
  }
}
```

### Accessible Routing

When navigating between routes in an SPA, screen reader users may not know the page has changed. Angular provides `TitleStrategy` to set page titles on navigation:

```typescript
import { TitleStrategy, RouterStateSnapshot } from '@angular/router';

@Injectable({ providedIn: 'root' })
export class PageTitleStrategy extends TitleStrategy {
  override updateTitle(routerState: RouterStateSnapshot): void {
    const title = this.buildTitle(routerState);
    if (title) {
      document.title = `${title} | My App`;
    }
  }
}

// Provide in app config
providers: [
  { provide: TitleStrategy, useClass: PageTitleStrategy },
]
```

### Accessibility Best Practices

- Use semantic HTML elements (`<button>`, `<nav>`, `<main>`) instead of generic `<div>` elements with ARIA roles.
- Provide meaningful `aria-label` attributes for icon-only buttons and interactive elements without visible text.
- Manage focus intentionally — move focus into dialogs when opened and return it when closed.
- Test with keyboard-only navigation to ensure all functionality is accessible.
- Use Angular Material components, which are built with accessibility in mind.

### Angular Aria

Angular 19 introduces the `Angular Aria` guide, which provides a comprehensive reference for using ARIA attributes correctly in Angular applications.

Source: [angular.dev/guide/accessibility](https://angular.dev/guide/accessibility)

## Style Guide

Angular's official style guide provides conventions for naming, file structure, and coding patterns that help teams maintain consistency.

Source: [angular.dev/style-guide](https://angular.dev/style-guide)

### File Naming

Follow the pattern `feature.type.ts`:

```
user-list.component.ts
user-list.component.html
user-list.component.css
user-list.component.spec.ts
user.service.ts
user.model.ts
auth.guard.ts
format-date.pipe.ts
highlight.directive.ts
```

### Folder Structure

Organize by feature, not by type:

```
src/app/
├── users/
│   ├── user-list.component.ts
│   ├── user-detail.component.ts
│   ├── user.service.ts
│   └── user.model.ts
├── auth/
│   ├── login.component.ts
│   ├── auth.service.ts
│   └── auth.guard.ts
├── shared/
│   ├── components/
│   ├── pipes/
│   └── directives/
├── app.component.ts
├── app.config.ts
└── app.routes.ts
```

### Naming Conventions

| Symbol | Naming Convention | Example |
|---|---|---|
| Components | PascalCase with `Component` suffix | `UserListComponent` |
| Services | PascalCase with `Service` suffix | `UserService` |
| Directives | PascalCase with `Directive` suffix | `HighlightDirective` |
| Pipes | PascalCase with `Pipe` suffix | `TruncatePipe` |
| Interfaces/Models | PascalCase, no `I` prefix | `User`, `CreateUserDto` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Selectors | kebab-case with a prefix | `app-user-list`, `app-highlight` |

### Coding Conventions

- Use `inject()` function instead of constructor injection (Angular 19+ recommended pattern).
- Use standalone components by default (no NgModules for new code).
- Use the new control flow syntax (`@if`, `@for`, `@switch`) instead of structural directives.
- Use signals for state management.
- Use functional guards and resolvers instead of class-based ones.
- Keep components small and focused — extract logic into services.
- Use `OnPush` change detection strategy for all components when possible.

## Error Handling

Angular provides built-in mechanisms for handling errors that occur during rendering, change detection, and within event handlers.

Source: [angular.dev/best-practices/error-handling](https://angular.dev/best-practices/error-handling)

### Global Error Handler

Provide a custom `ErrorHandler` to catch unhandled errors globally:

```typescript
import { ErrorHandler, Injectable } from '@angular/core';

@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  handleError(error: unknown): void {
    console.error('Unhandled error:', error);
    // Send to error tracking service (Sentry, DataDog, etc.)
  }
}

// Provide in app config
providers: [
  { provide: ErrorHandler, useClass: GlobalErrorHandler },
]
```

### HTTP Error Handling

Handle HTTP errors with interceptors for consistent error processing:

```typescript
import { HttpInterceptorFn } from '@angular/common/http';
import { catchError, throwError } from 'rxjs';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError(error => {
      if (error.status === 401) {
        // Redirect to login
        inject(Router).navigate(['/login']);
      }
      if (error.status === 403) {
        // Show access denied message
        inject(NotificationService).error('Access denied');
      }
      return throwError(() => error);
    }),
  );
};
```

### Error Boundaries

Angular does not have React-style error boundaries, but you can handle rendering errors by:

- Using `ErrorHandler` for global catches
- Using `try/catch` in lifecycle hooks and event handlers
- Using RxJS `catchError` in async pipes and HTTP calls

## Keeping Up to Date

Angular follows a predictable, time-based release schedule:

- **Major releases** — Every 6 months (e.g., v18 → v19 → v20)
- **Minor releases** — Every month
- **Patch releases** — Every week

### ng update

The Angular CLI's `ng update` command runs automated code transformations (schematics) that handle routine breaking changes:

```bash
# Check for available updates
ng update

# Update Angular core and CLI
ng update @angular/core @angular/cli

# Update Angular Material
ng update @angular/material
```

`ng update` automatically:

- Updates package versions in `package.json`
- Runs migration schematics that refactor your code for breaking changes
- Reports any manual changes you need to make

### Long-Term Support (LTS)

Each major Angular version receives:

- **Active support** — 6 months of regular updates
- **LTS** — 12 months of critical fixes and security patches

Source: [angular.dev/update](https://angular.dev/update)
