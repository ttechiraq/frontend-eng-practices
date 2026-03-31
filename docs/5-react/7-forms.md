---
sidebar_position: 7
---

# Forms

React provides two approaches to handling form data: **controlled components** where React state drives the form, and **uncontrolled components** where the DOM itself holds the form state. React 19 introduces built-in form actions for streamlined submission handling.

## Controlled Components

In a controlled component, form data is handled by React state. The input's value is always driven by state, and every change goes through a state setter:

```js
import { useState } from 'react';

function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    console.log('Submitting:', { email, password });
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Email
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
      </label>
      <label>
        Password
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
      </label>
      <button type="submit">Log in</button>
    </form>
  );
}
```

### Handling Multiple Inputs

Use a single state object and computed property names to handle multiple inputs:

```js
function SignupForm() {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
  });

  function handleChange(e) {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  }

  return (
    <form>
      <input name="firstName" value={formData.firstName} onChange={handleChange} />
      <input name="lastName" value={formData.lastName} onChange={handleChange} />
      <input name="email" value={formData.email} onChange={handleChange} />
    </form>
  );
}
```

### Different Input Types

```js
function Preferences() {
  const [agreed, setAgreed] = useState(false);
  const [color, setColor] = useState('blue');
  const [bio, setBio] = useState('');

  return (
    <form>
      {/* Checkbox */}
      <label>
        <input
          type="checkbox"
          checked={agreed}
          onChange={(e) => setAgreed(e.target.checked)}
        />
        I agree to the terms
      </label>

      {/* Select */}
      <label>
        Favorite color
        <select value={color} onChange={(e) => setColor(e.target.value)}>
          <option value="red">Red</option>
          <option value="blue">Blue</option>
          <option value="green">Green</option>
        </select>
      </label>

      {/* Textarea */}
      <label>
        Bio
        <textarea value={bio} onChange={(e) => setBio(e.target.value)} />
      </label>
    </form>
  );
}
```

## Uncontrolled Components

Uncontrolled components let the DOM handle form state. Use `useRef` to read values when needed:

```js
import { useRef } from 'react';

function FileUpload() {
  const fileInput = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    const file = fileInput.current.files[0];
    console.log('Selected file:', file.name);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="file" ref={fileInput} />
      <button type="submit">Upload</button>
    </form>
  );
}
```

File inputs are always uncontrolled because their value can only be set by the user.

## Form Actions (React 19)

React 19 adds native support for the `action` prop on `<form>`. You can pass an async function that handles submission, along with new hooks for pending state and form status:

### useActionState

Manages state updates from form actions:

```js
import { useActionState } from 'react';

async function submitForm(prevState, formData) {
  const name = formData.get('name');
  const error = await saveToServer(name);
  if (error) {
    return { error };
  }
  return { success: true };
}

function NameForm() {
  const [state, formAction, isPending] = useActionState(submitForm, {});

  return (
    <form action={formAction}>
      <input name="name" />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Saving...' : 'Save'}
      </button>
      {state.error && <p className="error">{state.error}</p>}
    </form>
  );
}
```

### useFormStatus

Reads the status of a parent `<form>` without prop drilling. Must be called from a component rendered inside a `<form>`:

```js
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}
```

### useOptimistic

Shows an optimistic value while an async action is in progress:

```js
import { useOptimistic } from 'react';

function MessageList({ messages, sendMessage }) {
  const [optimisticMessages, addOptimistic] = useOptimistic(
    messages,
    (state, newMessage) => [...state, { text: newMessage, sending: true }]
  );

  async function handleSend(formData) {
    const text = formData.get('message');
    addOptimistic(text);
    await sendMessage(text);
  }

  return (
    <div>
      {optimisticMessages.map((msg, i) => (
        <div key={i} style={{ opacity: msg.sending ? 0.5 : 1 }}>
          {msg.text}
        </div>
      ))}
      <form action={handleSend}>
        <input name="message" />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

Source: [react.dev/reference/react/useActionState](https://react.dev/reference/react/useActionState)

## Validation

### Client-Side Validation

Use HTML5 validation attributes together with React state for custom error messages:

```js
function RegistrationForm() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  function validate(value) {
    if (!value) return 'Email is required';
    if (!value.includes('@')) return 'Enter a valid email';
    return '';
  }

  function handleBlur() {
    setError(validate(email));
  }

  function handleSubmit(e) {
    e.preventDefault();
    const err = validate(email);
    if (err) {
      setError(err);
      return;
    }
    console.log('Valid submission:', email);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Email
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          onBlur={handleBlur}
          required
        />
      </label>
      {error && <p className="error">{error}</p>}
      <button type="submit">Register</button>
    </form>
  );
}
```

### Form Libraries

For complex forms, consider a dedicated library:

| Library | Description |
|---|---|
| [React Hook Form](https://react-hook-form.com/) | Performant, flexible forms with minimal re-renders |
| [Formik](https://formik.org/) | Feature-rich form management with validation |
| [Zod](https://zod.dev/) + React Hook Form | Type-safe schema validation integrated with forms |
