# Routing & Navigation

## Overview

Nuxt provides a powerful file-based routing system that automatically generates routes based on the directory structure in the `pages/` directory. This document covers routing patterns, middleware, navigation guards, and best practices for both SPA and SSR environments.

## File-Based Routing

### Basic Route Structure

```
app/pages/
├── index.vue              # /
├── about.vue              # /about
├── contact.vue            # /contact
├── login.vue              # /login
├── dashboard/
│   ├── index.vue          # /dashboard
│   ├── settings.vue       # /dashboard/settings
│   └── analytics.vue      # /dashboard/analytics
├── users/
│   ├── index.vue          # /users
│   └── [id].vue           # /users/:id (dynamic)
├── blog/
│   ├── index.vue          # /blog
│   ├── [slug].vue         # /blog/:slug
│   └── category/
│       └── [category].vue # /blog/category/:category
└── [...slug].vue          # Catch-all route (404)
```

### Route Naming Conventions

| Pattern | File | Route | Example URL |
|---------|------|-------|-------------|
| Static | `about.vue` | `/about` | `/about` |
| Index | `index.vue` | `/` | `/` |
| Dynamic | `[id].vue` | `/:id` | `/123` |
| Optional | `[[id]].vue` | `/:id?` | `/` or `/123` |
| Catch-all | `[...slug].vue` | `/:slug(.*)*` | `/any/path/here` |
| Named param | `user-[id].vue` | `/user-:id` | `/user-123` |

---

## Dynamic Routes

### Single Parameter

```vue
<!-- pages/users/[id].vue -->
<script setup lang="ts">
const route = useRoute()

// Type-safe parameter access
const userId = computed(() => route.params.id as string)

// Fetch user data
const { data: user, pending, error } = await useAsyncData(
  `user-${userId.value}`,
  () => $fetch(`/api/users/${userId.value}`)
)
</script>

<template>
  <div>
    <div v-if="pending">Loading...</div>
    <div v-else-if="error">Error: {{ error.message }}</div>
    <div v-else>
      <h1>{{ user?.name }}</h1>
      <p>{{ user?.email }}</p>
    </div>
  </div>
</template>
```

### Multiple Parameters

```vue
<!-- pages/[category]/[product].vue -->
<script setup lang="ts">
const route = useRoute()

const category = computed(() => route.params.category as string)
const productSlug = computed(() => route.params.product as string)

// Fetch with multiple params
const { data: product } = await useAsyncData(
  `product-${category.value}-${productSlug.value}`,
  () => $fetch(`/api/products/${category.value}/${productSlug.value}`)
)
</script>

<template>
  <div>
    <nav>
      <NuxtLink :to="`/${category}`">{{ category }}</NuxtLink>
      <span> / {{ product?.name }}</span>
    </nav>
    <article>
      <h1>{{ product?.name }}</h1>
    </article>
  </div>
</template>
```

### Catch-All Routes

```vue
<!-- pages/[...slug].vue -->
<script setup lang="ts">
const route = useRoute()

// slug is an array of path segments
const segments = computed(() => route.params.slug as string[])
const fullPath = computed(() => segments.value?.join('/') || '')

// Use for 404 or CMS pages
definePageMeta({
  layout: 'default',
})
</script>

<template>
  <div>
    <h1>Page Not Found</h1>
    <p>The path "{{ fullPath }}" does not exist.</p>
    <NuxtLink to="/">Go Home</NuxtLink>
  </div>
</template>
```

### Optional Parameters

```vue
<!-- pages/products/[[category]].vue -->
<!-- Matches both /products and /products/electronics -->
<script setup lang="ts">
const route = useRoute()

const category = computed(() => route.params.category as string | undefined)

const { data: products } = await useAsyncData(
  `products-${category.value || 'all'}`,
  () => {
    const params = category.value ? { category: category.value } : {}
    return $fetch('/api/products', { params })
  }
)
</script>
```

---

## Page Metadata

### definePageMeta

```vue
<script setup lang="ts">
definePageMeta({
  // Layout to use
  layout: 'dashboard',
  
  // Middleware to run
  middleware: ['auth', 'admin'],
  
  // Custom page key for caching
  key: route => route.fullPath,
  
  // Keep-alive configuration
  keepalive: true,
  
  // Page transition
  pageTransition: {
    name: 'slide',
    mode: 'out-in',
  },
  
  // Custom meta data
  title: 'Dashboard',
  requiresAuth: true,
  roles: ['admin', 'manager'],
})
</script>
```

### SEO Metadata

```vue
<script setup lang="ts">
// Dynamic SEO meta
const route = useRoute()
const { data: product } = await useAsyncData(() => fetchProduct(route.params.id))

useHead({
  title: () => product.value?.name || 'Product',
  meta: [
    { name: 'description', content: () => product.value?.description },
    { property: 'og:title', content: () => product.value?.name },
    { property: 'og:description', content: () => product.value?.description },
    { property: 'og:image', content: () => product.value?.image },
  ],
})

useSeoMeta({
  title: () => product.value?.name,
  ogTitle: () => product.value?.name,
  description: () => product.value?.description,
  ogDescription: () => product.value?.description,
  ogImage: () => product.value?.image,
  twitterCard: 'summary_large_image',
})
</script>
```

---

## Layouts

### Default Layout

```vue
<!-- layouts/default.vue -->
<script setup lang="ts">
const route = useRoute()
</script>

<template>
  <div class="app-layout">
    <AppHeader />
    
    <div class="app-content">
      <AppSidebar v-if="route.meta.showSidebar !== false" />
      
      <main class="main-content">
        <slot />
      </main>
    </div>
    
    <AppFooter />
  </div>
</template>

<style scoped>
.app-layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-content {
  flex: 1;
  display: flex;
}

.main-content {
  flex: 1;
  padding: 1rem;
}
</style>
```

### Custom Layouts

```vue
<!-- layouts/auth.vue -->
<template>
  <div class="auth-layout">
    <div class="auth-container">
      <div class="auth-logo">
        <NuxtLink to="/">
          <img src="/logo.svg" alt="Logo" />
        </NuxtLink>
      </div>
      
      <div class="auth-content">
        <slot />
      </div>
    </div>
  </div>
</template>

<!-- layouts/dashboard.vue -->
<script setup lang="ts">
const appStore = useAppStore()
</script>

<template>
  <div class="dashboard-layout">
    <DashboardSidebar :collapsed="appStore.sidebarCollapsed" />
    
    <div class="dashboard-main">
      <DashboardHeader />
      
      <div class="dashboard-content">
        <slot />
      </div>
    </div>
  </div>
</template>

<!-- layouts/blank.vue -->
<template>
  <div class="blank-layout">
    <slot />
  </div>
</template>
```

### Using Layouts in Pages

```vue
<script setup lang="ts">
// Specify layout in definePageMeta
definePageMeta({
  layout: 'dashboard',
})
</script>

<!-- Or use NuxtLayout component -->
<template>
  <NuxtLayout name="custom">
    <h1>Page Content</h1>
  </NuxtLayout>
</template>
```

### Dynamic Layout

```vue
<script setup lang="ts">
const layout = ref('default')

// Change layout based on condition
watch(() => userRole.value, (role) => {
  layout.value = role === 'admin' ? 'admin' : 'default'
})

// Use setPageLayout helper
function switchLayout(newLayout: string) {
  setPageLayout(newLayout)
}
</script>

<template>
  <NuxtLayout :name="layout">
    <slot />
  </NuxtLayout>
</template>
```

---

## Middleware

### Route Middleware Types

```
app/middleware/
├── auth.ts                 # Named middleware (manual)
├── admin.ts                # Named middleware (manual)
├── guest.ts                # Named middleware (manual)
├── setup.global.ts         # Global middleware (automatic)
└── analytics.global.ts     # Global middleware (automatic)
```

### Named Middleware

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const authStore = useAuthStore()

  // Check authentication
  if (!authStore.isAuthenticated) {
    // Save intended destination
    const redirectPath = to.fullPath
    
    return navigateTo({
      path: '/login',
      query: { redirect: redirectPath },
    })
  }
})

// middleware/admin.ts
export default defineNuxtRouteMiddleware((to) => {
  const userStore = useUserStore()

  if (!userStore.isAdmin) {
    return navigateTo('/dashboard')
  }
})

// middleware/guest.ts
export default defineNuxtRouteMiddleware(() => {
  const authStore = useAuthStore()

  if (authStore.isAuthenticated) {
    return navigateTo('/dashboard')
  }
})
```

### Global Middleware

```typescript
// middleware/setup.global.ts
export default defineNuxtRouteMiddleware(async () => {
  const authStore = useAuthStore()

  // Initialize auth state on first load
  if (!authStore.isInitialized) {
    await authStore.initialize()
  }
})

// middleware/analytics.global.ts
export default defineNuxtRouteMiddleware((to, from) => {
  // Track page views
  if (import.meta.client && from.path !== to.path) {
    trackPageView({
      path: to.path,
      title: to.meta.title as string,
    })
  }
})
```

### Using Middleware in Pages

```vue
<script setup lang="ts">
// Single middleware
definePageMeta({
  middleware: 'auth',
})

// Multiple middleware (executed in order)
definePageMeta({
  middleware: ['auth', 'admin'],
})

// Inline middleware
definePageMeta({
  middleware: [
    function (to, from) {
      // Custom logic
      if (someCondition) {
        return navigateTo('/other-page')
      }
    },
  ],
})
</script>
```

### Middleware with Data

```typescript
// middleware/role.ts
export default defineNuxtRouteMiddleware((to) => {
  const userStore = useUserStore()
  const requiredRoles = to.meta.roles as string[] | undefined

  if (requiredRoles && requiredRoles.length > 0) {
    const hasRole = requiredRoles.includes(userStore.currentUser?.role || '')
    
    if (!hasRole) {
      return abortNavigation({
        statusCode: 403,
        statusMessage: 'Access Denied',
        message: 'You do not have permission to access this page.',
      })
    }
  }
})
```

---

## Navigation

### NuxtLink Component

```vue
<template>
  <!-- Basic link -->
  <NuxtLink to="/about">About</NuxtLink>

  <!-- With params -->
  <NuxtLink :to="{ name: 'users-id', params: { id: user.id } }">
    {{ user.name }}
  </NuxtLink>

  <!-- With query -->
  <NuxtLink :to="{ path: '/search', query: { q: searchTerm } }">
    Search
  </NuxtLink>

  <!-- External link -->
  <NuxtLink to="https://example.com" external target="_blank">
    External Site
  </NuxtLink>

  <!-- With active class -->
  <NuxtLink
    to="/dashboard"
    active-class="text-primary font-bold"
    exact-active-class="border-b-2"
  >
    Dashboard
  </NuxtLink>

  <!-- Prefetch control -->
  <NuxtLink to="/heavy-page" :prefetch="false">
    Heavy Page (no prefetch)
  </NuxtLink>

  <!-- Replace history (no back) -->
  <NuxtLink to="/step-2" replace>
    Next Step
  </NuxtLink>
</template>
```

### Programmatic Navigation

```typescript
// Using navigateTo
async function goToUser(id: string) {
  await navigateTo(`/users/${id}`)
}

async function goToSearch(query: string) {
  await navigateTo({
    path: '/search',
    query: { q: query },
  })
}

// With options
async function goToDashboard() {
  await navigateTo('/dashboard', {
    replace: true,      // Replace current history entry
    redirectCode: 301,  // For SSR redirects
    external: false,    // External URL
    open: {             // Window.open options
      target: '_blank',
    },
  })
}

// Using useRouter
const router = useRouter()

function goBack() {
  router.back()
}

function goForward() {
  router.forward()
}

function goToPage(delta: number) {
  router.go(delta)
}

async function navigate() {
  await router.push('/new-page')
}

async function replaceRoute() {
  await router.replace('/new-page')
}
```

### Navigation Guards

```typescript
// Using onBeforeRouteLeave
<script setup lang="ts">
import { onBeforeRouteLeave } from 'vue-router'

const hasUnsavedChanges = ref(false)

onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges.value) {
    const answer = window.confirm('You have unsaved changes. Leave anyway?')
    if (!answer) {
      return false // Cancel navigation
    }
  }
})
</script>

// Using onBeforeRouteUpdate
<script setup lang="ts">
import { onBeforeRouteUpdate } from 'vue-router'

onBeforeRouteUpdate(async (to, from) => {
  // Called when route params change but component is reused
  // e.g., /users/1 -> /users/2
  if (to.params.id !== from.params.id) {
    await fetchUserData(to.params.id as string)
  }
})
</script>
```

---

## Data Fetching

### useAsyncData

```vue
<script setup lang="ts">
// Basic usage
const { data, pending, error, refresh } = await useAsyncData(
  'users',
  () => $fetch('/api/users')
)

// With options
const { data: products } = await useAsyncData(
  'products',
  () => $fetch('/api/products'),
  {
    // Only fetch on server
    server: true,
    
    // Lazy load (don't block navigation)
    lazy: false,
    
    // Default value
    default: () => [],
    
    // Transform response
    transform: (data) => data.items,
    
    // Custom key
    key: 'my-products',
    
    // Watch for changes
    watch: [categoryFilter],
  }
)

// Refresh data
async function refreshData() {
  await refresh()
}

// Clear cached data
function clearCache() {
  clearNuxtData('users')
}
</script>
```

### useFetch

```vue
<script setup lang="ts">
// Simple fetch
const { data, pending, error } = await useFetch('/api/users')

// With options
const { data: user } = await useFetch(`/api/users/${id}`, {
  method: 'GET',
  headers: {
    'Authorization': `Bearer ${token}`,
  },
  query: {
    include: 'profile',
  },
  // Only run on client
  server: false,
  
  // Watch for reactive dependencies
  watch: [id],
  
  // Immediate execution
  immediate: true,
})

// POST request
const { data: newUser } = await useFetch('/api/users', {
  method: 'POST',
  body: {
    name: 'John',
    email: 'john@example.com',
  },
})
</script>
```

### useLazyAsyncData / useLazyFetch

```vue
<script setup lang="ts">
// Non-blocking data fetch
const { data, pending } = useLazyAsyncData('heavy-data', () => 
  $fetch('/api/heavy-computation')
)

// Show loading state
</script>

<template>
  <div>
    <div v-if="pending" class="skeleton">Loading...</div>
    <div v-else>
      {{ data }}
    </div>
  </div>
</template>
```

---

## Route Validation

### validate Function

```vue
<script setup lang="ts">
definePageMeta({
  validate: async (route) => {
    // Validate route params
    const id = route.params.id as string
    
    // Check if valid format
    if (!/^\d+$/.test(id)) {
      return false // Returns 404
    }
    
    // Check if resource exists
    try {
      await $fetch(`/api/users/${id}`)
      return true
    } catch {
      return {
        statusCode: 404,
        statusMessage: 'User not found',
      }
    }
  },
})
</script>
```

### Custom Error Pages

```vue
<!-- error.vue (root level) -->
<script setup lang="ts">
interface Props {
  error: {
    statusCode: number
    statusMessage: string
    message?: string
    data?: any
  }
}

const props = defineProps<Props>()

const handleError = () => {
  clearError({ redirect: '/' })
}

// Different content based on error
const errorContent = computed(() => {
  switch (props.error.statusCode) {
    case 404:
      return {
        title: 'Page Not Found',
        message: 'The page you are looking for does not exist.',
        icon: '🔍',
      }
    case 403:
      return {
        title: 'Access Denied',
        message: 'You do not have permission to view this page.',
        icon: '🚫',
      }
    case 500:
      return {
        title: 'Server Error',
        message: 'Something went wrong on our end.',
        icon: '⚠️',
      }
    default:
      return {
        title: 'Error',
        message: props.error.message || 'An unexpected error occurred.',
        icon: '❌',
      }
  }
})
</script>

<template>
  <div class="error-page">
    <div class="error-content">
      <span class="error-icon">{{ errorContent.icon }}</span>
      <h1>{{ props.error.statusCode }}</h1>
      <h2>{{ errorContent.title }}</h2>
      <p>{{ errorContent.message }}</p>
      
      <div class="error-actions">
        <button @click="handleError">Go Home</button>
        <NuxtLink to="javascript:history.back()">Go Back</NuxtLink>
      </div>
    </div>
  </div>
</template>
```

---

## Route Rules

### Server-Side Route Rules

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // Pre-render at build time
    '/': { prerender: true },
    '/about': { prerender: true },
    '/blog/**': { prerender: true },
    
    // Client-side only (SPA)
    '/dashboard/**': { ssr: false },
    '/admin/**': { ssr: false },
    
    // ISR (Incremental Static Regeneration)
    '/products/**': { swr: 3600 }, // Revalidate every hour
    '/news/**': { isr: 600 }, // Revalidate every 10 minutes
    
    // Redirects
    '/old-page': { redirect: '/new-page' },
    '/old-blog/**': { redirect: '/blog/**' },
    
    // Headers
    '/api/**': {
      cors: true,
      headers: {
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE',
      },
    },
    
    // Cache control
    '/_nuxt/**': {
      headers: {
        'Cache-Control': 'public, max-age=31536000, immutable',
      },
    },
    
    // Proxy
    '/external-api/**': {
      proxy: 'https://api.example.com/**',
    },
  },
})
```

---

## Navigation Best Practices

### 1. Use NuxtLink for Internal Links

```vue
<!-- ✅ Good: Use NuxtLink -->
<NuxtLink to="/about">About</NuxtLink>

<!-- ❌ Bad: Use anchor tags -->
<a href="/about">About</a>
```

### 2. Handle Loading States

```vue
<script setup lang="ts">
const nuxtApp = useNuxtApp()
const isLoading = ref(false)

nuxtApp.hook('page:start', () => {
  isLoading.value = true
})

nuxtApp.hook('page:finish', () => {
  isLoading.value = false
})
</script>

<template>
  <div>
    <LoadingBar v-if="isLoading" />
    <NuxtPage />
  </div>
</template>
```

### 3. Scroll Behavior

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  router: {
    options: {
      scrollBehaviorType: 'smooth',
    },
  },
})

// Or in app/router.options.ts
export default {
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    }

    if (to.hash) {
      return {
        el: to.hash,
        behavior: 'smooth',
      }
    }

    return { top: 0, behavior: 'smooth' }
  },
}
```

### 4. Route Transitions

```vue
<!-- app.vue -->
<template>
  <NuxtLayout>
    <NuxtPage :transition="{
      name: 'page',
      mode: 'out-in',
    }" />
  </NuxtLayout>
</template>

<style>
.page-enter-active,
.page-leave-active {
  transition: all 0.3s;
}

.page-enter-from,
.page-leave-to {
  opacity: 0;
  transform: translateY(20px);
}
</style>
```

### 5. Breadcrumb Navigation

```vue
<!-- components/AppBreadcrumb.vue -->
<script setup lang="ts">
const route = useRoute()

interface Breadcrumb {
  label: string
  path: string
}

const breadcrumbs = computed<Breadcrumb[]>(() => {
  const paths = route.path.split('/').filter(Boolean)
  
  return paths.map((segment, index) => {
    const path = '/' + paths.slice(0, index + 1).join('/')
    const label = segment.charAt(0).toUpperCase() + segment.slice(1).replace(/-/g, ' ')
    
    return { label, path }
  })
})
</script>

<template>
  <nav aria-label="Breadcrumb" class="breadcrumb">
    <ol>
      <li>
        <NuxtLink to="/">Home</NuxtLink>
      </li>
      <li v-for="(crumb, index) in breadcrumbs" :key="crumb.path">
        <span class="separator">/</span>
        <NuxtLink
          v-if="index < breadcrumbs.length - 1"
          :to="crumb.path"
        >
          {{ crumb.label }}
        </NuxtLink>
        <span v-else aria-current="page">{{ crumb.label }}</span>
      </li>
    </ol>
  </nav>
</template>
```

---

## SPA vs SSR Routing Differences

### SPA Mode Considerations

```typescript
// nuxt.config.ts (SPA)
export default defineNuxtConfig({
  ssr: false,
  
  // Hash mode for static hosting without server config
  router: {
    options: {
      hashMode: true, // URLs become /#/path
    },
  },
  
  // All routes client-side
  routeRules: {
    '/**': { ssr: false },
  },
})
```

### SSR Mode Considerations

```typescript
// nuxt.config.ts (SSR)
export default defineNuxtConfig({
  ssr: true,
  
  // Hybrid rendering
  routeRules: {
    // Static pages
    '/': { prerender: true },
    '/about': { prerender: true },
    
    // Dynamic pages with SSR
    '/products/**': { swr: true },
    
    // Interactive pages without SSR
    '/dashboard/**': { ssr: false },
  },
})
```

---

## Testing Routes

### Route Testing

```typescript
// tests/routes.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { mountSuspended } from '@nuxt/test-utils/runtime'

describe('Routing', () => {
  it('navigates to about page', async () => {
    const router = useRouter()
    await router.push('/about')
    expect(router.currentRoute.value.path).toBe('/about')
  })

  it('handles dynamic routes', async () => {
    const router = useRouter()
    await router.push('/users/123')
    expect(router.currentRoute.value.params.id).toBe('123')
  })

  it('applies middleware', async () => {
    const authStore = useAuthStore()
    authStore.setAuthenticated(false)

    const router = useRouter()
    await router.push('/dashboard')

    expect(router.currentRoute.value.path).toBe('/login')
  })
})
```

---

## Next Steps

Continue to [Styling Architecture](./08-styling-architecture.md) to learn about CSS organization, theming, and UnoCSS patterns.