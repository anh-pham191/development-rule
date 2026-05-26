---
name: c-development
description: >
  C development philosophy, idioms, and patterns for writing safe, portable, modern C (C11/C17/C23).
  Use when: (1) Working on C projects, (2) Reviewing C code, (3) Writing systems-level or
  embedded code, (4) Handling memory management, undefined behaviour, or portability concerns,
  or (5) Setting up build/sanitiser/static-analysis tooling for a C codebase.
---

# C Development

Core principles for writing safe, portable, maintainable modern C.

## Philosophy

### 1. Readable
Optimise for human understanding.
- DO: Name functions and variables for what they mean; keep one concept per translation unit
- DON'T: Lean on single-letter names or clever macros

### 2. Explicit
Say what you mean — C will not infer intent.
- DO: Use fixed-width types, check every return, document ownership at the header
- DON'T: Rely on implicit promotions or "it works on x86_64"

### 3. Concise
Keep functions small enough to hold in your head.
- DO: Extract helpers; use early returns for input validation
- DON'T: Write 400-line functions with seven levels of nesting

### 4. Compositional
Build from small modules with narrow interfaces.
- DO: Use opaque pointers; keep translation units small
- DON'T: Expose struct internals to the world

### 5. Memory-Safe
Own every allocation; free every allocation.
- DO: Pair every `malloc` with a `free`; document ownership in the API
- DON'T: Return heap pointers without saying who frees them

### 6. Undefined-Behaviour Aware
Treat UB as a bug, not a feature.
- DO: Use `memcpy` for type-punning; initialise locals; guard against overflow
- DON'T: Cast incompatible pointers and dereference, or rely on signed overflow

### 7. Portable
Don't bake host assumptions into code.
- DO: Use `<stdint.h>`, `size_t`, explicit endianness conversions
- DON'T: Assume `int` is 32 bits or that structs have no padding

### 8. Defensive at Boundaries
Validate at the edge; trust the inside.
- DO: Null-check parameters in public functions; return clear error codes
- DON'T: Sprinkle defensive checks through every internal helper

### 9. Testable
If you can't test it, you can't trust it.
- DO: Unit-test under AddressSanitizer + UBSan; cover the error paths
- DON'T: Skip the `goto cleanup` branches

### 10. Tooled
The compiler is your first reviewer.
- DO: Build with `-Wall -Wextra -Werror`; run sanitisers and `clang-tidy` in CI
- DON'T: Silence warnings to make them go away

### 11. Pragmatic
Rules serve the code, not the reverse.
- DO: Profile before optimising; isolate clever code behind a clean interface
- DON'T: Micro-optimise on a hunch

### 12. Well-Commented
Explain *why*, not *what*.
- DO: Document ownership, lifetime, and invariants
- DON'T: Write `/* increment i */` above `i++`

## Error Handling

C has no exceptions. Use return codes and a single cleanup path.

```c
/* Returns 0 on success, negative errno-style code on failure.
 * On success, *out is a heap buffer owned by the caller. */
int load_config(const char *path, config_t **out)
{
    if (path == NULL || out == NULL) {
        return -EINVAL;
    }

    int rc = 0;
    FILE *fp = NULL;
    config_t *cfg = NULL;

    fp = fopen(path, "r");
    if (fp == NULL) {
        rc = -errno;
        goto cleanup;
    }

    cfg = calloc(1, sizeof(*cfg));
    if (cfg == NULL) {
        rc = -ENOMEM;
        goto cleanup;
    }

    if (parse_into(fp, cfg) != 0) {
        rc = -EIO;
        goto cleanup;
    }

    *out = cfg;
    cfg = NULL;  /* ownership transferred */

cleanup:
    if (fp != NULL) fclose(fp);
    free(cfg);   /* free(NULL) is safe */
    return rc;
}
```

- Public functions return an `int` status; output via out-parameters.
- One `cleanup:` label per function; resources freed in reverse order of acquisition.
- `errno` is volatile — capture it immediately after the failing call.

## Memory Management

Document ownership at the API boundary. Pair every allocator with its matching deallocator.

```c
/* Allocates and returns a new buffer. Caller owns it; free with buffer_free(). */
buffer_t *buffer_new(size_t cap);

/* Frees a buffer previously returned by buffer_new(). Safe on NULL. */
void buffer_free(buffer_t *buf);
```

Opaque-pointer pattern keeps struct internals out of the header:

```c
/* buffer.h */
typedef struct buffer buffer_t;
buffer_t *buffer_new(size_t cap);
void      buffer_free(buffer_t *buf);
size_t    buffer_len(const buffer_t *buf);

/* buffer.c */
struct buffer {
    uint8_t *data;
    size_t   len;
    size_t   cap;
};
```

- Set freed pointers to `NULL` if they remain in scope.
- Prefer `calloc` over `malloc` + `memset(0)`; it's clearer and sometimes faster.
- For temporary allocations in a hot path, consider an arena allocator.

## Testing

Use Criterion, Unity, or plain `assert` with a small runner. Always run tests under sanitisers.

```c
#include <criterion/criterion.h>
#include "buffer.h"

Test(buffer, new_returns_empty)
{
    buffer_t *b = buffer_new(16);
    cr_assert_not_null(b);
    cr_assert_eq(buffer_len(b), 0);
    buffer_free(b);
}

Test(buffer, free_null_is_safe)
{
    buffer_free(NULL);  /* must not crash */
}
```

Build tests with:

```
cc -std=c17 -Wall -Wextra -Werror -g -O1 \
   -fsanitize=address,undefined -fno-omit-frame-pointer \
   test_buffer.c buffer.c -lcriterion -o test_buffer
./test_buffer
```

- Cover error paths, NULL inputs, and boundary sizes — not just the happy path.
- A passing test that hasn't been run under ASan/UBSan tells you very little.

## Tooling

Minimum bar for every C codebase:

```
# Compiler flags (CMake or Make)
-std=c17 -Wall -Wextra -Werror
-Wshadow -Wconversion -Wstrict-prototypes
-Wmissing-prototypes -Wpointer-arith
-Wcast-align -Wformat=2

# Sanitisers (CI, debug builds)
-fsanitize=address,undefined -fno-omit-frame-pointer -g -O1

# Static analysis (every commit)
clang-tidy --warnings-as-errors=* src/*.c
scan-build make
cppcheck --enable=all --inconclusive --std=c17 src/
```

- Run AddressSanitizer + UBSan in CI on every PR. Add MemorySanitizer for uninitialised-read coverage.
- Pin a `.clang-format` and a `.clang-tidy` in the repo so the rules travel with the code.
- Treat new warnings as failing builds; suppress with a comment explaining why if you must.

## Header Discipline

Headers are part of the public API. Keep them tight.

```c
#ifndef MYLIB_BUFFER_H
#define MYLIB_BUFFER_H

#include <stddef.h>   /* size_t */
#include <stdint.h>   /* uint8_t */

/* Forward-declare instead of including the full header where possible. */
typedef struct buffer buffer_t;

buffer_t *buffer_new(size_t cap);
void      buffer_free(buffer_t *buf);
size_t    buffer_len(const buffer_t *buf);

#endif /* MYLIB_BUFFER_H */
```

- Use include guards (`#ifndef X_H`) or `#pragma once` — pick one and stay consistent.
- Headers include only what their declarations need; push implementation includes into the `.c` file.
- Forward-declare structs in headers when callers only hold pointers — it cuts compile times and breaks include cycles.
- No `static` functions, no definitions, no `using`-style namespace pollution in headers.
