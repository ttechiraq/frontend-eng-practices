---
sidebar_position: 2
---

# Components

Components are the core building blocks of React applications. A component is a piece of the UI that has its own logic and appearance. A component can be as small as a button, or as large as an entire page.

Source: [react.dev/learn/your-first-component](https://react.dev/learn/your-first-component)

## Defining a Component

A React component is a JavaScript function that returns markup (JSX). Component names must start with a capital letter:

```js
function Greeting() {
  return <h1>Hello, world!</h1>;
}
```

### Exporting Components

Use `export default` to mark the main component in a file, or named exports for multiple components:

```js
// Default export — one per file
export default function App() {
  return <h1>My App</h1>;
}

// Named exports — any number per file
export function Header() {
  return <header>Header</header>;
}

export function Footer() {
  return <footer>Footer</footer>;
}
```

Source: [react.dev/learn/importing-and-exporting-components](https://react.dev/learn/importing-and-exporting-components)

## JSX

JSX is a syntax extension for JavaScript that lets you write HTML-like markup inside JavaScript files. It is stricter than HTML:

- You must close all tags (e.g., `<img />`, `<br />`).
- A component can only return one root element. Use a fragment (`<>...</>`) to group multiple elements without adding an extra DOM node.

```js
function AboutPage() {
  return (
    <>
      <h1>About</h1>
      <p>Hello there.</p>
    </>
  );
}
```

### JavaScript in JSX with Curly Braces

Curly braces let you embed JavaScript expressions inside JSX:

```js
const user = { name: 'Hedy Lamarr', imageUrl: 'https://i.imgur.com/yXOvdOSs.jpg' };

function Profile() {
  return (
    <>
      <h1>{user.name}</h1>
      <img
        className="avatar"
        src={user.imageUrl}
        alt={'Photo of ' + user.name}
        style={{ width: 90, height: 90, borderRadius: '50%' }}
      />
    </>
  );
}
```

The double curly braces in `style={{}}` are a regular JavaScript object inside JSX curly braces.

Source: [react.dev/learn/javascript-in-jsx-with-curly-braces](https://react.dev/learn/javascript-in-jsx-with-curly-braces)

## Using Components

You use a component by rendering it as a JSX tag. Components can be nested inside other components:

```js
function Profile() {
  return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

React distinguishes components from HTML elements by casing: `<section>` is an HTML tag, `<Profile />` is a component.

### Nesting and Organizing Components

Components are regular JavaScript functions. You can keep multiple components in the same file when they are small or tightly related. As files grow, move components to separate files.

Never define a component inside another component. It causes bugs and poor performance because the inner component is recreated on every render:

```js
// Bad: nested definition
export default function Gallery() {
  function Profile() { /* ... */ }
  return <Profile />;
}

// Good: top-level definitions
export default function Gallery() {
  return <Profile />;
}

function Profile() { /* ... */ }
```

Source: [react.dev/learn/your-first-component](https://react.dev/learn/your-first-component)

## Props

React components use props to communicate with each other. Every parent component can pass information to its child components by giving them props. Props can be any JavaScript value: strings, numbers, objects, arrays, functions, and even other components.

### Passing Props

Pass props to a component like HTML attributes:

```js
function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={person.imageUrl}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageUrl: 'https://i.imgur.com/1bX5QH6.jpg' }}
      size={100}
    />
  );
}
```

### Default Props

Specify a default value for a prop using destructuring defaults:

```js
function Avatar({ person, size = 100 }) {
  return (
    <img
      src={person.imageUrl}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

### Forwarding Props with Spread Syntax

When a component forwards all its props to a child, use the spread syntax:

```js
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

### Passing JSX as Children

When you nest content inside a JSX tag, the parent component receives that content as a `children` prop:

```js
function Card({ children }) {
  return <div className="card">{children}</div>;
}

export default function Profile() {
  return (
    <Card>
      <Avatar />
      <p>Bio goes here</p>
    </Card>
  );
}
```

Source: [react.dev/learn/passing-props-to-a-component](https://react.dev/learn/passing-props-to-a-component)

## Adding Styles

In React, you specify a CSS class with `className`:

```js
<img className="avatar" />
```

Then write the CSS rules in a separate file:

```css
.avatar {
  border-radius: 50%;
}
```

React does not prescribe a specific approach for adding CSS. Common approaches include:

| Approach | Description |
|---|---|
| CSS files | Traditional `.css` files imported into components |
| CSS Modules | Scoped CSS files (`*.module.css`) that avoid class name collisions |
| Tailwind CSS | Utility-first framework using class names directly in JSX |
| CSS-in-JS | Libraries like styled-components or Emotion that co-locate styles with components |
| Inline styles | The `style` prop for dynamic, JavaScript-driven styles |

## Keeping Components Pure

A pure component always returns the same JSX given the same inputs (props, state, context). It does not change any objects or variables that existed before rendering. React relies on this rule to skip re-rendering components whose inputs have not changed:

```js
// Pure: same input always produces same output
function Recipe({ drinkers }) {
  return (
    <ol>
      <li>Boil {drinkers} cups of water.</li>
      <li>Add {drinkers} spoons of tea.</li>
      <li>Add {0.5 * drinkers} spoons of spice.</li>
    </ol>
  );
}
```

Side effects (network requests, DOM mutations, timers) should go inside event handlers or effects, not in the render function itself.

Source: [react.dev/learn/keeping-components-pure](https://react.dev/learn/keeping-components-pure)

## Responding to Events

React lets you add event handlers to your JSX. Event handlers are functions that fire in response to interactions like clicking, hovering, or focusing on form inputs:

```js
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

Pass the function itself (not the result of calling it). React calls your event handler when the user triggers the event:

| Syntax | Behavior |
|---|---|
| `onClick={handleClick}` | Passes the function (correct) |
| `onClick={handleClick()}` | Calls the function immediately during render (wrong) |

### Event Propagation

Events bubble up through the component tree. Use `e.stopPropagation()` to prevent bubbling:

```js
function Button({ onClick, children }) {
  return (
    <button onClick={(e) => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}
```

### Preventing Default Behavior

Some browser events have default behavior (e.g., form submission reloads the page). Call `e.preventDefault()` to stop it:

```js
function Signup() {
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      alert('Submitting!');
    }}>
      <input />
      <button>Send</button>
    </form>
  );
}
```

Source: [react.dev/learn/responding-to-events](https://react.dev/learn/responding-to-events)
