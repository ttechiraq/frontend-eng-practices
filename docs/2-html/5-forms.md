---
sidebar_position: 5
---

# Forms and validation

Forms are the primary mechanism for user input on the web. Prefer native HTML form controls before building custom widgets — native controls provide built-in keyboard support, screen reader compatibility, mobile-optimized keyboards, and browser autofill integration at no additional development cost.

Always pair controls with labels and provide both client-side and server-side validation.

## Basics

- **MUST** use `<label>` for every form control. Associate labels either explicitly via `for`/`id` or implicitly by wrapping the control inside the `<label>` element. Placeholders are NOT a substitute for labels — they disappear on input and are often poorly announced by assistive technologies.
- **SHOULD** use semantic input types to leverage native browser features. For example, `type="email"` triggers email-optimized virtual keyboards on mobile, provides built-in email format validation, and enables browser autofill.
- **MUST** include a `name` attribute on controls that submit data; the `name` is what gets sent to the server.

### Semantic input types

| Type | Purpose | Browser behavior |
|---|---|---|
| `email` | Email address input | Validates format; shows `@` keyboard on mobile |
| `url` | URL input | Validates URL format; shows URL keyboard on mobile |
| `tel` | Telephone number input | Shows numeric keypad on mobile (no format validation) |
| `number` | Numeric input | Shows numeric keyboard; supports `min`, `max`, `step` |
| `date` | Date picker | Native date picker UI on supported browsers |
| `datetime-local` | Date and time input | Combined date/time picker UI |
| `search` | Search field | May include a clear button; semantic meaning for assistive tech |
| `password` | Password input | Masks characters; triggers password manager autofill |
| `range` | Slider input | Native slider control with `min`, `max`, `step` |
| `color` | Color picker | Native color picker UI |

:::warning[bad]

```html
<!-- No label, using placeholder as label, generic type -->
<input type="text" placeholder="Email">

<!-- Label exists but is not associated -->
<span>Password</span>
<input type="text">
```

:::

:::tip[good]

```html
<!-- Explicit label association, semantic type, autocomplete -->
<label for="email">Email address</label>
<input id="email" name="email" type="email" autocomplete="email" required />

<!-- Implicit label association by wrapping -->
<label>
  Password
  <input name="password" type="password" autocomplete="current-password" required />
</label>
```

:::

## Grouping

Related form controls SHOULD be grouped using `<fieldset>` and `<legend>`. This is critical for screen reader users, who hear the legend text announced before each control within the group.

Common use cases:
- Radio button groups (the legend describes what the radio set is for).
- Checkbox groups.
- Address sections (shipping vs. billing).
- Multi-step form sections.

```html
<fieldset>
  <legend>Shipping address</legend>

  <label for="street">Street</label>
  <input id="street" name="street" type="text" autocomplete="street-address" />

  <label for="city">City</label>
  <input id="city" name="city" type="text" autocomplete="address-level2" />
</fieldset>

<fieldset>
  <legend>Preferred contact method</legend>

  <label><input type="radio" name="contact" value="email" /> Email</label>
  <label><input type="radio" name="contact" value="phone" /> Phone</label>
</fieldset>
```

## Validation

### Server-side validation is mandatory

**MUST** validate on the server. Client-side validation is a convenience for users — it provides immediate feedback — but it can be bypassed by disabling JavaScript or modifying requests directly. Never trust client-side validation as a security boundary.

### Native constraint attributes

Prefer built-in HTML validation attributes before writing custom JavaScript:

| Attribute | Purpose | Example |
|---|---|---|
| `required` | Field must not be empty | `<input required />` |
| `pattern` | Value must match a regular expression | `<input pattern="[A-Z]{2,}" />` |
| `minlength` / `maxlength` | Minimum / maximum character count | `<input minlength="8" maxlength="128" />` |
| `min` / `max` | Minimum / maximum numeric or date value | `<input type="number" min="1" max="100" />` |
| `step` | Granularity of numeric/date values | `<input type="number" step="0.01" />` |
| `type` | Implicit validation for `email`, `url`, `number` | `<input type="email" />` |

### Constraint Validation API

For custom validation logic beyond what HTML attributes provide, use the browser's built-in [Constraint Validation API](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation). Call `setCustomValidity()` to provide custom error messages and `reportValidity()` to trigger display.

### Validation feedback

- Style `:invalid` and `:valid` CSS pseudo-classes to give visual feedback. Consider using `:user-invalid` (available in modern browsers) to only show errors after the user has interacted with a field, avoiding premature error displays.
- Keep error messages concise and actionable: "Enter a valid email address" rather than "Invalid input."

```html
<form novalidate>
  <label for="pwd">Password</label>
  <input
    id="pwd"
    name="password"
    type="password"
    minlength="12"
    autocomplete="new-password"
    aria-describedby="pwd-help pwd-error"
    required
  />
  <p id="pwd-help">At least 12 characters.</p>
  <p id="pwd-error" role="alert" aria-live="polite" hidden></p>
</form>
```

The `novalidate` attribute on `<form>` disables native browser validation bubbles, allowing you to implement custom error display while still using constraint attributes for logic.

## Autocomplete

The `autocomplete` attribute improves UX (User Experience) by enabling browsers and password managers to fill in known values automatically. It also helps assistive technologies understand the purpose of a field.

Common tokens:

| Token | Purpose |
|---|---|
| `name` | Full name |
| `given-name` / `family-name` | First / last name |
| `email` | Email address |
| `tel` | Telephone number |
| `street-address` | Street address |
| `postal-code` | ZIP / postal code |
| `country` | Country |
| `cc-number` / `cc-exp` | Credit card number / expiration |
| `current-password` | Existing password (login) |
| `new-password` | New password (registration, change) |
| `one-time-code` | OTP (One-Time Password) code |

Avoid `autocomplete="off"` unless security or regulatory constraints genuinely require it — disabling autocomplete frustrates users and often does not improve security.

## Error messaging

Clear, accessible error communication is critical for usability:

- Tie field-level errors to the relevant input with `aria-describedby`. This ensures screen readers announce the error when the user focuses the field.
- Announce form-level error summaries in an `aria-live="polite"` region so screen reader users are notified without needing to search for errors.
- Place error messages near the field they describe, ideally directly below the input.

```html
<label for="username">Username</label>
<input
  id="username"
  name="username"
  type="text"
  aria-describedby="username-error"
  aria-invalid="true"
  required
/>
<p id="username-error" role="alert">Username must be at least 3 characters.</p>

<!-- Form-level summary -->
<div role="alert" aria-live="polite">
  Please fix 2 errors before submitting.
</div>
```

## Custom controls

Only build custom controls when native elements cannot meet the design or functional requirements. If you do, you **MUST** provide:

- Correct ARIA role (e.g., `role="combobox"`, `role="listbox"`, `role="slider"`) along with all required states and properties for that role.
- Full keyboard support matching the expected interaction pattern (see [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) for keyboard conventions per widget type).
- Focus management — visible focus styling, logical tab order, and programmatic focus control.
- Testing with at least one screen reader to verify the custom control is announced and operable correctly.

## Security and privacy

- Do not log, analytics-track, or persist sensitive fields (passwords, credit card numbers, OTP codes) beyond their immediate purpose.
- Use `autocomplete="one-time-code"` for OTP inputs to enable browser and keyboard autofill of incoming codes.
- Avoid splitting OTP entry into multiple single-character fields unless necessary — a single input with `autocomplete="one-time-code"` provides a better user experience and is easier to implement accessibly.
