---
name: php-development
description: >
  PHP development patterns for PHP 8.3+ code. Use when: (1) Working on PHP projects,
  (2) Writing new PHP code, (3) Reviewing PHP code, (4) Refactoring PHP code,
  (5) Configuring PHPStan or phpcs. Covers Symfony patterns with SilverStripe
  compatibility notes where relevant.
---

# PHP Development

Core principles for writing clean, maintainable, and modern PHP 8.3+ code.

## Philosophy

### 1. Modern & Standards-Compliant

Embrace PHP 8.x features and follow PSR standards.

- DO: Add `declare(strict_types=1);` to every file, use typed properties, follow PSR-12
- DON'T: Use deprecated features, write untyped code, rely on implicit coercion

### 2. Explicit

Make behavior clear and obvious. Avoid magic.

- DO: Use type declarations, return types, named arguments for clarity
- DON'T: Overuse magic methods (`__call`, `__get`), create hidden dependencies

### 3. Readable

Optimize for human understanding.

- DO: Use descriptive names, early returns, single responsibility methods
- DON'T: Write cryptic one-liners, nest deeply, use excessive abbreviations

### 4. Well-Structured

Apply proper design principles. OOP is primary, functional patterns where appropriate.

- DO: Use constructor property promotion, readonly properties, small focused classes
- DON'T: Create god classes, use global variables, rely on static for stateful logic

### 5. Compositional

Build through composition and interfaces rather than inheritance.

- DO: Define interfaces for contracts, use dependency injection, prefer composition
- DON'T: Create deep inheritance chains, use traits as a substitute for design

### 6. Testable

Design for testability with proper dependency injection.

- DO: Write unit and integration tests, inject dependencies, use mocking
- DON'T: Create untestable static dependencies, rely on global state

### 7. Secure

Follow security best practices.

- DO: Use prepared statements or ORM, validate input, escape output
- DON'T: Concatenate SQL, trust user input, expose sensitive data

### 8. Predictable

Handle errors explicitly. No surprises.

- DO: Throw specific exceptions, use custom hierarchies, document with `@throws`
- DON'T: Silently swallow errors, use exceptions for flow control

### 9. Statically Analyzed

Use static analysis to catch errors before runtime.

- DO: Run PHPStan at level 6+, use phpcs with PSR-12, leverage generics
- DON'T: Ignore warnings, disable rules without justification

## Error Handling

For detailed patterns, see [error-handling.md](references/error-handling.md).

Quick reference:

```php
<?php

declare(strict_types=1);

// Custom exception with context
final class UserNotFoundException extends \RuntimeException
{
    public function __construct(
        public readonly string $userId,
        ?\Throwable $previous = null,
    ) {
        parent::__construct(
            message: sprintf('User not found: %s', $userId),
            previous: $previous,
        );
    }
}

// Throw with context, catch specifically
public function getUser(string $id): User
{
    return $this->repository->find($id)
        ?? throw new UserNotFoundException($id);
}
```

## Testing

For detailed patterns, see [testing.md](references/testing.md).

Quick reference:

```php
<?php

declare(strict_types=1);

use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;

final class SluggerTest extends TestCase
{
    #[Test]
    #[DataProvider('slugCases')]
    public function generatesSlug(string $input, string $expected): void
    {
        $slugger = new Slugger();

        self::assertSame($expected, $slugger->slug($input));
    }

    /**
     * @return iterable<string, array{input: string, expected: string}>
     */
    public static function slugCases(): iterable
    {
        yield 'lowercase' => [
            'input' => 'Hello World',
            'expected' => 'hello-world',
        ];

        yield 'special characters' => [
            'input' => 'Héllo Wörld!',
            'expected' => 'hello-world',
        ];
    }
}
```

## Static Analysis

For detailed patterns, see [static-analysis.md](references/static-analysis.md).

Quick reference:

```php
<?php

declare(strict_types=1);

/**
 * @template T of object
 * @implements RepositoryInterface<T>
 */
abstract class AbstractRepository implements RepositoryInterface
{
    /**
     * @param class-string<T> $entityClass
     */
    public function __construct(
        private readonly EntityManagerInterface $em,
        private readonly string $entityClass,
    ) {}

    /**
     * @return T|null
     */
    public function find(string $id): ?object
    {
        return $this->em->find($this->entityClass, $id);
    }
}

/**
 * @param array{id: int, name: string, roles: list<string>} $data
 */
public function createUser(array $data): User
{
    return new User(...$data);
}
```

## Dependency Injection

Framework-agnostic principles for testable, decoupled code.

### Constructor Injection

Inject dependencies through the constructor:

```php
<?php

declare(strict_types=1);

final class OrderService
{
    public function __construct(
        private readonly OrderRepositoryInterface $orders,
        private readonly PaymentGatewayInterface $payments,
        private readonly LoggerInterface $logger,
    ) {}

    public function placeOrder(Order $order): void
    {
        $this->payments->charge($order->getTotal());
        $this->orders->save($order);
        $this->logger->info('Order placed', ['id' => $order->getId()]);
    }
}
```

### Interface Contracts

Define interfaces for dependencies:

```php
<?php

declare(strict_types=1);

interface PaymentGatewayInterface
{
    /**
     * @throws PaymentFailedException
     */
    public function charge(Money $amount): PaymentResult;

    public function refund(string $transactionId): void;
}

// Concrete implementations
final class StripePaymentGateway implements PaymentGatewayInterface
{
    public function __construct(
        private readonly StripeClient $client,
    ) {}

    public function charge(Money $amount): PaymentResult
    {
        // Stripe-specific implementation
    }

    public function refund(string $transactionId): void
    {
        // Stripe-specific implementation
    }
}
```

### Service Configuration

**Symfony:** Use autowiring with explicit bindings for interfaces:

```yaml
# config/services.yaml
services:
    _defaults:
        autowire: true
        autoconfigure: true

    App\:
        resource: '../src/'

    App\Payment\PaymentGatewayInterface:
        alias: App\Payment\StripePaymentGateway
```

**SilverStripe:** Use Injector for dependency management:

```yaml
# app/_config/services.yml
SilverStripe\Core\Injector\Injector:
  App\Payment\PaymentGatewayInterface:
    class: App\Payment\StripePaymentGateway
```

### Avoid Service Locator

Do not inject the container itself:

```php
// DON'T: Service locator anti-pattern
public function __construct(
    private readonly ContainerInterface $container,
) {}

public function process(): void
{
    $service = $this->container->get(SomeService::class); // Hidden dependency
}

// DO: Explicit injection
public function __construct(
    private readonly SomeService $service,
) {}

public function process(): void
{
    $this->service->doWork(); // Dependency is visible
}
```
