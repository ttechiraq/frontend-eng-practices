---
sidebar_position: 4
---

# Templates

In Angular, a template is a chunk of HTML that defines the DOM a component renders onto the page. Templates use special syntax to leverage Angular's data binding, event handling, control flow, and more.

Source: [angular.dev/guide/templates](https://angular.dev/guide/templates)

## How Templates Work

Templates are based on HTML syntax, with additional features such as built-in template functions, data binding, event listening, variables, and more.

Angular compiles templates into JavaScript to build an internal understanding of your application. One of the benefits is built-in rendering optimizations that Angular applies automatically.

### Differences from Standard HTML

- Angular may add comment nodes to a page as placeholders for dynamic content.
- Angular ignores and collapses unnecessary whitespace characters.
- The `@` character has special meaning for adding dynamic behavior (control flow). Use `&commat;` or `&#64;` for a literal `@`.
- Attributes with `[]`, `()`, etc. have special binding meaning.
- Component and directive elements can be self-closed (e.g., `<profile-photo />`).

## Binding Dynamic Text, Properties, and Attributes

### Text Interpolation

Render dynamic text in a template with double curly braces:

```html
<p>Hello, {{ userName }}!</p>
<p>Total: {{ price * quantity }}</p>
```

### Property Binding

Bind a DOM property to a component value using square brackets:

```html
<img [src]="imageUrl" [alt]="imageDescription" />
<button [disabled]="isFormInvalid">Submit</button>
```

### Attribute Binding

For HTML attributes that don't map to DOM properties, use `[attr.name]`:

```html
<td [attr.colspan]="tableSpan">Content</td>
<div [attr.role]="accessibleRole">...</div>
```

### Class and Style Binding

```html
<!-- Single class toggle -->
<div [class.active]="isActive">...</div>

<!-- Multiple classes from an object -->
<div [class]="{ active: isActive, disabled: isDisabled }">...</div>

<!-- Single style -->
<div [style.color]="textColor">...</div>

<!-- Style with unit -->
<div [style.width.px]="containerWidth">...</div>
```

Source: [angular.dev/guide/templates/binding](https://angular.dev/guide/templates/binding)

## Adding Event Listeners

Bind event handlers using parentheses:

```html
<button (click)="onSave()">Save</button>
<input (keyup.enter)="onSearch()" />
<input (input)="onInputChange($event)" />
```

The `$event` variable gives you access to the raw DOM event object:

```typescript
onInputChange(event: Event) {
  const input = event.target as HTMLInputElement;
  console.log(input.value);
}
```

Source: [angular.dev/guide/templates/event-listeners](https://angular.dev/guide/templates/event-listeners)

## Two-Way Binding

Two-way binding simultaneously binds a value and propagates changes. Angular uses the banana-in-a-box syntax `[()]`:

```html
<input [(ngModel)]="userName" />
```

For custom components, two-way binding works with a matching pair of input and output:

```typescript
@Component({
  selector: 'custom-slider',
  template: `...`,
})
export class CustomSlider {
  value = model(0); // Creates a two-way bindable signal
}
```

Usage:

```html
<custom-slider [(value)]="volume" />
```

Source: [angular.dev/guide/templates/two-way-binding](https://angular.dev/guide/templates/two-way-binding)

## Control Flow

Angular 19 uses a built-in control flow syntax with `@` blocks. These replace the older `*ngIf`, `*ngFor`, and `*ngSwitch` structural directives.

### Conditional Rendering with @if

```html
@if (user.isAdmin) {
  <admin-dashboard />
} @else if (user.isEditor) {
  <editor-dashboard />
} @else {
  <viewer-dashboard />
}
```

### Repeating Content with @for

```html
@for (item of items; track item.id) {
  <div class="item">{{ item.name }}</div>
} @empty {
  <p>No items found.</p>
}
```

The `track` expression is **required** and helps Angular identify items for efficient DOM updates. Use a unique identifier like `item.id`.

Contextual variables available inside `@for`:

| Variable | Description |
|---|---|
| `$index` | Index of the current item |
| `$first` | Whether the current item is the first |
| `$last` | Whether the current item is the last |
| `$even` | Whether the index is even |
| `$odd` | Whether the index is odd |
| `$count` | Total number of items in the collection |

### Selection with @switch

```html
@switch (userRole) {
  @case ('admin') {
    <admin-panel />
  }
  @case ('editor') {
    <editor-panel />
  }
  @default {
    <viewer-panel />
  }
}
```

Source: [angular.dev/guide/templates/control-flow](https://angular.dev/guide/templates/control-flow)

## Pipes

Pipes transform data declaratively in templates. Angular includes several built-in pipes:

```html
<p>{{ birthday | date:'longDate' }}</p>
<p>{{ price | currency:'USD' }}</p>
<p>{{ title | uppercase }}</p>
<p>{{ data | json }}</p>
```

### Built-in Pipes

| Pipe | Purpose |
|---|---|
| `DatePipe` | Formats dates |
| `CurrencyPipe` | Formats currency values |
| `DecimalPipe` | Formats decimal numbers |
| `PercentPipe` | Formats percentages |
| `UpperCasePipe` / `LowerCasePipe` | Case transforms |
| `JsonPipe` | Serializes to JSON (useful for debugging) |
| `AsyncPipe` | Subscribes to Observables/Promises |
| `SlicePipe` | Extracts a subset of an array or string |
| `KeyValuePipe` | Transforms objects/maps for iteration |

### Custom Pipes

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({ name: 'truncate' })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number = 50): string {
    return value.length > limit ? value.substring(0, limit) + '...' : value;
  }
}
```

Usage:

```html
<p>{{ description | truncate:100 }}</p>
```

### Pipe Chaining

Pipes can be chained:

```html
<p>{{ birthday | date:'shortDate' | uppercase }}</p>
```

Source: [angular.dev/guide/templates/pipes](https://angular.dev/guide/templates/pipes)

## Slotting Child Content with ng-content

`<ng-content>` is a placeholder element for rendering child content provided by a parent component. See the [Components](./2-components.md#content-projection-with-ng-content) chapter for details.

## Create Template Fragments with ng-template

`<ng-template>` defines a template fragment that is not rendered by default. It can be rendered programmatically or by structural directives:

```html
<ng-template #loading>
  <p>Loading...</p>
</ng-template>

@if (isLoading) {
  <ng-container *ngTemplateOutlet="loading" />
}
```

Source: [angular.dev/guide/templates/ng-template](https://angular.dev/guide/templates/ng-template)

## Grouping Elements with ng-container

`<ng-container>` is a grouping element that doesn't render a DOM element. Use it to apply structural logic without adding extra elements to the DOM:

```html
<ng-container *ngIf="showSection">
  <h2>Section Title</h2>
  <p>Section content</p>
</ng-container>
```

With the new control flow, `ng-container` is less frequently needed but still useful for template outlet rendering and grouping.

Source: [angular.dev/guide/templates/ng-container](https://angular.dev/guide/templates/ng-container)

## Template Variables

Declare variables in templates with `@let`:

```html
@let fullName = firstName + ' ' + lastName;
<p>Welcome, {{ fullName }}!</p>

@let userStream = user$ | async;
@if (userStream) {
  <p>{{ userStream.name }}</p>
}
```

Template reference variables (`#`) create references to DOM elements or component instances:

```html
<input #searchInput />
<button (click)="search(searchInput.value)">Search</button>
```

Source: [angular.dev/guide/templates/variables](https://angular.dev/guide/templates/variables)

## Deferred Loading with @defer

`@defer` creates lazily loaded sections of your template. The deferred content is not loaded until a trigger condition is met, reducing the initial bundle size:

```html
@defer (on viewport) {
  <heavy-chart-component />
} @placeholder {
  <p>Chart will load when scrolled into view</p>
} @loading (minimum 500ms) {
  <p>Loading chart...</p>
} @error {
  <p>Failed to load chart.</p>
}
```

### Trigger Conditions

| Trigger | Description |
|---|---|
| `on viewport` | Load when the placeholder enters the viewport |
| `on interaction` | Load when the user interacts with the placeholder |
| `on hover` | Load when the user hovers over the placeholder |
| `on idle` | Load when the browser is idle |
| `on timer(5s)` | Load after a specified delay |
| `on immediate` | Load immediately after non-deferred content |
| `when condition` | Load when a boolean expression becomes true |

### Sub-blocks

- `@placeholder` — Shown before the deferred content starts loading
- `@loading` — Shown while the deferred content is loading
- `@error` — Shown if loading fails

```html
@defer (on interaction; prefetch on idle) {
  <comments-section [postId]="post.id" />
} @placeholder {
  <button>Load Comments</button>
} @loading (after 100ms; minimum 500ms) {
  <spinner />
}
```

Source: [angular.dev/guide/templates/defer](https://angular.dev/guide/templates/defer)
