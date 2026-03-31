---
sidebar_position: 9
---

# State Management

As React applications grow, managing state becomes one of the most critical architectural decisions. React provides built-in tools for local and shared state, and the ecosystem offers external libraries for complex global state requirements.

## Local State with useState

For state that is only needed within a single component, `useState` is the simplest and most appropriate tool:

```js
function SearchBar() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

Source: [react.dev/reference/react/useState](https://react.dev/reference/react/useState)

## Complex Local State with useReducer

When a component has complex state logic with multiple sub-values or when the next state depends on the previous one, `useReducer` centralizes the update logic:

```js
import { useReducer } from 'react';

const initialState = { items: [], filter: 'all' };

function reducer(state, action) {
  switch (action.type) {
    case 'added':
      return { ...state, items: [...state.items, action.item] };
    case 'deleted':
      return { ...state, items: state.items.filter((i) => i.id !== action.id) };
    case 'filter_changed':
      return { ...state, filter: action.filter };
    default:
      throw new Error('Unknown action: ' + action.type);
  }
}

function TaskBoard() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <AddTask onAdd={(item) => dispatch({ type: 'added', item })} />
      <TaskList
        items={state.items}
        filter={state.filter}
        onDelete={(id) => dispatch({ type: 'deleted', id })}
      />
    </div>
  );
}
```

Source: [react.dev/learn/extracting-state-logic-into-a-reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)

## Sharing State with Context

Context lets you pass data through the component tree without manually threading props through every level. It is best for values that are infrequently updated and needed by many components (theme, locale, authentication).

### Creating and Providing Context

```js
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  function login(userData) { setUser(userData); }
  function logout() { setUser(null); }

  return (
    <AuthContext value={{ user, login, logout }}>
      {children}
    </AuthContext>
  );
}

function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
```

### Consuming Context

```js
function UserMenu() {
  const { user, logout } = useAuth();

  if (!user) return <Link to="/login">Log in</Link>;

  return (
    <div>
      <span>{user.name}</span>
      <button onClick={logout}>Log out</button>
    </div>
  );
}
```

Source: [react.dev/learn/passing-data-deeply-with-context](https://react.dev/learn/passing-data-deeply-with-context)

## Scaling Up with Reducer and Context

For medium-complexity state that is shared across a subtree, combine `useReducer` with Context. The reducer handles state logic while Context makes the state and dispatch function available without prop drilling:

```js
import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added':
      return [...tasks, { id: action.id, text: action.text, done: false }];
    case 'changed':
      return tasks.map((t) => (t.id === action.task.id ? action.task : t));
    case 'deleted':
      return tasks.filter((t) => t.id !== action.id);
    default:
      throw new Error('Unknown action: ' + action.type);
  }
}

function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, []);
  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}

function useTasks() { return useContext(TasksContext); }
function useTasksDispatch() { return useContext(TasksDispatchContext); }
```

Source: [react.dev/learn/scaling-up-with-reducer-and-context](https://react.dev/learn/scaling-up-with-reducer-and-context)

## External State Libraries

For large applications with significant global state, external libraries offer additional capabilities like devtools, middleware, persistence, and fine-grained subscriptions.

### Zustand

A minimal, unopinionated state management library:

```js
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

function Counter() {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);
  return <button onClick={increment}>{count}</button>;
}
```

### Redux Toolkit

The official Redux tooling for productive Redux development:

```js
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { Provider, useSelector, useDispatch } from 'react-redux';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    incremented: (state) => { state.value += 1; },
    decremented: (state) => { state.value -= 1; },
  },
});

const store = configureStore({ reducer: { counter: counterSlice.reducer } });

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(counterSlice.actions.incremented())}>{count}</button>;
}

function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}
```

### TanStack Query (React Query)

Manages server state (fetching, caching, synchronizing, and updating remote data):

```js
import { useQuery, QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

function Users() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then((res) => res.json()),
  });

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.map((user) => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Users />
    </QueryClientProvider>
  );
}
```

## Choosing the Right Approach

| Scenario | Recommended tool |
|---|---|
| Local component state | `useState` |
| Complex local state with many actions | `useReducer` |
| Theme, locale, auth (infrequent updates) | Context |
| Shared state in a subtree with complex logic | `useReducer` + Context |
| Large-scale global client state | Zustand, Redux Toolkit, Jotai |
| Server state (fetching, caching) | TanStack Query, SWR |
