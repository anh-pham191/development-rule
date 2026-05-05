---
name: vue-development
description: >
  Vue.js 3 development patterns, best practices, and component architecture for building maintainable Vue applications.
  Use when: (1) Working on vanilla Vue.js 3 projects (SPAs), (2) Creating Vue components, (3) Setting up Vue Router manually,
  (4) Managing state with Pinia, (5) Writing Vue tests, (6) Migrating from Options API to Composition API, (7) Debugging Vue applications,
  (8) Structuring Vue projects. Works with any build tool (Vite, Webpack, etc.) as build tools don't affect Vue code patterns.
  Note: Nuxt.js projects require different patterns (file-based routing, auto-imports, SSR) and are out of scope for this skill.
  Provides guidance on Composition API vs Options API, TypeScript integration, testing patterns, and modern Vue 3 workflows.
---

# Vue.js Development

Vue's declarative component model shapes how we think about UI development. We believe in leveraging Vue's reactivity system and composition patterns rather than fighting against them. This skill assumes front-end JavaScript development practices (testable, pragmatic, accessible). For a dedicated JavaScript skill, see `javascript-development` if available in your context.

## Scope and Framework Variations

This skill covers **vanilla Vue.js 3** (SPA applications). The core Vue patterns—Composition API, Options API, Pinia, Vue Router—are consistent across different setups.

### Build Tools (Vite, Webpack, etc.)

**Vite is the recommended build tool** for Vue 3 projects, but build tools don't affect Vue code patterns. Whether you use Vite, Webpack, or another tool, your Vue component code, Composition API usage, Pinia stores, and routing patterns remain the same.

**What varies:** Build configuration, dev server setup, and build optimisations. These are build-tool concerns, not Vue development patterns.

### Nuxt.js Framework

**Nuxt.js is out of scope for this skill.** Nuxt is a full-stack framework with different conventions:

- **File-based routing** (`pages/` directory) instead of manual Vue Router configuration
- **Auto-imports** for composables, components, and utilities
- **Server-side rendering** patterns and server API routes
- **Middleware system** for route protection
- **Layouts system** for page templates
- **Different project structure** (`pages/`, `layouts/`, `middleware/`, `server/`)

Nuxt requires a separate skill due to its distinct patterns and conventions. This skill focuses on vanilla Vue.js where you manually configure routing, imports, and project structure.

### When to Use This Skill

- ✅ Vanilla Vue.js 3 SPAs (Single Page Applications)
- ✅ Vue projects using Vue Router manually
- ✅ Projects with manual component/composable imports
- ✅ Any Vue 3 project regardless of build tool (Vite, Webpack, etc.)

### When NOT to Use This Skill

- ❌ Nuxt.js projects (different routing, auto-imports, SSR patterns)
- ❌ Vue 2 projects (different API, different patterns)
- ❌ Build tool configuration (Vite config, Webpack config, etc.)

## Philosophy

This skill is informed by [Vue Development Philosophy](../../../knowledge/philosophy/vue-development.md), which extends the front-end JavaScript philosophy. Summary:

### 1. Composition API First for New Code
Prefer Composition API with `<script setup>` for new components. It provides better TypeScript support, improved code organisation and reusability, clearer data flow, and easier testing.

### 2. TypeScript-First
Use TypeScript for new code. Type safety catches errors early and improves developer experience.

### 3. Pinia for State Management
Use Pinia for state management. It's the official recommendation for Vue 3 and provides better TypeScript support. Vuex is superseded by Pinia.

### 4. Component Composition
Build complex UIs from small, focused components. Follow atomic design principles when appropriate.

## Choosing Between Composition API and Options API

**Use Composition API (`<script setup>`) when:**
- Writing new components
- Component has complex logic or multiple concerns
- You need better TypeScript inference
- Logic needs to be shared across components
- Component will grow in complexity

**Use Options API when:**
- Working with existing Options API components
- Component is simple and primarily presentational
- Team is more familiar with Options API
- Maintaining consistency with existing codebase

**Migration strategy:** When modifying existing Options API components, consider migrating to Composition API if the component is being significantly refactored. For small changes, maintain the existing API style. For how to manage codebases during the transition, see the playbook [Vue: Options API to Composition API Migration](../../../knowledge/playbooks/vue-options-to-composition-migration.md) (when available in your context).

For detailed patterns and examples, see:
- [Composition API Patterns](references/composition-api.md)
- [Options API Patterns](references/options-api.md)

## Component Structure

### Composition API Component Template

```vue
<template>
  <div class="component-name">
    <!-- Template content -->
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { storeToRefs } from 'pinia';

// Props
interface Props {
  title: string;
  count?: number;
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
});

// Emits
const emit = defineEmits<{
  update: [value: string];
  close: [];
}>();

// State
const isLoading = ref(false);
const data = ref<string[]>([]);

// Computed
const displayCount = computed(() => props.count * 2);

// Methods
const handleClick = () => {
  emit('update', 'new value');
};

// Lifecycle
onMounted(async () => {
  isLoading.value = true;
  // Fetch data
  isLoading.value = false;
});
</script>

<style lang="scss" scoped>
.component-name {
  // Styles
}
</style>
```

### Options API Component Template

```vue
<template>
  <div class="component-name">
    <!-- Template content -->
  </div>
</template>

<script>
export default {
  name: 'ComponentName',
  props: {
    title: {
      type: String,
      required: true,
    },
    count: {
      type: Number,
      default: 0,
    },
  },
  emits: ['update', 'close'],
  data() {
    return {
      isLoading: false,
      data: [],
    };
  },
  computed: {
    displayCount() {
      return this.count * 2;
    },
  },
  methods: {
    handleClick() {
      this.$emit('update', 'new value');
    },
  },
  mounted() {
    this.isLoading = true;
    // Fetch data
    this.isLoading = false;
  },
};
</script>

<style lang="scss" scoped>
.component-name {
  // Styles
}
</style>
```

## State Management with Pinia

Use Pinia stores for shared state. Prefer Composition API setup stores:

```typescript
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

export const useMyStore = defineStore('myStore', () => {
  // State
  const items = ref<string[]>([]);
  const isLoading = ref(false);

  // Getters (computed)
  const itemCount = computed(() => items.value.length);

  // Actions
  const fetchItems = async () => {
    isLoading.value = true;
    try {
      items.value = await api.getItems();
    } finally {
      isLoading.value = false;
    }
  };

  return {
    items,
    isLoading,
    itemCount,
    fetchItems,
  };
});
```

**Using stores in components:**

```vue
<script setup lang="ts">
import { storeToRefs } from 'pinia';
import { useMyStore } from '@/stores/myStore';

const store = useMyStore();
// Use storeToRefs to maintain reactivity when destructuring
const { items, isLoading } = storeToRefs(store);
const { fetchItems } = store;

// Or access directly (also reactive)
const count = computed(() => store.itemCount);
</script>
```

For detailed Pinia patterns, see [Pinia State Management](references/pinia.md).

## Routing with Vue Router

Use Vue Router 4 with TypeScript:

```typescript
import { createRouter, createWebHashHistory, RouteRecordRaw } from 'vue-router';

const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    name: 'home',
    component: () => import('../views/Home.vue'),
    meta: { requiresAuth: true },
  },
];

const router = createRouter({
  history: createWebHashHistory(),
  routes,
});

// Navigation guards
router.beforeEach(async (to, from) => {
  if (to.meta.requiresAuth && !isAuthenticated()) {
    return { name: 'login' };
  }
});

export default router;
```

**Using router in components:**

```vue
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router';

const router = useRouter();
const route = useRoute();

const goToPage = () => {
  router.push({ name: 'home' });
};

// Access route params
const userId = route.params.id;
</script>
```

For detailed routing patterns, see [Vue Router Patterns](references/vue-router.md).

## Composables

Extract reusable logic into composables:

```typescript
// composables/useApi.ts
import { ref } from 'vue';

export function useApi<T>(url: string) {
  const data = ref<T | null>(null);
  const error = ref<Error | null>(null);
  const isLoading = ref(false);

  const fetch = async () => {
    isLoading.value = true;
    error.value = null;
    try {
      const response = await fetch(url);
      data.value = await response.json();
    } catch (e) {
      error.value = e as Error;
    } finally {
      isLoading.value = false;
    }
  };

  return {
    data,
    error,
    isLoading,
    fetch,
  };
}
```

**Using composables:**

```vue
<script setup lang="ts">
import { useApi } from '@/composables/useApi';
import { onMounted } from 'vue';

const { data, error, isLoading, fetch } = useApi<User[]>('/api/users');

onMounted(() => {
  fetch();
});
</script>
```

## Testing

Use Vitest and Vue Test Utils for component testing:

```typescript
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import MyComponent from '@/components/MyComponent.vue';

describe('MyComponent', () => {
  it('renders correctly', () => {
    const wrapper = mount(MyComponent, {
      props: {
        title: 'Test Title',
      },
    });
    expect(wrapper.text()).toContain('Test Title');
  });

  it('emits update event on click', async () => {
    const wrapper = mount(MyComponent);
    await wrapper.find('button').trigger('click');
    expect(wrapper.emitted('update')).toBeTruthy();
  });
});
```

For detailed testing patterns, see [Testing Patterns](references/testing.md).

## TypeScript Integration

### Props with TypeScript

```vue
<script setup lang="ts">
interface Props {
  title: string;
  count?: number;
  items: string[];
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
});
</script>
```

### Emits with TypeScript

```vue
<script setup lang="ts">
const emit = defineEmits<{
  update: [value: string];
  close: [];
  'item-selected': [id: number];
}>();
</script>
```

### Refs with TypeScript

```vue
<script setup lang="ts">
import { ref, Ref } from 'vue';

const count: Ref<number> = ref(0);
const user = ref<User | null>(null);
</script>
```

## Common Patterns

### Loading States

```vue
<script setup lang="ts">
const isLoading = ref(false);
const error = ref<Error | null>(null);

const fetchData = async () => {
  isLoading.value = true;
  error.value = null;
  try {
    // Fetch data
  } catch (e) {
    error.value = e as Error;
  } finally {
    isLoading.value = false;
  }
};
</script>

<template>
  <div v-if="isLoading">Loading...</div>
  <div v-else-if="error">Error: {{ error.message }}</div>
  <div v-else>Content</div>
</template>
```

### Conditional Rendering

```vue
<template>
  <!-- v-if for expensive toggles -->
  <ExpensiveComponent v-if="showExpensive" />
  
  <!-- v-show for frequent toggles -->
  <FrequentToggle v-show="showToggle" />
</template>
```

### List Rendering with Keys

```vue
<template>
  <div v-for="item in items" :key="item.id">
    {{ item.name }}
  </div>
</template>
```

### Event Handling

```vue
<template>
  <!-- Inline handler -->
  <button @click="count++">Increment</button>
  
  <!-- Method handler -->
  <button @click="handleClick">Click</button>
  
  <!-- With parameters -->
  <button @click="handleClick(id, $event)">Click</button>
  
  <!-- Event modifiers -->
  <form @submit.prevent="handleSubmit">
    <input @keyup.enter="submit" />
  </form>
</template>
```

## Project Structure

Organise Vue projects with clear separation. This structure applies regardless of build tool (Vite, Webpack, etc.):

```
src/
├── components/        # Reusable components
│   ├── atoms/        # Basic building blocks
│   ├── molecules/    # Composite components
│   └── organisms/    # Complex components
├── views/            # Page-level components
├── stores/           # Pinia stores
├── composables/      # Reusable composition functions
├── router/           # Vue Router configuration
├── services/         # API services
├── model/            # TypeScript models/interfaces
├── util/             # Utility functions
└── styles/           # Global styles
```

**Note:** Vite is the recommended build tool for Vue 3, but the project structure and Vue code patterns remain the same whether you use Vite, Webpack, Rollup, or another build tool. Build tools only affect configuration files (e.g., `vite.config.ts`, `webpack.config.js`) and build optimisations, not your Vue component code or application structure.

## Resources

For detailed patterns and examples, see:

- [Composition API Patterns](references/composition-api.md) - Detailed Composition API examples and patterns
- [Options API Patterns](references/options-api.md) - Options API patterns and migration guidance
- [Pinia State Management](references/pinia.md) - State management patterns and best practices
- [Vue Router Patterns](references/vue-router.md) - Routing patterns, guards, and navigation
- [Testing Patterns](references/testing.md) - Testing with Vitest and Vue Test Utils
