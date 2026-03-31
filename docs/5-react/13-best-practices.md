---
sidebar_position: 13
---

# Best Practices

This chapter covers recommended patterns, coding conventions, accessibility, security considerations, and project organization for React applications.

## Project Structure

### Feature-Based Organization

Organize code by feature rather than by file type:

```
src/
├── features/
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   ├── useAuth.ts
│   │   ├── authApi.ts
│   │   └── auth.test.tsx
│   ├── users/
│   │   ├── UserList.tsx
│   │   ├── UserProfile.tsx
│   │   ├── useUsers.ts
│   │   └── usersApi.ts
│   └── dashboard/
│       ├── Dashboard.tsx
│       └── DashboardStats.tsx
├── components/
│   ├── Button.tsx
│   ├── Modal.tsx
│   └── Spinner.tsx
├── hooks/
│   ├── useDebounce.ts
│   └── useMediaQuery.ts
├── lib/
│   ├── api.ts
│   └── utils.ts
├── App.tsx
└── main.tsx
```

- `features/` -- domain-specific components, hooks, and logic
- `components/` -- shared, reusable UI components
- `hooks/` -- shared custom hooks
- `lib/` -- utilities, API clients, and configuration

### Naming Conventions

| Symbol | Convention | Example |
|---|---|---|
| Components | PascalCase | `UserProfile.tsx` |
| Hooks | camelCase with `use` prefix | `useAuth.ts` |
| Utilities | camelCase | `formatDate.ts` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| Types/Interfaces | PascalCase | `User`, `CreateUserDto` |
| Test files | Same name with `.test.tsx` suffix | `UserProfile.test.tsx` |

## Component Patterns

### Keep Components Small and Focused

Each component should do one thing. If a component grows beyond 100-150 lines, consider extracting parts into child components or custom hooks:

```tsx
// Instead of one large component...
function UserDashboard() {
  // ...200 lines of mixed concerns
}

// ...break it into focused pieces
function UserDashboard() {
  return (
    <div>
      <UserHeader />
      <UserStats />
      <RecentActivity />
    </div>
  );
}
```

### Prefer Composition Over Props

Use the `children` prop and composition rather than passing large numbers of configuration props:

```tsx
// Less flexible: many config props
<Card title="Hello" subtitle="World" icon={<Icon />} footer={<Footer />} />

// More flexible: composition
<Card>
  <CardHeader>
    <Icon />
    <h2>Hello</h2>
    <p>World</p>
  </CardHeader>
  <CardBody>...</CardBody>
  <CardFooter>...</CardFooter>
</Card>
```

### Separate Logic from Presentation

Extract business logic and state management into custom hooks, keeping components focused on rendering:

```tsx
function useUserForm() {
  const [formData, setFormData] = useState({ name: '', email: '' });
  const [errors, setErrors] = useState({});

  function handleChange(e) {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  }

  function validate() {
    const newErrors = {};
    if (!formData.name) newErrors.name = 'Name is required';
    if (!formData.email) newErrors.email = 'Email is required';
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }

  return { formData, errors, handleChange, validate };
}

function UserForm() {
  const { formData, errors, handleChange, validate } = useUserForm();

  return (
    <form onSubmit={(e) => { e.preventDefault(); if (validate()) submit(formData); }}>
      <input name="name" value={formData.name} onChange={handleChange} />
      {errors.name && <span>{errors.name}</span>}
      <input name="email" value={formData.email} onChange={handleChange} />
      {errors.email && <span>{errors.email}</span>}
      <button type="submit">Save</button>
    </form>
  );
}
```

### Avoid Prop Drilling

When data needs to pass through many levels of components, consider:

1. **Component composition** -- restructure your tree so the data-consuming component is closer to the data source.
2. **Context** -- for infrequently changing values like theme or auth.
3. **State management library** -- for complex global state.

## Common Anti-Patterns

### Mutating State

Never modify state directly. Always produce a new value:

```tsx
// Bad
const handleClick = () => {
  user.name = 'New Name'; // mutation!
  setUser(user); // same reference, React skips re-render
};

// Good
const handleClick = () => {
  setUser({ ...user, name: 'New Name' });
};
```

### Unnecessary useEffect

Don't use effects for logic that can be computed during rendering or in event handlers:

```tsx
// Bad: effect to derive data
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');

useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// Good: compute during render
const fullName = firstName + ' ' + lastName;
```

```tsx
// Bad: effect to handle an event
useEffect(() => {
  if (submitted) {
    sendAnalytics('form_submitted');
  }
}, [submitted]);

// Good: handle in the event handler
function handleSubmit() {
  setSubmitted(true);
  sendAnalytics('form_submitted');
}
```

Source: [react.dev/learn/you-might-not-need-an-effect](https://react.dev/learn/you-might-not-need-an-effect)

### Defining Components Inside Components

Never define a component inside another component. It recreates the inner component on every render, destroying its state and DOM:

```tsx
// Bad
function Parent() {
  function Child() { return <div>child</div>; }
  return <Child />;
}

// Good
function Child() { return <div>child</div>; }
function Parent() { return <Child />; }
```

## TypeScript

### Typing Props

Use interfaces or type aliases for component props:

```tsx
interface UserCardProps {
  user: User;
  onSelect: (id: string) => void;
  variant?: 'compact' | 'full';
}

function UserCard({ user, onSelect, variant = 'full' }: UserCardProps) {
  return (
    <div onClick={() => onSelect(user.id)}>
      <h3>{user.name}</h3>
      {variant === 'full' && <p>{user.bio}</p>}
    </div>
  );
}
```

### Typing Hooks

```tsx
// useState with explicit type
const [user, setUser] = useState<User | null>(null);

// useRef with HTML element type
const inputRef = useRef<HTMLInputElement>(null);

// useReducer with typed action
type Action = { type: 'increment' } | { type: 'set'; value: number };
const [state, dispatch] = useReducer(reducer, { count: 0 });
```

### Typing Context

```tsx
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | null>(null);

function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}
```

## Accessibility (a11y)

### Semantic HTML

Use semantic elements instead of generic `<div>` and `<span>`:

```tsx
// Bad
<div onClick={handleClick}>Submit</div>

// Good
<button onClick={handleClick}>Submit</button>
```

### ARIA Attributes

Add ARIA attributes when semantic HTML is not sufficient:

```tsx
<button aria-label="Close dialog" onClick={onClose}>
  <XIcon />
</button>

<div role="alert" aria-live="polite">
  {errorMessage}
</div>
```

### Focus Management

Manage focus intentionally, especially in modals and dynamic content:

```tsx
function Dialog({ isOpen, onClose, children }) {
  const closeRef = useRef(null);

  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div role="dialog" aria-modal="true">
      {children}
      <button ref={closeRef} onClick={onClose}>Close</button>
    </div>
  );
}
```

### Keyboard Navigation

Ensure all interactive elements are keyboard-accessible. Test by navigating your app using only the Tab key and Enter/Space.

## Security

### XSS Prevention

React escapes all values embedded in JSX by default. Avoid using `dangerouslySetInnerHTML` unless you trust and sanitize the content:

```tsx
// Safe: React escapes this automatically
<p>{userInput}</p>

// Dangerous: bypasses React's escaping
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

If you must render raw HTML, sanitize it first with a library like [DOMPurify](https://github.com/cure53/DOMPurify):

```tsx
import DOMPurify from 'dompurify';

function RichContent({ html }) {
  const clean = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

### Environment Variables

Never expose secrets in client-side code. In Next.js, only variables prefixed with `NEXT_PUBLIC_` are included in the browser bundle. In Vite, only variables prefixed with `VITE_` are exposed:

```bash
# Server-only (not in browser bundle)
DATABASE_URL=postgres://...
API_SECRET=...

# Client-accessible
NEXT_PUBLIC_API_URL=https://api.example.com
VITE_API_URL=https://api.example.com
```

## Error Handling

### Error Boundaries

Error boundaries catch JavaScript errors during rendering and display a fallback UI instead of crashing the entire app:

```tsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error('Error caught by boundary:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

Usage:

```tsx
<ErrorBoundary>
  <UserProfile />
</ErrorBoundary>
```

Libraries like [react-error-boundary](https://github.com/bvaughn/react-error-boundary) provide a simpler API with hooks support.

## Keeping Up to Date

React follows semantic versioning. Major versions may include breaking changes, but the team provides codemods and detailed migration guides.

- Check the [React Blog](https://react.dev/blog) for release announcements.
- Use `npx react-codemod` to automate migration tasks.
- Update dependencies regularly to receive security patches and performance improvements.

Source: [react.dev/blog](https://react.dev/blog)
