---
sidebar_position: 4
---

# Accessibility (a11y)

Follow WCAG 2.2 AA at minimum. Prefer native HTML semantics; add ARIA only when necessary. First rule of ARIA: **no ARIA is better than bad ARIA**.

## Core principles

- **Perceivable**: Provide text alternatives and sensible structure.
- **Operable**: Full keyboard support; visible focus styles.
- **Understandable**: Clear labels, predictable navigation, consistent patterns.
- **Robust**: Compatible with assistive technologies; use valid HTML and tested ARIA patterns.

## Landmarks and structure

- Use one `<main>` per page; add a skip link to it.
- Use `<nav>` with `aria-label` when multiple navs exist (e.g., primary vs footer nav).
- Use headings in order to expose a usable outline to screen readers.

```html
<a class="skip-link" href="#main">Skip to main content</a>
<main id="main">...</main>
```

## Accessible names

- Every interactive control MUST have an accessible name.
- Prefer visible `<label>` associated via `for` or wrapping.
- Use `aria-label` only when a visible label is not possible; prefer `aria-labelledby` to reuse visible text.

:::warning[bad]
```html
<button></button>
```
:::

:::tip[good]
```html
<button aria-label="Close dialog">✕</button>
```
:::

## ARIA roles, states, properties

- Use semantic elements first; only add roles when the native element is unsuitable.
- Keep state in sync: `aria-expanded`, `aria-pressed`, `aria-selected`, `aria-disabled`, `aria-invalid`.
- Custom widgets MUST provide keyboard and focus management equal to their native equivalents.
- Use `role="alert"` or `aria-live` for dynamic updates; choose `polite` unless interruption is required.

```html
<div role="status" aria-live="polite">Saving…</div>
```

## Keyboard support

- All functionality MUST be available via keyboard alone.
- Focus order follows DOM order; avoid CSS reordering that breaks reading order.
- Provide visible focus indicators; do not remove focus outlines without replacements.
- Escape hatch for overlays: `Esc` SHOULD close dialogs/popovers; focus MUST return to the trigger.

## Forms and errors

- Associate labels with controls; include `aria-describedby` for help/error text.
- Errors SHOULD be announced via `aria-live`.
- Group related controls with `<fieldset>` and `<legend>`.

## Media alternatives

- Images MUST have meaningful `alt`. Decorative images SHOULD use `alt=""`.
- Provide captions for `<video>` and transcripts for audio.

## Color and contrast

- Meet WCAG 2.2 contrast ratios (4.5:1 for normal text; 3:1 for large text/icons).
- Do not rely on color alone to convey state; pair with text or icons.

## Testing

- Validate HTML and ARIA in dev (e.g., axe, Lighthouse, WAVE).
- Test with keyboard only and a screen reader for critical flows.
