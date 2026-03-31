---
sidebar_position: 3
---

# State and Lifecycle

Components often need to change what is on the screen as a result of an interaction. State is a component's memory -- it lets a component track information that changes over time and re-render when that information updates.

Source: [react.dev/learn/state-a-components-memory](https://react.dev/learn/state-a-components-memory)

## Adding State with useState

The `useState` hook declares a state variable. It takes an initial value and returns a pair: the current state value and a setter function:

```js
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Clicked {count} times
    </button>
  );
}
```

The convention is to name the pair `[something, setSomething]`.

### State is Isolated and Private

If you render the same component twice, each copy gets its own state. Changing state in one does not affect the other:

```js
export default function App() {
  return (
    <div>
      <Counter />
      <Counter />
    </div>
  );
}
```

Each `Counter` above maintains its own independent `count` value.

### State as a Snapshot

State behaves like a snapshot. Setting state does not change the variable in the current render; it triggers a new render with the updated value:

```js
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // count will be 1, not 3 — all three calls read the same snapshot
  }

  return <button onClick={handleClick}>+3 (actually +1)</button>;
}
```

To update state based on the previous value, pass an updater function:

```js
function handleClick() {
  setCount((prev) => prev + 1);
  setCount((prev) => prev + 1);
  setCount((prev) => prev + 1);
  // count will be 3
}
```

Source: [react.dev/learn/state-as-a-snapshot](https://react.dev/learn/state-as-a-snapshot)

## Updating Objects in State

State can hold any JavaScript value, including objects. But you should not mutate objects directly. Instead, create a new object and pass it to the setter:

```js
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
  },
});

function handleNameChange(e) {
  setPerson({ ...person, name: e.target.value });
}

function handleCityChange(e) {
  setPerson({
    ...person,
    artwork: { ...person.artwork, city: e.target.value },
  });
}
```

The spread syntax (`...`) copies the existing fields so you only override what changed.

Source: [react.dev/learn/updating-objects-in-state](https://react.dev/learn/updating-objects-in-state)

## Updating Arrays in State

Arrays in state should be treated as immutable too. Use non-mutating methods:

| Operation | Avoid (mutates) | Prefer (returns new array) |
|---|---|---|
| Adding | `push`, `unshift` | `[...arr, newItem]`, `concat` |
| Removing | `splice`, `pop`, `shift` | `filter`, `slice` |
| Replacing | `splice`, `arr[i] = ...` | `map` |
| Sorting | `sort`, `reverse` | Copy first, then sort |

```js
const [artists, setArtists] = useState([]);

// Adding
setArtists([...artists, { id: nextId++, name: name }]);

// Removing
setArtists(artists.filter((a) => a.id !== id));

// Updating
setArtists(artists.map((a) => (a.id === id ? { ...a, name: newName } : a)));
```

Source: [react.dev/learn/updating-arrays-in-state](https://react.dev/learn/updating-arrays-in-state)

## Lifting State Up

When two components need to share state, move the state to their closest common parent. The parent passes the state down as props and provides a callback to update it:

```js
import { useState } from 'react';

export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update together</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}

function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Clicked {count} times
    </button>
  );
}
```

This pattern is called "lifting state up." The information flows down through props and back up through event handlers.

Source: [react.dev/learn/sharing-state-between-components](https://react.dev/learn/sharing-state-between-components)

## Preserving and Resetting State

React preserves a component's state as long as it is rendered at the same position in the UI tree. When a component is removed or a different component is rendered at the same position, React destroys its state.

```js
// These two Counters have independent state because they
// occupy different positions in the tree:
<div>
  <Counter />  {/* position 0 */}
  <Counter />  {/* position 1 */}
</div>
```

### Resetting State with a Key

Use the `key` prop to force React to treat a component as a completely new instance:

```js
function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => setIsPlayerA(!isPlayerA)}>
        Next player!
      </button>
    </div>
  );
}
```

Switching the `key` resets the `Counter` state completely.

Source: [react.dev/learn/preserving-and-resetting-state](https://react.dev/learn/preserving-and-resetting-state)

## Choosing the State Structure

Well-structured state makes a component easy to modify and debug. Follow these principles:

1. **Group related state.** If two state variables always change together, merge them into one.
2. **Avoid contradictions in state.** Don't have multiple state variables that can disagree.
3. **Avoid redundant state.** If a value can be computed from props or other state, don't store it in state.
4. **Avoid duplication in state.** The same data in multiple state variables is hard to keep in sync.
5. **Avoid deeply nested state.** Flat state is easier to update.

```js
// Bad: redundant state
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState(''); // redundant!

// Good: compute derived values
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const fullName = firstName + ' ' + lastName;
```

Source: [react.dev/learn/choosing-the-state-structure](https://react.dev/learn/choosing-the-state-structure)
