---
sidebar_position: 4
---

# Hooks

Hooks let you use state and other React features from function components. You can use built-in hooks or combine them to build your own. Functions whose names start with `use` are called hooks.

Source: [react.dev/reference/react/hooks](https://react.dev/reference/react/hooks)

## Rules of Hooks

Hooks have two strict rules:

1. **Only call hooks at the top level.** Don't call hooks inside loops, conditions, or nested functions. This ensures hooks are called in the same order each render.
2. **Only call hooks from React functions.** Call hooks from function components or from custom hooks, not from regular JavaScript functions.

```js
// Bad: hook inside a condition
function Form() {
  if (showName) {
    const [name, setName] = useState(''); // breaks the rules
  }
}

// Good: condition inside the hook's usage
function Form() {
  const [name, setName] = useState('');
  // conditionally use `name` in the render
}
```

Source: [react.dev/reference/rules/rules-of-hooks](https://react.dev/reference/rules/rules-of-hooks)

## State Hooks

### useState

Declares a state variable. Returns the current value and a setter function:

```js
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### useReducer

An alternative to `useState` for complex state logic. Keeps update logic in a reducer function outside the component:

```js
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error('Unknown action: ' + action.type);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </>
  );
}
```

Source: [react.dev/reference/react/useReducer](https://react.dev/reference/react/useReducer)

## Context Hooks

### useContext

Reads and subscribes to a context. The component re-renders when the context value changes:

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');

function App() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext value={theme}>
      <Panel />
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle theme
      </button>
    </ThemeContext>
  );
}

function Panel() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>Current theme: {theme}</div>;
}
```

Source: [react.dev/reference/react/useContext](https://react.dev/reference/react/useContext)

## Ref Hooks

### useRef

Declares a ref -- a mutable value that persists across renders without causing re-renders. Most commonly used to hold a DOM node:

```js
import { useRef } from 'react';

function TextInput() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

### useImperativeHandle

Customizes the ref exposed by a component. Use it sparingly to limit what a parent can do with the ref:

```js
import { forwardRef, useImperativeHandle, useRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const realInputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      realInputRef.current.focus();
    },
  }));

  return <input {...props} ref={realInputRef} />;
});
```

Source: [react.dev/reference/react/useRef](https://react.dev/reference/react/useRef)

## Effect Hooks

### useEffect

Connects a component to an external system (network, browser API, third-party library). Effects run after rendering:

```js
import { useState, useEffect } from 'react';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to {roomId}!</h1>;
}
```

The dependency array (`[roomId]`) tells React to re-run the effect only when `roomId` changes. The returned cleanup function runs before the effect re-runs and when the component unmounts.

| Dependency array | When the effect runs |
|---|---|
| Not provided | After every render |
| `[]` | Only after the first render (mount) |
| `[a, b]` | After first render and whenever `a` or `b` change |

Effects are an "escape hatch." Don't use them to orchestrate data flow. If you are not interacting with an external system, you might not need an effect.

Source: [react.dev/learn/synchronizing-with-effects](https://react.dev/learn/synchronizing-with-effects)

### useLayoutEffect

Fires synchronously after all DOM mutations but before the browser repaints. Use it for measuring layout (e.g., tooltip positioning):

```js
import { useLayoutEffect, useRef, useState } from 'react';

function Tooltip({ children, targetRect }) {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);

  return <div ref={ref} style={{ top: targetRect.top - tooltipHeight }}>{children}</div>;
}
```

Source: [react.dev/reference/react/useLayoutEffect](https://react.dev/reference/react/useLayoutEffect)

## Performance Hooks

### useMemo

Caches the result of an expensive calculation between re-renders:

```js
import { useMemo } from 'react';

function TodoList({ todos, filter }) {
  const visibleTodos = useMemo(
    () => todos.filter((t) => t.status === filter),
    [todos, filter]
  );

  return <ul>{visibleTodos.map((t) => <li key={t.id}>{t.text}</li>)}</ul>;
}
```

### useCallback

Caches a function definition between re-renders. This is useful when passing callbacks to memoized child components:

```js
import { useCallback } from 'react';

function ProductPage({ productId }) {
  const handleSubmit = useCallback(
    (orderDetails) => {
      post('/product/' + productId + '/buy', { data: orderDetails });
    },
    [productId]
  );

  return <ShippingForm onSubmit={handleSubmit} />;
}
```

### useTransition

Marks a state update as non-blocking, allowing the UI to stay responsive during expensive re-renders:

```js
import { useState, useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }

  return (
    <>
      <TabButton onClick={() => selectTab('about')}>About</TabButton>
      <TabButton onClick={() => selectTab('posts')}>Posts</TabButton>
      {isPending && <Spinner />}
      <TabPanel tab={tab} />
    </>
  );
}
```

### useDeferredValue

Defers updating a non-critical part of the UI. Useful for keeping an input responsive while a results list updates:

```js
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <SearchResults query={deferredQuery} />
    </>
  );
}
```

Source: [react.dev/reference/react/useMemo](https://react.dev/reference/react/useMemo)

## Other Built-in Hooks

| Hook | Purpose |
|---|---|
| `useId` | Generates unique IDs for accessibility attributes |
| `useSyncExternalStore` | Subscribes to an external store (e.g., a third-party state library) |
| `useActionState` | Manages state returned by a form action |
| `useFormStatus` | Reads the status of a parent `<form>` |
| `useOptimistic` | Shows optimistic state during an async action |
| `use` | Reads a promise or context during render |
| `useDebugValue` | Customizes the label in React DevTools for a custom hook |

## Custom Hooks

Custom hooks extract reusable stateful logic into functions. A custom hook is any function whose name starts with `use` and that calls other hooks:

```js
import { useState, useEffect } from 'react';

function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function handleOnline() { setIsOnline(true); }
    function handleOffline() { setIsOnline(false); }

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}
```

Usage:

```js
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? 'Online' : 'Disconnected'}</h1>;
}
```

Custom hooks share stateful logic, not state itself. Each call to a custom hook is completely independent.

### When to Write a Custom Hook

- When you find yourself duplicating the same `useState` + `useEffect` pattern across components.
- When you want to wrap a browser API, third-party library, or subscription.
- When naming the hook makes your component code clearer by abstracting implementation details.

Source: [react.dev/learn/reusing-logic-with-custom-hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
