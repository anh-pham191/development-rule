## Node.js Development Philosophy

This codebase follows these core principles to ensure clean, maintainable, and idiomatic Node.js for server-side runtimes.

**Assumptions:** This guide assumes current LTS Node.js (22+) with `package.json` declaring `"type": "module"` for native ESM. Dependencies are managed with `pnpm` (preferred) or `npm`, and the lockfile (`pnpm-lock.yaml` or `package-lock.json`) is committed for applications. TypeScript is optional but encouraged at module boundaries (public exports, HTTP handlers, queue consumers); internal code may use JSDoc-annotated JavaScript where a build step is undesirable. The audience is backend engineers building APIs, CLIs, workers, and tooling — front-end JavaScript concerns are out of scope (see [Front-end JavaScript Development Philosophy](javascript-development.md)).

### 1. Async-First / Non-Blocking

Node.js runs your code on a single event-loop thread. Blocking that thread blocks every request, timer, and socket the process is handling.

- ✓ DO: Use `async`/`await`, native Promises, and non-blocking core APIs (`fs/promises`, `node:timers/promises`); offload CPU-bound work (hashing, parsing, image transforms) to `worker_threads` or a separate process
- ✗ DON'T: Call `fs.readFileSync`, `child_process.execSync`, or tight CPU loops in request handlers; busy-wait on a Promise; mix callback-style and Promise-style APIs in the same module

### 2. Streamed

Memory is finite and request bodies, file uploads, and database result sets can be large. Prefer streaming over buffering.

- ✓ DO: Use `stream/promises.pipeline()`, async iterators (`for await...of`), `Readable.from()`; stream HTTP responses; honour backpressure
- ✗ DON'T: Read whole files with `fs.readFile` when you only need to forward them; collect a database cursor into one giant array; manually `.on('data', ...)` without handling errors and end events

### 3. Bounded

Every external input is an opportunity for resource exhaustion. Put a ceiling on everything that crosses a process boundary.

- ✓ DO: Set request body size limits, per-request timeouts, connection pool maxima, and `AbortSignal` deadlines on outbound calls; cap concurrency on `Promise.all` fan-outs (`p-limit`, `Promise.allSettled` with a semaphore)
- ✗ DON'T: Trust upstreams to be fast or finite; spawn an unbounded number of in-flight Promises in a `for` loop; leave default Node timeouts in place for production HTTP clients

### 4. Concise

Keep code minimal without sacrificing clarity. Avoid unnecessary abstractions.

Functions should be short enough to hold in your head. If you can't see the entire function at once, or if you lose track of what variables hold, the function is probably doing too much. Deep nesting signals that the function is handling too many concerns. Consider extracting helper functions or restructuring.

- ✓ DO: Small focused functions; early returns; named helpers over deeply nested callbacks
- ✗ DON'T: 200-line `async` functions; pyramid `if`/`try`/`for` nesting; reaching for a class when a function closure suffices

### 5. Readable

Optimise for human understanding. Code should be self-explanatory.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your function and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Descriptive identifiers (`fetchOrderById`, not `getO`); module names that match their primary export; consistent file layout per module (imports, types, helpers, exported entry points)
- ✗ DON'T: Cryptic abbreviations; clever one-liners chaining six array methods; relying on tribal knowledge about which side effects a module triggers on import

### 6. Explicit

Make data flow and dependencies obvious. Node's module system and dynamic typing already give you enough rope — don't add more.

- ✓ DO: Use ESM `import`/`export` with named exports; inject configuration and clients (DB, logger, fetch) into modules rather than reading them from globals; declare `exports` and `engines` in `package.json`
- ✗ DON'T: Mutate `process.env` at runtime; rely on `require.cache` side effects; use default exports for modules with multiple public symbols; reach into `node_modules/<pkg>/lib/internal.js`

### 7. Compositional

Build server-side systems by composing small units — middlewares, handlers, transforms — not by extending base classes.

- ✓ DO: Compose middleware/handlers as plain functions returning Promises; use factory functions that close over dependencies; prefer functions and modules over deep class hierarchies
- ✗ DON'T: Build a five-level inheritance chain to share a logger; mix request-scoped state into singleton classes; reach for a DI container when a parameter would do

### 8. Testable

Design for testing from the start. Test boundaries with the real runtime, mock only what you must.

- ✓ DO: Write tests first using `node:test` or `vitest`; inject `fetch`, the clock, and the database; test handlers as functions, not via a running HTTP server, where practical
- ✗ DON'T: Hard-code `new Date()`, `Math.random()`, or `process.env` reads inside business logic; mock the module under test; create test setups that depend on test-execution order

### 9. Secure (Boundary-Validated)

Every byte that enters the process from outside is untrusted until validated. Every byte that leaves is a potential leak.

- ✓ DO: Validate request bodies, query strings, and env vars with `zod` or `valibot` at the edge; use parameterised queries / prepared statements; run with the least privilege (non-root user, restricted file system, `--permission` flag where appropriate); audit dependencies (`pnpm audit`, `npm audit`)
- ✗ DON'T: Trust client-supplied JSON shapes; interpolate user input into SQL, shell, or file paths; log secrets or full request bodies; store secrets in source or in `.env` files that get committed

### 10. Observable

You can't fix what you can't see. Treat logs, metrics, and traces as first-class outputs of the service.

- ✓ DO: Use structured JSON logging (`pino`); attach a correlation/request ID via `AsyncLocalStorage` and child loggers; emit metrics for latency, error rate, and queue depth; surface unhandled rejections (`process.on('unhandledRejection')`) and crash loudly
- ✗ DON'T: `console.log` in production handlers; swallow caught errors with empty `catch` blocks; log unstructured strings that can't be queried; let the process limp on after `unhandledRejection`

### 11. Packaged

A Node.js application or library is a `package.json` first and code second. Treat the manifest with the same care as the code.

- ✓ DO: Set `"type": "module"`, declare an `engines.node` range, define an `exports` map for libraries, commit the lockfile for applications, follow semver for libraries, pin dev tooling
- ✗ DON'T: Ship dual CJS/ESM unless you have a real consumer need; rely on `main` alone when `exports` is supported; let `engines` drift behind your CI Node version; commit `node_modules`

### 12. Typed at Boundaries

Whether you use TypeScript or JSDoc, the public surface of every module should describe what it accepts and returns. Internal code can be looser if it's well tested.

- ✓ DO: Type exported functions, HTTP handler input/output, queue message shapes, and database row types; parse external data into typed values with a schema validator; treat `unknown` as the default for incoming data
- ✗ DON'T: Sprinkle `any` to silence the type checker; trust an `interface` to validate runtime data; export functions whose argument types are inferred as `any`

### 13. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Start with the Node standard library and a small set of well-maintained dependencies; reach for a framework only when it pays for itself; profile before optimising
- ✗ DON'T: Adopt a microservice/event-sourced/CQRS architecture for a CRUD app; pull in a dependency for a five-line helper; tune V8 flags before you've measured

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 14. Predictable

Code should behave the same way on every run, in every environment. The runtime gives you enough non-determinism already.

- ✓ DO: Pure functions where practical; pass clocks and randomness as dependencies; handle Promise rejections; design idempotent handlers for retried jobs
- ✗ DON'T: Mutate shared module-level state from request handlers; rely on iteration order of `Set`/`Map` for behaviour; let unhandled rejections silently change process state

### 15. Well-Commented

Well-written code is self-evident about *what* it does. Comments add value when they explain *why*: the context, constraints, or reasoning that isn't obvious from the code itself.

- ✓ DO: Explain why a timeout has a particular value, why a stream is paused, or why a query is structured a certain way for a specific index; note workarounds for upstream bugs with a link
- ✗ DON'T: Narrate `// increment counter` above `counter++`; restate the function name in a JSDoc block; leave stale comments that contradict the code
