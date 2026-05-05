---
name: code-quality
description: >
  Language-agnostic code readability principles for writing clean, maintainable code.
  Use when: (1) Writing new code, (2) Reviewing code, (3) Refactoring existing code,
  (4) Discussing best practices, (5) Improving code readability. Applies to any
  programming language and promotes code that is easy to understand and maintain.
---

# Code Quality

Principles for writing code that is readable by human beings. These guidelines apply to any programming language.

## 1. Use Clear, Descriptive Names

Variables, functions, and classes should reveal their intent.

```
Good: calculateMonthlyPayment(), userEmailAddress, isValidInput
Bad:  calc(), data, doStuff(), x, temp
```

Avoid abbreviations unless universally understood in the domain.

## 2. Keep Functions Small and Focused

Each function should do one thing well. If you struggle to name a function without using "and," it probably does too much.

```
Good: validateEmail(), sendNotification(), calculateTax()
Bad:  validateAndSendEmailThenLog()
```

Aim for functions that fit on one screen.

## 3. Write Code That Reads Like Prose

Well-written code tells a story. Logic should flow naturally from top to bottom with minimal jumping around.

```python
# Good: reads naturally
def process_order(order):
    if not order.is_valid():
        return OrderError("Invalid order")
    
    inventory.reserve(order.items)
    payment = process_payment(order.total)
    
    if payment.failed:
        inventory.release(order.items)
        return PaymentError(payment.reason)
    
    return ship_order(order)
```

Use early returns to avoid deep nesting.

## 4. Be Consistent

Pick conventions for formatting, naming, and structure—then stick to them throughout your codebase.

- If you use `camelCase`, use it everywhere
- If you put braces on the same line, do it consistently
- If you use tabs, don't mix in spaces

Consistency reduces cognitive load.

## 5. Comment the "Why" Not the "What"

Code should be self-explanatory about what it does. Use comments to explain why.

```python
# Bad: explains what (obvious from code)
# Increment counter by 1
counter += 1

# Good: explains why (non-obvious business logic)
# IRD requires 7-day grace period for late payments
due_date = due_date.add_days(7)
```

Comment non-obvious trade-offs, workarounds, and business rules.

## 6. Avoid Deep Nesting

Multiple levels of indentation make code hard to follow.

```python
# Bad: deeply nested
def process(data):
    if data:
        if data.is_valid:
            if data.has_items:
                for item in data.items:
                    if item.active:
                        # actual logic buried here

# Good: early returns flatten the structure
def process(data):
    if not data:
        return None
    if not data.is_valid:
        return ValidationError()
    if not data.has_items:
        return EmptyError()
    
    for item in data.items:
        if not item.active:
            continue
        # actual logic at reasonable depth
```

Extract nested logic into separate functions or use guard clauses.

## 7. Handle Errors Explicitly

Make error conditions visible rather than hiding them.

```python
# Bad: silent failure
def get_user(id):
    try:
        return database.find(id)
    except:
        return None  # caller has no idea what went wrong

# Good: explicit error handling
def get_user(id):
    try:
        return database.find(id)
    except NotFoundError:
        raise UserNotFoundError(f"User {id} not found")
    except ConnectionError as e:
        raise DatabaseError(f"Database unavailable: {e}")
```

Clear error handling makes behavior predictable.

## 8. Don't Be Clever

Straightforward code beats clever code. That one-liner you're proud of might confuse someone (including future you) who needs to debug it at 2am.

```python
# Clever but confusing
result = [x for x in (y.process() for y in items if y.valid) if x]

# Clear and debuggable
result = []
for item in items:
    if not item.valid:
        continue
    processed = item.process()
    if processed:
        result.append(processed)
```

Optimize for human understanding first, then for elegance.
