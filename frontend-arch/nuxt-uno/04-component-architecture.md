# Component Architecture

## Overview

This document outlines the component architecture patterns for Nuxt applications using Vue 3's Composition API. It covers component organization, patterns, composition strategies, and best practices for building maintainable, reusable components.

## Component Categories

### 1. UI Primitives

Base-level components that serve as building blocks. These are highly reusable and have no business logic.

```
components/ui/
├── UiButton.vue
├── UiInput.vue
├── UiIcon.vue
├── UiSpinner.vue
├── UiBadge.vue
├── UiTooltip.vue
└── UiAvatar.vue
```

### 2. Common Components

Shared components used across multiple features with some application-specific logic.

```
components/common/
├── AppModal.vue
├── AppToast.vue
├── AppPaginator.vue
├── AppDropdown.vue
├── AppErrorBoundary.vue
└── AppConfirmDialog.vue
```

### 3. Layout Components

Components that define page structure and navigation.

```
components/layout/
├── LayoutHeader.vue
├── LayoutSidebar.vue
├── LayoutFooter.vue
├── LayoutBreadcrumb.vue
└── LayoutNavigation.vue
```

### 4. Domain Components

Feature-specific components organized by business domain.

```
components/user/
├── UserCard.vue
├── UserAvatar.vue
├── UserProfile.vue
└── UserSettings.vue

components/product/
├── ProductCard.vue
├── ProductList.vue
├── ProductDetail.vue
└── ProductFilter.vue
```

---

## Component Structure

### Standard Component Template

```vue
<script setup lang="ts">
/**
 * ComponentName
 * @description Brief description of the component's purpose
 */

// ==========================================
// Imports
// ==========================================
import { ref, computed, watch, onMounted } from 'vue'
import type { PropType } from 'vue'

// ==========================================
// Types
// ==========================================
interface Props {
  /** The title to display */
  title: string
  /** Optional description */
  description?: string
  /** Visual variant */
  variant?: 'primary' | 'secondary' | 'danger'
  /** Loading state */
  loading?: boolean
  /** Disabled state */
  disabled?: boolean
}

interface Emits {
  /** Emitted when action is triggered */
  (e: 'action', payload: { id: string }): void
  /** Emitted when component is closed */
  (e: 'close'): void
}

// ==========================================
// Props & Emits
// ==========================================
const props = withDefaults(defineProps<Props>(), {
  description: '',
  variant: 'primary',
  loading: false,
  disabled: false,
})

const emit = defineEmits<Emits>()

// ==========================================
// Composables
// ==========================================
const { t } = useI18n()

// ==========================================
// State
// ==========================================
const isOpen = ref(false)
const inputValue = ref('')

// ==========================================
// Computed
// ==========================================
const computedClass = computed(() => [
  'component-base',
  `variant-${props.variant}`,
  {
    'is-loading': props.loading,
    'is-disabled': props.disabled,
  },
])

const isActionDisabled = computed(() => props.disabled || props.loading)

// ==========================================
// Methods
// ==========================================
function handleAction() {
  if (isActionDisabled.value) return
  emit('action', { id: 'some-id' })
}

function handleClose() {
  isOpen.value = false
  emit('close')
}

// ==========================================
// Watchers
// ==========================================
watch(
  () => props.title,
  (newTitle) => {
    console.log('Title changed:', newTitle)
  },
)

// ==========================================
// Lifecycle
// ==========================================
onMounted(() => {
  // Initialization logic
})

// ==========================================
// Expose (if needed)
// ==========================================
defineExpose({
  isOpen,
  handleClose,
})
</script>

<template>
  <div :class="computedClass">
    <header class="component-header">
      <h3 class="component-title">{{ title }}</h3>
      <p v-if="description" class="component-description">
        {{ description }}
      </p>
    </header>

    <main class="component-body">
      <slot />
    </main>

    <footer class="component-footer">
      <slot name="footer">
        <button
          type="button"
          :disabled="isActionDisabled"
          @click="handleAction"
        >
          {{ t('common.action') }}
        </button>
      </slot>
    </footer>
  </div>
</template>

<style scoped>
.component-base {
  /* Component styles */
}

.variant-primary {
  /* Primary variant */
}

.variant-secondary {
  /* Secondary variant */
}

.variant-danger {
  /* Danger variant */
}

.is-loading {
  /* Loading state */
}

.is-disabled {
  /* Disabled state */
}
</style>
```

---

## Props Patterns

### Typed Props with Defaults

```vue
<script setup lang="ts">
interface Props {
  // Required props
  id: string
  title: string
  
  // Optional props with specific types
  size?: 'sm' | 'md' | 'lg' | 'xl'
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost'
  
  // Boolean props
  disabled?: boolean
  loading?: boolean
  fullWidth?: boolean
  
  // Complex types
  items?: Array<{ id: string; label: string }>
  config?: Record<string, unknown>
  
  // Function props
  onAction?: (id: string) => void | Promise<void>
}

const props = withDefaults(defineProps<Props>(), {
  size: 'md',
  variant: 'primary',
  disabled: false,
  loading: false,
  fullWidth: false,
  items: () => [],
  config: () => ({}),
})
</script>
```

### Prop Validation

```vue
<script setup lang="ts">
interface Props {
  // Validated through TypeScript
  email: string
  age: number
  status: 'active' | 'inactive' | 'pending'
}

const props = defineProps<Props>()

// Runtime validation (if needed)
const isValidEmail = computed(() => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(props.email)
})
</script>
```

### Complex Object Props

```vue
<script setup lang="ts">
// Define interfaces in a separate types file
interface User {
  id: string
  name: string
  email: string
  avatar?: string
  role: 'admin' | 'user' | 'guest'
}

interface Props {
  user: User
  settings?: Partial<User>
}

const props = defineProps<Props>()

// Destructure with computed for reactivity
const userName = computed(() => props.user.name)
</script>
```

---

## Emits Patterns

### Typed Emits

```vue
<script setup lang="ts">
interface Emits {
  // Simple events
  (e: 'close'): void
  (e: 'open'): void
  
  // Events with payloads
  (e: 'update', value: string): void
  (e: 'select', item: { id: string; label: string }): void
  
  // Events with multiple arguments
  (e: 'change', value: string, previousValue: string): void
  
  // Events matching v-model
  (e: 'update:modelValue', value: string): void
}

const emit = defineEmits<Emits>()

// Usage
function handleUpdate(newValue: string) {
  emit('update', newValue)
}
</script>
```

### v-model Integration

```vue
<script setup lang="ts">
interface Props {
  modelValue: string
}

const props = defineProps<Props>()
const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()

// Local ref synced with v-model
const localValue = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value),
})
</script>

<template>
  <input v-model="localValue" type="text" />
</template>
```

### Multiple v-model

```vue
<script setup lang="ts">
interface Props {
  modelValue: string
  title: string
  visible: boolean
}

const props = defineProps<Props>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
  (e: 'update:title', value: string): void
  (e: 'update:visible', value: boolean): void
}>()
</script>

<!-- Usage -->
<!-- <MyComponent v-model="content" v-model:title="title" v-model:visible="isVisible" /> -->
```

---

## Slots Patterns

### Basic Slots

```vue
<script setup lang="ts">
// Define slot types for documentation
interface Slots {
  default?: () => any
  header?: () => any
  footer?: (props: { loading: boolean }) => any
}

defineSlots<Slots>()
</script>

<template>
  <div class="card">
    <header v-if="$slots.header" class="card-header">
      <slot name="header" />
    </header>

    <main class="card-body">
      <slot />
    </main>

    <footer v-if="$slots.footer" class="card-footer">
      <slot name="footer" :loading="loading" />
    </footer>
  </div>
</template>
```

### Scoped Slots

```vue
<script setup lang="ts">
interface Item {
  id: string
  title: string
  description: string
}

interface Props {
  items: Item[]
}

interface Slots {
  default?: (props: { item: Item; index: number }) => any
  empty?: () => any
}

const props = defineProps<Props>()
defineSlots<Slots>()
</script>

<template>
  <div class="list">
    <template v-if="items.length">
      <div v-for="(item, index) in items" :key="item.id" class="list-item">
        <slot :item="item" :index="index">
          <!-- Default rendering -->
          <span>{{ item.title }}</span>
        </slot>
      </div>
    </template>
    
    <div v-else class="list-empty">
      <slot name="empty">
        <p>No items found</p>
      </slot>
    </div>
  </div>
</template>
```

### Render Functions with Slots

```vue
<script setup lang="ts">
import { h, type VNode } from 'vue'

interface Props {
  tag?: string
  items: string[]
}

const props = withDefaults(defineProps<Props>(), {
  tag: 'div',
})

const slots = defineSlots<{
  default?: (props: { items: string[] }) => VNode[]
}>()

// Render function usage
function renderContent() {
  if (slots.default) {
    return slots.default({ items: props.items })
  }
  return props.items.map((item) => h('span', { key: item }, item))
}
</script>

<template>
  <component :is="tag">
    <component :is="renderContent" />
  </component>
</template>
```

---

## Component Composition

### Using Composables

```vue
<script setup lang="ts">
// Import composables
import { useUser } from '~/composables/useUser'
import { useToast } from '~/composables/useToast'
import { useApi } from '~/composables/useApi'

// Use composables
const { user, loading: userLoading, fetchUser } = useUser()
const { showToast } = useToast()
const { post, loading: apiLoading } = useApi()

// Combined loading state
const isLoading = computed(() => userLoading.value || apiLoading.value)

async function handleSubmit(data: FormData) {
  try {
    await post('/api/submit', data)
    showToast('Success!', 'success')
  } catch (error) {
    showToast('Error occurred', 'error')
  }
}
</script>
```

### Component with Provide/Inject

```vue
<!-- Parent Component -->
<script setup lang="ts">
import { provide, ref } from 'vue'
import type { InjectionKey } from 'vue'

// Define injection key with type
interface ThemeContext {
  theme: Ref<'light' | 'dark'>
  toggleTheme: () => void
}

export const ThemeKey: InjectionKey<ThemeContext> = Symbol('theme')

// Provide context
const theme = ref<'light' | 'dark'>('light')

function toggleTheme() {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

provide(ThemeKey, { theme, toggleTheme })
</script>

<!-- Child Component -->
<script setup lang="ts">
import { inject } from 'vue'
import { ThemeKey } from './ParentComponent.vue'

// Inject context
const themeContext = inject(ThemeKey)

if (!themeContext) {
  throw new Error('ThemeKey not provided')
}

const { theme, toggleTheme } = themeContext
</script>
```

### Compound Components

```vue
<!-- Tabs.vue (Parent) -->
<script setup lang="ts">
import { provide, ref } from 'vue'

interface TabsContext {
  activeTab: Ref<string>
  setActiveTab: (id: string) => void
}

const TabsKey = Symbol('tabs') as InjectionKey<TabsContext>

const activeTab = ref('')

function setActiveTab(id: string) {
  activeTab.value = id
}

provide(TabsKey, { activeTab, setActiveTab })

onMounted(() => {
  // Set first tab as active if none selected
  if (!activeTab.value) {
    // Logic to set first tab
  }
})
</script>

<template>
  <div class="tabs">
    <slot />
  </div>
</template>

<!-- TabList.vue -->
<script setup lang="ts">
</script>

<template>
  <div class="tab-list" role="tablist">
    <slot />
  </div>
</template>

<!-- Tab.vue -->
<script setup lang="ts">
import { inject } from 'vue'

interface Props {
  id: string
  label: string
}

const props = defineProps<Props>()
const { activeTab, setActiveTab } = inject(TabsKey)!

const isActive = computed(() => activeTab.value === props.id)
</script>

<template>
  <button
    role="tab"
    :aria-selected="isActive"
    :class="{ active: isActive }"
    @click="setActiveTab(id)"
  >
    {{ label }}
  </button>
</template>

<!-- TabPanel.vue -->
<script setup lang="ts">
import { inject } from 'vue'

interface Props {
  id: string
}

const props = defineProps<Props>()
const { activeTab } = inject(TabsKey)!

const isVisible = computed(() => activeTab.value === props.id)
</script>

<template>
  <div v-show="isVisible" role="tabpanel">
    <slot />
  </div>
</template>
```

---

## State Management in Components

### Local State

```vue
<script setup lang="ts">
// Simple refs
const count = ref(0)
const name = ref('')
const isOpen = ref(false)

// Reactive objects
const form = reactive({
  email: '',
  password: '',
  remember: false,
})

// Shallow refs for performance
const largeData = shallowRef<LargeObject[]>([])

// Computed properties
const doubleCount = computed(() => count.value * 2)

const isFormValid = computed(() => {
  return form.email.length > 0 && form.password.length >= 8
})
</script>
```

### Form State Management

```vue
<script setup lang="ts">
interface FormData {
  email: string
  password: string
  confirmPassword: string
}

interface FormErrors {
  email?: string
  password?: string
  confirmPassword?: string
}

// Form state
const form = reactive<FormData>({
  email: '',
  password: '',
  confirmPassword: '',
})

const errors = reactive<FormErrors>({})
const isSubmitting = ref(false)

// Validation
function validate(): boolean {
  errors.email = undefined
  errors.password = undefined
  errors.confirmPassword = undefined

  if (!form.email) {
    errors.email = 'Email is required'
  } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email)) {
    errors.email = 'Invalid email format'
  }

  if (!form.password) {
    errors.password = 'Password is required'
  } else if (form.password.length < 8) {
    errors.password = 'Password must be at least 8 characters'
  }

  if (form.password !== form.confirmPassword) {
    errors.confirmPassword = 'Passwords do not match'
  }

  return !errors.email && !errors.password && !errors.confirmPassword
}

// Submit handler
async function handleSubmit() {
  if (!validate()) return

  isSubmitting.value = true
  try {
    await submitForm(form)
  } finally {
    isSubmitting.value = false
  }
}

// Reset form
function resetForm() {
  form.email = ''
  form.password = ''
  form.confirmPassword = ''
  Object.keys(errors).forEach((key) => {
    errors[key as keyof FormErrors] = undefined
  })
}
</script>
```

---

## Accessibility Best Practices

### ARIA Attributes

```vue
<script setup lang="ts">
const isOpen = ref(false)
const buttonId = useId()
const panelId = useId()
</script>

<template>
  <div class="accordion">
    <button
      :id="buttonId"
      type="button"
      :aria-expanded="isOpen"
      :aria-controls="panelId"
      @click="isOpen = !isOpen"
    >
      <span>Toggle Section</span>
      <UiIcon :name="isOpen ? 'chevron-up' : 'chevron-down'" aria-hidden="true" />
    </button>

    <div
      :id="panelId"
      role="region"
      :aria-labelledby="buttonId"
      :hidden="!isOpen"
    >
      <slot />
    </div>
  </div>
</template>
```

### Keyboard Navigation

```vue
<script setup lang="ts">
interface Props {
  items: Array<{ id: string; label: string }>
}

const props = defineProps<Props>()
const focusedIndex = ref(0)

function handleKeydown(event: KeyboardEvent) {
  switch (event.key) {
    case 'ArrowDown':
      event.preventDefault()
      focusedIndex.value = Math.min(focusedIndex.value + 1, props.items.length - 1)
      break
    case 'ArrowUp':
      event.preventDefault()
      focusedIndex.value = Math.max(focusedIndex.value - 1, 0)
      break
    case 'Home':
      event.preventDefault()
      focusedIndex.value = 0
      break
    case 'End':
      event.preventDefault()
      focusedIndex.value = props.items.length - 1
      break
    case 'Enter':
    case ' ':
      event.preventDefault()
      selectItem(props.items[focusedIndex.value])
      break
  }
}
</script>

<template>
  <ul role="listbox" @keydown="handleKeydown">
    <li
      v-for="(item, index) in items"
      :key="item.id"
      role="option"
      :aria-selected="index === focusedIndex"
      :tabindex="index === focusedIndex ? 0 : -1"
      @click="selectItem(item)"
    >
      {{ item.label }}
    </li>
  </ul>
</template>
```

### Focus Management

```vue
<script setup lang="ts">
const inputRef = ref<HTMLInputElement>()
const isOpen = ref(false)

// Focus input when modal opens
watch(isOpen, (open) => {
  if (open) {
    nextTick(() => {
      inputRef.value?.focus()
    })
  }
})

// Expose focus method
defineExpose({
  focus: () => inputRef.value?.focus(),
})
</script>

<template>
  <div v-if="isOpen" role="dialog" aria-modal="true">
    <input ref="inputRef" type="text" />
  </div>
</template>
```

---

## Performance Patterns

### Lazy Loading Components

```vue
<script setup lang="ts">
// Lazy load heavy components
const HeavyChart = defineAsyncComponent(() =>
  import('~/components/HeavyChart.vue')
)

const HeavyTable = defineAsyncComponent({
  loader: () => import('~/components/HeavyTable.vue'),
  loadingComponent: () => import('~/components/LoadingSpinner.vue'),
  delay: 200,
  timeout: 10000,
})
</script>

<template>
  <Suspense>
    <template #default>
      <HeavyChart :data="chartData" />
    </template>
    <template #fallback>
      <div class="loading">Loading chart...</div>
    </template>
  </Suspense>
</template>
```

### Virtual Scrolling

```vue
<script setup lang="ts">
import { useVirtualList } from '@vueuse/core'

interface Item {
  id: string
  name: string
}

const props = defineProps<{
  items: Item[]
}>()

const { list, containerProps, wrapperProps } = useVirtualList(
  computed(() => props.items),
  {
    itemHeight: 50,
    overscan: 5,
  },
)
</script>

<template>
  <div v-bind="containerProps" class="virtual-list-container">
    <div v-bind="wrapperProps">
      <div
        v-for="{ data, index } in list"
        :key="data.id"
        class="virtual-list-item"
      >
        {{ data.name }}
      </div>
    </div>
  </div>
</template>
```

### Memoization

```vue
<script setup lang="ts">
import { computed, shallowRef } from 'vue'

// Use shallowRef for large arrays/objects
const items = shallowRef<Item[]>([])

// Memoize expensive computations
const expensiveResult = computed(() => {
  return items.value.reduce((acc, item) => {
    // Expensive calculation
    return acc + complexCalculation(item)
  }, 0)
})

// Use v-memo in template for list items
</script>

<template>
  <div>
    <div
      v-for="item in items"
      :key="item.id"
      v-memo="[item.id, item.updatedAt]"
    >
      <ExpensiveChild :item="item" />
    </div>
  </div>
</template>
```

---

## Testing Components

### Component Test Structure

```typescript
// components/user/__tests__/UserCard.test.ts
import { mount } from '@vue/test-utils'
import { describe, it, expect, vi } from 'vitest'
import UserCard from '../UserCard.vue'

describe('UserCard', () => {
  const defaultProps = {
    user: {
      id: '1',
      name: 'John Doe',
      email: 'john@example.com',
    },
  }

  it('renders user name', () => {
    const wrapper = mount(UserCard, {
      props: defaultProps,
    })

    expect(wrapper.text()).toContain('John Doe')
  })

  it('emits select event when clicked', async () => {
    const wrapper = mount(UserCard, {
      props: defaultProps,
    })

    await wrapper.trigger('click')

    expect(wrapper.emitted('select')).toBeTruthy()
    expect(wrapper.emitted('select')![0]).toEqual([defaultProps.user])
  })

  it('applies loading state correctly', () => {
    const wrapper = mount(UserCard, {
      props: { ...defaultProps, loading: true },
    })

    expect(wrapper.classes()).toContain('is-loading')
    expect(wrapper.find('.spinner').exists()).toBe(true)
  })

  it('renders slot content', () => {
    const wrapper = mount(UserCard, {
      props: defaultProps,
      slots: {
        actions: '<button>Edit</button>',
      },
    })

    expect(wrapper.find('button').text()).toBe('Edit')
  })
})
```

---

## Best Practices Summary

### Component Design

1. **Single Responsibility**: One component, one purpose
2. **Props Down, Events Up**: Clear data flow direction
3. **Type Everything**: Full TypeScript coverage
4. **Document Props**: Add JSDoc comments for complex props
5. **Default Values**: Always provide sensible defaults

### Organization

1. **Domain-Driven**: Group by feature, not type
2. **Consistent Naming**: Follow naming conventions strictly
3. **Colocate Tests**: Keep tests near components
4. **Extract Logic**: Move complex logic to composables

### Performance

1. **Lazy Load**: Heavy components loaded on demand
2. **Virtual Lists**: For large data sets
3. **Memoization**: Cache expensive computations
4. **Shallow Refs**: For large reactive objects

### Accessibility

1. **Semantic HTML**: Use correct elements
2. **ARIA Labels**: For custom widgets
3. **Keyboard Navigation**: Full keyboard support
4. **Focus Management**: Proper focus handling

---

## Next Steps

Continue to [Composables Patterns](./05-composables-patterns.md) to learn about creating and organizing Vue 3 composables for logic reuse.