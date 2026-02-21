---
sidebar_position: 2
---

# Components

Every Angular component must have three elements: a CSS selector that defines how the component is used in HTML, an HTML template that controls what renders into the DOM, and a TypeScript class with behaviors such as handling user input and fetching data from a server.

Source: [angular.dev/guide/components](https://angular.dev/guide/components)

## Anatomy of a Component

You provide Angular-specific information for a component by adding a `@Component` decorator on top of the TypeScript class:

```typescript
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo" />`,
})
export class ProfilePhoto {}
```

The object passed to the `@Component` decorator is called the component's **metadata**. This includes the `selector`, `template`, and other properties.

### Styles

Components can optionally include a list of CSS styles that apply to that component's DOM. By default, a component's styles only affect elements defined in that component's template (view encapsulation).

```typescript
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo" />`,
  styles: `
    img {
      border-radius: 50%;
    }
  `,
})
export class ProfilePhoto {}
```

### Separate Files

You can choose to write your template and styles in separate files:

```typescript
@Component({
  selector: 'profile-photo',
  templateUrl: 'profile-photo.html',
  styleUrl: 'profile-photo.css',
})
export class ProfilePhoto {}
```

Both `templateUrl` and `styleUrl` are relative to the directory in which the component resides.

## Using Components

### Imports in the @Component Decorator

To use a component, directive, or pipe, you must add it to the `imports` array in the `@Component` decorator:

```typescript
import { ProfilePhoto } from './profile-photo';

@Component({
  imports: [ProfilePhoto],
  template: `
    <profile-photo />
    <p>Welcome to your profile!</p>
  `,
})
export class UserProfile {}
```

By default, Angular components are standalone (as of Angular 19), meaning you can directly add them to the `imports` array of other components.

### Showing Components in a Template

Every component defines a CSS selector. You show a component by creating a matching HTML element in the template of other components:

```typescript
@Component({
  selector: 'profile-photo',
  template: `...`,
})
export class ProfilePhoto {}

@Component({
  imports: [ProfilePhoto],
  template: `<profile-photo />`,
})
export class UserProfile {}
```

Angular creates an instance of the component for every matching HTML element it encounters. The DOM element that matches a component's selector is referred to as that component's **host element**.

## Selectors

Angular supports several types of CSS selectors for components:

| Selector type | Example | Usage |
|---|---|---|
| Type selector | `selector: 'profile-photo'` | Matches by element name |
| Attribute selector | `selector: '[dropzone]'` | Matches by attribute presence |
| Class selector | `selector: '.menu-item'` | Matches by CSS class |

Source: [angular.dev/guide/components/selectors](https://angular.dev/guide/components/selectors)

## Accepting Data with Input Properties

When using a component, you commonly pass data to it through input properties. Define inputs using the `input` function:

```typescript
import { Component, input } from '@angular/core';

@Component({
  selector: 'user-card',
  template: `<p>User: {{ name() }}</p>`,
})
export class UserCard {
  name = input<string>();          // Optional input
  role = input('viewer');          // Input with default value
  id = input.required<number>();   // Required input
}
```

Usage in a parent template:

```html
<user-card [name]="'Alice'" [id]="42" />
```

### Input Transforms

You can transform input values by providing a `transform` function:

```typescript
@Component({ ... })
export class CustomSlider {
  label = input('', { transform: trimString });
}

function trimString(value: string | undefined) {
  return value?.trim() ?? '';
}
```

### Input Aliases

You can alias an input to a different public name:

```typescript
@Component({ ... })
export class CustomSlider {
  value = input(0, { alias: 'sliderValue' });
}
```

Source: [angular.dev/guide/components/inputs](https://angular.dev/guide/components/inputs)

## Custom Events with Outputs

Components can emit custom events using the `output` function:

```typescript
import { Component, output } from '@angular/core';

@Component({
  selector: 'expandable-panel',
  template: `<button (click)="onToggle()">Toggle</button>`,
})
export class ExpandablePanel {
  panelClosed = output<void>();

  onToggle() {
    this.panelClosed.emit();
  }
}
```

Listening to outputs in a parent template:

```html
<expandable-panel (panelClosed)="onPanelClosed()" />
```

### Output with Data

```typescript
export class ExpandablePanel {
  panelClosed = output<string>();

  close() {
    this.panelClosed.emit('panel closed by user');
  }
}
```

Source: [angular.dev/guide/components/outputs](https://angular.dev/guide/components/outputs)

## Styling

Angular provides several mechanisms for component styling:

### View Encapsulation

By default, Angular scopes styles to the component's view. This means styles defined in a component only apply to elements in that component's template. Angular achieves this by adding unique attributes to the component's DOM elements.

### Special Selectors

- `:host` — Targets the component's host element itself
- `:host-context()` — Applies styles based on ancestor conditions
- `::ng-deep` — Disables view encapsulation for a rule (use sparingly, deprecated)

```css
:host {
  display: block;
  border: 1px solid #ccc;
}

:host(.active) {
  border-color: blue;
}
```

Source: [angular.dev/guide/components/styling](https://angular.dev/guide/components/styling)

## Lifecycle

Angular components have a lifecycle managed by the framework. Angular provides lifecycle hooks that let you run code at specific moments:

| Hook | Timing |
|---|---|
| `constructor` | When Angular instantiates the component |
| `ngOnInit` | After Angular sets all inputs for the first time |
| `ngOnChanges` | Every time inputs change (before `ngOnInit` as well) |
| `ngDoCheck` | Every change detection cycle |
| `ngAfterViewInit` | After the component's view (and child views) are initialized |
| `ngAfterViewChecked` | After every check of the component's view |
| `ngAfterContentInit` | After projected content is initialized |
| `ngAfterContentChecked` | After every check of projected content |
| `ngOnDestroy` | Before the component is destroyed |

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';

@Component({ selector: 'user-dashboard', template: `...` })
export class UserDashboard implements OnInit, OnDestroy {
  ngOnInit() {
    // Fetch initial data, set up subscriptions
  }

  ngOnDestroy() {
    // Clean up subscriptions, timers
  }
}
```

### afterRender and afterNextRender

Angular also provides `afterRender` and `afterNextRender` for running code after the DOM has been rendered. These are useful for DOM manipulation and third-party library integration:

```typescript
import { Component, afterNextRender, ElementRef } from '@angular/core';

@Component({ ... })
export class ChartComponent {
  constructor(elementRef: ElementRef) {
    afterNextRender(() => {
      // Safe to access DOM here
      initChart(elementRef.nativeElement);
    });
  }
}
```

Source: [angular.dev/guide/components/lifecycle](https://angular.dev/guide/components/lifecycle)

## Content Projection with ng-content

Content projection allows a component to accept and render content provided by its parent. Use the `<ng-content>` element as a placeholder:

```typescript
@Component({
  selector: 'custom-card',
  template: `
    <div class="card">
      <div class="card-header">
        <ng-content select="[card-title]" />
      </div>
      <div class="card-body">
        <ng-content />
      </div>
    </div>
  `,
})
export class CustomCard {}
```

Usage:

```html
<custom-card>
  <h2 card-title>Card Title</h2>
  <p>This goes into the default slot.</p>
</custom-card>
```

The `select` attribute on `<ng-content>` uses CSS selectors to determine which projected content goes where. An `<ng-content>` without a `select` attribute receives all projected content that doesn't match other selectors.

Source: [angular.dev/guide/components/content-projection](https://angular.dev/guide/components/content-projection)

## Referencing Component Children with Queries

Angular provides query functions to access child elements, components, or directives:

### View Queries

- `viewChild()` — Finds a single element or component in the component's own template
- `viewChildren()` — Finds all matching elements or components

```typescript
import { Component, viewChild, ElementRef } from '@angular/core';

@Component({
  template: `<input #nameInput />`,
})
export class SearchComponent {
  nameInput = viewChild<ElementRef>('nameInput');

  focusInput() {
    this.nameInput()?.nativeElement.focus();
  }
}
```

### Content Queries

- `contentChild()` — Finds a single projected content element
- `contentChildren()` — Finds all matching projected content elements

```typescript
import { Component, contentChildren } from '@angular/core';

@Component({
  selector: 'tab-group',
  template: `...`,
})
export class TabGroup {
  tabs = contentChildren(Tab);
}
```

Source: [angular.dev/guide/components/queries](https://angular.dev/guide/components/queries)

## Host Elements

Angular creates an instance of a component for every matching HTML element. The DOM element that matches a component's selector is that component's host element.

### Binding to the Host Element

A component can bind properties, attributes, and events on its host element using the `host` property in the component metadata:

```typescript
@Component({
  selector: 'profile-photo',
  template: `...`,
  host: {
    'role': 'img',
    '[attr.aria-label]': 'photoLabel()',
    '(click)': 'onHostClick()',
  },
})
export class ProfilePhoto {
  photoLabel = input('Profile photo');

  onHostClick() {
    // Handle click on the host element
  }
}
```

Source: [angular.dev/guide/components/host-elements](https://angular.dev/guide/components/host-elements)

## Programmatic Rendering

In most cases, you render components via templates. However, Angular also supports creating components dynamically at runtime using `ViewContainerRef`:

```typescript
import { Component, ViewContainerRef, inject } from '@angular/core';
import { DynamicComponent } from './dynamic.component';

@Component({ ... })
export class AppComponent {
  private vcr = inject(ViewContainerRef);

  loadComponent() {
    this.vcr.clear();
    const ref = this.vcr.createComponent(DynamicComponent);
    ref.instance.someInput = 'Hello';
  }
}
```

Source: [angular.dev/guide/components/programmatic-rendering](https://angular.dev/guide/components/programmatic-rendering)
