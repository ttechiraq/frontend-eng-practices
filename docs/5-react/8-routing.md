---
sidebar_position: 8
---

# Routing

React itself does not include a built-in router. For single-page applications, [React Router](https://reactrouter.com/) is the most widely used routing library. Framework-based React apps (Next.js, Remix) have their own file-system-based routing built in.

## React Router Setup

Install React Router:

```bash
npm install react-router
```

### Defining Routes

Use `createBrowserRouter` and `RouterProvider` to define routes declaratively:

```js
import { createBrowserRouter, RouterProvider } from 'react-router';

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    children: [
      { index: true, element: <Home /> },
      { path: 'about', element: <About /> },
      { path: 'users/:userId', element: <UserProfile /> },
    ],
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

### Layout Routes

Layout routes render shared UI (navigation, sidebar) around nested routes. Use `<Outlet />` to render child route content:

```js
import { Outlet, Link } from 'react-router';

function RootLayout() {
  return (
    <div>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      <main>
        <Outlet />
      </main>
    </div>
  );
}
```

## Navigation

### Link and NavLink

Use `<Link>` for standard navigation and `<NavLink>` for links that need active styling:

```js
import { Link, NavLink } from 'react-router';

function Navigation() {
  return (
    <nav>
      <NavLink to="/" className={({ isActive }) => isActive ? 'active' : ''}>
        Home
      </NavLink>
      <Link to="/about">About</Link>
    </nav>
  );
}
```

### Programmatic Navigation

Use the `useNavigate` hook to navigate in response to events:

```js
import { useNavigate } from 'react-router';

function LoginForm() {
  const navigate = useNavigate();

  async function handleSubmit(e) {
    e.preventDefault();
    await login();
    navigate('/dashboard');
  }

  return <form onSubmit={handleSubmit}>...</form>;
}
```

## Route Parameters

Use `:paramName` in the path and `useParams` to read it:

```js
import { useParams } from 'react-router';

function UserProfile() {
  const { userId } = useParams();
  return <h1>User ID: {userId}</h1>;
}
```

## Search Parameters

Use `useSearchParams` to read and update query strings:

```js
import { useSearchParams } from 'react-router';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const category = searchParams.get('category') ?? 'all';

  function filterByCategory(cat) {
    setSearchParams({ category: cat });
  }

  return (
    <div>
      <button onClick={() => filterByCategory('electronics')}>Electronics</button>
      <button onClick={() => filterByCategory('books')}>Books</button>
      <p>Current category: {category}</p>
    </div>
  );
}
```

## Data Loading

React Router supports loading data before a route renders using `loader` functions:

```js
import { useLoaderData } from 'react-router';

async function userLoader({ params }) {
  const res = await fetch(`/api/users/${params.userId}`);
  if (!res.ok) throw new Response('Not Found', { status: 404 });
  return res.json();
}

function UserProfile() {
  const user = useLoaderData();
  return <h1>{user.name}</h1>;
}

const router = createBrowserRouter([
  {
    path: 'users/:userId',
    element: <UserProfile />,
    loader: userLoader,
  },
]);
```

## Error Handling

Define an `errorElement` to handle errors thrown by loaders, actions, or rendering:

```js
const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      { index: true, element: <Home /> },
    ],
  },
]);

function ErrorPage() {
  const error = useRouteError();
  return (
    <div>
      <h1>Oops!</h1>
      <p>{error.statusText || error.message}</p>
    </div>
  );
}
```

## Protected Routes

Guard routes by checking authentication in a layout or loader:

```js
import { Navigate, Outlet } from 'react-router';

function ProtectedLayout() {
  const { isAuthenticated } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return <Outlet />;
}

const router = createBrowserRouter([
  {
    element: <ProtectedLayout />,
    children: [
      { path: 'dashboard', element: <Dashboard /> },
      { path: 'settings', element: <Settings /> },
    ],
  },
  { path: 'login', element: <Login /> },
]);
```

## Lazy Loading Routes

Split route components into separate bundles with `React.lazy` or React Router's `lazy` option:

```js
const router = createBrowserRouter([
  {
    path: 'admin',
    lazy: () => import('./admin/AdminPage'),
  },
]);
```

Or with `React.lazy`:

```js
import { lazy, Suspense } from 'react';

const AdminPage = lazy(() => import('./admin/AdminPage'));

const router = createBrowserRouter([
  {
    path: 'admin',
    element: (
      <Suspense fallback={<div>Loading...</div>}>
        <AdminPage />
      </Suspense>
    ),
  },
]);
```

## Framework Routing

### Next.js App Router

Next.js uses a file-system-based router. Each folder inside `app/` becomes a route segment, and `page.tsx` files define the UI for that route:

```
app/
├── layout.tsx          # Root layout
├── page.tsx            # / route
├── about/
│   └── page.tsx        # /about route
├── blog/
│   ├── page.tsx        # /blog route
│   └── [slug]/
│       └── page.tsx    # /blog/:slug dynamic route
└── (auth)/
    ├── login/
    │   └── page.tsx    # /login route (grouped)
    └── register/
        └── page.tsx    # /register route (grouped)
```

Source: [reactrouter.com](https://reactrouter.com/)
