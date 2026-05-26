---
name: r-development
description: >
  R development philosophy, idioms, and patterns for writing clean, reproducible R code
  across analyses, packages, and Shiny apps. Use when: (1) Working on R projects,
  (2) Writing or refactoring R scripts, packages, or Shiny apps, (3) Reviewing R code,
  (4) Setting up renv, testthat, lintr, or styler, (5) Discussing base R vs tidyverse
  trade-offs. Covers both base R and the tidyverse pragmatically.
---

# R Development

Core principles for writing clean, reproducible, and idiomatic R 4.x code.

## Philosophy

### 1. Readable
Optimise for human understanding. Code should be self-documenting.
- DO: `snake_case` names, descriptive variables, native pipe `|>` for data flow
- DON'T: Cryptic abbreviations (`df1`, `tmp`), mixed naming conventions, deeply nested calls

### 2. Explicit
Make behaviour clear and obvious. Avoid magic and implicit coercion.
- DO: Use `pkg::fun()` in package code, named arguments at call sites, `stopifnot()` for invariants
- DON'T: Use `<<-` or `attach()`, rely on silent recycling and coercion

### 3. Compositional
Build pipelines from small, focused functions.
- DO: Short single-purpose functions chained with `|>`, `purrr::map_*` or `vapply()` for iteration
- DON'T: 200-line top-level scripts, five-deep `lapply()` nests where a named helper would clarify intent

### 4. Vectorised
Favour vectorised operations over element-wise loops.
- DO: `ifelse()`, `dplyr::case_when()`, logical indexing, `rowSums()`; pre-allocate if a loop is required
- DON'T: Grow vectors in a loop with `c()` / `rbind()`, reach for `apply()` when a direct vectorised op exists

### 5. Tidy
For rectangular data: one variable per column, one observation per row.
- DO: Reshape with `pivot_longer`/`pivot_wider`, store dates and factors with explicit types
- DON'T: Encode variables in column names, rely on row order to carry meaning

### 6. Functional
Prefer pure functions over global mutation.
- DO: Return new objects, pass dependencies as arguments, isolate side effects at the edges
- DON'T: Use `<<-`, `attach()`, or `setwd()` inside helpers

### 7. Testable
Design for testing from the start.
- DO: Put logic in named functions in `R/`, write `testthat` tests, snapshot printed/plotted output
- DON'T: Bury logic in Rmd chunks or Shiny `server` blocks, hard-code paths or credentials

### 8. Reproducible
Pin dependencies, seed randomness, capture environment.
- DO: `renv::init()` + commit `renv.lock`, `set.seed()` before stochastic steps, `here::here()` for paths
- DON'T: Rely on ambient package versions, use `setwd()`, leave simulations unseeded

### 9. Pragmatic
Solve the problem at hand without unnecessary complexity.
- DO: Start simple, refactor when patterns emerge, profile with `profvis` before optimising
- DON'T: Build S4 hierarchies for a one-off analysis, switch dialects mid-script

### 10. Predictable
Handle errors and `NA` deliberately.
- DO: `rlang::abort()` with classed conditions, `vapply()` for stable return types, decide `NA` handling per function
- DON'T: Swallow errors with bare `try()`, let `sapply()` shape-shift silently

### 11. Documented
Document the interface and the non-obvious "why".
- DO: `roxygen2` tags on every exported function, examples that run under `R CMD check`
- DON'T: Ship a package without `man/` pages, document only the obvious

## Error Handling

```r
# Base R: signal with stop() / warning() / message()
parse_amount <- function(x) {
  if (!is.numeric(x)) {
    stop("`x` must be numeric, got ", class(x)[1], call. = FALSE)
  }
  x
}

# Preferred: rlang classed conditions with structured data
parse_amount <- function(x) {
  if (!is.numeric(x)) {
    rlang::abort(
      message = c(
        "`x` must be numeric.",
        i = paste0("Got <", class(x)[1], ">.")
      ),
      class = "amount_parse_error",
      x = x
    )
  }
  x
}

# Catch specific classes, not everything
result <- tryCatch(
  parse_amount(input),
  amount_parse_error = function(cnd) {
    rlang::warn(conditionMessage(cnd))
    NA_real_
  }
)
```

## Testing

```r
# tests/testthat/test-slug.R
library(testthat)

test_that("slug() lowercases and dashes spaces", {
  expect_equal(slug("Hello World"), "hello-world")
})

test_that("slug() strips accents and punctuation", {
  expect_equal(slug("Héllo Wörld!"), "hello-world")
})

test_that("slug() errors on non-character input", {
  expect_error(slug(123), class = "slug_input_error")
})

# Snapshot tests for printed output / ggplots
test_that("summary print stays stable", {
  expect_snapshot(print(summarise_sales(sales_2024)))
})

# Run from the package root
# devtools::test()
# devtools::check()
```

## Reproducibility

```r
# Project setup (run once, at the project root)
renv::init()           # creates renv/, renv.lock, .Rprofile
renv::snapshot()       # update lockfile after adding deps
renv::restore()        # reproduce the locked environment elsewhere

# Stochastic work: seed at the top of the script
set.seed(20260526)
samples <- rnorm(1000)

# Paths: use here::here(), never setwd()
data <- readr::read_csv(here::here("data", "raw", "sales.csv"))

# Capture environment with long-running outputs
saveRDS(
  list(result = model, session = sessionInfo()),
  here::here("output", "model_2026-05-26.rds")
)
```

## Style

```r
# Use the tidyverse style guide as a baseline:
#   - snake_case for objects and functions
#   - <- for assignment, = for named arguments
#   - 2-space indent, ~80 char line limit
#   - one statement per line; pipe steps on their own line

monthly_revenue <- sales |>
  dplyr::filter(status == "settled") |>
  dplyr::group_by(month = lubridate::floor_date(date, "month")) |>
  dplyr::summarise(revenue = sum(amount), .groups = "drop")

# Tooling: run as part of dev workflow / CI
styler::style_pkg()    # auto-format to the style guide
lintr::lint_package()  # static analysis: naming, complexity, undefined symbols

# Configure project-wide rules in .lintr at the project root:
# linters: linters_with_defaults(
#   line_length_linter = line_length_linter(100L),
#   object_name_linter = object_name_linter(styles = "snake_case")
# )
```

## Package Structure

```
mypackage/
├── DESCRIPTION          # metadata, deps under Imports/Suggests
├── NAMESPACE            # generated by roxygen2 — do not edit by hand
├── R/                   # all exported and internal functions
│   ├── slug.R
│   └── summarise-sales.R
├── man/                 # generated .Rd files from roxygen2
├── tests/
│   └── testthat/
│       ├── test-slug.R
│       └── _snaps/      # snapshot fixtures
├── vignettes/           # long-form usage docs (.Rmd)
├── inst/                # extra files shipped with the package
├── renv.lock            # pinned dependencies (for apps/analyses)
└── README.Rmd
```

```r
# R/slug.R — roxygen2 documents the interface
#' Slugify a string
#'
#' Lowercases, strips accents, and replaces non-alphanumeric runs with `-`.
#'
#' @param x A character vector.
#' @return A character vector of slugs, same length as `x`.
#' @examples
#' slug("Hello World")
#' slug(c("Héllo", "Wörld!"))
#' @export
slug <- function(x) {
  stopifnot(is.character(x))
  x |>
    stringi::stri_trans_general("Latin-ASCII") |>
    tolower() |>
    gsub("[^a-z0-9]+", "-", x = _) |>
    gsub("^-|-$", "", x = _)
}
```

```r
# DESCRIPTION (minimal)
# Package: mypackage
# Title: Tools for Sales Analysis
# Version: 0.1.0
# Authors@R: person("Anh", "Pham", email = "anh@example.com", role = c("aut", "cre"))
# Description: Reusable helpers for cleaning and summarising sales data.
# License: MIT + file LICENSE
# Encoding: UTF-8
# Imports:
#     dplyr,
#     stringi,
#     rlang
# Suggests:
#     testthat (>= 3.0.0),
#     withr
# Config/testthat/edition: 3
# Roxygen: list(markdown = TRUE)
# RoxygenNote: 7.3.2

# Typical dev loop
# devtools::load_all()    # source everything under R/
# devtools::document()    # regenerate man/ + NAMESPACE from roxygen
# devtools::test()        # run testthat
# devtools::check()       # full R CMD check
```
