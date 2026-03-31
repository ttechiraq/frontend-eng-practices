---
sidebar_position: 1
---

# What is React?

React is a JavaScript library for building user interfaces. It lets you compose complex UIs from small, isolated pieces of code called "components." Maintained by Meta and a community of individual developers and companies, React is the most widely adopted frontend library in the ecosystem.

Source: [react.dev](https://react.dev/)

## Key Features

### Components

React lets you build encapsulated components that manage their own state, then compose them to make complex UIs. Since component logic is written in JavaScript instead of templates, you can pass rich data through your app and keep state out of the DOM.

### Declarative Views

React makes it painless to create interactive UIs. Design simple views for each state in your application, and React will efficiently update and render just the right components when your data changes. Declarative views make your code more predictable, simpler to understand, and easier to debug.

### JSX

JSX is a syntax extension for JavaScript that lets you write HTML-like markup inside JavaScript. Most React projects use JSX for its convenience, though it is not required.

### Hooks

Hooks let you use state and other React features from function components. Built-in hooks like `useState`, `useEffect`, and `useContext` cover the most common use cases, and you can build custom hooks to reuse stateful logic across components.

### Server Components

React Server Components let you build apps that span the server and client, combining the rich interactivity of client-side apps with the improved performance of traditional server rendering.

## Installation

### Prerequisites

- [Node.js](https://nodejs.org/) - v18.0 or newer
- Text editor - [VS Code](https://code.visualstudio.com/) recommended
- Terminal

### Creating a New Project

The React team recommends starting new projects with a framework. The most popular options are:

**Next.js (recommended for most projects):**

```bash
npx create-next-app@latest my-app
cd my-app
npm run dev
```

**Vite (for single-page applications):**

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npm run dev
```

Source: [react.dev/learn/creating-a-react-app](https://react.dev/learn/creating-a-react-app)

### Adding React to an Existing Project

If you have an existing project, you can add React via npm:

```bash
npm install react react-dom
```

Then import and render a component:

```js
import { createRoot } from 'react-dom/client';

function App() {
  return <h1>Hello, world!</h1>;
}

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

Source: [react.dev/learn/add-react-to-an-existing-project](https://react.dev/learn/add-react-to-an-existing-project)

## Project Structure

A typical React project created with Vite contains:

```
my-app/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   ├── App.tsx            # Root component
│   ├── App.css            # Root styles
│   ├── main.tsx           # Application entry point
│   └── index.css          # Global styles
├── index.html             # Main HTML page
├── package.json           # Dependencies
├── tsconfig.json          # TypeScript configuration
└── vite.config.ts         # Vite configuration
```

A Next.js project uses a file-system based router:

```
my-app/
├── app/
│   ├── layout.tsx         # Root layout
│   ├── page.tsx           # Home page (/)
│   ├── about/
│   │   └── page.tsx       # About page (/about)
│   └── globals.css        # Global styles
├── public/                # Static assets
├── next.config.ts         # Next.js configuration
├── package.json           # Dependencies
└── tsconfig.json          # TypeScript configuration
```

## React 19 Highlights

React 19 introduced several significant features and improvements:

### Actions

Actions simplify handling async transitions. They automatically manage pending states, errors, and optimistic updates:

```js
function UpdateName() {
  const [name, setName] = useState('');
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      redirect('/profile');
    });
  };

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

### New Hooks

- **`useActionState`** - manages state for form actions, tracking pending state and the last returned result.
- **`useFormStatus`** - reads the status of a parent `<form>` without prop drilling.
- **`useOptimistic`** - shows an optimistic state while an async action is in progress.
- **`use`** - reads resources (promises, context) during render.

### Server Components

React Server Components allow components to be rendered ahead of time, in an environment separate from the client application or SSR server. They can run at build time or on a per-request basis on a web server.

### Document Metadata

React 19 supports rendering `<title>`, `<meta>`, and `<link>` tags anywhere in your component tree. React automatically hoists them into the `<head>` section of the document:

```js
function BlogPost({ post }) {
  return (
    <article>
      <title>{post.title}</title>
      <meta name="author" content={post.author} />
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

### Ref as a Prop

Function components can now accept `ref` as a regular prop. `forwardRef` is no longer needed:

```js
function MyInput({ placeholder, ref }) {
  return <input placeholder={placeholder} ref={ref} />;
}
```

### Stylesheet Support

React 19 has built-in support for stylesheet precedence, ensuring styles are inserted in the correct order regardless of where they are rendered:

```js
function Component() {
  return (
    <Suspense fallback="loading...">
      <link rel="stylesheet" href="styles.css" precedence="default" />
      <article>...</article>
    </Suspense>
  );
}
```

### React Compiler

The React Compiler (currently in beta) automatically optimizes your React code. It memoizes components and hooks, reducing the need for manual `useMemo`, `useCallback`, and `React.memo` calls.

Source: [react.dev/blog/2024/12/05/react-19](https://react.dev/blog/2024/12/05/react-19)

## Essentials

Before diving into the in-depth guides, you should be familiar with these concepts:

- [JavaScript ES6+](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) (arrow functions, destructuring, modules, template literals)
- [TypeScript fundamentals](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
- [HTML and the DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)

The following chapters cover each React concept in depth, starting with [Components](./2-components.md).
