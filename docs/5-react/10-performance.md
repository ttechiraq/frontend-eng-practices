---
sidebar_position: 10
---

# Performance

React is fast by default, but as applications grow you may need to optimize both how quickly your app loads and how responsive it feels during use.

## Rendering Optimization

### React.memo

Wrap a component in `memo` to skip re-rendering when its props have not changed:

```js
import { memo } from 'react';

const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});
```

A memoized component still re-renders when:
- Its own state changes
- A context it subscribes to changes

`memo` compares props using `Object.is` (shallow equality). New objects, arrays, or functions created during rendering break memoization.

Source: [react.dev/reference/react/memo](https://react.dev/reference/react/memo)

### useMemo

Caches the result of an expensive calculation so it only recomputes when its dependencies change:

```js
import { useMemo } from 'react';

function TodoList({ todos, filter }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, filter),
    [todos, filter]
  );

  return <ul>{visibleTodos.map((t) => <li key={t.id}>{t.text}</li>)}</ul>;
}
```

Source: [react.dev/reference/react/useMemo](https://react.dev/reference/react/useMemo)

### useCallback

Caches a function definition so the same reference is preserved between re-renders. Pair it with `memo` on child components to prevent unnecessary re-renders:

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

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // Only re-renders when onSubmit changes
  return <form onSubmit={onSubmit}>...</form>;
});
```

Source: [react.dev/reference/react/useCallback](https://react.dev/reference/react/useCallback)

### React Compiler

The React Compiler (currently in beta) automatically applies memoization across your entire app. When enabled, you can remove most manual `useMemo`, `useCallback`, and `memo` calls:

```js
// Without compiler — manual memoization needed
const visibleTodos = useMemo(() => filterTodos(todos, filter), [todos, filter]);
const handleSubmit = useCallback((data) => submit(data), []);

// With compiler — no manual memoization needed
const visibleTodos = filterTodos(todos, filter);
const handleSubmit = (data) => submit(data);
```

Source: [react.dev/learn/react-compiler](https://react.dev/learn/react-compiler)

## Transition and Deferred Updates

### useTransition

Marks a state update as non-blocking. The UI stays responsive while React renders the update in the background:

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
      <TabButtons onSelectTab={selectTab} />
      {isPending ? <Spinner /> : <TabPanel tab={tab} />}
    </>
  );
}
```

### useDeferredValue

Defers updating a value until higher-priority renders are complete:

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

Source: [react.dev/reference/react/useTransition](https://react.dev/reference/react/useTransition)

## Code Splitting and Lazy Loading

### React.lazy and Suspense

Split components into separate bundles that load on demand:

```js
import { lazy, Suspense } from 'react';

const MarkdownPreview = lazy(() => import('./MarkdownPreview'));

function Editor() {
  const [showPreview, setShowPreview] = useState(false);

  return (
    <div>
      <textarea />
      <button onClick={() => setShowPreview(true)}>Preview</button>
      {showPreview && (
        <Suspense fallback={<p>Loading preview...</p>}>
          <MarkdownPreview />
        </Suspense>
      )}
    </div>
  );
}
```

### Route-Based Code Splitting

The most natural place to introduce code splitting is at the route level. See the [Routing chapter](./8-routing.md) for lazy-loading routes.

Source: [react.dev/reference/react/lazy](https://react.dev/reference/react/lazy)

## Virtualization

For long lists (hundreds or thousands of items), render only the items currently visible in the viewport. Libraries like [TanStack Virtual](https://tanstack.com/virtual) or [react-window](https://github.com/bvaughn/react-window) provide this:

```js
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 35,
  });

  return (
    <div ref={parentRef} style={{ height: 400, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map((virtualRow) => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: virtualRow.start,
              height: virtualRow.size,
              width: '100%',
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Profiling

### React DevTools Profiler

The React DevTools Profiler records render timings and helps identify components that re-render unnecessarily:

1. Open React DevTools in your browser.
2. Switch to the **Profiler** tab.
3. Click **Record**, interact with your app, then click **Stop**.
4. Review the flame chart to find slow components.

### Why Did This Render?

Enable "Record why each component rendered while profiling" in React DevTools settings. This shows whether a component re-rendered because of state, props, hooks, or a parent re-render.

## What to Optimize First

| Symptom | Investigation |
|---|---|
| Slow initial load | Code-split large routes and components with `lazy`; check bundle size with a bundle analyzer |
| Janky interactions | Profile with React DevTools; look for unnecessary re-renders; use `memo`/`useMemo`/`useCallback` or enable the React Compiler |
| Slow lists | Virtualize long lists; ensure list items have stable `key` props |
| Input lag | Use `useDeferredValue` or `useTransition` to keep the input responsive |

As a general rule, profile first, then optimize. Premature optimization makes code harder to read without measurable benefit.
