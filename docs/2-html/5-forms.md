---
sidebar_position: 5
---

# Forms and validation

Prefer native HTML form controls before custom widgets. Always pair controls with labels and provide both client-side and server-side validation.

## Basics

- **MUST** use `<label>` for every form control; link via `for` or wrap the control.
- **SHOULD** use semantic input types to leverage native keyboards and validation (`email`, `url`, `tel`, `number`, `date`, `datetime-local`, `search`, `password`).
- **MUST** include `name` on controls that submit data.

:::warning[bad]
```html
<input type="text" placeholder="Email">
```
:::

:::tip[good]
```html
<label for="email">Email</label>
<input id="email" name="email" type="email" autocomplete="email" required />
```
:::

## Grouping

- Use `<fieldset>` and `<legend>` for related inputs (e.g., contact info, shipping vs billing).
- Group radios with the same `name`; label each option.

## Validation

- **MUST** validate on the server; client-side is defense-in-depth.
- Prefer native constraints: `required`, `pattern`, `min`, `max`, `step`, `minlength`, `maxlength`.
- Use Constraint Validation API for custom messages; keep them concise and specific.
- Style `:invalid` and `:valid` states to give immediate feedback.

```html
<form novalidate>
  <label for="pwd">Password</label>
  <input
    id="pwd"
    name="password"
    type="password"
    minlength="12"
    aria-describedby="pwd-help"
    required
  />
  <p id="pwd-help">At least 12 characters.</p>
</form>
```

## Autocomplete

- Use `autocomplete` tokens to improve UX and accuracy (`name`, `email`, `street-address`, `cc-number`, `one-time-code`).
- Avoid `autocomplete="off"` unless security or regulatory constraints require it.

## Error messaging

- Tie errors to inputs via `aria-describedby`.
- Announce form-level errors in an `aria-live="polite"` region.

```html
<div role="alert" aria-live="polite">Please fix the highlighted fields.</div>
```

## Custom controls

Only build custom controls when native elements cannot meet requirements. If you do, you **MUST** provide:
- Correct role (e.g., `role="combobox"`), state, and property attributes.
- Full keyboard support (tab, arrow keys, Enter/Space, Esc).
- Focus management and visible focus styling.

## Security and privacy

- Do not log or analytics-track sensitive fields (passwords, tokens).
- Use `autocomplete="one-time-code"` for OTP inputs; avoid forcing multi-field OTP splits unless necessary.
