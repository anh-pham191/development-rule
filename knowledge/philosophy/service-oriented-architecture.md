# Service-Oriented Architecture Philosophy

This document provides guidance for designing and building services within our ecosystem. It establishes principles that ensure our services are well-bounded, maintainable, and work together effectively.

## Audience

- **Primary:** Developers designing and building services
- **Secondary:** AI agents working on service architecture decisions

## Scope

This document covers:

- When to create a service
- Service boundaries and naming
- Communication patterns between services
- Data ownership and access
- Security considerations
- Observability requirements

The following are **out of scope**:

- Migration and refactoring strategies (separate document)
- Detailed API contract standards (see [API Design Philosophy](api-design.md))
- Infrastructure and deployment specifics

---

## Philosophy

These principles answer: *"What do we value in service architecture?"*

### 1. Domain-Driven Boundaries

Services should align with business domains, not technical layers or initial consumers.

A service owns a coherent set of data and behaviour that makes sense as a unit. The boundary should be drawn where the domain naturally separates: where concepts, terminology, and rules change.

When deciding service boundaries, prioritise:

1. **Domain clarity**: What is the *type* of data or calculation? Does this belong together conceptually?
2. **Reuse potential**: Will multiple products need this functionality?
3. **Technology fit**: Does this workload have distinct performance or scaling characteristics?

Avoid creating services for purely technical reasons (e.g., "this is a different framework") when the domain doesn't warrant separation.

### 2. Explicit Scope and Naming

Services should have clear, documented boundaries that prevent scope creep.

**Name services after their domain, not their initial consumer.** A service named after what it *does* rather than who first needed it will attract appropriate functionality. A service named after a consumer will attract unrelated functionality.

Each service should document:

- What it is responsible for
- What is explicitly out of scope
- The domain concepts it owns

When new functionality is proposed, ask: "Does this belong to this service's domain, or does it deserve its own home?"

### 3. Data Ownership

Each service owns its data. The API is the only path to that data.

- Services have their own databases, private to themselves
- No direct database access from other services or products
- All consumers use the service's API

This ensures services can evolve their internal data structures without breaking consumers, and provides a clear point for validation, access control, and audit.

### 4. Services on a Spectrum

Services exist on a spectrum from data access to domain logic. This is expected and acceptable.

Some services are primarily API layers providing structured access to data. Others encapsulate business domain calculations with minimal persistence. Many are a blend.

Do not force artificial classification. Design each service for its purpose:

- **Data-oriented services** focus on CRUD operations, query flexibility, and data integrity
- **Calculation-oriented services** focus on stateless computation, receiving inputs and returning results
- **Hybrid services** combine both, which is often the natural shape of a domain

Aim for services to be stateless where possible. Side effects like audit logging and error tracking are expected, but avoid services that manage complex workflow state.

### 5. Synchronous by Default

Default to synchronous communication. Use asynchronous patterns when the situation demands it.

Synchronous REST calls are simpler to understand, debug, and trace. They provide immediate feedback and clear error handling.

Consider asynchronous patterns (queues, events, parallel request pools) when:

- Customer experience requires non-blocking operations
- Data processing tasks can run independently
- Load patterns benefit from decoupling producer and consumer

The choice should be driven by customer experience and system behaviour, not technical preference.

### 6. Contract-First Design

Define service interfaces before implementation. The API is the contract.

- Services should have OpenAPI documentation
- Maintain backward compatibility where possible
- When breaking changes are necessary, coordinate with consumers
- Version APIs to manage breaking changes

Changes to service contracts require consideration of all consumers. Document changes, communicate in advance, and deploy in coordination when compatibility cannot be maintained.

See [API Design Philosophy](api-design.md) for detailed guidance on API contracts.

### 7. Independent Deployability

Services should be independently deployable.

Each service should:

- Live in its own repository
- Have its own CI/CD pipeline
- Be deployable without requiring simultaneous deployment of other services

This enables teams to ship changes quickly and reduces coordination overhead. It also requires discipline around backward compatibility and contract management.

### 8. Resilient Communication

Assume network failures. Design for graceful degradation.

When a service call fails, the caller should handle it appropriately. Strategies include:

- **Retry logic** for transient failures
- **Graceful degradation** when a dependency is unavailable
- **Clear error responses** when the operation cannot proceed

The appropriate strategy depends on context:

- Customer-facing operations may need graceful degradation
- Background processing may retry with backoff
- Critical operations may need to fail fast and clearly

Services should not assume their dependencies are always available.

### 9. Observable Systems

Observability is a first-class concern, not an afterthought.

Distributed systems are difficult to debug without proper instrumentation. Services should provide:

- **Structured logging** for error tracking and debugging
- **Distributed tracing** to follow requests across service boundaries
- **Metrics** for understanding system health and performance
- **Audit trails** for security and compliance

Invest in observability infrastructure. When something goes wrong in production, you should be able to understand what happened.

### 10. Secure by Design

Security is foundational, not optional.

**Authentication:**

- Internal services require machine-to-machine (M2M) authentication
- Use an Identity Provider (IdP) for credential storage, issuance, and verification
- User-facing products may use OAuth user credentials

**Authorisation:**

- Services trust authenticated callers to enforce user-level access controls
- User context does not propagate to internal services
- Products are responsible for ensuring appropriate user access and confidentiality

**Data Protection:**

- Encryption is required at rest and in transit
- Handle sensitive data according to security policies
- Consider audit and compliance requirements in service design

### 11. Pragmatic Decomposition

Extract services when there is clear benefit. Avoid premature distribution.

Creating a service introduces operational overhead: deployment pipelines, monitoring, contract management, network latency. This overhead is worth it when:

- Multiple products need the same functionality
- The domain boundary is clear and stable
- The service can be genuinely independent

Do not extract a service because it *might* be reused someday, or because the code is getting large. Extract when you have concrete reuse, clear boundaries, and the operational overhead is justified.

### 12. Smart Client, Thin Service

Services should be thin: data access and domain validation, not business orchestration.

Complexity belongs in clients (products) that understand their context, not in shared services that must remain general-purpose. When services try to anticipate every client need, they become bloated, opinionated, and difficult to evolve.

**Services should:**

- Provide clean data access with appropriate validation
- Enforce data integrity rules that apply universally
- Remain agnostic about how clients use the data

**Clients should:**

- Compose data from multiple service calls
- Apply business logic specific to their use case
- Own the user experience and workflow orchestration

This distribution keeps services reusable and clients flexible. A service that knows too much about its callers cannot easily serve new ones.

**Example:** A tax calculation service returns rates and thresholds. The product combines these with customer data and applies business rules for a specific workflow. The service does not need to know about the workflow.

---

## Anti-Patterns

These patterns cause problems in practice. Avoid them.

### Scope Creep Through Ambiguous Naming

A service with a vague or consumer-based name attracts unrelated functionality. Developers add features because the name sounds applicable, even when the domain doesn't fit.

**Example:** A service named "Integration Service" or "Helper Service" becomes a dumping ground. A service named "Taxpayer Tax Inventory" has a clearer scope.

**Prevention:** Name services after their domain. Document what is in and out of scope. Challenge additions that don't fit.

### Premature Service Extraction

Extracting a service before the domain is understood leads to wrong boundaries. Refactoring distributed systems is harder than refactoring monoliths.

**Prevention:** Let boundaries emerge from use. Extract when you have clear reuse across multiple products and understand the domain well.

### Shared Databases

Multiple services accessing the same database creates hidden coupling. Changes to the schema affect multiple services. Data integrity rules are scattered.

**Prevention:** Each service owns its data. If multiple services need the same data, one service owns it and exposes an API.

---

## Decision Framework

When facing architectural decisions, use this framework:

### Should this be a new service?

1. **Is the domain boundary clear?** Can you articulate what this service owns and what it doesn't?
2. **Is there concrete reuse?** Do multiple products need this now, or are you speculating?
3. **Can it be truly independent?** Can you deploy it without coordinating with other services?
4. **Is the overhead justified?** Is the operational cost worth the architectural benefit?

If you answer "no" to these questions, consider keeping the functionality in an existing service or product until the situation changes.

### Where does this functionality belong?

1. **What domain owns this concept?** Follow the data and business rules.
2. **Does a service already own this domain?** Extend it rather than creating overlap.
3. **Would adding this cause scope creep?** If it doesn't fit the existing service's domain, it may need a new home.

### How should these services communicate?

1. **Does the caller need an immediate response?** Use synchronous calls.
2. **Can the work happen in the background?** Consider async patterns.
3. **What happens if the dependency is unavailable?** Plan for failure.

---

## Related Documents

- [API Design Philosophy](api-design.md): Detailed guidance on API contracts and conventions
