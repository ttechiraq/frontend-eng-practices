---
sidebar_position: 9
---

# Modern HTML features

Use modern, standardized elements before reaching for custom JavaScript components. Ensure accessibility and progressive enhancement.

## Dialog

Use `<dialog>` for modal or non-modal dialogs; it handles focus trapping and inert background.

```html
<dialog id="confirmDelete">
  <p>Delete this item?</p>
  <button type="button" onclick="confirmDelete.close()">Cancel</button>
  <button type="button">Delete</button>
</dialog>

<button type="button" onclick="confirmDelete.showModal()">Open dialog</button>
```

- Provide explicit close controls; Esc SHOULD close the dialog.
- Return focus to the trigger after close.

## Popover API

Use the popover attribute for non-modal overlays (menus, toasts, hints).

```html
<button popovertarget="menu1">Open menu</button>
<div id="menu1" popover>
  <button type="button">Action</button>
  <button type="button">Dismiss</button>
</div>
```

- `popover="auto"` is light-dismissable; use `"manual"` for controlled overlays.
- Keep popovers small and contextual; prefer `<dialog>` for modal flows.

## Details and summary

Use `<details>/<summary>` for disclosure widgets; they are keyboard-accessible by default.

```html
<details>
  <summary>More info</summary>
  <p>Additional context...</p>
</details>
```

## Template and slot

- `<template>` holds inert markup for cloning.
- `<slot>` exposes composition points in Web Components.

```html
<template id="card">
  <article class="card">
    <h3></h3>
    <p></p>
  </article>
</template>
```

## Search element

Use `<search>` to wrap search interfaces when available; otherwise use a `<form role="search">` pattern.

```html
<search>
  <form role="search">
    <label for="query">Search</label>
    <input id="query" name="q" type="search" />
    <button type="submit">Go</button>
  </form>
</search>
```

## Inert

Use `inert` to make background content unfocusable when overlays are active; pair with dialog/popover logic.

```html
<main id="page" inert>...</main>
```

## contenteditable

Use sparingly. If required:
- Set `role` and accessible name.
- Manage keyboard interactions and focus intentionally.
- Sanitize user input on save to avoid XSS.

## Progressive enhancement

- Check support for newer attributes (e.g., `popover`) and provide fallbacks.
- Prefer native features over heavy custom components for performance and accessibility.
