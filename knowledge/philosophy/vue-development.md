## Vue Development Philosophy

This document extends [Front-end JavaScript Development Philosophy](javascript-development.md).

Vue's declarative component model shapes how we think about UI development. We believe in leveraging Vue's reactivity system and composition patterns rather than fighting against them.

### 1. Composition API First

Prefer Composition API with `<script setup>` for new code. It provides better TypeScript support, improved code organisation, and clearer data flow.

- ✓ DO: Use `<script setup>` for new components
- ✓ DO: Extract composables for reusable logic
- ✓ DO: Leverage TypeScript inference
- ✗ DON'T: Mix Options and Composition API in one component
- ✗ DON'T: Avoid Composition API just because it's "new"

### 2. TypeScript-First

Use TypeScript for new code. Type safety catches errors early and improves developer experience.

- ✓ DO: Define interfaces for props and emits
- ✓ DO: Use type inference where possible
- ✓ DO: Type composables and stores
- ✗ DON'T: Use `any` without justification
- ✗ DON'T: Skip type definitions for complex data structures

### 3. Component Composition

Build complex UIs from small, focused components. Compose functionality rather than creating monolithic components.

- ✓ DO: Create single-purpose components
- ✓ DO: Extract reusable logic into composables
- ✓ DO: Follow atomic design principles when appropriate
- ✗ DON'T: Create god components that do everything
- ✗ DON'T: Put business logic in templates
- ✗ DON'T: Build deeply nested hierarchies

### 4. Explicit State Management

Use Pinia for shared state. Keep state management explicit and traceable.

- ✓ DO: Use Pinia stores for shared state
- ✓ DO: Keep stores focused on a single domain
- ✓ DO: Use getters for derived state
- ✗ DON'T: Create monolithic stores
- ✗ DON'T: Mutate state outside actions
- ✗ DON'T: Duplicate state across stores
- ✗ DON'T: Use Vuex (superseded by Pinia)

### 5. Declarative Templates

Templates should be declarative and readable. Keep logic out of templates.

- ✓ DO: Use template expressions for simple display logic
- ✓ DO: Use computed properties for derived values
- ✓ DO: Use methods for event handlers
- ✗ DON'T: Put complex logic in templates
- ✗ DON'T: Use `v-html` with untrusted data
- ✗ DON'T: Create deeply nested template expressions

### 6. Reactive by Default

Leverage Vue's reactivity system. Make state changes predictable and traceable.

- ✓ DO: Use `ref()` and `reactive()` appropriately
- ✓ DO: Understand reactivity boundaries
- ✓ DO: Use `storeToRefs()` when destructuring Pinia stores
- ✗ DON'T: Mutate props directly
- ✗ DON'T: Break reactivity by replacing entire objects unnecessarily
- ✗ DON'T: Ignore reactivity caveats
