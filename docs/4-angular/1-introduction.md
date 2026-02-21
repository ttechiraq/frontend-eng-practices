---
sidebar_position: 1
---

# What is Angular?

Angular is a web framework that empowers developers to build fast, reliable applications. Maintained by a dedicated team at Google, Angular provides a broad suite of tools, APIs, and libraries to simplify and streamline your development workflow. Angular gives you a solid platform on which to build fast, reliable applications that scale with both the size of your team and the size of your codebase.

Source: [angular.dev/overview](https://angular.dev/overview)

## Key Features

### Components

Angular components make it easy to split your code into well-encapsulated parts. Every component has a TypeScript class, an HTML template, and optional CSS styles.

### Angular Signals

Angular's fine-grained reactivity model, combined with compile-time optimizations, simplifies development and helps build faster apps by default.

### Server-Side Rendering

Angular supports both server-side rendering (SSR) and static site generation (SSG) along with full DOM hydration.

### Dependency Injection

Angular's DI system allows you to easily share code across components throughout your entire application.

### Routing

Angular Router provides a feature-rich navigation toolkit, including support for route guards, data resolution, lazy-loading, and much more.

### Forms

Angular provides a standardized system for form participation and validation, including both reactive forms and template-driven forms.

## Installation

### Prerequisites

- [Node.js](https://nodejs.org/) — v18.19.1 or newer
- Text editor — [VS Code](https://code.visualstudio.com/) recommended
- Terminal

### Creating a New Project

To create a new Angular project, run the following command in your terminal:

```bash
ng new my-app
```

To create a project with server-side rendering (SSR) enabled:

```bash
ng new my-app --ssr
```

The Angular CLI prompts you for configuration options such as stylesheet format and SSR. After setup, navigate into the project and start the development server:

```bash
cd my-app
ng serve
```

Your application will be running at `http://localhost:4200`.

### Adding Angular to an Existing Project

If you have an existing project, you can add individual Angular packages via npm:

```bash
npm install @angular/core @angular/common @angular/compiler @angular/platform-browser
```

## Project Structure

A typical Angular CLI project contains:

```
my-app/
├── src/
│   ├── app/
│   │   ├── app.component.ts      # Root component
│   │   ├── app.component.html    # Root template
│   │   ├── app.component.css     # Root styles
│   │   ├── app.config.ts         # Application configuration
│   │   └── app.routes.ts         # Route definitions
│   ├── index.html                # Main HTML page
│   ├── main.ts                   # Application entry point
│   └── styles.css                # Global styles
├── angular.json                  # CLI configuration
├── package.json                  # Dependencies
└── tsconfig.json                 # TypeScript configuration
```

## Angular 19 Highlights

Angular 19 introduced several significant changes and stabilizations:

### Standalone Components by Default

As of Angular 19, the `standalone` option defaults to `true`. Components, directives, and pipes no longer need `standalone: true` in their decorator — they are standalone by default. Components created with earlier versions may specify `standalone: false` for backwards compatibility.

```typescript
// Angular 19+: standalone by default
@Component({
  selector: 'app-profile',
  template: `<h1>{{ name }}</h1>`,
})
export class ProfileComponent {
  name = 'Angular';
}
```

### Signals Stabilized

Core reactivity primitives reached stable status. New additions include:

- **`linkedSignal`** — for dependent/derived writable state
- **`resource` API** — for async reactivity (integrating async data into signals)

### New Control Flow

The built-in control flow syntax (`@if`, `@for`, `@switch`) replaced the legacy `*ngIf`, `*ngFor`, and `*ngSwitch` structural directives with a cleaner, more performant template syntax.

### Incremental Hydration

Server-side rendered applications can now hydrate incrementally, deferring hydration of specific sections until they are needed — improving initial load performance.

### Vitest as Default Test Runner

New Angular CLI projects use [Vitest](https://vitest.dev/) as the default test runner, replacing Karma. Vitest runs tests in Node.js with `jsdom` for DOM simulation, providing faster test execution.

### Zoneless Change Detection

Angular 19 introduced experimental zoneless change detection, removing the ZoneJS dependency and triggering change detection only when signals or events indicate a change. This becomes the default in Angular v21+.

### Quality of Life Improvements

- HMR (Hot Module Replacement) for styles
- Schematics for adopting best practices (e.g., `inject`-based DI)
- Removal of unused imports
- Time picker component in Angular Material

## Essentials

Before diving into the in-depth guides, you should be familiar with these concepts:

- [TypeScript Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
- [TypeScript fundamentals](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
- [JavaScript Classes](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Classes)

The following chapters cover each Angular concept in depth, starting with [Components](./2-components.md).
