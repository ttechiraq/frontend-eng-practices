---
sidebar_position: 11
---

# Testing

Testing React components helps you verify that they render correctly, respond to user interactions as expected, and handle edge cases gracefully.

## Recommended Tools

| Tool | Purpose |
|---|---|
| [Vitest](https://vitest.dev/) | Fast test runner with native ESM support |
| [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) | Renders components and queries the DOM the way users see it |
| [jsdom](https://github.com/jsdom/jsdom) / [happy-dom](https://github.com/nicedoc/happy-dom) | DOM environment for Node.js |
| [MSW](https://mswjs.io/) | Mocks network requests at the service worker level |
| [Playwright](https://playwright.dev/) / [Cypress](https://www.cypress.io/) | End-to-end browser testing |

## Setup with Vitest

Install the required packages:

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

Configure Vitest in `vite.config.ts`:

```ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    globals: true,
  },
});
```

Create the setup file:

```ts
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
```

Run tests:

```bash
npx vitest
```

## Testing Components

### Basic Rendering Test

```tsx
import { render, screen } from '@testing-library/react';
import { Greeting } from './Greeting';

test('renders a greeting', () => {
  render(<Greeting name="Alice" />);
  expect(screen.getByText('Hello, Alice!')).toBeInTheDocument();
});
```

### Testing User Interactions

Use `@testing-library/user-event` for realistic user interaction simulation:

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from './Counter';

test('increments on click', async () => {
  const user = userEvent.setup();
  render(<Counter />);

  const button = screen.getByRole('button', { name: /clicked 0 times/i });
  await user.click(button);

  expect(screen.getByRole('button', { name: /clicked 1 times/i })).toBeInTheDocument();
});
```

### Testing Form Submissions

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

test('submits the form with user input', async () => {
  const user = userEvent.setup();
  const handleSubmit = vi.fn();
  render(<LoginForm onSubmit={handleSubmit} />);

  await user.type(screen.getByLabelText(/email/i), 'test@example.com');
  await user.type(screen.getByLabelText(/password/i), 'secret123');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'secret123',
  });
});
```

### Testing Async Behavior

Use `findBy*` queries (which return promises) to wait for elements that appear after asynchronous operations:

```tsx
import { render, screen } from '@testing-library/react';
import { UserProfile } from './UserProfile';

test('loads and displays user data', async () => {
  render(<UserProfile userId="1" />);

  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  const heading = await screen.findByRole('heading', { name: /alice/i });
  expect(heading).toBeInTheDocument();
});
```

## Querying the DOM

React Testing Library provides queries ordered by priority. Prefer accessible queries that reflect how users find elements:

| Priority | Query | Use case |
|---|---|---|
| 1 | `getByRole` | Buttons, headings, links, inputs (most preferred) |
| 2 | `getByLabelText` | Form fields with labels |
| 3 | `getByPlaceholderText` | Inputs with placeholder text |
| 4 | `getByText` | Non-interactive text content |
| 5 | `getByDisplayValue` | Inputs showing a specific value |
| 6 | `getByAltText` | Images |
| 7 | `getByTestId` | Last resort when no accessible query works |

Each query has variants:

| Variant | Behavior |
|---|---|
| `getBy*` | Throws if not found (synchronous) |
| `queryBy*` | Returns `null` if not found (for asserting absence) |
| `findBy*` | Returns a promise; waits for the element to appear |

## Mocking Network Requests

Use MSW (Mock Service Worker) to intercept network requests:

```ts
// src/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/users/:id', ({ params }) => {
    return HttpResponse.json({ id: params.id, name: 'Alice' });
  }),
];
```

```ts
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```ts
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
import { server } from '../mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Testing Custom Hooks

Use `renderHook` from React Testing Library:

```tsx
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('increments counter', () => {
  const { result } = renderHook(() => useCounter());

  expect(result.current.count).toBe(0);

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

## Testing with Routing

Wrap components in a `MemoryRouter` for testing:

```tsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter } from 'react-router';
import { App } from './App';

test('navigates to about page', async () => {
  const user = userEvent.setup();
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );

  await user.click(screen.getByRole('link', { name: /about/i }));
  expect(screen.getByRole('heading', { name: /about/i })).toBeInTheDocument();
});
```

## Testing Patterns

### Arrange-Act-Assert

Structure every test with three clear phases:

```tsx
test('adds item to cart', async () => {
  // Arrange
  const user = userEvent.setup();
  render(<ProductPage product={mockProduct} />);

  // Act
  await user.click(screen.getByRole('button', { name: /add to cart/i }));

  // Assert
  expect(screen.getByText(/1 item in cart/i)).toBeInTheDocument();
});
```

### What to Test

- **Do test** user-visible behavior: rendered output, interactions, navigation, form submissions.
- **Don't test** implementation details: internal state values, component method calls, hook internals.

### Code Coverage

Generate a coverage report:

```bash
npx vitest --coverage
```

Use coverage as a guide, not a target. High coverage does not guarantee correct behavior, and chasing 100% often leads to brittle tests.
