# Testing Strategy

## Overview

This document covers the comprehensive testing strategy for Nuxt applications using **Vitest** as the primary testing framework, combined with **@nuxt/test-utils** for Nuxt-specific testing utilities and **@vue/test-utils** for component testing.

## Testing Stack

| Tool | Purpose |
|------|---------|
| **Vitest** | Test runner and assertion library |
| **@nuxt/test-utils** | Nuxt environment and utilities |
| **@vue/test-utils** | Vue component mounting and testing |
| **happy-dom** | Lightweight DOM implementation |
| **MSW** | API mocking (optional) |

---

## Setup and Configuration

### Installation

```bash
# Core testing dependencies
pnpm add -D vitest @vue/test-utils @nuxt/test-utils happy-dom

# Optional: API mocking
pnpm add -D msw
```

### Vitest Configuration

```typescript
// vitest.config.ts
import { defineVitestProject } from '@nuxt/test-utils/config'
import { defineConfig } from 'vitest/config'

const isCI = process.env.CI === 'true'

export default defineConfig({
  test: {
    // Global settings
    globals: true,
    environment: 'happy-dom',
    
    // Reporter configuration
    reporters: isCI ? ['default', 'junit', 'hanging-process'] : ['default'],
    outputFile: {
      junit: './test-results/junit.xml',
    },
    
    // Coverage configuration
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      reportsDirectory: './coverage',
      exclude: [
        'node_modules/',
        '.nuxt/',
        '.output/',
        'tests/',
        '**/*.d.ts',
        '**/*.config.*',
      ],
      thresholds: {
        lines: 80,
        branches: 80,
        functions: 80,
        statements: 80,
      },
    },
    
    // Test file patterns
    include: ['tests/**/*.test.ts', 'tests/**/*.spec.ts'],
    exclude: ['node_modules', '.nuxt', '.output'],
    
    // Setup files
    setupFiles: ['./tests/setup.ts'],
    
    // Timeout
    testTimeout: 10000,
    hookTimeout: 10000,
    
    // Projects for different test types
    projects: [
      // Unit tests
      {
        extends: true,
        test: {
          name: 'unit',
          include: ['tests/unit/**/*.test.ts'],
        },
      },
      // Integration tests with Nuxt
      await defineVitestProject({
        test: {
          name: 'nuxt',
          include: ['tests/integration/**/*.test.ts'],
          setupFiles: ['./tests/setup.ts'],
          environmentOptions: {
            nuxt: {
              mock: {
                indexedDb: true,
                intersectionObserver: true,
              },
            },
          },
        },
      }),
      // Component tests
      {
        extends: true,
        test: {
          name: 'components',
          include: ['tests/components/**/*.test.ts'],
          environment: 'happy-dom',
        },
      },
    ],
  },
})
```

### Test Setup File

```typescript
// tests/setup.ts
import { vi } from 'vitest'

// ==========================================
// Global Mocks
// ==========================================

// Mock browser APIs not available in happy-dom
if (typeof window !== 'undefined') {
  // AbortSignal.timeout
  if (!AbortSignal.timeout) {
    AbortSignal.timeout = (ms: number) => {
      const controller = new AbortController()
      setTimeout(() => controller.abort(new DOMException('TimeoutError')), ms)
      return controller.signal
    }
  }

  // matchMedia
  Object.defineProperty(window, 'matchMedia', {
    writable: true,
    value: vi.fn().mockImplementation((query: string) => ({
      matches: false,
      media: query,
      onchange: null,
      addListener: vi.fn(),
      removeListener: vi.fn(),
      addEventListener: vi.fn(),
      removeEventListener: vi.fn(),
      dispatchEvent: vi.fn(),
    })),
  })

  // IntersectionObserver
  class MockIntersectionObserver {
    observe = vi.fn()
    disconnect = vi.fn()
    unobserve = vi.fn()
  }
  window.IntersectionObserver = MockIntersectionObserver as any

  // ResizeObserver
  class MockResizeObserver {
    observe = vi.fn()
    disconnect = vi.fn()
    unobserve = vi.fn()
  }
  window.ResizeObserver = MockResizeObserver as any
}

// ==========================================
// Vue Mocks
// ==========================================

// Mock vue-router
vi.mock('vue-router', async () => {
  const actual = await vi.importActual<typeof import('vue-router')>('vue-router')
  return {
    ...actual,
    useRouter: vi.fn(() => ({
      push: vi.fn(),
      replace: vi.fn(),
      go: vi.fn(),
      back: vi.fn(),
      forward: vi.fn(),
    })),
    useRoute: vi.fn(() => ({
      params: {},
      query: {},
      path: '/',
      name: 'index',
      meta: {},
    })),
  }
})

// ==========================================
// Global Test Utilities
// ==========================================

// Wait for next tick
export const flushPromises = () => new Promise((resolve) => setTimeout(resolve, 0))

// Wait for specific time
export const wait = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms))

// ==========================================
// Cleanup
// ==========================================

afterEach(() => {
  vi.clearAllMocks()
})
```

### Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui",
    "test:unit": "vitest run --project unit",
    "test:integration": "vitest run --project nuxt",
    "test:components": "vitest run --project components"
  }
}
```

---

## Test Directory Structure

```
tests/
├── setup.ts                    # Global test setup
├── utils/                      # Test utilities
│   ├── index.ts               # Utility exports
│   ├── mocks.ts               # Mock factories
│   ├── fixtures.ts            # Test fixtures
│   └── helpers.ts             # Helper functions
├── unit/                       # Unit tests
│   ├── utils/                 # Utility function tests
│   │   ├── format.test.ts
│   │   └── validation.test.ts
│   └── composables/           # Composable tests
│       ├── useCounter.test.ts
│       └── useForm.test.ts
├── integration/                # Integration tests
│   ├── api/                   # API integration tests
│   │   └── users.test.ts
│   ├── stores/                # Store tests
│   │   └── auth.test.ts
│   └── pages/                 # Page tests
│       └── home.test.ts
└── components/                 # Component tests
    ├── common/
    │   ├── AppButton.test.ts
    │   └── AppModal.test.ts
    └── user/
        └── UserCard.test.ts
```

---

## Unit Testing

### Testing Utility Functions

```typescript
// tests/unit/utils/format.test.ts
import { describe, it, expect } from 'vitest'
import { formatDate, formatCurrency, truncate } from '~/utils/format'

describe('formatDate', () => {
  it('formats date with default options', () => {
    const date = new Date('2024-01-15T10:30:00Z')
    expect(formatDate(date)).toBe('January 15, 2024')
  })

  it('formats date with custom format', () => {
    const date = new Date('2024-01-15T10:30:00Z')
    expect(formatDate(date, { format: 'short' })).toBe('1/15/24')
  })

  it('handles invalid date', () => {
    expect(formatDate(new Date('invalid'))).toBe('Invalid date')
  })

  it('handles null/undefined', () => {
    expect(formatDate(null as any)).toBe('-')
    expect(formatDate(undefined as any)).toBe('-')
  })
})

describe('formatCurrency', () => {
  it('formats currency with default locale', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56')
  })

  it('formats currency with custom locale', () => {
    expect(formatCurrency(1234.56, { locale: 'de-DE', currency: 'EUR' }))
      .toBe('1.234,56 €')
  })

  it('handles negative values', () => {
    expect(formatCurrency(-100)).toBe('-$100.00')
  })

  it('handles zero', () => {
    expect(formatCurrency(0)).toBe('$0.00')
  })
})

describe('truncate', () => {
  it('truncates long strings', () => {
    const text = 'This is a very long string that should be truncated'
    expect(truncate(text, 20)).toBe('This is a very long...')
  })

  it('does not truncate short strings', () => {
    const text = 'Short'
    expect(truncate(text, 20)).toBe('Short')
  })

  it('handles empty string', () => {
    expect(truncate('', 10)).toBe('')
  })

  it('uses custom suffix', () => {
    expect(truncate('Long text here', 8, '…')).toBe('Long tex…')
  })
})
```

### Testing Validation Functions

```typescript
// tests/unit/utils/validation.test.ts
import { describe, it, expect } from 'vitest'
import { isEmail, isPhone, isRequired, minLength, maxLength } from '~/utils/validation'

describe('validation utilities', () => {
  describe('isEmail', () => {
    it.each([
      ['test@example.com', true],
      ['user.name@domain.org', true],
      ['user+tag@example.com', true],
      ['invalid-email', false],
      ['@nodomain.com', false],
      ['no@tld', false],
      ['', false],
    ])('isEmail("%s") returns %s', (email, expected) => {
      expect(isEmail(email)).toBe(expected)
    })
  })

  describe('isPhone', () => {
    it('validates phone numbers', () => {
      expect(isPhone('1234567890')).toBe(true)
      expect(isPhone('+1-234-567-8900')).toBe(true)
      expect(isPhone('(123) 456-7890')).toBe(true)
      expect(isPhone('123')).toBe(false)
      expect(isPhone('abcdefghij')).toBe(false)
    })
  })

  describe('isRequired', () => {
    it('validates required fields', () => {
      expect(isRequired('value')).toBe(true)
      expect(isRequired('')).toBe(false)
      expect(isRequired(null)).toBe(false)
      expect(isRequired(undefined)).toBe(false)
      expect(isRequired(0)).toBe(true)
      expect(isRequired(false)).toBe(true)
    })
  })

  describe('minLength', () => {
    it('validates minimum length', () => {
      const validate = minLength(5)
      expect(validate('hello')).toBe(true)
      expect(validate('hello world')).toBe(true)
      expect(validate('hi')).toBe(false)
      expect(validate('')).toBe(false)
    })
  })

  describe('maxLength', () => {
    it('validates maximum length', () => {
      const validate = maxLength(10)
      expect(validate('hello')).toBe(true)
      expect(validate('hello world')).toBe(false)
      expect(validate('')).toBe(true)
    })
  })
})
```

### Testing Composables

```typescript
// tests/unit/composables/useCounter.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { useCounter } from '~/composables/useCounter'

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { count } = useCounter()
    expect(count.value).toBe(0)
  })

  it('initializes with custom value', () => {
    const { count } = useCounter(10)
    expect(count.value).toBe(10)
  })

  it('increments count', () => {
    const { count, increment } = useCounter(0)
    increment()
    expect(count.value).toBe(1)
  })

  it('decrements count', () => {
    const { count, decrement } = useCounter(5)
    decrement()
    expect(count.value).toBe(4)
  })

  it('increments by custom amount', () => {
    const { count, increment } = useCounter(0)
    increment(5)
    expect(count.value).toBe(5)
  })

  it('resets to initial value', () => {
    const { count, increment, reset } = useCounter(10)
    increment()
    increment()
    expect(count.value).toBe(12)
    reset()
    expect(count.value).toBe(10)
  })

  it('computes isPositive correctly', () => {
    const { count, isPositive, decrement } = useCounter(1)
    expect(isPositive.value).toBe(true)
    decrement()
    expect(isPositive.value).toBe(false)
    decrement()
    expect(isPositive.value).toBe(false)
  })
})
```

```typescript
// tests/unit/composables/useForm.test.ts
import { describe, it, expect, vi } from 'vitest'
import { useForm } from '~/composables/useForm'

describe('useForm', () => {
  const createForm = () => useForm({
    initialValues: {
      email: '',
      password: '',
    },
    validate: (values) => ({
      email: !values.email ? 'Email required' : undefined,
      password: values.password.length < 8 ? 'Min 8 characters' : undefined,
    }),
    onSubmit: vi.fn(),
  })

  it('initializes with default values', () => {
    const { values } = createForm()
    expect(values.email).toBe('')
    expect(values.password).toBe('')
  })

  it('updates field values', () => {
    const { values, setFieldValue } = createForm()
    setFieldValue('email', 'test@example.com')
    expect(values.email).toBe('test@example.com')
  })

  it('validates form fields', async () => {
    const { errors, validateForm } = createForm()
    await validateForm()
    expect(errors.email).toBe('Email required')
    expect(errors.password).toBe('Min 8 characters')
  })

  it('clears errors on valid input', async () => {
    const { values, errors, validateForm } = createForm()
    values.email = 'test@example.com'
    values.password = 'password123'
    await validateForm()
    expect(errors.email).toBeUndefined()
    expect(errors.password).toBeUndefined()
  })

  it('tracks dirty state', () => {
    const { values, isDirty } = createForm()
    expect(isDirty.value).toBe(false)
    values.email = 'test@example.com'
    expect(isDirty.value).toBe(true)
  })

  it('tracks touched state', () => {
    const { touched, setFieldTouched } = createForm()
    expect(touched.email).toBeFalsy()
    setFieldTouched('email')
    expect(touched.email).toBe(true)
  })

  it('resets form to initial values', () => {
    const { values, reset } = createForm()
    values.email = 'test@example.com'
    reset()
    expect(values.email).toBe('')
  })

  it('submits form when valid', async () => {
    const form = createForm()
    form.values.email = 'test@example.com'
    form.values.password = 'password123'
    
    await form.handleSubmit()
    
    expect(form.onSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123',
    })
  })

  it('does not submit when invalid', async () => {
    const form = createForm()
    await form.handleSubmit()
    expect(form.onSubmit).not.toHaveBeenCalled()
  })
})
```

---

## Component Testing

### Basic Component Test

```typescript
// tests/components/common/AppButton.test.ts
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import AppButton from '~/components/common/AppButton.vue'

describe('AppButton', () => {
  it('renders slot content', () => {
    const wrapper = mount(AppButton, {
      slots: {
        default: 'Click me',
      },
    })
    expect(wrapper.text()).toBe('Click me')
  })

  it('applies variant class', () => {
    const wrapper = mount(AppButton, {
      props: { variant: 'primary' },
    })
    expect(wrapper.classes()).toContain('btn-primary')
  })

  it('applies size class', () => {
    const wrapper = mount(AppButton, {
      props: { size: 'lg' },
    })
    expect(wrapper.classes()).toContain('btn-lg')
  })

  it('is disabled when prop is true', () => {
    const wrapper = mount(AppButton, {
      props: { disabled: true },
    })
    expect(wrapper.attributes('disabled')).toBeDefined()
  })

  it('shows loading spinner when loading', () => {
    const wrapper = mount(AppButton, {
      props: { loading: true },
    })
    expect(wrapper.find('.animate-spin').exists()).toBe(true)
  })

  it('emits click event', async () => {
    const wrapper = mount(AppButton)
    await wrapper.trigger('click')
    expect(wrapper.emitted('click')).toBeTruthy()
  })

  it('does not emit click when disabled', async () => {
    const wrapper = mount(AppButton, {
      props: { disabled: true },
    })
    await wrapper.trigger('click')
    expect(wrapper.emitted('click')).toBeFalsy()
  })

  it('does not emit click when loading', async () => {
    const wrapper = mount(AppButton, {
      props: { loading: true },
    })
    await wrapper.trigger('click')
    expect(wrapper.emitted('click')).toBeFalsy()
  })
})
```

### Component with Props and Events

```typescript
// tests/components/user/UserCard.test.ts
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import UserCard from '~/components/user/UserCard.vue'

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
    avatar: 'https://example.com/avatar.jpg',
    role: 'admin' as const,
  }

  it('renders user information', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser },
    })

    expect(wrapper.text()).toContain('John Doe')
    expect(wrapper.text()).toContain('john@example.com')
  })

  it('displays user avatar', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser },
    })

    const avatar = wrapper.find('img')
    expect(avatar.attributes('src')).toBe(mockUser.avatar)
    expect(avatar.attributes('alt')).toBe(mockUser.name)
  })

  it('shows admin badge for admin users', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser },
    })

    expect(wrapper.find('[data-testid="admin-badge"]').exists()).toBe(true)
  })

  it('hides admin badge for regular users', () => {
    const wrapper = mount(UserCard, {
      props: { user: { ...mockUser, role: 'user' } },
    })

    expect(wrapper.find('[data-testid="admin-badge"]').exists()).toBe(false)
  })

  it('emits select event with user on click', async () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser },
    })

    await wrapper.trigger('click')

    expect(wrapper.emitted('select')).toBeTruthy()
    expect(wrapper.emitted('select')![0]).toEqual([mockUser])
  })

  it('emits edit event when edit button clicked', async () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser, showActions: true },
    })

    await wrapper.find('[data-testid="edit-button"]').trigger('click')

    expect(wrapper.emitted('edit')).toBeTruthy()
    expect(wrapper.emitted('edit')![0]).toEqual([mockUser])
  })

  it('emits delete event when delete button clicked', async () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser, showActions: true },
    })

    await wrapper.find('[data-testid="delete-button"]').trigger('click')

    expect(wrapper.emitted('delete')).toBeTruthy()
    expect(wrapper.emitted('delete')![0]).toEqual([mockUser])
  })

  it('hides action buttons when showActions is false', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser, showActions: false },
    })

    expect(wrapper.find('[data-testid="edit-button"]').exists()).toBe(false)
    expect(wrapper.find('[data-testid="delete-button"]').exists()).toBe(false)
  })
})
```

### Testing Components with Slots

```typescript
// tests/components/common/AppModal.test.ts
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import AppModal from '~/components/common/AppModal.vue'

describe('AppModal', () => {
  const createWrapper = (props = {}, slots = {}) => {
    return mount(AppModal, {
      props: {
        modelValue: true,
        title: 'Test Modal',
        ...props,
      },
      slots: {
        default: '<p>Modal content</p>',
        ...slots,
      },
      global: {
        stubs: {
          teleport: true,
        },
      },
    })
  }

  it('renders when modelValue is true', () => {
    const wrapper = createWrapper()
    expect(wrapper.find('[role="dialog"]').exists()).toBe(true)
  })

  it('does not render when modelValue is false', () => {
    const wrapper = createWrapper({ modelValue: false })
    expect(wrapper.find('[role="dialog"]').exists()).toBe(false)
  })

  it('renders title', () => {
    const wrapper = createWrapper({ title: 'Custom Title' })
    expect(wrapper.text()).toContain('Custom Title')
  })

  it('renders default slot content', () => {
    const wrapper = createWrapper()
    expect(wrapper.text()).toContain('Modal content')
  })

  it('renders header slot', () => {
    const wrapper = createWrapper({}, {
      header: '<h2>Custom Header</h2>',
    })
    expect(wrapper.text()).toContain('Custom Header')
  })

  it('renders footer slot', () => {
    const wrapper = createWrapper({}, {
      footer: '<button>Custom Footer</button>',
    })
    expect(wrapper.text()).toContain('Custom Footer')
  })

  it('emits update:modelValue when close button clicked', async () => {
    const wrapper = createWrapper()
    await wrapper.find('[data-testid="close-button"]').trigger('click')
    expect(wrapper.emitted('update:modelValue')).toBeTruthy()
    expect(wrapper.emitted('update:modelValue')![0]).toEqual([false])
  })

  it('emits update:modelValue when backdrop clicked', async () => {
    const wrapper = createWrapper({ closeOnBackdrop: true })
    await wrapper.find('[data-testid="backdrop"]').trigger('click')
    expect(wrapper.emitted('update:modelValue')![0]).toEqual([false])
  })

  it('does not close on backdrop click when closeOnBackdrop is false', async () => {
    const wrapper = createWrapper({ closeOnBackdrop: false })
    await wrapper.find('[data-testid="backdrop"]').trigger('click')
    expect(wrapper.emitted('update:modelValue')).toBeFalsy()
  })

  it('closes on Escape key', async () => {
    const wrapper = createWrapper()
    await wrapper.trigger('keydown', { key: 'Escape' })
    expect(wrapper.emitted('update:modelValue')![0]).toEqual([false])
  })

  it('has correct ARIA attributes', () => {
    const wrapper = createWrapper()
    const dialog = wrapper.find('[role="dialog"]')
    expect(dialog.attributes('aria-modal')).toBe('true')
    expect(dialog.attributes('aria-labelledby')).toBeDefined()
  })
})
```

---

## Integration Testing

### Testing with Nuxt Context

```typescript
// tests/integration/pages/home.test.ts
import { describe, it, expect } from 'vitest'
import { mountSuspended } from '@nuxt/test-utils/runtime'
import HomePage from '~/pages/index.vue'

describe('HomePage', () => {
  it('renders page content', async () => {
    const wrapper = await mountSuspended(HomePage)
    expect(wrapper.text()).toContain('Welcome')
  })

  it('navigates on button click', async () => {
    const wrapper = await mountSuspended(HomePage)
    const router = useRouter()
    
    await wrapper.find('[data-testid="get-started-button"]').trigger('click')
    
    expect(router.push).toHaveBeenCalledWith('/dashboard')
  })
})
```

### Testing API Integration

```typescript
// tests/integration/api/users.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import { useUsers } from '~/composables/api/useUsers'

// Mock $fetch
const mockFetch = vi.fn()
vi.stubGlobal('$fetch', mockFetch)

describe('useUsers API', () => {
  beforeEach(() => {
    mockFetch.mockReset()
  })

  afterEach(() => {
    vi.restoreAllMocks()
  })

  describe('fetchUsers', () => {
    it('fetches users successfully', async () => {
      const mockUsers = [
        { id: '1', name: 'User 1' },
        { id: '2', name: 'User 2' },
      ]
      mockFetch.mockResolvedValueOnce(mockUsers)

      const { users, fetchUsers } = useUsers()
      await fetchUsers()

      expect(mockFetch).toHaveBeenCalledWith('/api/users', expect.any(Object))
      expect(users.value).toEqual(mockUsers)
    })

    it('handles fetch error', async () => {
      const error = new Error('Network error')
      mockFetch.mockRejectedValueOnce(error)

      const { error: apiError, fetchUsers } = useUsers()
      
      await expect(fetchUsers()).rejects.toThrow('Network error')
      expect(apiError.value).toBe(error)
    })

    it('sets loading state correctly', async () => {
      mockFetch.mockImplementation(() => new Promise(resolve => 
        setTimeout(() => resolve([]), 100)
      ))

      const { loading, fetchUsers } = useUsers()
      
      expect(loading.value).toBe(false)
      
      const promise = fetchUsers()
      expect(loading.value).toBe(true)
      
      await promise
      expect(loading.value).toBe(false)
    })
  })

  describe('createUser', () => {
    it('creates user successfully', async () => {
      const newUser = { id: '3', name: 'New User' }
      mockFetch.mockResolvedValueOnce(newUser)

      const { users, createUser } = useUsers()
      await createUser({ name: 'New User' })

      expect(mockFetch).toHaveBeenCalledWith('/api/users', {
        method: 'POST',
        body: { name: 'New User' },
      })
      expect(users.value).toContainEqual(newUser)
    })
  })

  describe('deleteUser', () => {
    it('deletes user successfully', async () => {
      mockFetch
        .mockResolvedValueOnce([{ id: '1', name: 'User 1' }])
        .mockResolvedValueOnce({})

      const { users, fetchUsers, deleteUser } = useUsers()
      await fetchUsers()
      expect(users.value).toHaveLength(1)

      await deleteUser('1')
      expect(users.value).toHaveLength(0)
    })
  })
})
```

### Testing Stores

```typescript
// tests/integration/stores/auth.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useAuthStore } from '~/stores/auth'

vi.stubGlobal('$fetch', vi.fn())

describe('Auth Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
    vi.clearAllMocks()
  })

  describe('login', () => {
    it('authenticates user successfully', async () => {
      const mockResponse = {
        tokens: { accessToken: 'token', refreshToken: 'refresh', expiresAt: Date.now() + 3600000 },
        user: { id: '1', name: 'Test User' },
      }
      vi.mocked($fetch).mockResolvedValueOnce(mockResponse)

      const authStore = useAuthStore()
      await authStore.login({ email: 'test@example.com', password: 'password' })

      expect(authStore.isAuthenticated).toBe(true)
      expect(authStore.accessToken).toBe('token')
    })

    it('handles login failure', async () => {
      vi.mocked($fetch).mockRejectedValueOnce(new Error('Invalid credentials'))

      const authStore = useAuthStore()
      
      await expect(authStore.login({ email: 'test@example.com', password: 'wrong' }))
        .rejects.toThrow('Invalid credentials')
      
      expect(authStore.isAuthenticated).toBe(false)
    })
  })

  describe('logout', () => {
    it('clears authentication state', async () => {
      // Setup authenticated state
      const authStore = useAuthStore()
      vi.mocked($fetch).mockResolvedValueOnce({
        tokens: { accessToken: 'token', refreshToken: 'refresh', expiresAt: Date.now() + 3600000 },
        user: { id: '1', name: 'Test User' },
      })
      await authStore.login({ email: 'test@example.com', password: 'password' })
      expect(authStore.isAuthenticated).toBe(true)

      // Logout
      vi.mocked($fetch).mockResolvedValueOnce({})
      await authStore.logout()

      expect(authStore.isAuthenticated).toBe(false)
      expect(authStore.accessToken).toBeUndefined()
    })
  })

  describe('isAuthenticated', () => {
    it('returns false when no tokens', () => {
      const authStore = useAuthStore()
      expect(authStore.isAuthenticated).toBe(false)
    })

    it('returns false when token expired', async () => {
      vi.mocked($fetch).mockResolvedValueOnce({
        tokens: { accessToken: 'token', refreshToken: 'refresh', expiresAt: Date.now() - 1000 },
        user: { id: '1', name: 'Test User' },
      })

      const authStore = useAuthStore()
      await authStore.login({ email: 'test@example.com', password: 'password' })

      expect(authStore.isAuthenticated).toBe(false)
    })
  })
})
```

---

## Test Utilities

### Mock Factories

```typescript
// tests/utils/mocks.ts
import type { User, Product, Order } from '~/types'

export function createMockUser(overrides: Partial<User> = {}): User {
  return {
    id: crypto.randomUUID(),
    name: 'Test User',
    email: 'test@example.com',
    avatar: 'https://example.com/avatar.jpg',
    role: 'user',
    createdAt: new Date().toISOString(),
    ...overrides,
  }
}

export function createMockProduct(overrides: Partial<Product> = {}): Product {
  return {
    id: crypto.randomUUID(),
    name: 'Test Product',
    description: 'A test product',
    price: 99.99,
    category: 'electronics',
    inStock: true,
    ...overrides,
  }
}

export function createMockOrder(overrides: Partial<Order> = {}): Order {
  return {
    id: crypto.randomUUID(),
    userId: crypto.randomUUID(),
    items: [],
    total: 0,
    status: 'pending',
    createdAt: new Date().toISOString(),
    ...overrides,
  }
}

// Create multiple mocks
export function createMockUsers(count: number, overrides: Partial<User> = {}): User[] {
  return Array.from({ length: count }, (_, i) =>
    createMockUser({ name: `User ${i + 1}`, ...overrides })
  )
}
```

### Component Test Helpers

```typescript
// tests/utils/helpers.ts
import { mount, VueWrapper } from '@vue/test-utils'
import { createPinia, setActivePinia } from 'pinia'
import type { Component } from 'vue'

interface MountOptions {
  props?: Record<string, any>
  slots?: Record<string, any>
  stubs?: Record<string, any>
  mocks?: Record<string, any>
}

export function mountWithContext<T extends Component>(
  component: T,
  options: MountOptions = {},
): VueWrapper {
  setActivePinia(createPinia())

  return mount(component, {
    props: options.props,
    slots: options.slots,
    global: {
      stubs: {
        NuxtLink: {
          template: '<a :href="to"><slot /></a>',
          props: ['to'],
        },
        ...options.stubs,
      },
      mocks: {
        $t: (key: string) => key,
        ...options.mocks,
      },
    },
  })
}

// Wait for component updates
export async function flushPromises(): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, 0))
}

// Find by data-testid
export function findByTestId(wrapper: VueWrapper, testId: string) {
  return wrapper.find(`[data-testid="${testId}"]`)
}

// Find all by data-testid
export function findAllByTestId(wrapper: VueWrapper, testId: string) {
  return wrapper.findAll(`[data-testid="${testId}"]`)
}
```

---

## Best Practices

### 1. Test Organization

```typescript
// Group related tests
describe('UserCard', () => {
  describe('rendering', () => {
    it('renders user name', () => { /* ... */ })
    it('renders user email', () => { /* ... */ })
  })

  describe('interactions', () => {
    it('emits click event', () => { /* ... */ })
    it('emits edit event', () => { /* ... */ })
  })

  describe('conditional rendering', () => {
    it('shows badge for admin', () => { /* ... */ })
    it('hides actions when disabled', () => { /* ... */ })
  })
})
```

### 2. Descriptive Test Names

```typescript
// ✅ Good: Describes behavior
it('shows error message when email is invalid', () => { /* ... */ })
it('disables submit button while form is submitting', () => { /* ... */ })

// ❌ Bad: Vague descriptions
it('works correctly', () => { /* ... */ })
it('handles error', () => { /* ... */ })
```

### 3. Arrange-Act-Assert Pattern

```typescript
it('increments counter when button clicked', async () => {
  // Arrange
  const wrapper = mount(Counter)
  
  // Act
  await wrapper.find('button').trigger('click')
  
  // Assert
  expect(wrapper.text()).toContain('1')
})
```

### 4. Test Data-TestId Attributes

```vue
<!-- Component -->
<button data-testid="submit-button">Submit</button>

<!-- Test -->
await wrapper.find('[data-testid="submit-button"]').trigger('click')
```

### 5. Avoid Testing Implementation Details

```typescript
// ✅ Good: Test behavior
it('displays error when validation fails', async () => {
  const wrapper = mount(Form)
  await wrapper.find('input').setValue('invalid')
  await wrapper.find('form').trigger('submit')
  expect(wrapper.text()).toContain('Invalid input')
})

// ❌ Bad: Testing internal state
it('sets isError to true', async () => {
  const wrapper = mount(Form)
  // Don't test internal implementation
  expect(wrapper.vm.isError).toBe(true)
})
```

---

## Next Steps

Continue to [Code Quality](./10-code-quality.md) to learn about linting, formatting, and git hooks configuration.