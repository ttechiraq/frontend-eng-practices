---
sidebar_position: 9
---

# Modern HTML features

Modern HTML provides native solutions for UI patterns that previously required significant JavaScript. Using these built-in features improves performance, reduces bundle size, and provides better accessibility out of the box. Always ensure accessibility and consider progressive enhancement for features with limited browser support.

## Dialog element

The `<dialog>` element provides a native way to create **modal** and **non-modal** dialog boxes (alerts, confirmations, forms overlays). It handles focus trapping, backdrop rendering, and making background content inert — features that custom dialog implementations frequently get wrong.

### Modal dialog

A modal dialog blocks interaction with the rest of the page. The browser automatically:
- Adds a `::backdrop` pseudo-element behind the dialog.
- Traps focus inside the dialog (Tab does not leave it).
- Makes the rest of the page inert (not clickable/focusable).
- Closes on `Esc` keypress by default.

```html
<dialog id="confirmDelete">
  <h2>Confirm deletion</h2>
  <p>Are you sure you want to delete this item? This action cannot be undone.</p>
  <form method="dialog">
    <button value="cancel">Cancel</button>
    <button value="confirm">Delete</button>
  </form>
</dialog>

<button type="button" onclick="document.getElementById('confirmDelete').showModal()">
  Delete item
</button>
```

- Call `.showModal()` to open as a modal (with backdrop and focus trap).
- Call `.show()` to open as a non-modal dialog (no backdrop, no focus trap).
- Use `<form method="dialog">` inside the dialog so that submit buttons close it automatically and set the dialog's `returnValue` to the button's `value`.
- **MUST** return focus to the triggering element after close.
- **SHOULD** provide an explicit close mechanism (button or form) in addition to `Esc`.

### Styling the backdrop

```css
dialog::backdrop {
  background-color: rgba(0, 0, 0, 0.5);
}
```

### When to use dialog vs other patterns

| Pattern | Element | Use for |
|---|---|---|
| Modal overlay (blocking) | `<dialog>` + `.showModal()` | Confirmations, form overlays, critical actions |
| Non-modal overlay | `<dialog>` + `.show()` | Persistent panels, tool windows |
| Tooltip / menu / toast | `popover` attribute | Lightweight, non-blocking overlays |

## Popover API

The [Popover API](https://developer.mozilla.org/en-US/docs/Web/API/Popover_API) (Baseline 2024) provides a standardized mechanism for non-modal overlays using HTML attributes alone — no JavaScript required for basic functionality.

### Basic usage

```html
<button popovertarget="actions-menu">Open menu</button>

<div id="actions-menu" popover>
  <ul>
    <li><button type="button">Edit</button></li>
    <li><button type="button">Duplicate</button></li>
    <li><button type="button">Delete</button></li>
  </ul>
</div>
```

The `popovertarget` attribute on the button automatically toggles the popover. No event listeners needed.

### Popover types

| Value | Behavior | Use for |
|---|---|---|
| `popover="auto"` (default) | Light-dismissable: closes when clicking outside or pressing `Esc`. Only one `auto` popover open at a time (unless nested). | Action menus, dropdowns |
| `popover="hint"` | Light-dismissable. Does not close `auto` popovers but closes other `hint` popovers. | Tooltips, contextual help |
| `popover="manual"` | Not light-dismissable. Must be explicitly shown/hidden. Multiple can be open simultaneously. | Toasts, persistent notifications |

### Styling and positioning

Popovers are placed in the **top layer** by the browser, meaning they automatically appear above all other content without needing `z-index` manipulation.

```css
[popover] {
  margin: auto;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 0.5rem;
}
```

For precise positioning relative to the trigger, use the [CSS Anchor Positioning API](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning) (emerging specification) or JavaScript-based positioning libraries.

## Details and summary

The `<details>` and `<summary>` elements create native disclosure widgets — expandable/collapsible content sections. They are keyboard-accessible by default (Space/Enter toggles) and communicate open/closed state to screen readers.

```html
<details>
  <summary>System requirements</summary>
  <ul>
    <li>Node.js 20 or later</li>
    <li>npm 10 or later</li>
    <li>macOS, Windows, or Linux</li>
  </ul>
</details>

<!-- Open by default -->
<details open>
  <summary>Release notes</summary>
  <p>Version 2.1 includes performance improvements and bug fixes.</p>
</details>
```

- The `<summary>` MUST be the first child of `<details>`. If omitted, the browser provides a default "Details" label.
- The `open` attribute controls the initial state and can be toggled programmatically.
- Use for FAQ sections, supplementary information, code samples, and any content that benefits from progressive disclosure.
- Listen for the `toggle` event to react to open/close changes.

## Template and slot

### Template

The `<template>` element holds HTML markup that is **not rendered** when the page loads. It is parsed by the browser but remains inert — scripts inside do not run, images do not load, and styles do not apply. Content is activated by cloning it with JavaScript.

```html
<template id="notification-template">
  <div class="notification" role="alert">
    <strong class="notification-title"></strong>
    <p class="notification-message"></p>
    <button type="button" class="notification-dismiss">Dismiss</button>
  </div>
</template>
```

```javascript
const template = document.getElementById('notification-template');
const clone = template.content.cloneNode(true);
clone.querySelector('.notification-title').textContent = 'Success';
clone.querySelector('.notification-message').textContent = 'Your changes were saved.';
document.body.appendChild(clone);
```

### Slot

The `<slot>` element is used inside **Shadow DOM** (a Web Components technology) to define insertion points where the consumer of a custom element can provide their own content.

```html
<!-- Custom element definition (simplified) -->
<template id="card-template">
  <div class="card">
    <slot name="title">Default title</slot>
    <slot>Default content</slot>
  </div>
</template>

<!-- Usage -->
<custom-card>
  <h3 slot="title">Card Heading</h3>
  <p>Card body content goes here.</p>
</custom-card>
```

Named slots (`<slot name="...">`) allow multiple insertion points. The default slot (no name) captures any content not assigned to a named slot.

## Search element

The `<search>` element (Baseline 2023) is a semantic landmark for search interfaces. It communicates to assistive technologies that the enclosed content is a search facility.

```html
<search>
  <form action="/search" method="get">
    <label for="site-search">Search this site</label>
    <input id="site-search" name="q" type="search" />
    <button type="submit">Search</button>
  </form>
</search>
```

- `<search>` maps to `role="search"` implicitly.
- For broader browser support, you can also add `role="search"` to a `<form>` element as a fallback.
- Use `type="search"` on the input — it provides semantic meaning and may show a clear button in some browsers.

## Inert attribute

The `inert` attribute makes an element and all its descendants **non-interactive** — they cannot be focused, clicked, or found by assistive technology. The browser removes them from the tab order and the accessibility tree.

```html
<!-- When a modal is open, mark the background as inert -->
<main id="page-content" inert>
  <h1>Main page content</h1>
  <p>This content is not interactable while the modal is open.</p>
</main>

<dialog id="modal" open>
  <p>Modal content here.</p>
</dialog>
```

- `<dialog>.showModal()` automatically makes the rest of the page inert — you do not need to set `inert` manually when using modal dialogs.
- Use `inert` manually for custom overlay patterns that do not use `<dialog>`, or for temporarily disabling sections of a page (e.g., during a loading state).
- Remember to **remove** `inert` when the overlay is dismissed.

## contenteditable

The `contenteditable` attribute makes an element's content editable by the user directly in the browser. Use it sparingly and with caution — it provides a raw editing surface without any built-in structure or validation.

If `contenteditable` is required:
- Set an appropriate ARIA role (e.g., `role="textbox"`) and provide an accessible name via `aria-label` or `aria-labelledby`.
- Manage keyboard interactions intentionally — the default behavior can vary between browsers.
- **MUST** sanitize all user-generated content on save to prevent XSS (Cross-Site Scripting) attacks. Content from `contenteditable` regions can contain arbitrary HTML.
- Consider using a structured rich-text editor library instead of raw `contenteditable` for non-trivial editing experiences.

```html
<label id="notes-label">Notes</label>
<div
  contenteditable="true"
  role="textbox"
  aria-labelledby="notes-label"
  aria-multiline="true"
></div>
```

## Progressive enhancement

Not all modern HTML features have universal browser support. Apply progressive enhancement so core functionality works everywhere, with enhanced experiences for capable browsers.

- **Feature detection** — Check for support before relying on newer APIs. For example, `HTMLElement.prototype.popover !== undefined` or `typeof HTMLDialogElement !== 'undefined'`.
- **Fallbacks** — Provide functional alternatives when a feature is unsupported. For example, a `<details>/<summary>` pattern can degrade to visible content in browsers that do not support the elements.
- **Avoid polyfill bloat** — Only polyfill features that are critical to the user experience. Many modern features (dialog, popover, details) have sufficient browser support that polyfills are unnecessary for most audiences.
- Prefer native HTML features over heavy JavaScript custom components for better performance, smaller bundle sizes, and built-in accessibility.
