## TypeScript Development Philosophy

This codebase follows these core principles to ensure clean, maintainable, and type-safe TypeScript.

**Assumptions:** This guide assumes TypeScript 5.x with `strict: true` (ideally extending `@tsconfig/strictest` or equivalent), ES modules with `NodeNext` or `Bundler` module resolution, and `tsc --noEmit` running in CI on every commit. ESLint with `@typescript-eslint/strict-type-checked` and Prettier are expected to run cleanly. The principles apply equally to front-end (browser), Node.js, and library code — what changes is the runtime, not the type discipline. For framework-specific guidance, see [React Development Philosophy](react-development.md), [Vue Development Philosophy](vue-development.md), or [Node.js Development Philosophy](node-development.md).

### 1. Strict by Default

Turn the compiler on, all of it. Strict mode is the floor, not the ceiling.

- ✓ DO: Enable `strict: true`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noImplicitOverride`, `noFallthroughCasesInSwitch`; treat `tsc --noEmit` failures as build failures
- ✗ DON'T: Disable strict flags to silence noise; ship `// @ts-ignore` or `// @ts-expect-error` without an explanatory comment; rely on a non-strict base config

### 2. Readable

Optimise for human understanding. Code should be self-explanatory.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your function and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Descriptive names that convey intent; types that read like documentation; code that reads like prose
- ✗ DON'T: One-letter generics where a real name (`TUser`, `TItem`) clarifies; cryptic abbreviations; conditional-type gymnastics that require a whiteboard to decode

### 3. Concise

Keep code minimal without sacrificing clarity. Avoid unnecessary abstractions.

Functions should be short enough to hold in your head. If you can't see the entire function at once, or if you lose track of what variables hold, the function is probably doing too much. Deep nesting signals that the function is handling too many concerns. Consider extracting helper functions or restructuring.

- ✓ DO: Small focused functions; let inference work for locals; prefer `type` aliases over re-declaring shapes inline
- ✗ DON'T: Build elaborate generic wrappers for code that's used in one place; write functions that scroll for pages

### 4. Explicit

Prefer clarity over implicit behaviour. Make data flow and types obvious at the boundaries.

- ✓ DO: Annotate exported functions, public class methods, and module boundaries; use named exports; declare return types on anything crossing a module/package edge
- ✗ DON'T: Rely on inference for public APIs (a refactor can silently widen them); use default exports; lean on declaration merging where a plain type would do

### 5. Domain-Modelled

Encode invariants in the type system. If the compiler can rule out a bug, it will.

- ✓ DO: Discriminated unions for state machines and variant data; branded/nominal types for IDs and validated primitives; `readonly` on data that shouldn't be mutated; `as const` for literal unions
- ✗ DON'T: Pass raw `string` and `number` everywhere (`UserId` and `OrderId` aren't the same thing); model state as flat optional fields where a union would express it precisely

### 6. Inferred Where Helpful, Annotated at Boundaries

Trust inference inside a function; require explicit types where contracts live.

- ✓ DO: Let TS infer local variable types; annotate function parameters, return types of exported functions, and component props; use `satisfies` to check shape without widening
- ✗ DON'T: Annotate every `const x: string = "hello"`; leave the return type of an exported function to inference (it becomes part of the API by accident)

### 7. Narrowing-First

Write code the compiler can follow. Narrow with control flow rather than asserting your way through.

- ✓ DO: Use `typeof`, `instanceof`, `in`, equality with literals, and discriminated tags; write user-defined type guards (`x is Foo`) when narrowing logic is non-trivial
- ✗ DON'T: Reach for `as Foo` to "make the error go away"; cast through `as unknown as Foo`; pretend `null` isn't there

### 8. Total Functions

Handle every case of a union. Let the compiler tell you when a new variant arrives.

- ✓ DO: Use exhaustive `switch` on discriminants with a `never`-typed default branch (`const _exhaustive: never = value`); handle every key of a string-literal union
- ✗ DON'T: Leave silent fall-throughs that work today and break the moment someone adds a variant; rely on runtime guards where the type system already proves the case

### 9. No `any`, Rare `unknown`

`any` opts out of typechecking — every value it touches is poisoned. `unknown` keeps the discipline; reach for it at I/O boundaries and narrow before use.

- ✓ DO: Type external data as `unknown` and validate (Zod, Valibot, hand-written guards) before using it; use `unknown` for catch clauses (`catch (err: unknown)`); ban `any` via lint (`@typescript-eslint/no-explicit-any`)
- ✗ DON'T: Use `any` to silence the compiler; use `as any` as a refactor escape hatch; type a `fetch` response as `any` and hope the shape is right

### 10. Errors as Values or Typed Exceptions — Pick One

Either model failure in the return type (`Result<T, E>`) or throw with a typed error class. Mixing both in the same module makes call sites unpredictable.

- ✓ DO: Pick a convention per package or domain; if throwing, define an error class hierarchy and document what each function can throw; if returning, model `Result` consistently
- ✗ DON'T: Throw from some functions and return `{ ok, error }` from sibling functions in the same module; throw `string` or plain objects; swallow errors with empty `catch` blocks

### 11. Compositional

Build flexible systems through small, focused types and functions.

- ✓ DO: Small composable utility types; intersection (`A & B`) and union (`A | B`) over deep inheritance; constrained generics that say what they need (`<T extends { id: string }>`)
- ✗ DON'T: Deep class hierarchies; god-types with twenty optional fields; generic parameters that aren't constrained or used meaningfully

### 12. Testable

Design for testability from the start. Strong types make tests shorter; they don't replace them.

- ✓ DO: Inject dependencies behind interfaces; keep pure logic separate from I/O; type test fixtures with `satisfies` so they stay valid under refactors
- ✗ DON'T: Rely on global singletons that can't be substituted; type-cast mocks (`as unknown as Service`) — define a real interface instead

### 13. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Start simple; reach for a conditional type or mapped type only when it earns its place; let runtime validation handle what static types can't
- ✗ DON'T: Over-engineer for hypothetical requirements; write type-level Turing machines because you can; encode every business rule in the type system when a unit test would do

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 14. Predictable

Code should behave as expected without surprises. The same input should produce the same output and the same types.

- ✓ DO: Prefer pure functions and immutable data (`readonly`, `ReadonlyArray<T>`, `as const`); make state transitions explicit; resolve `Promise` rejections
- ✗ DON'T: Mutate function parameters; mix sync and async return paths in the same function; let `Promise` rejections go unhandled

### 15. Tooled

Let `tsc`, ESLint, and Prettier enforce the baseline so reviews can focus on design.

- ✓ DO: Run `tsc --noEmit`, `eslint` (with `@typescript-eslint/strict-type-checked`), and `prettier --check` in CI; fail the build on errors; enable editor integration
- ✗ DON'T: Disable rules without a `// eslint-disable-next-line ... -- reason: ...` justification; commit code Prettier would reformat; treat type errors as "we'll fix it later"

### 16. Well-Commented

Well-written code is self-evident about *what* it does. Comments add value when they explain *why*: the context, constraints, or reasoning that isn't obvious from the code itself.

- ✓ DO: Write TSDoc (`/** ... */`) on exported APIs; explain business rules, non-obvious invariants encoded in the types, and the reasoning behind a design choice; justify every `@ts-expect-error`
- ✗ DON'T: Narrate what the code does step-by-step; leave comments that duplicate the type signature; ship `@ts-ignore` without a reason
