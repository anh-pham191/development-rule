---
name: react-development
description: >
  React 18/19 development patterns, hooks idioms, and component architecture for
  building maintainable, accessible React applications. Use when: (1) Working on
  React projects (SPAs, Next.js, Remix), (2) Writing or reviewing function
  components and hooks, (3) Choosing state management (server vs URL vs local),
  (4) Wiring up data fetching with TanStack Query / RTK Query / SWR, (5) Writing
  React Testing Library tests, (6) Debugging effects, re-renders, or stale
  closures, (7) Typing props and event handlers with TypeScript, (8) Migrating
  class components to hooks. Assumes function components + hooks only and
  TypeScript strict mode where practical. Extends the front-end JavaScript skill.
---

# React Development

Core principles for writing clean, maintainable, accessible React 18/19 code with function components and hooks.

## Philosophy

### 1. Function-First
Function components and hooks only. No new class components.
- DO: `function UserCard({ user }: Props) { ... }` with hooks
- DON'T: Introduce `class extends React.Component` in new code

### 2. Readable
Optimise for human understanding. JSX should mirror the UI it produces.
- DO: Descriptive component and hook names; flat JSX with extracted children
- DON'T: Deeply nested ternaries inside JSX

### 3. Composable Components
Small components, single responsibility. Lift state up; compose via props and `children`.
- DO: Extract a subtree the moment it earns its own name
- DON'T: God components that fetch, transform, render and animate

### 4. Explicit
Make data flow visible — props in, events out, dependencies declared.
- DO: List every reactive value in effect dependency arrays
- DON'T: Capture stale values via closures or hidden refs

### 5. State-Discipline
Server state, URL state, and local UI state are different problems.
- DO: TanStack Query for server state; router for URL state; `useState` for UI
- DON'T: Cache server data in `useState` and reinvent refetch/loading/error

### 6. Hooks-Pure
Hooks rely on call order — keep them at the top level.
- DO: Enable `eslint-plugin-react-hooks` and obey both rules
- DON'T: Call hooks in conditions, loops, or after an early return

### 7. Effect-Minimal
`useEffect` is for syncing with external systems, not for deriving state.
- DO: Compute derived values during render; use event handlers for user actions
- DON'T: Copy props into state inside an effect

### 8. Accessible
Semantic HTML and keyboard support are baseline, not optional.
- DO: Real `<button>`s, labelled inputs, focus management on route changes
- DON'T: `<div onClick>` without role, tabIndex, and key handlers

### 9. Performant
Memoise only when profiling shows a cost.
- DO: Stable list keys; `React.lazy` for heavy routes; measure first
- DON'T: Reflexively wrap every callback in `useCallback`

### 10. Typed
TypeScript strict mode; type props, state, and event handlers.
- DO: `React.ChangeEvent<HTMLInputElement>`, prop interfaces
- DON'T: `any`, `as unknown as T`, or `{}` for prop types

### 11. Testable
Test what the user sees with React Testing Library.
- DO: `getByRole`, `userEvent`, MSW for network
- DON'T: Snapshot everything; assert on class names or internals

### 12. Pragmatic
Start simple, add machinery when concrete pain appears.
- DO: `useState` first, reach for context or libraries when justified
- DON'T: Adopt Redux on day one of a five-screen app

## Component Composition

Keep components small. Compose with `children` and dedicated slot props rather than configuration explosions.

```tsx
type CardProps = React.PropsWithChildren<{
  title: string;
  actions?: React.ReactNode;
}>;

export function Card({ title, actions, children }: CardProps) {
  return (
    <section className="card">
      <header className="card__header">
        <h2>{title}</h2>
        {actions}
      </header>
      <div className="card__body">{children}</div>
    </section>
  );
}

// Usage — composition over configuration
<Card title="Invoices" actions={<Button>New</Button>}>
  <InvoiceList />
</Card>;
```

Prefer extracting a named child component the moment a JSX subtree earns a name (`InvoiceRow`, `EmptyState`).

## Hooks Patterns

### `useState` — local UI state

```tsx
const [isOpen, setIsOpen] = useState(false);
const toggle = () => setIsOpen(prev => !prev);
```

Use the updater form (`setX(prev => ...)`) whenever the next value depends on the previous.

### `useEffect` — synchronise with external systems

`useEffect` is for the DOM, subscriptions, timers, and integrations with non-React libraries. It is **not** for deriving values from props/state.

```tsx
// ✓ Synchronising with an external system
useEffect(() => {
  const id = window.setInterval(tick, 1000);
  return () => window.clearInterval(id);
}, []);

// ✗ Deriving state — compute during render instead
const fullName = `${firstName} ${lastName}`; // not an effect
```

### Custom hooks — reusable behaviour

```tsx
export function useDebouncedValue<T>(value: T, delayMs = 300): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = window.setTimeout(() => setDebounced(value), delayMs);
    return () => window.clearTimeout(id);
  }, [value, delayMs]);

  return debounced;
}
```

Custom hooks must start with `use` so the React Hooks ESLint rules apply.

## Data Fetching

Don't fetch in `useEffect` for application data — use a library that handles caching, deduplication, retries, and revalidation.

```tsx
import { useQuery } from '@tanstack/react-query';

type Invoice = { id: string; total: number; paid: boolean };

async function fetchInvoices(): Promise<Invoice[]> {
  const res = await fetch('/api/invoices');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

export function InvoiceList() {
  const { data, isPending, isError, error } = useQuery({
    queryKey: ['invoices'],
    queryFn: fetchInvoices,
  });

  if (isPending) return <Spinner aria-label="Loading invoices" />;
  if (isError) return <ErrorBanner message={error.message} />;

  return (
    <ul>
      {data.map(invoice => (
        <li key={invoice.id}>{invoice.total}</li>
      ))}
    </ul>
  );
}
```

For mutations, use `useMutation` and invalidate the affected query keys on success.

## State Management

Pick the smallest tool that fits.

| State kind | Tool |
|---|---|
| Local UI state (open/closed, hover, input value) | `useState` / `useReducer` |
| Cross-component but local subtree | Lift state up; pass props |
| Server data (lists, entities, paginated results) | TanStack Query / RTK Query / SWR |
| URL-shareable state (filters, page, tab) | Router (`useSearchParams`) |
| Genuinely global app state (auth, theme) | Context, or a library (Zustand, Redux Toolkit) |

Reach for context when prop-drilling becomes painful and the value is genuinely tree-wide. Reach for a global store only when context churn causes real re-render problems.

```tsx
// Context for genuinely tree-wide values
const ThemeContext = createContext<'light' | 'dark'>('light');

export function useTheme(): 'light' | 'dark' {
  return useContext(ThemeContext);
}
```

## Testing

Use React Testing Library with `userEvent`. Query by accessible role and name — the same way a user (or screen reader) navigates.

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('submits with the entered credentials', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();

    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'anh@example.com');
    await user.type(screen.getByLabelText(/password/i), 'hunter2');
    await user.click(screen.getByRole('button', { name: /sign in/i }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: 'anh@example.com',
      password: 'hunter2',
    });
  });

  it('shows an error when the email is missing', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={vi.fn()} />);

    await user.click(screen.getByRole('button', { name: /sign in/i }));

    expect(screen.getByRole('alert')).toHaveTextContent(/email is required/i);
  });
});
```

Mock the network with MSW so the components under test exercise the same code paths as production.

## Performance

Memoise when you measure a cost, not before.

```tsx
// React.memo — only useful if the parent re-renders often AND props are referentially stable
export const ExpensiveRow = React.memo(function ExpensiveRow({ item }: { item: Item }) {
  return <Row item={item} />;
});

// useMemo — for expensive pure computations
const sortedItems = useMemo(
  () => [...items].sort((a, b) => a.name.localeCompare(b.name)),
  [items],
);

// useCallback — for stable references passed to memoised children or effect deps
const handleSelect = useCallback((id: string) => {
  setSelectedId(id);
}, []);
```

What to avoid:

- Wrapping every callback in `useCallback` "for safety" — the wrapper itself costs memory and inhibits optimisations
- `useMemo` for cheap computations like `a + b` or `items.length`
- `React.memo` on components whose props are new objects/arrays each render (the memo never hits)
- Using array index as a key for lists that reorder, insert, or remove

Route-level code splitting is almost always worth it:

```tsx
const SettingsPage = React.lazy(() => import('./SettingsPage'));

<Suspense fallback={<PageSpinner />}>
  <SettingsPage />
</Suspense>;
```

## Typing Props & Events

Type props with an `interface` or `type`. Use `React.PropsWithChildren` when you accept arbitrary children.

```tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
  children: React.ReactNode;
}

export function Button({ variant = 'primary', disabled, onClick, children }: ButtonProps) {
  return (
    <button
      type="button"
      className={`btn btn--${variant}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

Common event types:

```tsx
function handleChange(e: React.ChangeEvent<HTMLInputElement>) { /* e.target.value */ }
function handleSubmit(e: React.FormEvent<HTMLFormElement>) { e.preventDefault(); }
function handleKey(e: React.KeyboardEvent<HTMLInputElement>) { /* e.key */ }
function handleClick(e: React.MouseEvent<HTMLButtonElement>) { /* ... */ }
```

Refs:

```tsx
const inputRef = useRef<HTMLInputElement>(null);
// later: inputRef.current?.focus();
```

Generic components:

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyOf: (item: T) => React.Key;
}

export function List<T>({ items, renderItem, keyOf }: ListProps<T>) {
  return <ul>{items.map(item => <li key={keyOf(item)}>{renderItem(item)}</li>)}</ul>;
}
```

Avoid `React.FC` — it implies `children` even when the component doesn't accept them and adds little value. Declare the prop type and use a plain function.
