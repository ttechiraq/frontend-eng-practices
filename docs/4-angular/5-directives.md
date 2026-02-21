---
sidebar_position: 5
---

# Directives

Angular directives are classes that add behavior to elements in your Angular applications. There are three kinds of directives: components (directives with templates), attribute directives, and structural directives.

Source: [angular.dev/guide/directives](https://angular.dev/guide/directives)

## Types of Directives

| Type | Description | Example |
|---|---|---|
| Components | Directives with a template | `@Component({ ... })` |
| Attribute directives | Change the appearance or behavior of an element | `NgClass`, `NgStyle` |
| Structural directives | Change the DOM layout by adding/removing elements | `@if`, `@for`, `@switch` (built-in control flow) |

## Built-in Attribute Directives

### NgClass

Adds and removes CSS classes on an element:

```html
<!-- Toggle a single class -->
<div [ngClass]="'active'">...</div>

<!-- Conditional classes with an object -->
<div [ngClass]="{ active: isActive, disabled: isDisabled }">...</div>

<!-- Array of class names -->
<div [ngClass]="['bordered', 'rounded']">...</div>
```

:::info
With Angular 19's class binding, you can often use `[class]` directly instead of `NgClass`:

```html
<div [class.active]="isActive">...</div>
```

:::

### NgStyle

Sets inline styles dynamically:

```html
<div [ngStyle]="{ 'font-size.px': fontSize, color: fontColor }">...</div>
```

With Angular 19's style binding, you can often use `[style]` directly:

```html
<div [style.font-size.px]="fontSize" [style.color]="fontColor">...</div>
```

### NgModel

Adds two-way data binding to a form element. Requires importing `FormsModule`:

```typescript
import { FormsModule } from '@angular/forms';

@Component({
  imports: [FormsModule],
  template: `<input [(ngModel)]="name" />`,
})
export class NameEditor {
  name = '';
}
```

## Custom Attribute Directives

Create your own attribute directives by decorating a class with `@Directive`:

```typescript
import { Directive, ElementRef, inject } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
})
export class HighlightDirective {
  private el = inject(ElementRef);

  constructor() {
    this.el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

Usage:

```html
<p appHighlight>This text is highlighted.</p>
```

### Responding to Events

Use `host` to listen for events on the host element:

```typescript
@Directive({
  selector: '[appHighlight]',
  host: {
    '(mouseenter)': 'onMouseEnter()',
    '(mouseleave)': 'onMouseLeave()',
  },
})
export class HighlightDirective {
  private el = inject(ElementRef);
  color = input('yellow');

  onMouseEnter() {
    this.el.nativeElement.style.backgroundColor = this.color();
  }

  onMouseLeave() {
    this.el.nativeElement.style.backgroundColor = '';
  }
}
```

### Directive with Inputs

```html
<p appHighlight="cyan">Hover to highlight in cyan.</p>
```

## Structural Directives

Structural directives modify the DOM structure by adding, removing, or manipulating elements. In Angular 19, the built-in control flow (`@if`, `@for`, `@switch`) replaces the older `*ngIf`, `*ngFor`, and `*ngSwitch` structural directives.

### Legacy Structural Directives

For reference, the legacy structural directives are still available:

```html
<!-- *ngIf (replaced by @if) -->
<div *ngIf="isVisible">Content</div>

<!-- *ngFor (replaced by @for) -->
<div *ngFor="let item of items; trackBy: trackById">{{ item.name }}</div>

<!-- *ngSwitch (replaced by @switch) -->
<div [ngSwitch]="role">
  <p *ngSwitchCase="'admin'">Admin</p>
  <p *ngSwitchCase="'editor'">Editor</p>
  <p *ngSwitchDefault>Viewer</p>
</div>
```

### Custom Structural Directives

You can create custom structural directives using `TemplateRef` and `ViewContainerRef`:

```typescript
import { Directive, TemplateRef, ViewContainerRef, inject, input, effect } from '@angular/core';

@Directive({
  selector: '[appUnless]',
})
export class UnlessDirective {
  private templateRef = inject(TemplateRef);
  private viewContainer = inject(ViewContainerRef);

  appUnless = input.required<boolean>();

  constructor() {
    effect(() => {
      if (!this.appUnless()) {
        this.viewContainer.createEmbeddedView(this.templateRef);
      } else {
        this.viewContainer.clear();
      }
    });
  }
}
```

Usage:

```html
<p *appUnless="isHidden">This is shown when isHidden is false.</p>
```

## Host Directives

The `hostDirectives` property allows a component or directive to apply other directives to its host element. This enables directive composition:

```typescript
@Component({
  selector: 'admin-menu',
  hostDirectives: [MenuBehavior, Tooltip],
  template: `...`,
})
export class AdminMenu {}
```

When Angular creates `AdminMenu`, it also creates instances of `MenuBehavior` and `Tooltip` on the host element.

### Forwarding Inputs and Outputs

You can expose inputs and outputs from host directives:

```typescript
@Component({
  selector: 'admin-menu',
  hostDirectives: [
    {
      directive: Tooltip,
      inputs: ['tooltipText: adminTooltip'],
      outputs: ['tooltipVisibility'],
    },
  ],
  template: `...`,
})
export class AdminMenu {}
```

Usage:

```html
<admin-menu [adminTooltip]="'Edit settings'" (tooltipVisibility)="onTooltipChange($event)" />
```

Source: [angular.dev/guide/directives](https://angular.dev/guide/directives)
