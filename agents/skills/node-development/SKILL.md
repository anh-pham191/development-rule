---
name: node-development
description: >
  Node.js development philosophy, runtime idioms, and server-side patterns for writing clean,
  performant, observable Node.js services and tools. Use when: (1) Working on Node.js backends,
  APIs, CLIs, workers, or build tooling, (2) Writing new Node.js code on current LTS (22+) with ESM,
  (3) Reviewing Node.js code for event-loop, streaming, security, or packaging concerns,
  (4) Choosing between `node:test` and `vitest`, (5) Setting up `pino` logging, `zod` validation,
  or `AbortController`-based cancellation, (6) Structuring `package.json` (`type`, `engines`,
  `exports`) and lockfile policy. Focuses on the Node *runtime* — for browser/front-end JavaScript
  concerns, see `javascript-development` or `vue-development`.
---

# Node.js Development

Core principles and patterns for writing clean, maintainable, observable Node.js services on current LTS (22+) with ESM.

## Philosophy

### 1. Async-First / Non-Blocking
Never block the event loop; offload CPU work to worker threads.
- DO: `async`/`await`, `fs/promises`, `worker_threads` for CPU-bound tasks
- DON'T: `*Sync` APIs or tight CPU loops in request handlers

### 2. Streamed
Prefer streams and async iterators over buffering whole payloads.
- DO: `pipeline()`, `for await...of`, honour backpressure
- DON'T: `fs.readFile` for large files you only forward; collect cursors into arrays

### 3. Bounded
Cap memory, concurrency, and time at every boundary.
- DO: Body size limits, per-request timeouts, `AbortSignal`, `p-limit` on fan-out
- DON'T: Trust upstreams; spawn unbounded in-flight Promises

### 4. Concise
Small focused functions; early returns.
- DO: Named helpers; flat control flow
- DON'T: 200-line async functions or pyramid nesting

### 5. Readable
Self-explanatory code; write for the newcomer.
- DO: Descriptive names; consistent module layout
- DON'T: Cryptic abbreviations; chained one-liners

### 6. Explicit
Make data flow and dependencies obvious.
- DO: Named ESM exports; inject config/clients; declare `exports` and `engines`
- DON'T: Mutate `process.env` at runtime; reach into a package's internals

### 7. Compositional
Compose middleware and handlers as plain functions.
- DO: Factory functions closing over dependencies
- DON'T: Deep class hierarchies to share a logger

### 8. Testable
Inject the clock, randomness, `fetch`, and the database.
- DO: `node:test` or `vitest`; test handlers as functions
- DON'T: Hard-code `new Date()` or `process.env` reads in business logic

### 9. Secure (Boundary-Validated)
Validate at the edge; least privilege everywhere.
- DO: `zod`/`valibot` on input; parameterised queries; audit dependencies
- DON'T: Trust client JSON shapes; interpolate user input into SQL or shell

### 10. Observable
Structured logs, metrics, traces; never swallow errors.
- DO: `pino` child loggers; correlation IDs via `AsyncLocalStorage`; crash on `unhandledRejection`
- DON'T: `console.log` in production; empty `catch` blocks

### 11. Packaged
Treat `package.json` with the same care as the code.
- DO: `"type": "module"`, `engines.node`, `exports` map, commit lockfile for apps
- DON'T: Ship dual CJS/ESM without a reason; commit `node_modules`

### 12. Typed at Boundaries
Type the public surface; parse external data into typed values.
- DO: TypeScript or JSDoc on exports; `unknown` for incoming data
- DON'T: Trust an `interface` to validate runtime input

### 13. Pragmatic
Start simple; reach for frameworks only when they pay for themselves.
- DO: Node standard library first; profile before optimising
- DON'T: CQRS/event-sourcing a CRUD app; add a dep for a five-line helper

### 14. Predictable
Same behaviour every run, every environment.
- DO: Pass clock/random as dependencies; idempotent job handlers
- DON'T: Mutate module-level state from handlers

### 15. Well-Commented
Comments explain *why*, not *what*.
- DO: Note why a timeout is 30s, why a query hints a specific index
- DON'T: `// increment counter` above `counter++`

## Async & Error Handling

Use `async`/`await` end-to-end. Surface failures; never silence them.

```js
// Always await or return — bare Promises lose their stack on rejection.
export async function loadUser(id, { db, logger }) {
  const row = await db.query('SELECT id, email FROM users WHERE id = $1', [id]);
  if (!row) {
    const err = new Error('user not found');
    err.code = 'USER_NOT_FOUND';
    throw err;
  }
  return row;
}

// Centralised error middleware (Fastify-style)
app.setErrorHandler((err, req, reply) => {
  req.log.error({ err, reqId: req.id }, 'request failed');
  const status = err.statusCode ?? (err.code === 'USER_NOT_FOUND' ? 404 : 500);
  reply.code(status).send({ error: err.code ?? 'INTERNAL_ERROR' });
});

// Crash loudly on unexpected programmer errors.
process.on('unhandledRejection', (reason) => {
  console.error('unhandledRejection', reason);
  process.exit(1);
});
```

**Cancellation with `AbortController`:**

```js
export async function fetchWithTimeout(url, { ms = 5_000 } = {}) {
  const ac = new AbortController();
  const timer = setTimeout(() => ac.abort(new Error('timeout')), ms);
  try {
    const res = await fetch(url, { signal: ac.signal });
    if (!res.ok) throw new Error(`upstream ${res.status}`);
    return await res.json();
  } finally {
    clearTimeout(timer);
  }
}
```

## Streams & Backpressure

Use `stream/promises.pipeline()` — it propagates errors and honours backpressure automatically.

```js
import { pipeline } from 'node:stream/promises';
import { createReadStream, createWriteStream } from 'node:fs';
import { createGzip } from 'node:zlib';

await pipeline(
  createReadStream('input.log'),
  createGzip(),
  createWriteStream('input.log.gz'),
);
```

**Async iterators for large datasets:**

```js
// Stream rows from a DB cursor without loading them all into memory.
export async function* iterateOrders(db) {
  const cursor = db.query(new Cursor('SELECT id, total FROM orders'));
  for await (const row of cursor) {
    yield row;
  }
}

for await (const order of iterateOrders(db)) {
  await process(order);
}
```

## Validation at the Edge

Parse incoming data into typed, trusted shapes with `zod`. The schema is the contract.

```ts
import { z } from 'zod';

const CreateOrderBody = z.object({
  customerId: z.string().uuid(),
  items: z
    .array(z.object({ sku: z.string().min(1), quantity: z.number().int().positive() }))
    .min(1)
    .max(100),
  notes: z.string().max(500).optional(),
});

export type CreateOrder = z.infer<typeof CreateOrderBody>;

app.post('/orders', async (req, reply) => {
  const body = CreateOrderBody.parse(req.body); // throws on invalid input
  const order = await createOrder(body, { db: req.db });
  reply.code(201).send(order);
});
```

Validate environment variables the same way at boot — fail fast if config is wrong.

```ts
const Env = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']),
  DATABASE_URL: z.string().url(),
  PORT: z.coerce.number().int().positive().default(3000),
});

export const env = Env.parse(process.env);
```

## Testing

Two reasonable choices. Prefer `node:test` for libraries and small services (zero dependencies); prefer `vitest` for codebases already using a Vite/TS toolchain or that want richer mocking.

**`node:test` (built-in):**

```js
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { loadUser } from './users.js';

test('loadUser returns the row when found', async () => {
  const db = { query: async () => ({ id: '1', email: 'a@b.test' }) };
  const user = await loadUser('1', { db, logger: console });
  assert.equal(user.email, 'a@b.test');
});

test('loadUser throws USER_NOT_FOUND when missing', async () => {
  const db = { query: async () => null };
  await assert.rejects(() => loadUser('x', { db, logger: console }), { code: 'USER_NOT_FOUND' });
});
```

Run with `node --test` (or `node --test --watch` during development).

**`vitest`:**

```ts
import { describe, it, expect, vi } from 'vitest';
import { loadUser } from './users.js';

describe('loadUser', () => {
  it('returns the row when found', async () => {
    const db = { query: vi.fn().mockResolvedValue({ id: '1', email: 'a@b.test' }) };
    await expect(loadUser('1', { db, logger: console })).resolves.toMatchObject({ id: '1' });
  });

  it('rejects when missing', async () => {
    const db = { query: vi.fn().mockResolvedValue(null) };
    await expect(loadUser('x', { db, logger: console })).rejects.toMatchObject({
      code: 'USER_NOT_FOUND',
    });
  });
});
```

Test async code by `await`-ing the assertion (`resolves`/`rejects`). Never trust a test that doesn't `await` its Promise.

## Observability

Use `pino` for structured JSON logging. Attach a request/correlation ID with a child logger so every log line for a request is queryable together.

```ts
import { pino } from 'pino';
import { AsyncLocalStorage } from 'node:async_hooks';
import { randomUUID } from 'node:crypto';

export const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  redact: ['req.headers.authorization', '*.password', '*.token'],
});

const als = new AsyncLocalStorage<{ reqId: string; log: pino.Logger }>();

export function withRequestContext(req, res, next) {
  const reqId = req.headers['x-request-id'] ?? randomUUID();
  const log = logger.child({ reqId, method: req.method, url: req.url });
  als.run({ reqId, log }, () => next());
}

export function ctx() {
  const store = als.getStore();
  if (!store) throw new Error('no request context');
  return store;
}

// Anywhere inside the request, just:
ctx().log.info({ orderId }, 'order created');
```

What to emit:

- **Logs:** structured key/value pairs, not interpolated strings
- **Metrics:** request latency, error rate, queue depth, in-flight requests, event-loop lag (`perf_hooks.monitorEventLoopDelay`)
- **Traces:** OpenTelemetry (`@opentelemetry/sdk-node`) for cross-service correlation when you have more than one service

## Tooling & Packaging

Minimal `package.json` for a modern ESM Node.js app:

```json
{
  "name": "@acme/orders-api",
  "version": "1.4.2",
  "type": "module",
  "engines": { "node": ">=22.0.0" },
  "scripts": {
    "dev": "node --watch --env-file=.env src/server.ts",
    "build": "tsc -p tsconfig.build.json",
    "start": "node dist/server.js",
    "test": "node --test --experimental-test-coverage",
    "lint": "eslint . && prettier --check ."
  },
  "dependencies": {
    "fastify": "^5.0.0",
    "pino": "^9.0.0",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "tsx": "^4.19.0",
    "typescript": "^5.6.0"
  }
}
```

For a **library**, add an `exports` map and avoid pinning runtime dependencies tightly:

```json
{
  "name": "@acme/tax-utils",
  "version": "0.3.1",
  "type": "module",
  "engines": { "node": ">=22.0.0" },
  "exports": {
    ".": { "types": "./dist/index.d.ts", "import": "./dist/index.js" },
    "./schemas": { "types": "./dist/schemas.d.ts", "import": "./dist/schemas.js" }
  },
  "files": ["dist"],
  "peerDependencies": { "zod": "^3.23.0" }
}
```

**Lockfile policy:**

- **Applications:** commit `pnpm-lock.yaml` / `package-lock.json`. Install with `pnpm install --frozen-lockfile` / `npm ci` in CI.
- **Libraries:** commit the lockfile for reproducible local dev and CI, but rely on semver ranges (not the lockfile) for downstream consumers.

**Running TypeScript directly in dev:** use `tsx` (`tsx watch src/server.ts`) or Node's native loader (`node --experimental-strip-types`) for fast iteration without a build step. Always compile to JS with `tsc` for production.

**Hot reload without third-party tools:** `node --watch src/server.ts` restarts the process on file changes — sufficient for most dev workflows.

**Loading `.env` without a library:** `node --env-file=.env src/server.ts` (Node 20.6+).
