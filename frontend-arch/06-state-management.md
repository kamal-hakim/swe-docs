# State Management

## Overview

This document covers state management patterns for Nuxt applications using **Pinia**, Vue's official state management library. It includes store architecture, patterns for common use cases, and best practices for both SPA and SSR environments.

## State Management Options

### When to Use Each Approach

| Approach | Use Case |
|----------|----------|
| **Local State (ref/reactive)** | Component-specific state |
| **Props/Emits** | Parent-child communication |
| **Provide/Inject** | Deep component tree sharing |
| **Composables** | Reusable logic with state |
| **useState (Nuxt)** | SSR-safe shared state |
| **Pinia Stores** | Global application state |

---

## Pinia Setup

### Installation and Configuration

```bash
# Install Pinia (usually included with @pinia/nuxt)
pnpm add pinia @pinia/nuxt
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@pinia/nuxt'],
  
  pinia: {
    storesDirs: ['./app/stores/**'],
  },
})
```

### Store Directory Structure

```
app/stores/
├── auth.ts                     # Authentication state
├── user.ts                     # User profile state
├── app.ts                      # Application settings
├── notification.ts             # Notifications
├── cart.ts                     # Shopping cart (example)
└── index.ts                    # Store exports (optional)
```

---

## Store Patterns

### Setup Store Pattern (Recommended)

The Setup Store pattern uses Vue's Composition API syntax, providing better TypeScript support and more flexibility.

```typescript
// stores/user.ts
import { defineStore } from 'pinia'

interface User {
  id: string
  name: string
  email: string
  avatar?: string
  role: 'admin' | 'user' | 'guest'
}

interface UserPreferences {
  theme: 'light' | 'dark' | 'system'
  language: string
  notifications: boolean
}

export const useUserStore = defineStore('user', () => {
  // ==========================================
  // State
  // ==========================================
  const currentUser = ref<User | null>(null)
  const preferences = ref<UserPreferences>({
    theme: 'system',
    language: 'en',
    notifications: true,
  })
  const loading = ref(false)
  const error = ref<Error | null>(null)

  // ==========================================
  // Getters (Computed)
  // ==========================================
  const isAuthenticated = computed(() => !!currentUser.value)
  const isAdmin = computed(() => currentUser.value?.role === 'admin')
  const displayName = computed(() => currentUser.value?.name || 'Guest')
  const userInitials = computed(() => {
    const name = currentUser.value?.name || ''
    return name
      .split(' ')
      .map(n => n[0])
      .join('')
      .toUpperCase()
      .slice(0, 2)
  })

  // ==========================================
  // Actions
  // ==========================================
  async function fetchUser() {
    if (currentUser.value) return currentUser.value

    loading.value = true
    error.value = null

    try {
      const response = await $fetch<User>('/api/auth/me')
      currentUser.value = response
      return response
    } catch (err) {
      error.value = err as Error
      throw err
    } finally {
      loading.value = false
    }
  }

  async function updateProfile(updates: Partial<User>) {
    if (!currentUser.value) throw new Error('Not authenticated')

    loading.value = true
    error.value = null

    try {
      const response = await $fetch<User>('/api/user/profile', {
        method: 'PATCH',
        body: updates,
      })
      currentUser.value = response
      return response
    } catch (err) {
      error.value = err as Error
      throw err
    } finally {
      loading.value = false
    }
  }

  function updatePreferences(updates: Partial<UserPreferences>) {
    preferences.value = { ...preferences.value, ...updates }
  }

  function setUser(user: User | null) {
    currentUser.value = user
  }

  function clearUser() {
    currentUser.value = null
    preferences.value = {
      theme: 'system',
      language: 'en',
      notifications: true,
    }
  }

  // ==========================================
  // Return
  // ==========================================
  return {
    // State
    currentUser: readonly(currentUser),
    preferences,
    loading: readonly(loading),
    error: readonly(error),

    // Getters
    isAuthenticated,
    isAdmin,
    displayName,
    userInitials,

    // Actions
    fetchUser,
    updateProfile,
    updatePreferences,
    setUser,
    clearUser,
  }
})
```

### Options Store Pattern

For simpler stores or those migrating from Vuex:

```typescript
// stores/counter.ts
import { defineStore } from 'pinia'

interface CounterState {
  count: number
  history: number[]
}

export const useCounterStore = defineStore('counter', {
  state: (): CounterState => ({
    count: 0,
    history: [],
  }),

  getters: {
    doubleCount: (state) => state.count * 2,
    
    isPositive: (state) => state.count > 0,
    
    lastHistoryItem(): number | undefined {
      return this.history[this.history.length - 1]
    },
  },

  actions: {
    increment() {
      this.count++
      this.history.push(this.count)
    },

    decrement() {
      this.count--
      this.history.push(this.count)
    },

    async incrementAsync() {
      await new Promise(resolve => setTimeout(resolve, 1000))
      this.increment()
    },

    reset() {
      this.count = 0
      this.history = []
    },
  },
})
```

---

## Common Store Patterns

### Authentication Store

```typescript
// stores/auth.ts
import { defineStore } from 'pinia'

interface AuthCredentials {
  email: string
  password: string
}

interface AuthTokens {
  accessToken: string
  refreshToken: string
  expiresAt: number
}

interface AuthState {
  tokens: AuthTokens | null
  isInitialized: boolean
}

export const useAuthStore = defineStore('auth', () => {
  // State
  const tokens = ref<AuthTokens | null>(null)
  const isInitialized = ref(false)
  const loading = ref(false)

  // Other stores
  const userStore = useUserStore()

  // Computed
  const isAuthenticated = computed(() => {
    if (!tokens.value) return false
    return tokens.value.expiresAt > Date.now()
  })

  const accessToken = computed(() => tokens.value?.accessToken)

  // Actions
  async function login(credentials: AuthCredentials) {
    loading.value = true

    try {
      const response = await $fetch<{ tokens: AuthTokens; user: any }>('/api/auth/login', {
        method: 'POST',
        body: credentials,
      })

      tokens.value = response.tokens
      userStore.setUser(response.user)

      // Persist tokens
      if (import.meta.client) {
        localStorage.setItem('auth_tokens', JSON.stringify(response.tokens))
      }

      return response
    } finally {
      loading.value = false
    }
  }

  async function logout() {
    try {
      await $fetch('/api/auth/logout', { method: 'POST' })
    } catch {
      // Ignore logout errors
    } finally {
      tokens.value = null
      userStore.clearUser()

      if (import.meta.client) {
        localStorage.removeItem('auth_tokens')
      }
    }
  }

  async function refreshTokens() {
    if (!tokens.value?.refreshToken) {
      throw new Error('No refresh token available')
    }

    try {
      const response = await $fetch<AuthTokens>('/api/auth/refresh', {
        method: 'POST',
        body: { refreshToken: tokens.value.refreshToken },
      })

      tokens.value = response

      if (import.meta.client) {
        localStorage.setItem('auth_tokens', JSON.stringify(response))
      }

      return response
    } catch (error) {
      // Refresh failed, logout user
      await logout()
      throw error
    }
  }

  async function initialize() {
    if (isInitialized.value) return

    if (import.meta.client) {
      const stored = localStorage.getItem('auth_tokens')
      if (stored) {
        try {
          const parsed = JSON.parse(stored) as AuthTokens
          if (parsed.expiresAt > Date.now()) {
            tokens.value = parsed
            await userStore.fetchUser()
          } else if (parsed.refreshToken) {
            await refreshTokens()
            await userStore.fetchUser()
          }
        } catch {
          localStorage.removeItem('auth_tokens')
        }
      }
    }

    isInitialized.value = true
  }

  return {
    // State
    tokens: readonly(tokens),
    loading: readonly(loading),
    isInitialized: readonly(isInitialized),

    // Computed
    isAuthenticated,
    accessToken,

    // Actions
    login,
    logout,
    refreshTokens,
    initialize,
  }
})
```

### Application Settings Store

```typescript
// stores/app.ts
import { defineStore } from 'pinia'

interface AppSettings {
  sidebarCollapsed: boolean
  language: string
  timezone: string
  dateFormat: string
  itemsPerPage: number
}

const defaultSettings: AppSettings = {
  sidebarCollapsed: false,
  language: 'en',
  timezone: 'UTC',
  dateFormat: 'YYYY-MM-DD',
  itemsPerPage: 20,
}

export const useAppStore = defineStore('app', () => {
  // State
  const settings = ref<AppSettings>({ ...defaultSettings })
  const isLoading = ref(false)
  const isMobile = ref(false)

  // Persist settings
  const persistedSettings = import.meta.client
    ? useLocalStorage<AppSettings>('app_settings', defaultSettings)
    : ref(defaultSettings)

  // Initialize from persisted
  settings.value = { ...defaultSettings, ...persistedSettings.value }

  // Computed
  const sidebarOpen = computed(() => !settings.value.sidebarCollapsed && !isMobile.value)

  // Actions
  function updateSettings(updates: Partial<AppSettings>) {
    settings.value = { ...settings.value, ...updates }
    persistedSettings.value = settings.value
  }

  function toggleSidebar() {
    updateSettings({ sidebarCollapsed: !settings.value.sidebarCollapsed })
  }

  function setMobile(mobile: boolean) {
    isMobile.value = mobile
  }

  function resetSettings() {
    settings.value = { ...defaultSettings }
    persistedSettings.value = defaultSettings
  }

  return {
    settings: readonly(settings),
    isLoading,
    isMobile: readonly(isMobile),
    sidebarOpen,
    updateSettings,
    toggleSidebar,
    setMobile,
    resetSettings,
  }
})
```

### Notification Store

```typescript
// stores/notification.ts
import { defineStore } from 'pinia'

interface Notification {
  id: string
  type: 'success' | 'error' | 'warning' | 'info'
  title: string
  message?: string
  duration?: number
  dismissible?: boolean
  action?: {
    label: string
    handler: () => void
  }
}

type NotificationInput = Omit<Notification, 'id'>

export const useNotificationStore = defineStore('notification', () => {
  // State
  const notifications = ref<Notification[]>([])

  // Computed
  const hasNotifications = computed(() => notifications.value.length > 0)

  const latestNotification = computed(() => 
    notifications.value[notifications.value.length - 1]
  )

  // Actions
  function add(notification: NotificationInput): string {
    const id = crypto.randomUUID()
    const duration = notification.duration ?? 5000
    const dismissible = notification.dismissible ?? true

    notifications.value.push({
      id,
      ...notification,
      duration,
      dismissible,
    })

    // Auto-dismiss
    if (duration > 0) {
      setTimeout(() => {
        dismiss(id)
      }, duration)
    }

    return id
  }

  function dismiss(id: string) {
    const index = notifications.value.findIndex(n => n.id === id)
    if (index !== -1) {
      notifications.value.splice(index, 1)
    }
  }

  function dismissAll() {
    notifications.value = []
  }

  // Convenience methods
  function success(title: string, message?: string) {
    return add({ type: 'success', title, message })
  }

  function error(title: string, message?: string) {
    return add({ type: 'error', title, message, duration: 0 })
  }

  function warning(title: string, message?: string) {
    return add({ type: 'warning', title, message })
  }

  function info(title: string, message?: string) {
    return add({ type: 'info', title, message })
  }

  return {
    notifications: readonly(notifications),
    hasNotifications,
    latestNotification,
    add,
    dismiss,
    dismissAll,
    success,
    error,
    warning,
    info,
  }
})
```

### Data Collection Store Pattern

```typescript
// stores/products.ts
import { defineStore } from 'pinia'

interface Product {
  id: string
  name: string
  price: number
  category: string
  inStock: boolean
}

interface ProductFilters {
  search: string
  category: string | null
  inStockOnly: boolean
  priceRange: [number, number] | null
}

interface ProductsState {
  items: Product[]
  filters: ProductFilters
  sortBy: 'name' | 'price' | 'category'
  sortOrder: 'asc' | 'desc'
  pagination: {
    page: number
    perPage: number
    total: number
  }
}

export const useProductsStore = defineStore('products', () => {
  // State
  const items = ref<Product[]>([])
  const loading = ref(false)
  const error = ref<Error | null>(null)

  const filters = ref<ProductFilters>({
    search: '',
    category: null,
    inStockOnly: false,
    priceRange: null,
  })

  const sortBy = ref<'name' | 'price' | 'category'>('name')
  const sortOrder = ref<'asc' | 'desc'>('asc')

  const pagination = ref({
    page: 1,
    perPage: 20,
    total: 0,
  })

  // Computed
  const filteredItems = computed(() => {
    let result = [...items.value]

    // Apply search filter
    if (filters.value.search) {
      const search = filters.value.search.toLowerCase()
      result = result.filter(item =>
        item.name.toLowerCase().includes(search) ||
        item.category.toLowerCase().includes(search)
      )
    }

    // Apply category filter
    if (filters.value.category) {
      result = result.filter(item => item.category === filters.value.category)
    }

    // Apply stock filter
    if (filters.value.inStockOnly) {
      result = result.filter(item => item.inStock)
    }

    // Apply price range filter
    if (filters.value.priceRange) {
      const [min, max] = filters.value.priceRange
      result = result.filter(item => item.price >= min && item.price <= max)
    }

    return result
  })

  const sortedItems = computed(() => {
    const sorted = [...filteredItems.value]
    
    sorted.sort((a, b) => {
      let comparison = 0
      
      switch (sortBy.value) {
        case 'name':
          comparison = a.name.localeCompare(b.name)
          break
        case 'price':
          comparison = a.price - b.price
          break
        case 'category':
          comparison = a.category.localeCompare(b.category)
          break
      }
      
      return sortOrder.value === 'asc' ? comparison : -comparison
    })

    return sorted
  })

  const paginatedItems = computed(() => {
    const start = (pagination.value.page - 1) * pagination.value.perPage
    const end = start + pagination.value.perPage
    return sortedItems.value.slice(start, end)
  })

  const totalPages = computed(() =>
    Math.ceil(filteredItems.value.length / pagination.value.perPage)
  )

  const categories = computed(() => {
    const cats = new Set(items.value.map(item => item.category))
    return Array.from(cats).sort()
  })

  // Actions
  async function fetchProducts() {
    loading.value = true
    error.value = null

    try {
      const response = await $fetch<{ items: Product[]; total: number }>('/api/products')
      items.value = response.items
      pagination.value.total = response.total
    } catch (err) {
      error.value = err as Error
      throw err
    } finally {
      loading.value = false
    }
  }

  function setFilter<K extends keyof ProductFilters>(key: K, value: ProductFilters[K]) {
    filters.value[key] = value
    pagination.value.page = 1 // Reset to first page on filter change
  }

  function clearFilters() {
    filters.value = {
      search: '',
      category: null,
      inStockOnly: false,
      priceRange: null,
    }
    pagination.value.page = 1
  }

  function setSort(field: typeof sortBy.value, order?: typeof sortOrder.value) {
    if (sortBy.value === field && !order) {
      sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc'
    } else {
      sortBy.value = field
      sortOrder.value = order || 'asc'
    }
  }

  function setPage(page: number) {
    pagination.value.page = Math.max(1, Math.min(page, totalPages.value))
  }

  function getById(id: string) {
    return items.value.find(item => item.id === id)
  }

  return {
    // State
    items: readonly(items),
    loading: readonly(loading),
    error: readonly(error),
    filters,
    sortBy: readonly(sortBy),
    sortOrder: readonly(sortOrder),
    pagination,

    // Computed
    filteredItems,
    sortedItems,
    paginatedItems,
    totalPages,
    categories,

    // Actions
    fetchProducts,
    setFilter,
    clearFilters,
    setSort,
    setPage,
    getById,
  }
})
```

---

## Store Composition

### Using Stores Together

```typescript
// stores/checkout.ts
import { defineStore } from 'pinia'

export const useCheckoutStore = defineStore('checkout', () => {
  // Use other stores
  const authStore = useAuthStore()
  const cartStore = useCartStore()
  const notificationStore = useNotificationStore()

  // State
  const step = ref<'cart' | 'shipping' | 'payment' | 'confirmation'>('cart')
  const shippingAddress = ref<ShippingAddress | null>(null)
  const paymentMethod = ref<PaymentMethod | null>(null)
  const processing = ref(false)

  // Computed
  const canProceed = computed(() => {
    switch (step.value) {
      case 'cart':
        return cartStore.itemCount > 0
      case 'shipping':
        return !!shippingAddress.value
      case 'payment':
        return !!paymentMethod.value
      default:
        return false
    }
  })

  // Actions
  async function processOrder() {
    if (!authStore.isAuthenticated) {
      throw new Error('Must be authenticated to checkout')
    }

    if (!shippingAddress.value || !paymentMethod.value) {
      throw new Error('Missing required information')
    }

    processing.value = true

    try {
      const order = await $fetch('/api/orders', {
        method: 'POST',
        body: {
          items: cartStore.items,
          shippingAddress: shippingAddress.value,
          paymentMethod: paymentMethod.value,
        },
      })

      cartStore.clearCart()
      notificationStore.success('Order placed successfully!')
      step.value = 'confirmation'

      return order
    } catch (error) {
      notificationStore.error('Failed to process order')
      throw error
    } finally {
      processing.value = false
    }
  }

  return {
    step,
    shippingAddress,
    paymentMethod,
    processing: readonly(processing),
    canProceed,
    processOrder,
  }
})
```

---

## SSR Considerations

### useState for Simple State

For simple SSR-safe state, use Nuxt's `useState`:

```typescript
// composables/useCount.ts
export function useCount() {
  // SSR-safe state
  const count = useState<number>('count', () => 0)

  function increment() {
    count.value++
  }

  return { count, increment }
}
```

### Hydration-Safe Stores

```typescript
// stores/ssr-safe.ts
import { defineStore, skipHydrate } from 'pinia'

export const useSSRStore = defineStore('ssr', () => {
  // State that should be hydrated from server
  const serverData = ref<string | null>(null)

  // State that should NOT be hydrated (client-only)
  const clientOnlyState = ref({
    mousePosition: { x: 0, y: 0 },
    windowSize: { width: 0, height: 0 },
  })

  return {
    serverData,
    // Skip hydration for client-only state
    clientOnlyState: skipHydrate(clientOnlyState),
  }
})
```

### Store Initialization

```typescript
// plugins/store-init.ts
export default defineNuxtPlugin(async () => {
  const authStore = useAuthStore()
  const appStore = useAppStore()

  // Initialize stores on app start
  if (import.meta.client) {
    await authStore.initialize()
  }

  // Watch for auth changes
  watch(
    () => authStore.isAuthenticated,
    (isAuth) => {
      if (!isAuth) {
        // Reset other stores on logout
        // ...
      }
    },
  )
})
```

---

## Store Persistence

### With Pinia Plugin

```typescript
// plugins/pinia-persist.ts
import { defineNuxtPlugin } from '#app'

export default defineNuxtPlugin(({ $pinia }) => {
  if (import.meta.server) return

  // Restore state from localStorage
  const stored = localStorage.getItem('pinia-state')
  if (stored) {
    try {
      $pinia.state.value = JSON.parse(stored)
    } catch {
      localStorage.removeItem('pinia-state')
    }
  }

  // Save state to localStorage on change
  watch(
    () => $pinia.state.value,
    (state) => {
      // Only persist specific stores
      const toPersist = {
        app: state.app,
        user: { preferences: state.user?.preferences },
      }
      localStorage.setItem('pinia-state', JSON.stringify(toPersist))
    },
    { deep: true },
  )
})
```

### Selective Persistence

```typescript
// stores/persisted.ts
import { defineStore } from 'pinia'

export const usePersistedStore = defineStore('persisted', () => {
  // Persisted state
  const settings = useLocalStorage('user-settings', {
    theme: 'system',
    language: 'en',
  })

  // Non-persisted state
  const tempData = ref<string | null>(null)

  return {
    settings,
    tempData,
  }
})
```

---

## Testing Stores

### Unit Testing

```typescript
// stores/__tests__/user.test.ts
import { setActivePinia, createPinia } from 'pinia'
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { useUserStore } from '../user'

describe('useUserStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('initializes with null user', () => {
    const store = useUserStore()
    expect(store.currentUser).toBeNull()
    expect(store.isAuthenticated).toBe(false)
  })

  it('sets user correctly', () => {
    const store = useUserStore()
    const user = { id: '1', name: 'Test', email: 'test@example.com', role: 'user' as const }

    store.setUser(user)

    expect(store.currentUser).toEqual(user)
    expect(store.isAuthenticated).toBe(true)
    expect(store.displayName).toBe('Test')
  })

  it('clears user on logout', () => {
    const store = useUserStore()
    store.setUser({ id: '1', name: 'Test', email: 'test@example.com', role: 'user' })

    store.clearUser()

    expect(store.currentUser).toBeNull()
    expect(store.isAuthenticated).toBe(false)
  })

  it('fetches user from API', async () => {
    const mockUser = { id: '1', name: 'API User', email: 'api@example.com', role: 'user' }
    
    vi.stubGlobal('$fetch', vi.fn().mockResolvedValue(mockUser))

    const store = useUserStore()
    await store.fetchUser()

    expect(store.currentUser).toEqual(mockUser)
  })
})
```

### Integration Testing

```typescript
// stores/__tests__/integration.test.ts
import { setActivePinia, createPinia } from 'pinia'
import { describe, it, expect, beforeEach } from 'vitest'
import { useAuthStore } from '../auth'
import { useUserStore } from '../user'

describe('Store Integration', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('clears user store on auth logout', async () => {
    const authStore = useAuthStore()
    const userStore = useUserStore()

    // Setup authenticated state
    userStore.setUser({ id: '1', name: 'Test', email: 'test@example.com', role: 'user' })

    // Logout
    await authStore.logout()

    // Verify user is cleared
    expect(userStore.currentUser).toBeNull()
  })
})
```

---

## Best Practices

### 1. Store Organization

```typescript
// ✅ Good: Focused, single-responsibility stores
const useAuthStore = defineStore('auth', () => { /* auth logic */ })
const useUserStore = defineStore('user', () => { /* user logic */ })

// ❌ Bad: Monolithic store with everything
const useAppStore = defineStore('app', () => { 
  /* auth + user + settings + notifications + ... */ 
})
```

### 2. Readonly State

```typescript
// ✅ Good: Expose readonly refs
return {
  users: readonly(users),
  loading: readonly(loading),
  fetchUsers,
}

// ❌ Bad: Allow direct mutations
return {
  users,
  loading,
}
```

### 3. Async Actions

```typescript
// ✅ Good: Proper loading and error handling
async function fetchData() {
  loading.value = true
  error.value = null
  
  try {
    data.value = await $fetch('/api/data')
  } catch (err) {
    error.value = err as Error
    throw err
  } finally {
    loading.value = false
  }
}

// ❌ Bad: No error handling
async function fetchData() {
  data.value = await $fetch('/api/data')
}
```

### 4. Store Composition Rules

```typescript
// ✅ Good: Use other stores inside actions/getters
const useCheckoutStore = defineStore('checkout', () => {
  const cartStore = useCartStore()
  
  async function processOrder() {
    const items = cartStore.items
    // ...
  }
})

// ❌ Bad: Circular dependencies
// Store A imports Store B, Store B imports Store A
```

### 5. SSR Safety

```typescript
// ✅ Good: Check for client
if (import.meta.client) {
  localStorage.setItem('key', value)
}

// ❌ Bad: Direct browser API access
localStorage.setItem('key', value) // Fails on server
```

---

## Next Steps

Continue to [Routing & Navigation](./07-routing-and-navigation.md) to learn about Nuxt's file-based routing, middleware, and navigation patterns.