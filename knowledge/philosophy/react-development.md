## React Development Philosophy

This document extends [Front-end JavaScript Development Philosophy](javascript-development.md).

React's component model and hooks shape how we think about UI. We believe in leaning into React's data-down/events-up flow and its declarative rendering model rather than fighting it with imperative escapes.

**Assumptions:**

- React 18 or 19 with concurrent rendering enabled
- Function components and hooks only — no class components in new code
- TypeScript (strict mode) is strongly recommended for any non-trivial app
- Build via Vite for SPAs, or Next.js when SSR/SSG is required
- `package.json` with a committed lockfile (`package-lock.json`, `pnpm-lock.yaml`, or `yarn.lock`) for reproducible builds

### 1. Function-First

Write function components with hooks. Class components are a legacy form that should not appear in new code, and existing classes should migrate when touched.

- ✓ DO: Use function components, `useState`, `useReducer`, custom hooks
- ✗ DON'T: Introduce new class components, lifecycle methods, or `this`-bound handlers

### 2. Readable

Optimise for human understanding. Components should be self-explanatory.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your component and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Descriptive component and hook names (`UserProfileCard`, `useDebouncedValue`); JSX that reads like the UI it produces
- ✗ DON'T: Cryptic prop names; rendering logic buried under conditional ternaries five levels deep

### 3. Composable Components

Build complex UIs from small components that each do one thing. Lift state up to the lowest common ancestor; pass it down via props.

Functions should be short enough to hold in your head. If you can't see the entire component at once, or if you lose track of which props feed which JSX, the component is probably doing too much. Extract child components or custom hooks.

- ✓ DO: Single-purpose components; `children` and render-prop patterns for flexible composition; extract subtrees that have an obvious name
- ✗ DON'T: God components that fetch, transform, render, and animate; prop-drilling through five layers when a child component would lift the concern naturally

### 4. Explicit

Prefer clarity over implicit behaviour. Make data flow obvious.

- ✓ DO: Pass data via props; show effect dependencies explicitly in the dependency array; name custom hooks `use*` so React can lint them
- ✗ DON'T: Hidden global mutable state; effects with stale closures; reaching into refs to bypass React's rendering model

### 5. State-Discipline

Not all state is the same. Distinguish server state, URL state, form state, and local UI state, and use the right tool for each.

- ✓ DO: TanStack Query / RTK Query / SWR for server state; the URL (router params, search params) for shareable state; `useState`/`useReducer` for ephemeral UI state; a form library (React Hook Form) for forms
- ✗ DON'T: Cache server responses in `useState` and re-implement loading/error/refetch yourself; put everything into a single global store; duplicate the URL into local state

### 6. Hooks-Pure

Respect the Rules of Hooks. They are not a style preference — React relies on call order to associate state with components.

- ✓ DO: Call hooks at the top level of components and custom hooks; extract reusable behaviour into `use*` functions; enable `eslint-plugin-react-hooks`
- ✗ DON'T: Call hooks inside conditions, loops, or after early returns; call hooks from regular functions or class components

### 7. Effect-Minimal

`useEffect` is for synchronising React state with an external system (DOM, subscriptions, network), not for deriving values or chaining state updates.

- ✓ DO: Compute derived values during render; use event handlers for user-initiated work; reach for `useEffect` only when an external system needs to know
- ✗ DON'T: Use effects to copy props into state; chain `setState` calls in effects to "react" to other state changes; fetch in `useEffect` when a data-fetching library would do it better

### 8. Accessible

Accessibility is a baseline, not a feature. Build with semantic HTML and assistive-technology users in mind from the start.

- ✓ DO: Use the right element (`<button>`, `<a>`, `<label>`, `<nav>`); manage focus on route and modal transitions; support keyboard interaction; pair icons with text or `aria-label`
- ✗ DON'T: Click-handlers on `<div>` without role/tabIndex/keyboard support; ARIA attributes that duplicate or contradict semantic HTML; positive `tabindex` values

### 9. Performant

Render performance matters, but premature memoisation costs more than it saves. Measure, then optimise.

- ✓ DO: Profile with React DevTools before optimising; key lists by stable IDs; code-split heavy routes with `React.lazy` + `Suspense`; reach for `useMemo`/`useCallback`/`React.memo` only when profiling shows a real cost
- ✗ DON'T: Wrap every callback in `useCallback` "just in case"; use array index as a key for lists that reorder; ship a single megabundle when route-level splitting is trivial

### 10. Typed

TypeScript catches a large class of UI bugs at compile time — wrong prop shapes, missing event types, narrowed-incorrectly state.

- ✓ DO: `strict: true`; type props with interfaces or type aliases; use `React.FormEvent`, `React.ChangeEvent<HTMLInputElement>` etc. for handlers; let inference work where it can
- ✗ DON'T: Use `any` to silence the compiler; cast through `as unknown as T`; type props as `object` or `{}`

### 11. Testable

Test components the way users use them. React Testing Library queries by accessible name and role for a reason.

- ✓ DO: Assert on what the user sees (`screen.getByRole('button', { name: /save/i })`); drive interactions with `userEvent`; mock the network at the boundary (MSW), not at the hook
- ✗ DON'T: Snapshot every component as a substitute for behavioural tests; assert on internal state, class names, or component instance methods; test implementation details that refactors should be free to change

### 12. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Start with `useState` and local components; reach for context, reducers, or a state library when concrete pain emerges; write components that are immediately understandable
- ✗ DON'T: Reach for Redux on day one of a five-screen app; build a generic abstraction for a single caller; write "clever" hooks that require a tour to understand

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 13. Predictable

Components should behave the same way given the same props and state. Surprises break user trust and complicate debugging.

- ✓ DO: Pure render functions; stable list keys; idempotent reducers; treat props and state as immutable; derive values during render so React's diffing stays consistent
- ✗ DON'T: Mutate props or state in place (`state.items.push(...)`); read or write the DOM directly inside render; rely on render-order side effects between sibling components

### 14. Well-Commented

Well-written components are self-evident about *what* they render. Comments add value when they explain *why*: the constraint, the workaround, the design choice that isn't obvious from the JSX.

- ✓ DO: Explain why an effect has an unusual dependency list; note when a `useMemo` exists because profiling showed a cost; document an accessibility workaround for a specific screen reader
- ✗ DON'T: Narrate the JSX (`// render the button`); duplicate the component name in a comment above it
