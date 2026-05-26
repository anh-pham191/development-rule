## R Development Philosophy

This codebase follows these core principles to ensure clean, maintainable, and reproducible R code for analyses, packages, and Shiny apps.

**Assumptions:** This guide assumes a recent R (4.x) with projects using `renv` (or a similar lockfile tool) for reproducibility, RStudio or Positron with `lintr` and `styler` for tooling, and the [tidyverse style guide](https://style.tidyverse.org/) as a baseline — even when using base R rather than tidyverse packages. Both base R and the tidyverse are valid; pick the dialect that best fits the task and keep it consistent within a script or package.

### 1. Readable

Optimise for human understanding. Code should be self-documenting.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your function and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Use `snake_case` for variables and functions, descriptive names (`monthly_revenue`, not `mr`), one statement per line, the native pipe `|>` to make data flow read top-to-bottom
- ✗ DON'T: Use cryptic abbreviations (`df1`, `tmp`, `x2`), deeply nested function calls, mix `.`, `_`, and camelCase in the same project

### 2. Explicit

Make behaviour clear and obvious. Avoid magic and implicit coercion.

- ✓ DO: Use `::` for namespace lookups in package code (`dplyr::filter`), name arguments at call sites for clarity (`mean(x, na.rm = TRUE)`), declare expected types with `stopifnot()` or `vctrs::vec_assert()`
- ✗ DON'T: Rely on `library()` calls inside functions, depend on R's silent recycling and coercion rules, write functions that read or write the global environment via `<<-`

### 3. Compositional

Build flexible pipelines from small, focused functions that compose together.

Functions should be short enough to hold in your head. If you can't see the entire function at once, or if you lose track of what variables hold, the function is probably doing too much. Deep nesting signals that the function is handling too many concerns. Consider extracting helper functions or restructuring.

- ✓ DO: Write small single-purpose functions, chain them with `|>`, use `purrr::map_*` or `vapply()` to compose over collections
- ✗ DON'T: Write 200-line analysis blocks at the top level, nest five `lapply()` calls when a named helper would explain intent; functions that scroll for pages

### 4. Vectorised

Favour vectorised operations over element-wise loops. R is fastest and clearest when you think in vectors.

- ✓ DO: Use `ifelse()`, `dplyr::case_when()`, `pmin()`/`pmax()`, logical indexing, `rowSums()`/`colMeans()`; pre-allocate when a loop is genuinely needed
- ✗ DON'T: Grow a vector inside a `for` loop with `c()` or `rbind()`, reach for `apply()` when a direct vectorised op exists, use a loop just because it's familiar from another language

### 5. Tidy

When working with rectangular data, embrace tidy-data conventions: one variable per column, one observation per row, one type per cell.

- ✓ DO: Reshape with `tidyr::pivot_longer()`/`pivot_wider()`, keep dates as `Date`/`POSIXct`, store categoricals as `factor` with explicit levels
- ✗ DON'T: Encode variables in column names (`sales_2023`, `sales_2024` as separate columns when "year" is really a variable), mix units or types within a column, rely on row order to carry meaning

### 6. Functional

Prefer pure functions that take inputs and return outputs over code that mutates global state.

- ✓ DO: Return new objects rather than modifying in place, pass dependencies as arguments, isolate side effects (file I/O, plotting, DB writes) to a thin outer layer
- ✗ DON'T: Use `<<-` to reach up the scope chain, rely on `attach()`, set options or change working directories deep inside helper functions

### 7. Testable

Design for testing from the start. Code that's hard to test is usually hard to reason about.

- ✓ DO: Put logic in named functions in `R/`, write `testthat` tests in `tests/testthat/`, use snapshot tests for printed output and plots, mock external services with `httptest2` or `withr::local_*`
- ✗ DON'T: Bury logic inside an Rmd or Shiny `server` function where it can't be unit-tested, hard-code file paths or credentials, depend on the contents of the global environment

### 8. Reproducible

A result that can't be reproduced isn't a result. Pin dependencies, seed randomness, and capture environment.

- ✓ DO: Initialise projects with `renv::init()` and commit `renv.lock`, call `set.seed()` before any stochastic step, record `sessionInfo()` in long-running outputs, use `here::here()` for file paths
- ✗ DON'T: Rely on whatever package versions happen to be installed, use `setwd()` in scripts, leave random simulations unseeded, hard-code absolute paths like `/Users/me/...`

### 9. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Start with a flat script, refactor into functions when patterns emerge, reach for `data.table` only when `dplyr` is genuinely too slow, profile with `profvis` before optimising
- ✗ DON'T: Build an S4 class hierarchy for a one-off analysis, switch dialects mid-script, prematurely optimise; write "clever" code that requires explanation to understand

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 10. Predictable

Code should behave as expected without surprises. Handle errors and missing values explicitly.

- ✓ DO: Decide deliberately how `NA` flows through each function, use `stop()` / `rlang::abort()` with informative messages and classed conditions, prefer `vapply()` over `sapply()` for stable return types, set `stringsAsFactors = FALSE` mental model (the default since R 4.0)
- ✗ DON'T: Swallow errors with `try()` and ignore the result, let `sapply()` silently return a list when you expected a vector, mix `=` and `<-` for assignment within a project

### 11. Documented

Code is read more than it is written. Document the interface and the non-obvious "why".

- ✓ DO: Use `roxygen2` `@param`, `@return`, `@examples` for every exported function, keep a top-of-script comment explaining intent and inputs/outputs, maintain a `README.Rmd` for packages
- ✗ DON'T: Ship a package without `man/` pages, document only the obvious, let examples bit-rot until they no longer run under `R CMD check`

### 12. Well-Commented

Well-written code is self-evident about *what* it does. Comments add value when they explain *why*: the context, constraints, or reasoning that isn't obvious from the code itself.

- ✓ DO: Explain business rules, statistical assumptions, why a particular threshold or seed was chosen, or note when code works around an upstream bug
- ✗ DON'T: Narrate what the code does step-by-step; leave comments that duplicate the function name or restate the operator
