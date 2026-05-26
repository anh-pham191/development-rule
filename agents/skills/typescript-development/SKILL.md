---
name: typescript-development
description: >
  TypeScript development philosophy, idioms, and patterns for writing clean, type-safe,
  maintainable TypeScript code. Use when: (1) Working on TypeScript projects (front-end,
  Node.js, or library), (2) Writing new TypeScript code, (3) Reviewing TypeScript code,
  (4) Refactoring JavaScript to TypeScript, (5) Configuring `tsconfig.json`, ESLint, or
  type-checking in CI, (6) Designing type-safe APIs and domain models. Covers strict-mode
  configuration, discriminated unions, branded types, narrowing, generics, and tooling
  for TypeScript 5.x on ESM.
---

# TypeScript Development

Core principles for writing clean, maintainable, and type-safe TypeScript on the 5.x line.

## Philosophy

### 1. Strict by Default
Turn the compiler on, all of it. Strict mode is the floor.
- DO: Enable `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`; fail CI on `tsc --noEmit` errors
- DON'T: Disable strict flags; ship `@ts-ignore` without a reason

### 2. Readable
Optimise for human understanding. Code should be self-explanatory.
- DO: Descriptive names; types that read like documentation
- DON'T: Cryptic generics or conditional-type gymnastics

### 3. Concise
Keep code minimal without sacrificing clarity.
- DO: Small focused functions; let inference work locally
- DON'T: Generic wrappers around code used in one place

### 4. Explicit
Prefer clarity over implicit behaviour at module boundaries.
- DO: Annotate exported functions, public methods, component props; use named exports
- DON'T: Rely on inference for public APIs; use default exports

### 5. Domain-Modelled
Encode invariants in the type system.
- DO: Discriminated unions, branded types for IDs, `readonly`, `as const`
- DON'T: Pass raw `string`/`number` everywhere; flatten state into optional fields

### 6. Inferred Where Helpful, Annotated at Boundaries
Trust inference inside a function; require explicit types where contracts live.
- DO: Annotate parameters and exported return types; use `satisfies` to check without widening
- DON'T: Annotate every local; leave exported return types to inference

### 7. Narrowing-First
Write code the compiler can follow.
- DO: Narrow with `typeof`, `in`, discriminants, custom type guards
- DON'T: Reach for `as Foo` to silence errors

### 8. Total Functions
Handle every case of a union.
- DO: Exhaustive `switch` with a `never` default branch
- DON'T: Silent fall-throughs that break when a variant is added

### 9. No `any`, Rare `unknown`
- DO: Type external input as `unknown` and validate; ban `any` via lint
- DON'T: Use `any` or `as any` to silence the compiler

### 10. Errors as Values or Typed Exceptions
Pick one convention per package.
- DO: Either return `Result<T, E>` or throw typed errors — consistently
- DON'T: Mix both styles in the same module; throw strings or plain objects

### 11. Compositional
Build through small composable types.
- DO: Constrained generics, intersections/unions, small utility types
- DON'T: Deep class hierarchies or god-types

### 12. Testable
- DO: Inject dependencies behind interfaces; keep pure logic separate from I/O
- DON'T: Type-cast mocks; rely on global singletons

### 13. Pragmatic
- DO: Start simple; let runtime validation handle what static types can't
- DON'T: Write type-level Turing machines just because you can

### 14. Predictable
- DO: Prefer pure functions and `readonly` data; handle `Promise` rejections
- DON'T: Mutate parameters; let rejections go unhandled

### 15. Tooled
- DO: Run `tsc --noEmit`, ESLint (`@typescript-eslint/strict-type-checked`), Prettier in CI
- DON'T: Disable lints without justification

### 16. Well-Commented
- DO: TSDoc on exports; explain *why* and justify each `@ts-expect-error`
- DON'T: Restate the type signature in prose

## tsconfig essentials

The non-negotiable flags for a new TypeScript 5.x project:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],

    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "noPropertyAccessFromIndexSignature": true,

    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "esModuleInterop": true,
    "skipLibCheck": true,

    "noEmit": true
  },
  "include": ["src/**/*"]
}
```

For browser code, swap `lib` to `["ES2022", "DOM", "DOM.Iterable"]` and `module`/`moduleResolution` to `"Bundler"`.

Extending `@tsconfig/strictest` is a reasonable shortcut — but read what it enables before relying on it.

## Discriminated Unions & Exhaustiveness

Model variant data with a literal discriminant. The compiler enforces totality.

```ts
type Result<T, E> =
  | { kind: "ok"; value: T }
  | { kind: "err"; error: E };

type FetchState =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; message: string };

function render(state: FetchState): string {
  switch (state.status) {
    case "idle":
      return "Press load.";
    case "loading":
      return "Loading…";
    case "success":
      return `Hello, ${state.data.name}`;
    case "error":
      return `Failed: ${state.message}`;
    default: {
      // If a new variant is added, this line becomes a type error.
      const _exhaustive: never = state;
      throw new Error(`Unhandled state: ${_exhaustive as string}`);
    }
  }
}
```

Avoid flat shapes like `{ loading: boolean; data?: User; error?: string }` — they admit illegal combinations (loading and error at once).

## Branded / Nominal Types

TypeScript is structural; brands give you nominal-style safety for primitives.

```ts
type Brand<T, B> = T & { readonly __brand: B };

export type UserId = Brand<string, "UserId">;
export type OrderId = Brand<string, "OrderId">;
export type EmailAddress = Brand<string, "EmailAddress">;

// Smart constructor — the only way to mint one
export function emailAddress(raw: string): EmailAddress {
  if (!raw.includes("@")) throw new Error(`Invalid email: ${raw}`);
  return raw as EmailAddress;
}

function sendInvoice(to: EmailAddress, order: OrderId): void {
  // ...
}

const u: UserId = "u_123" as UserId;
const o: OrderId = "o_456" as OrderId;

// sendInvoice(u, o); // ✗ Type error — UserId is not an EmailAddress
sendInvoice(emailAddress("a@b.com"), o); // ✓
```

The brand exists only at compile time — zero runtime cost.

## Type Narrowing

Narrow with control flow, not assertions.

```ts
// typeof
function len(x: string | string[]): number {
  if (typeof x === "string") return x.length;
  return x.length; // x is string[]
}

// in operator
type Cat = { meow: () => void };
type Dog = { bark: () => void };
function speak(animal: Cat | Dog): void {
  if ("meow" in animal) animal.meow();
  else animal.bark();
}

// User-defined type guard
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    typeof (value as Record<string, unknown>).id === "string"
  );
}

// satisfies — check shape without widening
const routes = {
  home: "/",
  profile: "/profile",
  settings: "/settings",
} as const satisfies Record<string, `/${string}`>;

type RouteName = keyof typeof routes; // "home" | "profile" | "settings"
```

Prefer `satisfies` over annotation when you want both checking *and* the precise inferred type.

## Generics

Generics earn their place when the same logic operates on multiple types. Constrain them to say what they need.

```ts
// Constrained generic — T must have an id
function indexById<T extends { id: string }>(items: readonly T[]): Map<string, T> {
  return new Map(items.map((item) => [item.id, item]));
}

// Multiple type parameters with a relationship
function pluck<T, K extends keyof T>(items: readonly T[], key: K): Array<T[K]> {
  return items.map((item) => item[key]);
}

const users = [{ id: "1", name: "Anh" }, { id: "2", name: "Sam" }];
const names = pluck(users, "name"); // string[]
// const bad = pluck(users, "age"); // ✗ "age" is not a key of the user type
```

Avoid generics that are used only once, generics with unused parameters, or generics that exist only to enable an unsafe cast inside.

## Errors as Values

If you prefer not to throw, a small `Result` type goes a long way. Pick this *or* typed exceptions per package — don't mix.

```ts
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

const ok = <T>(value: T): Result<T, never> => ({ ok: true, value });
const err = <E>(error: E): Result<never, E> => ({ ok: false, error });

async function parseConfig(path: string): Promise<Result<Config, ConfigError>> {
  const raw = await readFile(path, "utf8").catch(() => null);
  if (raw === null) return err({ kind: "not-found", path });

  try {
    return ok(configSchema.parse(JSON.parse(raw)));
  } catch (cause) {
    return err({ kind: "parse-failed", cause });
  }
}

const result = await parseConfig("app.json");
if (!result.ok) {
  console.error(result.error);
  return;
}
useConfig(result.value); // narrowed to Config
```

If you'd rather throw, define a typed error hierarchy and document it on the function — but commit to the choice.

## Public API Annotations

Annotate what crosses module/package boundaries; let inference handle internals.

```ts
// ✓ Exported — annotate the contract
export interface CreateUserInput {
  readonly email: string;
  readonly name: string;
}

export async function createUser(input: CreateUserInput): Promise<User> {
  const id = await generateId();         // inferred: string
  const createdAt = new Date();          // inferred: Date
  return persist({ id, createdAt, ...input });
}

// ✓ Internal — inference is fine
function generateId() {
  return crypto.randomUUID();
}
```

Inferring an exported return type means a refactor can silently widen your public API. Lock it down with an annotation.

## Tooling

The expected baseline on every TypeScript project:

```bash
# Type-check — must pass with zero errors
npx tsc --noEmit

# Lint — strict, type-aware
npx eslint . --max-warnings 0

# Format — must produce no diff
npx prettier --check .

# Run all in CI on every PR
```

A reasonable ESLint config (`eslint.config.js`, flat config):

```js
import tseslint from "typescript-eslint";

export default tseslint.config(
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
      },
    },
    rules: {
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/consistent-type-imports": "error",
      "@typescript-eslint/switch-exhaustiveness-check": "error",
      "@typescript-eslint/no-unnecessary-condition": "error",
    },
  },
);
```

When suppressing a check, explain why:

```ts
// @ts-expect-error -- upstream types lag the 5.4 release; tracked in #1234
import { newApi } from "some-library";

// eslint-disable-next-line @typescript-eslint/no-non-null-assertion
// reason: guarded by the `length > 0` check above
const first = items[0]!;
```

If you can't write a convincing justification, the suppression shouldn't be there.
