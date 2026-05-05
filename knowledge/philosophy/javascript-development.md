## Front-end JavaScript Development Philosophy

This codebase follows these core principles to ensure clean, maintainable, and modern front-end JavaScript.

**Assumptions:**

- Modern ES6+ JavaScript targeting evergreen browsers
- ES modules (`import`/`export`) as the standard module system
- Vue with its ecosystem (Pinia, Vite) for SPAs—Vue-specific principles: see [Vue Development Philosophy](vue-development.md)
- TypeScript is preferred for applications with significant complexity; conventions are documented separately
- Commit `package-lock.json` to ensure reproducible builds

### 1. Modern & Standards-Compliant

Embrace ES6+ features and modern browser APIs. Avoid legacy patterns and polyfills for obsolete browsers.

- ✓ DO: Use `const`/`let`, arrow functions, destructuring, template literals, `async`/`await`, optional chaining (`?.`), nullish coalescing (`??`)
- ✗ DON'T: Use `var`, callback pyramids, string concatenation for templates, target IE11 or other legacy browsers

### 2. Readable

Optimise for human understanding. Code should be self-documenting.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your function and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Use descriptive names, early returns, small focused functions, consistent formatting; code that reads like prose
- ✗ DON'T: Write clever one-liners, chain excessive methods, use cryptic abbreviations, nest deeply; code that requires tribal knowledge to understand

### 3. Explicit

Make behavior clear and obvious. Avoid magic and implicit side effects.

- ✓ DO: Use ES modules (`import`/`export`), show data flow clearly, declare dependencies explicitly, use named exports for better tree-shaking, destructure function parameters
- ✗ DON'T: Use CommonJS (`require`/`module.exports`), rely on hoisting, use implicit globals, create hidden side effects, overuse default exports

### 4. Compositional

Build flexible systems through small, focused units that compose together.

Functions should be short enough to hold in your head. If you can't see the entire function at once, or if you lose track of what variables hold, the function is probably doing too much. Deep nesting signals that the function is handling too many concerns. Consider extracting helper functions or restructuring.

- ✓ DO: Write small pure functions, use composition over inheritance, create focused single-purpose modules; short functions with few variables
- ✗ DON'T: Create god objects, use deep class hierarchies, build monolithic components; functions that scroll for pages

### 5. Testable

Design for testability from the start with proper separation of concerns.

- ✓ DO: Write unit tests, inject dependencies, separate side effects from pure logic, mock external services
- ✗ DON'T: Create untestable DOM dependencies, rely on global state, hardcode API endpoints

### 6. Accessible

Accessibility is a core requirement, not an afterthought.

- ✓ DO: Use semantic HTML, add ARIA attributes where needed, ensure keyboard navigation, maintain focus management, provide text alternatives
- ✗ DON'T: Use `<div>` for everything, rely solely on color for meaning, trap keyboard focus, skip heading levels

### 7. Performant

Be mindful of performance from the start, especially bundle size and runtime efficiency.

- ✓ DO: Lazy load routes/components, tree-shake imports, debounce expensive operations, use `requestAnimationFrame` for visual updates
- ✗ DON'T: Import entire libraries for single functions, cause layout thrashing, block the main thread, ignore bundle size

### 8. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Start simple, refactor when patterns emerge, use vanilla JS for simple interactions, profile before optimising; write code that is immediately understandable
- ✗ DON'T: Over-engineer for hypothetical requirements, reach for a framework when a few lines suffice, prematurely optimise; write "clever" code that requires explanation to understand

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 9. Predictable

Code should behave as expected without surprises. Manage state and errors explicitly.

- ✓ DO: Use pure functions where possible, use try/catch with async/await, handle Promise rejections, provide meaningful error messages, use error boundaries in components, make state changes traceable, prefer immutable updates
- ✗ DON'T: Mutate shared state unexpectedly, ignore unhandled Promise rejections, use empty catch blocks, let errors propagate silently, create hidden state dependencies

### 10. Linted & Formatted

Use automated tooling to enforce consistency and catch errors before runtime.

- ✓ DO: Configure ESLint with strict rules, use Prettier for formatting, run linting in CI, enable editor integration, fix linting errors before committing
- ✗ DON'T: Disable rules without justification, commit unlinted code, debate formatting manually, ignore linting warnings

### 11. Secure

Follow security best practices to protect users and data.

- ✓ DO: Sanitise user input before rendering, use `textContent` over `innerHTML`, keep dependencies updated, run `npm audit` regularly, validate on the server
- ✗ DON'T: Use `v-html` or `innerHTML` with untrusted data, store sensitive tokens in localStorage, trust client-side validation alone, ignore dependency vulnerabilities

### 12. Well-Commented

Well-written code is self-evident about *what* it does. Comments add value when they explain *why*: the context, constraints, or reasoning that isn't obvious from the code itself.

- ✓ DO: Explain business rules, non-obvious constraints, or the reasoning behind a design choice; note when code works around an external limitation
- ✗ DON'T: Narrate what the code does step-by-step; leave comments that duplicate the function name

---

### Framework Guidance

#### When to Use What

- **Vanilla JavaScript** - Simple interactions, progressive enhancement, small utilities
- **Vue SPA** - Complex applications with significant state management and routing needs

#### Vue Ecosystem

When building SPAs, we use Vue with its standard ecosystem (Vue 3, Pinia, Vue Router, Vite). For Vue and Pinia principles and best practices, see [Vue Development Philosophy](vue-development.md).
