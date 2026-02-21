---
sidebar_position: 4
---

# Accessibility (a11y)

Accessibility — abbreviated **a11y** (a + 11 letters + y) — means making web content usable by everyone, including people with visual, auditory, motor, or cognitive disabilities. It is both a legal requirement in many jurisdictions and a fundamental quality attribute of professional web development.

Follow [WCAG 2.2](https://www.w3.org/TR/WCAG22/) (Web Content Accessibility Guidelines) **AA** conformance at minimum. Prefer native HTML semantics; add [ARIA](https://www.w3.org/WAI/ARIA/apg/) (Accessible Rich Internet Applications) only when necessary.

The first rule of ARIA: **no ARIA is better than bad ARIA**. A native `<button>` is inherently accessible; a `<div role="button">` requires you to manually implement keyboard handling, focus management, and state communication — and mistakes are common.

## Core principles (POUR)

WCAG is organized around four principles, commonly referred to by the acronym **POUR**:

- **Perceivable** — Information must be presentable in ways all users can perceive. Provide text alternatives for images, captions for video, and sufficient color contrast.
- **Operable** — UI components must be operable by all users. Support full keyboard navigation, provide visible focus indicators, and allow enough time for interactions.
- **Understandable** — Content and interfaces must be understandable. Use clear labels, predictable navigation patterns, and helpful error messages.
- **Robust** — Content must be robust enough to work with current and future assistive technologies. Use valid HTML, tested ARIA patterns, and standard web platform features.

## WCAG 2.2 conformance levels

WCAG defines three conformance levels, each building on the previous:

| Level | Description | Requirement |
|---|---|---|
| **A** | Minimum accessibility; removes the most critical barriers. | MUST meet for any public-facing content. |
| **AA** | Addresses the most common barriers for the widest range of disabilities. | SHOULD be the standard target for all projects. |
| **AAA** | Highest level; may not be achievable for all content types. | MAY target for specific content where feasible. |

## Landmarks and structure

Landmarks allow screen reader users to jump directly to major page sections without reading through every element.

- Use one `<main>` per page for the primary content area.
- Provide a **skip navigation link** as the first focusable element so keyboard users can bypass repetitive header content.
- Use `<nav>` for navigation blocks. When multiple `<nav>` elements exist on one page, distinguish them with `aria-label` (e.g., `aria-label="Primary navigation"`, `aria-label="Footer navigation"`).
- Use headings in sequential order to expose a usable outline to screen readers.

```html
<a class="skip-link" href="#main">Skip to main content</a>

<header>
  <nav aria-label="Primary navigation">...</nav>
</header>

<main id="main">
  <h1>Page Title</h1>
  ...
</main>

<footer>
  <nav aria-label="Footer navigation">...</nav>
</footer>
```

## Accessible names

Every interactive element (buttons, links, inputs, selects) MUST have an **accessible name** — the text that assistive technologies announce to identify the control.

The accessible name computation follows a priority order:
1. `aria-labelledby` — references visible text elsewhere on the page.
2. `aria-label` — provides a name when visible text is not available.
3. Native labeling — `<label>` for inputs, text content for buttons/links.
4. `title` attribute — lowest priority; generally avoid as the sole name.

- Prefer visible `<label>` elements associated via `for`/`id` or by wrapping the control.
- Use `aria-label` only when a visible label is not possible (e.g., icon-only buttons).
- Use `aria-labelledby` to compose a name from multiple visible text elements.
- Use `aria-describedby` for supplementary information (help text, error messages) that should not be the primary name.

:::warning[bad]

```html
<!-- No accessible name — screen reader announces "button" with no context -->
<button><svg>...</svg></button>

<!-- Placeholder is not a substitute for a label -->
<input type="email" placeholder="Enter your email">
```

:::

:::tip[good]

```html
<!-- Icon button with aria-label -->
<button aria-label="Close dialog">
  <svg aria-hidden="true">...</svg>
</button>

<!-- Explicit label association -->
<label for="user-email">Email address</label>
<input id="user-email" type="email" name="email" autocomplete="email" />
```

:::

## ARIA roles, states, and properties

ARIA attributes extend native semantics for custom widgets that have no native HTML equivalent.

### Roles

Roles define what an element **is**. Common roles include `dialog`, `tablist`, `tab`, `tabpanel`, `alert`, `status`, `combobox`, `menu`, `menuitem`, `tree`, and `treeitem`. Use semantic HTML elements first — only add a role when no native element provides the needed semantics.

### States and properties

States communicate the **current condition** of a widget. Keep them synchronized with the actual UI state:

| Attribute | Purpose | Example use |
|---|---|---|
| `aria-expanded` | Whether a collapsible section is open | Accordion, dropdown menu |
| `aria-pressed` | Toggle button state | Bold/italic toolbar buttons |
| `aria-selected` | Selected item in a set | Tab in a tablist, option in a listbox |
| `aria-disabled` | Control is not currently interactive | Disabled form submit button |
| `aria-invalid` | Input value does not pass validation | Form field with an error |
| `aria-hidden` | Element is hidden from assistive technology | Decorative icons, duplicate content |
| `aria-live` | Region whose content updates dynamically | Chat messages, status notifications |

### Live regions

Use `aria-live` to announce dynamic content changes to screen readers without requiring focus to move:

- `aria-live="polite"` — Announces updates at the next convenient pause. Use for non-urgent status messages (e.g., "Saved", "3 results found").
- `aria-live="assertive"` — Interrupts the current announcement. Reserve for urgent messages (e.g., session timeout warnings, critical errors).
- `role="alert"` — Implicitly assertive; use for important notifications.
- `role="status"` — Implicitly polite; use for routine status updates.

```html
<!-- Polite status update -->
<div role="status" aria-live="polite">3 items in your cart.</div>

<!-- Urgent alert -->
<div role="alert">Your session will expire in 2 minutes.</div>
```

## Keyboard support

All functionality MUST be available via keyboard alone. Many users cannot use a mouse due to motor impairments, and screen reader users navigate primarily by keyboard.

- **Focus order** follows the DOM (Document Object Model) order. Avoid CSS reordering (`order`, `flex-direction: row-reverse`) that creates visual/DOM mismatches.
- **Visible focus indicators** MUST be present on all focusable elements. Do not set `outline: none` without providing an equivalent or better alternative.
- **Custom widgets** MUST implement keyboard patterns matching their native counterparts. For example, a custom dropdown should support Arrow keys for navigation, Enter/Space for selection, and Escape to close.
- **Focus management** — When opening a dialog, move focus into it. When closing, return focus to the element that triggered it. Use `tabindex="-1"` to make non-interactive elements programmatically focusable when needed.
- `tabindex="0"` adds an element to the natural tab order. `tabindex="-1"` removes it from tab order but allows programmatic focus. Avoid positive `tabindex` values — they override the natural DOM order and create maintenance problems.

## Forms and errors

Accessible forms ensure all users can complete tasks independently:

- Associate every input with a `<label>` via `for`/`id` pairing.
- Link help text and error messages to inputs with `aria-describedby`.
- Announce form-level or field-level errors via `aria-live` regions.
- Group related inputs with `<fieldset>` and `<legend>` (e.g., "Shipping Address", radio button groups).

See the [forms chapter](./5-forms.md) for detailed rules.

## Media alternatives

- Images MUST have meaningful `alt` text that conveys the image's purpose or content.
- Decorative images (purely visual, no informational content) SHOULD use `alt=""` so screen readers skip them entirely.
- `<video>` MUST include captions via `<track kind="captions">` for deaf and hard-of-hearing users.
- Audio content SHOULD have a text transcript available.
- Complex images (charts, diagrams) SHOULD have extended descriptions via `aria-describedby` linking to nearby descriptive text.

## Color and contrast

Color alone MUST NOT be the only means of conveying information, state, or distinguishing UI elements. Always pair color with text labels, icons, patterns, or underlines.

Minimum contrast ratios per WCAG 2.2:

| Content type | Minimum ratio |
|---|---|
| Normal text (< 18pt / < 14pt bold) | 4.5:1 |
| Large text (≥ 18pt / ≥ 14pt bold) | 3:1 |
| UI components and graphical objects | 3:1 |

## Testing

Accessibility testing SHOULD be integrated into the development workflow, not deferred to a final audit:

- **Automated tools** — Run [axe](https://www.deque.com/axe/), [Lighthouse](https://developer.chrome.com/docs/lighthouse/), or [WAVE](https://wave.webaim.org/) during development to catch common violations (missing alt text, color contrast, invalid ARIA).
- **Keyboard testing** — Tab through every interactive flow using only the keyboard. Verify focus order, visibility, and that all actions are reachable.
- **Screen reader testing** — Test critical user flows with at least one screen reader (VoiceOver on macOS, NVDA on Windows) to verify announcements and navigation.
- **Manual review** — Check heading hierarchy, landmark structure, and form label associations in the browser accessibility tree (DevTools > Accessibility panel).
