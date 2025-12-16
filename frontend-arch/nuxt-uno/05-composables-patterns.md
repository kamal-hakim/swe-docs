# Composables Patterns

## Overview

Composables are the primary mechanism for logic reuse in Vue 3 applications. They encapsulate reactive state, computed properties, watchers, and lifecycle hooks into reusable functions. This document covers patterns, best practices, and guidelines for creating effective composables.

## What Are Composables?

Composables are functions that:
- Leverage Vue's Composition API
- Encapsulate and reuse stateful logic
- Return reactive state and methods
- Can be composed together

```typescript
// Basic composable structure
export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  return {
    count: readonly(count),
    increment,
    decrement,
  }
}
```

---

## Composables Organization

### Directory Structure

```
app/composables/
├── api/                        # API integration composables
│   ├── useApi.ts              # Base API client
│   ├── useFetch.ts            # Data fetching wrapper
│   └── useInfiniteScroll.ts   # Infinite pagination
├── auth/                       # Authentication composables
│   ├── useAuth.ts             # Authentication state
│   ├── useSession.ts          # Session management
│   └── usePermissions.ts      # Permission checking
├── ui/                         # UI-related composables
│   ├── useModal.ts            # Modal state
│   ├── useToast.ts            # Toast notifications
│   ├── useDialog.ts           # Dialog confirmations
│   └── useDropdown.ts         # Dropdown state
├── utils/                      # Utility composables
│   ├── useDebounce.ts         # Debounce functionality
│   ├── useThrottle.ts         # Throttle functionality
│   ├── useLocalStorage.ts     # Local storage wrapper
│   ├── useClipboard.ts        # Clipboard operations
│   └── useMediaQuery.ts       # Responsive breakpoints
├── form/                       # Form handling composables
│   ├── useForm.ts             # Form state management
│   ├── useValidation.ts       # Form validation
│   └── useFormField.ts        # Individual field logic
└── [domain]/                   # Domain-specific composables
    ├── useUser.ts             # User operations
    ├── useProduct.ts          # Product operations
    └── useOrder.ts            # Order operations
```

---

## Core Patterns

### 1. Return Object Pattern

Always return an object with named properties for maximum flexibility:

```typescript
// composables/useCounter.ts
export function useCounter(initialValue = 0) {
  // State
  const count = ref(initialValue)
  const history = ref<number[]>([initialValue])
  
  // Computed
  const isPositive = computed(() => count.value > 0)
  const isNegative = computed(() => count.value < 0)
  const isZero = computed(() => count.value === 0)
  
  // Methods
  function increment(amount = 1) {
    count.value += amount
    history.value.push(count.value)
  }
  
  function decrement(amount = 1) {
    count.value -= amount
    history.value.push(count.value)
  }
  
  function reset() {
    count.value = initialValue
    history.value = [initialValue]
  }
  
  // Return object with readonly state
  return {
    // State (readonly to prevent external mutations)
    count: readonly(count),
    history: readonly(history),
    
    // Computed
    isPositive,
    isNegative,
    isZero,
    
    // Methods
    increment,
    decrement,
    reset,
  }
}
```

### 2. Options Pattern

For composables with complex configuration:

```typescript
// composables/usePagination.ts
interface UsePaginationOptions {
  initialPage?: number
  pageSize?: number
  total?: Ref<number> | number
  onPageChange?: (page: number) => void | Promise<void>
}

export function usePagination(options: UsePaginationOptions = {}) {
  const {
    initialPage = 1,
    pageSize = 10,
    total = 0,
    onPageChange,
  } = options
  
  // State
  const currentPage = ref(initialPage)
  const itemsPerPage = ref(pageSize)
  
  // Computed
  const totalRef = isRef(total) ? total : ref(total)
  
  const totalPages = computed(() => 
    Math.ceil(totalRef.value / itemsPerPage.value)
  )
  
  const offset = computed(() => 
    (currentPage.value - 1) * itemsPerPage.value
  )
  
  const hasNextPage = computed(() => 
    currentPage.value < totalPages.value
  )
  
  const hasPrevPage = computed(() => 
    currentPage.value > 1
  )
  
  // Methods
  async function goToPage(page: number) {
    if (page < 1 || page > totalPages.value) return
    currentPage.value = page
    await onPageChange?.(page)
  }
  
  async function nextPage() {
    if (hasNextPage.value) {
      await goToPage(currentPage.value + 1)
    }
  }
  
  async function prevPage() {
    if (hasPrevPage.value) {
      await goToPage(currentPage.value - 1)
    }
  }
  
  function setPageSize(size: number) {
    itemsPerPage.value = size
    currentPage.value = 1
  }
  
  return {
    currentPage: readonly(currentPage),
    itemsPerPage: readonly(itemsPerPage),
    totalPages,
    offset,
    hasNextPage,
    hasPrevPage,
    goToPage,
    nextPage,
    prevPage,
    setPageSize,
  }
}
```

### 3. Factory Pattern

Create composable instances with shared configuration:

```typescript
// composables/createApi.ts
interface ApiConfig {
  baseURL: string
  headers?: Record<string, string>
  timeout?: number
}

export function createApi(config: ApiConfig) {
  const { baseURL, headers = {}, timeout = 30000 } = config
  
  return function useApi<T>() {
    const data = ref<T | null>(null)
    const error = ref<Error | null>(null)
    const loading = ref(false)
    
    async function get(endpoint: string): Promise<T> {
      loading.value = true
      error.value = null
      
      try {
        const response = await fetch(`${baseURL}${endpoint}`, {
          headers,
          signal: AbortSignal.timeout(timeout),
        })
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`)
        }
        
        const result = await response.json()
        data.value = result
        return result
      } catch (err) {
        error.value = err as Error
        throw err
      } finally {
        loading.value = false
      }
    }
    
    async function post<R = T>(endpoint: string, body: unknown): Promise<R> {
      loading.value = true
      error.value = null
      
      try {
        const response = await fetch(`${baseURL}${endpoint}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json', ...headers },
          body: JSON.stringify(body),
          signal: AbortSignal.timeout(timeout),
        })
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`)
        }
        
        return await response.json()
      } catch (err) {
        error.value = err as Error
        throw err
      } finally {
        loading.value = false
      }
    }
    
    return {
      data: readonly(data),
      error: readonly(error),
      loading: readonly(loading),
      get,
      post,
    }
  }
}

// Usage
const useApi = createApi({ baseURL: '/api' })

// In component
const { data, loading, get } = useApi<User[]>()
await get('/users')
```

---

## Common Composables

### useAsyncData

```typescript
// composables/useAsyncData.ts
interface UseAsyncDataOptions<T> {
  immediate?: boolean
  default?: T
  onSuccess?: (data: T) => void
  onError?: (error: Error) => void
}

export function useAsyncData<T>(
  fetcher: () => Promise<T>,
  options: UseAsyncDataOptions<T> = {},
) {
  const {
    immediate = true,
    default: defaultValue,
    onSuccess,
    onError,
  } = options
  
  const data = ref<T | undefined>(defaultValue) as Ref<T | undefined>
  const error = ref<Error | null>(null)
  const loading = ref(false)
  const loaded = ref(false)
  
  async function execute(): Promise<T | undefined> {
    loading.value = true
    error.value = null
    
    try {
      const result = await fetcher()
      data.value = result
      loaded.value = true
      onSuccess?.(result)
      return result
    } catch (err) {
      error.value = err as Error
      onError?.(err as Error)
    } finally {
      loading.value = false
    }
  }
  
  function reset() {
    data.value = defaultValue
    error.value = null
    loading.value = false
    loaded.value = false
  }
  
  // Execute immediately if option is set
  if (immediate) {
    execute()
  }
  
  return {
    data,
    error: readonly(error),
    loading: readonly(loading),
    loaded: readonly(loaded),
    execute,
    refresh: execute,
    reset,
  }
}
```

### useDebounce

```typescript
// composables/useDebounce.ts
export function useDebounce<T>(value: Ref<T>, delay = 300): Ref<T> {
  const debouncedValue = ref(value.value) as Ref<T>
  let timeout: ReturnType<typeof setTimeout>
  
  watch(value, (newValue) => {
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      debouncedValue.value = newValue
    }, delay)
  })
  
  onUnmounted(() => {
    clearTimeout(timeout)
  })
  
  return readonly(debouncedValue)
}

// Function debounce version
export function useDebounceFn<T extends (...args: any[]) => any>(
  fn: T,
  delay = 300,
): (...args: Parameters<T>) => void {
  let timeout: ReturnType<typeof setTimeout>
  
  const debouncedFn = (...args: Parameters<T>) => {
    clearTimeout(timeout)
    timeout = setTimeout(() => fn(...args), delay)
  }
  
  onUnmounted(() => {
    clearTimeout(timeout)
  })
  
  return debouncedFn
}
```

### useLocalStorage

```typescript
// composables/useLocalStorage.ts
interface UseLocalStorageOptions<T> {
  serializer?: {
    read: (value: string) => T
    write: (value: T) => string
  }
  onError?: (error: Error) => void
}

export function useLocalStorage<T>(
  key: string,
  defaultValue: T,
  options: UseLocalStorageOptions<T> = {},
): Ref<T> {
  const {
    serializer = {
      read: JSON.parse,
      write: JSON.stringify,
    },
    onError = console.error,
  } = options
  
  // Read initial value
  function read(): T {
    if (typeof window === 'undefined') {
      return defaultValue
    }
    
    try {
      const stored = localStorage.getItem(key)
      return stored ? serializer.read(stored) : defaultValue
    } catch (error) {
      onError(error as Error)
      return defaultValue
    }
  }
  
  // Create reactive ref
  const data = ref(read()) as Ref<T>
  
  // Write to storage on change
  watch(
    data,
    (value) => {
      if (typeof window === 'undefined') return
      
      try {
        if (value === null || value === undefined) {
          localStorage.removeItem(key)
        } else {
          localStorage.setItem(key, serializer.write(value))
        }
      } catch (error) {
        onError(error as Error)
      }
    },
    { deep: true },
  )
  
  // Listen for storage events (cross-tab sync)
  if (typeof window !== 'undefined') {
    useEventListener(window, 'storage', (event) => {
      if (event.key === key) {
        data.value = event.newValue ? serializer.read(event.newValue) : defaultValue
      }
    })
  }
  
  return data
}
```

### useEventListener

```typescript
// composables/useEventListener.ts
type EventTarget = Window | Document | HTMLElement | null

export function useEventListener<K extends keyof WindowEventMap>(
  target: Window,
  event: K,
  handler: (event: WindowEventMap[K]) => void,
  options?: AddEventListenerOptions,
): void

export function useEventListener<K extends keyof DocumentEventMap>(
  target: Document,
  event: K,
  handler: (event: DocumentEventMap[K]) => void,
  options?: AddEventListenerOptions,
): void

export function useEventListener<K extends keyof HTMLElementEventMap>(
  target: Ref<HTMLElement | null> | HTMLElement,
  event: K,
  handler: (event: HTMLElementEventMap[K]) => void,
  options?: AddEventListenerOptions,
): void

export function useEventListener(
  target: EventTarget | Ref<EventTarget>,
  event: string,
  handler: EventListener,
  options?: AddEventListenerOptions,
): void {
  const targetRef = isRef(target) ? target : ref(target)
  
  function cleanup() {
    targetRef.value?.removeEventListener(event, handler, options)
  }
  
  watch(
    targetRef,
    (el, _, onCleanup) => {
      el?.addEventListener(event, handler, options)
      onCleanup(cleanup)
    },
    { immediate: true },
  )
  
  onUnmounted(cleanup)
}
```

### useMediaQuery

```typescript
// composables/useMediaQuery.ts
export function useMediaQuery(query: string): Ref<boolean> {
  const matches = ref(false)
  
  if (typeof window === 'undefined') {
    return readonly(matches)
  }
  
  const mediaQuery = window.matchMedia(query)
  matches.value = mediaQuery.matches
  
  function handler(event: MediaQueryListEvent) {
    matches.value = event.matches
  }
  
  mediaQuery.addEventListener('change', handler)
  
  onUnmounted(() => {
    mediaQuery.removeEventListener('change', handler)
  })
  
  return readonly(matches)
}

// Breakpoint composable
export function useBreakpoints() {
  const isMobile = useMediaQuery('(max-width: 639px)')
  const isTablet = useMediaQuery('(min-width: 640px) and (max-width: 1023px)')
  const isDesktop = useMediaQuery('(min-width: 1024px)')
  const isLargeDesktop = useMediaQuery('(min-width: 1280px)')
  
  const current = computed(() => {
    if (isMobile.value) return 'mobile'
    if (isTablet.value) return 'tablet'
    if (isLargeDesktop.value) return 'large-desktop'
    return 'desktop'
  })
  
  return {
    isMobile,
    isTablet,
    isDesktop,
    isLargeDesktop,
    current,
  }
}
```

---

## API Integration Composables

### useApi

```typescript
// composables/api/useApi.ts
interface ApiOptions {
  baseURL?: string
  headers?: Record<string, string>
}

interface RequestOptions extends RequestInit {
  params?: Record<string, string>
}

export function useApi(options: ApiOptions = {}) {
  const config = useRuntimeConfig()
  const baseURL = options.baseURL || config.public.apiBaseUrl
  
  const loading = ref(false)
  const error = ref<Error | null>(null)
  
  async function request<T>(
    endpoint: string,
    requestOptions: RequestOptions = {},
  ): Promise<T> {
    const { params, ...fetchOptions } = requestOptions
    
    // Build URL with query params
    let url = `${baseURL}${endpoint}`
    if (params) {
      const searchParams = new URLSearchParams(params)
      url += `?${searchParams.toString()}`
    }
    
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch(url, {
        ...fetchOptions,
        headers: {
          'Content-Type': 'application/json',
          ...options.headers,
          ...fetchOptions.headers,
        },
      })
      
      if (!response.ok) {
        throw new ApiError(response.status, await response.text())
      }
      
      return await response.json()
    } catch (err) {
      error.value = err as Error
      throw err
    } finally {
      loading.value = false
    }
  }
  
  return {
    loading: readonly(loading),
    error: readonly(error),
    
    get: <T>(endpoint: string, params?: Record<string, string>) =>
      request<T>(endpoint, { method: 'GET', params }),
    
    post: <T>(endpoint: string, body: unknown) =>
      request<T>(endpoint, { method: 'POST', body: JSON.stringify(body) }),
    
    put: <T>(endpoint: string, body: unknown) =>
      request<T>(endpoint, { method: 'PUT', body: JSON.stringify(body) }),
    
    patch: <T>(endpoint: string, body: unknown) =>
      request<T>(endpoint, { method: 'PATCH', body: JSON.stringify(body) }),
    
    delete: <T>(endpoint: string) =>
      request<T>(endpoint, { method: 'DELETE' }),
  }
}

// Custom error class
class ApiError extends Error {
  constructor(
    public status: number,
    public message: string,
  ) {
    super(message)
    this.name = 'ApiError'
  }
}
```

### Domain-Specific API Composable

```typescript
// composables/api/useUsers.ts
interface User {
  id: string
  name: string
  email: string
  role: 'admin' | 'user'
  createdAt: string
}

interface CreateUserInput {
  name: string
  email: string
  role?: 'admin' | 'user'
}

export function useUsers() {
  const api = useApi()
  
  // State
  const users = ref<User[]>([])
  const currentUser = ref<User | null>(null)
  
  // Computed
  const admins = computed(() => users.value.filter(u => u.role === 'admin'))
  const regularUsers = computed(() => users.value.filter(u => u.role === 'user'))
  
  // Methods
  async function fetchUsers(params?: { role?: string; search?: string }) {
    const result = await api.get<User[]>('/users', params)
    users.value = result
    return result
  }
  
  async function fetchUser(id: string) {
    const result = await api.get<User>(`/users/${id}`)
    currentUser.value = result
    return result
  }
  
  async function createUser(input: CreateUserInput) {
    const result = await api.post<User>('/users', input)
    users.value.push(result)
    return result
  }
  
  async function updateUser(id: string, input: Partial<CreateUserInput>) {
    const result = await api.patch<User>(`/users/${id}`, input)
    const index = users.value.findIndex(u => u.id === id)
    if (index !== -1) {
      users.value[index] = result
    }
    if (currentUser.value?.id === id) {
      currentUser.value = result
    }
    return result
  }
  
  async function deleteUser(id: string) {
    await api.delete(`/users/${id}`)
    users.value = users.value.filter(u => u.id !== id)
    if (currentUser.value?.id === id) {
      currentUser.value = null
    }
  }
  
  return {
    // State
    users: readonly(users),
    currentUser: readonly(currentUser),
    loading: api.loading,
    error: api.error,
    
    // Computed
    admins,
    regularUsers,
    
    // Methods
    fetchUsers,
    fetchUser,
    createUser,
    updateUser,
    deleteUser,
  }
}
```

---

## UI Composables

### useModal

```typescript
// composables/ui/useModal.ts
interface ModalState {
  isOpen: boolean
  data: any
}

const modals = reactive<Map<string, ModalState>>(new Map())

export function useModal(id: string) {
  // Initialize modal state
  if (!modals.has(id)) {
    modals.set(id, { isOpen: false, data: null })
  }
  
  const state = computed(() => modals.get(id)!)
  const isOpen = computed(() => state.value.isOpen)
  const data = computed(() => state.value.data)
  
  function open(modalData?: any) {
    modals.set(id, { isOpen: true, data: modalData ?? null })
  }
  
  function close() {
    modals.set(id, { isOpen: false, data: null })
  }
  
  function toggle() {
    if (isOpen.value) {
      close()
    } else {
      open()
    }
  }
  
  return {
    isOpen,
    data,
    open,
    close,
    toggle,
  }
}
```

### useToast

```typescript
// composables/ui/useToast.ts
interface Toast {
  id: string
  message: string
  type: 'success' | 'error' | 'warning' | 'info'
  duration: number
}

const toasts = ref<Toast[]>([])

export function useToast() {
  function show(
    message: string,
    type: Toast['type'] = 'info',
    duration = 5000,
  ) {
    const id = crypto.randomUUID()
    
    toasts.value.push({ id, message, type, duration })
    
    if (duration > 0) {
      setTimeout(() => {
        dismiss(id)
      }, duration)
    }
    
    return id
  }
  
  function dismiss(id: string) {
    const index = toasts.value.findIndex(t => t.id === id)
    if (index !== -1) {
      toasts.value.splice(index, 1)
    }
  }
  
  function dismissAll() {
    toasts.value = []
  }
  
  return {
    toasts: readonly(toasts),
    show,
    success: (message: string, duration?: number) => show(message, 'success', duration),
    error: (message: string, duration?: number) => show(message, 'error', duration),
    warning: (message: string, duration?: number) => show(message, 'warning', duration),
    info: (message: string, duration?: number) => show(message, 'info', duration),
    dismiss,
    dismissAll,
  }
}
```

### useConfirm

```typescript
// composables/ui/useConfirm.ts
interface ConfirmOptions {
  title?: string
  message: string
  confirmText?: string
  cancelText?: string
  type?: 'danger' | 'warning' | 'info'
}

interface ConfirmState {
  isOpen: boolean
  options: ConfirmOptions | null
  resolve: ((value: boolean) => void) | null
}

const state = reactive<ConfirmState>({
  isOpen: false,
  options: null,
  resolve: null,
})

export function useConfirm() {
  function confirm(options: ConfirmOptions): Promise<boolean> {
    return new Promise((resolve) => {
      state.isOpen = true
      state.options = {
        title: 'Confirm',
        confirmText: 'Confirm',
        cancelText: 'Cancel',
        type: 'info',
        ...options,
      }
      state.resolve = resolve
    })
  }
  
  function handleConfirm() {
    state.resolve?.(true)
    reset()
  }
  
  function handleCancel() {
    state.resolve?.(false)
    reset()
  }
  
  function reset() {
    state.isOpen = false
    state.options = null
    state.resolve = null
  }
  
  return {
    isOpen: computed(() => state.isOpen),
    options: computed(() => state.options),
    confirm,
    handleConfirm,
    handleCancel,
  }
}
```

---

## Form Composables

### useForm

```typescript
// composables/form/useForm.ts
interface UseFormOptions<T> {
  initialValues: T
  validate?: (values: T) => Record<keyof T, string | undefined> | Promise<Record<keyof T, string | undefined>>
  onSubmit: (values: T) => void | Promise<void>
}

export function useForm<T extends Record<string, any>>(options: UseFormOptions<T>) {
  const { initialValues, validate, onSubmit } = options
  
  // State
  const values = reactive({ ...initialValues }) as T
  const errors = reactive<Record<keyof T, string | undefined>>(
    {} as Record<keyof T, string | undefined>
  )
  const touched = reactive<Record<keyof T, boolean>>(
    {} as Record<keyof T, boolean>
  )
  const isSubmitting = ref(false)
  const isValidating = ref(false)
  
  // Computed
  const isDirty = computed(() => {
    return Object.keys(initialValues).some(
      key => values[key as keyof T] !== initialValues[key as keyof T]
    )
  })
  
  const isValid = computed(() => {
    return Object.values(errors).every(error => !error)
  })
  
  const canSubmit = computed(() => {
    return isValid.value && isDirty.value && !isSubmitting.value
  })
  
  // Methods
  function setFieldValue<K extends keyof T>(field: K, value: T[K]) {
    values[field] = value
  }
  
  function setFieldError<K extends keyof T>(field: K, error: string | undefined) {
    errors[field] = error
  }
  
  function setFieldTouched<K extends keyof T>(field: K, isTouched = true) {
    touched[field] = isTouched
  }
  
  async function validateForm() {
    if (!validate) return true
    
    isValidating.value = true
    try {
      const validationErrors = await validate(values)
      Object.assign(errors, validationErrors)
      return Object.values(validationErrors).every(error => !error)
    } finally {
      isValidating.value = false
    }
  }
  
  async function handleSubmit() {
    // Touch all fields
    Object.keys(values).forEach(key => {
      touched[key as keyof T] = true
    })
    
    // Validate
    const isFormValid = await validateForm()
    if (!isFormValid) return
    
    // Submit
    isSubmitting.value = true
    try {
      await onSubmit(values)
    } finally {
      isSubmitting.value = false
    }
  }
  
  function reset() {
    Object.assign(values, initialValues)
    Object.keys(errors).forEach(key => {
      errors[key as keyof T] = undefined
    })
    Object.keys(touched).forEach(key => {
      touched[key as keyof T] = false
    })
  }
  
  return {
    values,
    errors: readonly(errors) as Record<keyof T, string | undefined>,
    touched: readonly(touched) as Record<keyof T, boolean>,
    isSubmitting: readonly(isSubmitting),
    isValidating: readonly(isValidating),
    isDirty,
    isValid,
    canSubmit,
    setFieldValue,
    setFieldError,
    setFieldTouched,
    validateForm,
    handleSubmit,
    reset,
  }
}
```

---

## Nuxt-Specific Patterns

### Server-Safe Composables

```typescript
// composables/useClientOnly.ts
export function useClientOnly<T>(fn: () => T, fallback?: T): T | undefined {
  if (import.meta.server) {
    return fallback
  }
  return fn()
}

// Usage
const windowWidth = useClientOnly(
  () => window.innerWidth,
  0
)
```

### useAsyncData Integration

```typescript
// composables/api/useResource.ts
export function useResource<T>(
  key: string,
  fetcher: () => Promise<T>,
) {
  const nuxtApp = useNuxtApp()
  
  // Use Nuxt's built-in data fetching
  const { data, pending, error, refresh } = useAsyncData(key, fetcher)
  
  // Additional reactive state
  const isStale = ref(false)
  
  // Mark as stale after certain time
  let staleTimeout: ReturnType<typeof setTimeout>
  
  watch(data, () => {
    isStale.value = false
    clearTimeout(staleTimeout)
    staleTimeout = setTimeout(() => {
      isStale.value = true
    }, 5 * 60 * 1000) // 5 minutes
  })
  
  // Cleanup
  onUnmounted(() => {
    clearTimeout(staleTimeout)
  })
  
  return {
    data,
    loading: pending,
    error,
    isStale: readonly(isStale),
    refresh,
    invalidate: () => nuxtApp.runWithContext(() => refreshNuxtData(key)),
  }
}
```

---

## Testing Composables

### Unit Testing

```typescript
// composables/__tests__/useCounter.test.ts
import { describe, it, expect } from 'vitest'
import { useCounter } from '../useCounter'

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { count } = useCounter()
    expect(count.value).toBe(0)
  })

  it('initializes with provided value', () => {
    const { count } = useCounter(10)
    expect(count.value).toBe(10)
  })

  it('increments count', () => {
    const { count, increment } = useCounter()
    increment()
    expect(count.value).toBe(1)
  })

  it('decrements count', () => {
    const { count, decrement } = useCounter(5)
    decrement()
    expect(count.value).toBe(4)
  })

  it('resets to initial value', () => {
    const { count, increment, reset } = useCounter(5)
    increment()
    increment()
    expect(count.value).toBe(7)
    reset()
    expect(count.value).toBe(5)
  })
})
```

### Testing with Vue Test Utils

```typescript
// composables/__tests__/useForm.test.ts
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import { defineComponent } from 'vue'
import { useForm } from '../useForm'

describe('useForm', () => {
  const TestComponent = defineComponent({
    setup() {
      return useForm({
        initialValues: { email: '', password: '' },
        validate: (values) => ({
          email: !values.email ? 'Email required' : undefined,
          password: values.password.length < 8 ? 'Min 8 chars' : undefined,
        }),
        onSubmit: vi.fn(),
      })
    },
    template: '<div></div>',
  })

  it('validates form fields', async () => {
    const wrapper = mount(TestComponent)
    const vm = wrapper.vm as any

    await vm.validateForm()

    expect(vm.errors.email).toBe('Email required')
    expect(vm.errors.password).toBe('Min 8 chars')
  })

  it('tracks dirty state', () => {
    const wrapper = mount(TestComponent)
    const vm = wrapper.vm as any

    expect(vm.isDirty).toBe(false)

    vm.values.email = 'test@example.com'

    expect(vm.isDirty).toBe(true)
  })
})
```

---

## Best Practices

### 1. Always Clean Up

```typescript
export function useInterval(callback: () => void, delay: number) {
  const intervalId = ref<ReturnType<typeof setInterval>>()
  
  function start() {
    stop()
    intervalId.value = setInterval(callback, delay)
  }
  
  function stop() {
    if (intervalId.value) {
      clearInterval(intervalId.value)
      intervalId.value = undefined
    }
  }
  
  // Always clean up on unmount
  onUnmounted(stop)
  
  return { start, stop }
}
```

### 2. Return Readonly State

```typescript
export function useData() {
  const data = ref<Data | null>(null)
  
  // ✅ Good: Return readonly to prevent external mutations
  return {
    data: readonly(data),
    setData: (newData: Data) => { data.value = newData },
  }
  
  // ❌ Bad: Exposing mutable ref directly
  // return { data }
}
```

### 3. Handle SSR Properly

```typescript
export function useWindowSize() {
  // Default values for SSR
  const width = ref(0)
  const height = ref(0)
  
  // Only run on client
  if (import.meta.client) {
    width.value = window.innerWidth
    height.value = window.innerHeight
    
    useEventListener(window, 'resize', () => {
      width.value = window.innerWidth
      height.value = window.innerHeight
    })
  }
  
  return { width, height }
}
```

### 4. Type Everything

```typescript
// ✅ Good: Full type coverage
interface UseCounterReturn {
  count: Readonly<Ref<number>>
  increment: () => void
  decrement: () => void
  reset: () => void
}

export function useCounter(initial = 0): UseCounterReturn {
  // ...implementation
}
```

---

## Next Steps

Continue to [State Management](./06-state-management.md) to learn about Pinia stores and global state management patterns.