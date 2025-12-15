# Security and Performance

## Overview

This document covers security best practices and performance optimization strategies for Nuxt applications. Security is implemented through the **nuxt-security** module, Content Security Policy (CSP), and various security headers. Performance optimization includes bundle optimization, caching strategies, lazy loading, and rendering optimization.

---

## Security Configuration

### Installation

```bash
# Install nuxt-security module
pnpm add -D nuxt-security
```

### Module Setup

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    'nuxt-security',
  ],
  
  security: {
    // Enable in all environments (disable in dev if causing issues)
    enabled: true,
    
    // Security headers
    headers: {
      // Content Security Policy
      contentSecurityPolicy: {
        'base-uri': ["'self'"],
        'default-src': ["'self'"],
        'connect-src': ["'self'", 'https:', 'wss:'],
        'font-src': ["'self'", 'https:', 'data:'],
        'form-action': ["'self'"],
        'frame-ancestors': ["'none'"],
        'frame-src': ["'self'", 'https:'],
        'img-src': ["'self'", 'https:', 'data:', 'blob:'],
        'manifest-src': ["'self'"],
        'media-src': ["'self'", 'https:'],
        'object-src': ["'none'"],
        'script-src': ["'self'", "'unsafe-inline'", "'wasm-unsafe-eval'"],
        'script-src-attr': ["'none'"],
        'style-src': ["'self'", "'unsafe-inline'"],
        'upgrade-insecure-requests': true,
        'worker-src': ["'self'", 'blob:'],
      },
      
      // Cross-Origin settings
      crossOriginEmbedderPolicy: false, // Set to 'require-corp' for strict mode
      crossOriginOpenerPolicy: 'same-origin',
      crossOriginResourcePolicy: 'same-origin',
      
      // Other security headers
      originAgentCluster: '?1',
      referrerPolicy: 'strict-origin-when-cross-origin',
      strictTransportSecurity: {
        maxAge: 31536000,
        includeSubdomains: true,
        preload: true,
      },
      xContentTypeOptions: 'nosniff',
      xDNSPrefetchControl: 'off',
      xDownloadOptions: 'noopen',
      xFrameOptions: 'DENY',
      xPermittedCrossDomainPolicies: 'none',
      xXSSProtection: '1; mode=block',
      
      // Permissions Policy
      permissionsPolicy: {
        'accelerometer': [],
        'ambient-light-sensor': [],
        'autoplay': ['self'],
        'camera': [],
        'display-capture': [],
        'document-domain': [],
        'encrypted-media': ['self'],
        'fullscreen': ['self'],
        'geolocation': [],
        'gyroscope': [],
        'magnetometer': [],
        'microphone': [],
        'midi': [],
        'payment': [],
        'picture-in-picture': ['self'],
        'publickey-credentials-get': [],
        'screen-wake-lock': [],
        'usb': [],
        'xr-spatial-tracking': [],
      },
    },
    
    // Rate limiting (server routes)
    rateLimiter: {
      tokensPerInterval: 150,
      interval: 300000, // 5 minutes
      headers: true,
      driver: {
        name: 'memory',
      },
    },
    
    // Request size limits
    requestSizeLimiter: {
      maxRequestSizeInBytes: 2000000, // 2MB
      maxUploadFileRequestInBytes: 10000000, // 10MB
    },
    
    // XSS validator
    xssValidator: {
      stripIgnoreTag: true,
      stripIgnoreTagBody: ['script'],
    },
    
    // CORS configuration
    corsHandler: {
      origin: '*',
      methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
      allowHeaders: ['Content-Type', 'Authorization'],
      exposeHeaders: ['Content-Length', 'X-Request-Id'],
      credentials: false,
      maxAge: 86400,
    },
    
    // CSRF protection
    csrf: {
      enabled: true,
      cookie: {
        name: '__csrf',
        sameSite: 'strict',
      },
      methodsToProtect: ['POST', 'PUT', 'PATCH', 'DELETE'],
    },
    
    // Nonce for inline scripts
    nonce: true,
  },
})
```

### Route-Specific Security

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // API routes - strict CSP
    '/api/**': {
      security: {
        headers: {
          contentSecurityPolicy: {
            'default-src': ["'none'"],
            'frame-ancestors': ["'none'"],
          },
        },
      },
    },
    
    // Public pages - allow external resources
    '/public/**': {
      security: {
        headers: {
          contentSecurityPolicy: {
            'img-src': ["'self'", 'https:', 'data:'],
            'script-src': ["'self'", "'unsafe-inline'"],
          },
        },
      },
    },
    
    // Admin pages - additional protection
    '/admin/**': {
      security: {
        rateLimiter: {
          tokensPerInterval: 50,
          interval: 300000,
        },
      },
    },
  },
})
```

---

## XSS Prevention

### HTML Sanitization

```typescript
// composables/useSanitize.ts
import DOMPurify from 'isomorphic-dompurify'

export function useSanitize() {
  const sanitizeHtml = (dirty: string, options?: DOMPurify.Config): string => {
    return DOMPurify.sanitize(dirty, {
      ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br', 'ul', 'ol', 'li'],
      ALLOWED_ATTR: ['href', 'target', 'rel'],
      ALLOW_DATA_ATTR: false,
      ...options,
    })
  }

  const sanitizeUrl = (url: string): string => {
    try {
      const parsed = new URL(url)
      if (!['http:', 'https:', 'mailto:'].includes(parsed.protocol)) {
        return ''
      }
      return url
    }
    catch {
      return ''
    }
  }

  const escapeHtml = (text: string): string => {
    const map: Record<string, string> = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      '\'': '&#039;',
    }
    return text.replace(/[&<>"']/g, m => map[m])
  }

  return {
    sanitizeHtml,
    sanitizeUrl,
    escapeHtml,
  }
}
```

### Safe Content Rendering

```vue
<!-- components/SafeHtml.vue -->
<script setup lang="ts">
interface Props {
  content: string
  tag?: string
}

const props = withDefaults(defineProps<Props>(), {
  tag: 'div',
})

const { sanitizeHtml } = useSanitize()

const sanitizedContent = computed(() => sanitizeHtml(props.content))
</script>

<template>
  <component
    :is="tag"
    v-html="sanitizedContent"
  />
</template>
```

### Input Validation

```typescript
// utils/validation.ts
export const validators = {
  // Prevent script injection
  noScript: (value: string): boolean => {
    return !/<script|javascript:|on\w+=/i.test(value)
  },

  // Validate email format
  email: (value: string): boolean => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    return emailRegex.test(value) && value.length <= 254
  },

  // Validate URL
  url: (value: string): boolean => {
    try {
      const url = new URL(value)
      return ['http:', 'https:'].includes(url.protocol)
    }
    catch {
      return false
    }
  },

  // Alphanumeric only
  alphanumeric: (value: string): boolean => {
    return /^[a-zA-Z0-9]+$/.test(value)
  },

  // Safe filename
  filename: (value: string): boolean => {
    return /^[\w\-. ]+$/.test(value) && !value.includes('..')
  },
}
```

---

## Authentication Security

### Secure Token Storage

```typescript
// composables/useAuth.ts
export function useAuth() {
  // Store tokens in httpOnly cookies (set by server)
  // Never store tokens in localStorage or sessionStorage
  
  const accessToken = useCookie('access_token', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 60 * 15, // 15 minutes
  })

  const refreshToken = useCookie('refresh_token', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 60 * 60 * 24 * 7, // 7 days
  })

  const csrf = useCookie('csrf_token', {
    secure: true,
    sameSite: 'strict',
  })

  return {
    accessToken: readonly(accessToken),
    refreshToken: readonly(refreshToken),
    csrf: readonly(csrf),
  }
}
```

### API Request Security

```typescript
// composables/api/useSecureFetch.ts
export function useSecureFetch() {
  const csrf = useCookie('csrf_token')
  
  const secureFetch = async <T>(
    url: string,
    options: RequestInit = {},
  ): Promise<T> => {
    const headers = new Headers(options.headers)
    
    // Add CSRF token for mutating requests
    if (['POST', 'PUT', 'PATCH', 'DELETE'].includes(options.method || '')) {
      headers.set('X-CSRF-Token', csrf.value || '')
    }
    
    // Add security headers
    headers.set('X-Requested-With', 'XMLHttpRequest')
    headers.set('X-Content-Type-Options', 'nosniff')
    
    const response = await $fetch<T>(url, {
      ...options,
      headers,
      credentials: 'include',
    })
    
    return response
  }

  return { secureFetch }
}
```

---

## Performance Optimization

### Bundle Optimization

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Build optimization
  vite: {
    build: {
      // Target modern browsers
      target: 'esnext',
      
      // Chunk splitting
      rollupOptions: {
        output: {
          manualChunks: {
            // Vendor chunks
            'vue-vendor': ['vue', 'vue-router', 'pinia'],
            'ui-vendor': ['@vueuse/core'],
          },
        },
      },
      
      // Minimize CSS
      cssMinify: 'lightningcss',
      
      // Report bundle size
      reportCompressedSize: true,
    },
    
    // Optimize dependencies
    optimizeDeps: {
      include: [
        'vue',
        'vue-router',
        'pinia',
        '@vueuse/core',
      ],
    },
  },
  
  // Experimental features
  experimental: {
    // Enable view transitions
    viewTransition: true,
    
    // Render JSON payloads
    renderJsonPayloads: true,
    
    // Async context
    asyncContext: true,
  },
  
  // Build options
  build: {
    // Transpile dependencies if needed
    transpile: [],
    
    // Analyze bundle (set via env)
    analyze: process.env.ANALYZE === 'true',
  },
})
```

### Image Optimization

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/image'],
  
  image: {
    // Provider configuration
    provider: 'ipx',
    
    // Quality settings
    quality: 80,
    
    // Format preferences
    format: ['avif', 'webp'],
    
    // Screen sizes for srcset
    screens: {
      xs: 320,
      sm: 640,
      md: 768,
      lg: 1024,
      xl: 1280,
      '2xl': 1536,
    },
    
    // Presets
    presets: {
      avatar: {
        modifiers: {
          format: 'webp',
          width: 80,
          height: 80,
          fit: 'cover',
        },
      },
      thumbnail: {
        modifiers: {
          format: 'webp',
          width: 200,
          height: 200,
          fit: 'cover',
        },
      },
      hero: {
        modifiers: {
          format: 'webp',
          width: 1920,
          height: 1080,
          fit: 'cover',
          quality: 90,
        },
      },
    },
    
    // Domains for external images
    domains: ['images.example.com', 'cdn.example.com'],
  },
})
```

### Component Usage

```vue
<template>
  <!-- Optimized image with srcset -->
  <NuxtImg
    src="/images/hero.jpg"
    alt="Hero image"
    width="1920"
    height="1080"
    loading="lazy"
    placeholder
    sizes="sm:100vw md:80vw lg:1200px"
  />
  
  <!-- Avatar with preset -->
  <NuxtImg
    :src="user.avatar"
    :alt="user.name"
    preset="avatar"
    loading="lazy"
  />
</template>
```

### Lazy Loading Components

```vue
<script setup lang="ts">
// Lazy load heavy components
const HeavyChart = defineAsyncComponent(() =>
  import('~/components/charts/HeavyChart.vue')
)

const LazyModal = defineAsyncComponent({
  loader: () => import('~/components/common/Modal.vue'),
  loadingComponent: LoadingSkeleton,
  delay: 200,
  timeout: 10000,
})

// Conditional lazy loading
const AdvancedEditor = computed(() => {
  if (useAdvancedEditor.value) {
    return defineAsyncComponent(() =>
      import('~/components/editor/AdvancedEditor.vue')
    )
  }
  return BasicEditor
})
</script>

<template>
  <div>
    <!-- Lazy component -->
    <Suspense>
      <template #default>
        <HeavyChart :data="chartData" />
      </template>
      <template #fallback>
        <ChartSkeleton />
      </template>
    </Suspense>
    
    <!-- Conditional render -->
    <component :is="AdvancedEditor" v-model="content" />
  </div>
</template>
```

### Code Splitting

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Route-level code splitting
  routeRules: {
    // Don't prerender admin routes
    '/admin/**': {
      ssr: false,
      experimentalNoScripts: false,
    },
  },
  
  // Component prefetching
  experimental: {
    componentIslands: true,
  },
})
```

---

## Caching Strategies

### Static Asset Caching

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    // Public assets with caching
    publicAssets: [
      {
        dir: 'public/images',
        baseURL: '/images',
        maxAge: 60 * 60 * 24 * 365, // 1 year
      },
      {
        dir: 'public/fonts',
        baseURL: '/fonts',
        maxAge: 60 * 60 * 24 * 365, // 1 year
      },
    ],
    
    // Route caching rules
    routeRules: {
      // Static pages - long cache
      '/': { prerender: true, cache: { maxAge: 60 * 60 } },
      '/about': { prerender: true, cache: { maxAge: 60 * 60 * 24 } },
      
      // API routes - short cache
      '/api/public/**': { cache: { maxAge: 60 } },
      
      // Dynamic content - SWR
      '/api/posts/**': { 
        swr: true, 
        cache: { 
          maxAge: 60, 
          staleMaxAge: 60 * 60,
        },
      },
      
      // No cache for authenticated
      '/api/user/**': { cache: false },
    },
  },
})
```

### API Response Caching

```typescript
// server/api/posts/[id].get.ts
import { defineEventHandler, getRouterParam } from 'h3'

export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')
  
  // Set cache headers
  setHeaders(event, {
    'Cache-Control': 'public, max-age=60, s-maxage=300, stale-while-revalidate=600',
    'CDN-Cache-Control': 'public, max-age=300',
    'Vary': 'Accept-Encoding',
  })
  
  const post = await fetchPost(id)
  
  // Set ETag for conditional requests
  const etag = `"${hashContent(post)}"`
  setHeaders(event, { ETag: etag })
  
  // Check if client has cached version
  const ifNoneMatch = getHeader(event, 'if-none-match')
  if (ifNoneMatch === etag) {
    setResponseStatus(event, 304)
    return null
  }
  
  return post
})
```

### Client-Side Caching

```typescript
// composables/useCache.ts
export function useCache<T>() {
  const cache = new Map<string, { data: T; timestamp: number }>()
  
  const get = (key: string, maxAge: number = 60000): T | null => {
    const entry = cache.get(key)
    if (!entry) return null
    
    if (Date.now() - entry.timestamp > maxAge) {
      cache.delete(key)
      return null
    }
    
    return entry.data
  }
  
  const set = (key: string, data: T): void => {
    cache.set(key, { data, timestamp: Date.now() })
  }
  
  const invalidate = (pattern?: string): void => {
    if (!pattern) {
      cache.clear()
      return
    }
    
    const regex = new RegExp(pattern)
    for (const key of cache.keys()) {
      if (regex.test(key)) {
        cache.delete(key)
      }
    }
  }
  
  return { get, set, invalidate }
}
```

---

## Rendering Performance

### Virtual Scrolling

```vue
<!-- components/VirtualList.vue -->
<script setup lang="ts">
import { useVirtualList } from '@vueuse/core'

interface Props {
  items: any[]
  itemHeight: number
}

const props = defineProps<Props>()

const { list, containerProps, wrapperProps } = useVirtualList(
  computed(() => props.items),
  {
    itemHeight: props.itemHeight,
    overscan: 5,
  },
)
</script>

<template>
  <div v-bind="containerProps" class="h-96 overflow-auto">
    <div v-bind="wrapperProps">
      <div
        v-for="{ data, index } in list"
        :key="index"
        :style="{ height: `${itemHeight}px` }"
      >
        <slot :item="data" :index="index" />
      </div>
    </div>
  </div>
</template>
```

### Debounced Inputs

```vue
<script setup lang="ts">
const searchQuery = ref('')
const debouncedQuery = refDebounced(searchQuery, 300)

const { data: results } = await useFetch('/api/search', {
  query: { q: debouncedQuery },
  watch: [debouncedQuery],
})
</script>

<template>
  <input v-model="searchQuery" placeholder="Search..." />
  <SearchResults :results="results" />
</template>
```

### Computed Properties Optimization

```vue
<script setup lang="ts">
// ❌ Bad: Expensive computation in template
// <div>{{ items.filter(i => i.active).map(i => i.name).join(', ') }}</div>

// ✅ Good: Memoized with computed
const activeItemNames = computed(() => {
  return items.value
    .filter(i => i.active)
    .map(i => i.name)
    .join(', ')
})

// ✅ Better: Use shallowRef for large objects
const largeData = shallowRef<LargeObject[]>([])

// ✅ Lazy computed (only computed when accessed)
const expensiveData = computedAsync(async () => {
  return await fetchExpensiveData()
})
</script>
```

---

## Network Performance

### Prefetching

```vue
<script setup lang="ts">
// Prefetch on hover
const prefetchPost = (id: string) => {
  $fetch(`/api/posts/${id}`, { method: 'HEAD' })
}

// Preload critical resources
useHead({
  link: [
    { rel: 'preload', as: 'font', href: '/fonts/main.woff2', crossorigin: '' },
    { rel: 'preconnect', href: 'https://api.example.com' },
    { rel: 'dns-prefetch', href: 'https://cdn.example.com' },
  ],
})
</script>

<template>
  <NuxtLink
    v-for="post in posts"
    :key="post.id"
    :to="`/posts/${post.id}`"
    @mouseenter="prefetchPost(post.id)"
  >
    {{ post.title }}
  </NuxtLink>
</template>
```

### Request Batching

```typescript
// composables/useBatchedFetch.ts
export function useBatchedFetch<T>(
  batchSize: number = 10,
  delay: number = 50,
) {
  const pending = ref<string[]>([])
  const cache = new Map<string, T>()
  let timeout: NodeJS.Timeout | null = null

  const fetch = (id: string): Promise<T> => {
    // Return cached
    if (cache.has(id)) {
      return Promise.resolve(cache.get(id)!)
    }

    // Add to pending
    pending.value.push(id)

    // Schedule batch
    if (!timeout) {
      timeout = setTimeout(executeBatch, delay)
    }

    // Wait for batch result
    return new Promise((resolve) => {
      watch(
        () => cache.get(id),
        (value) => {
          if (value) resolve(value)
        },
        { immediate: true },
      )
    })
  }

  const executeBatch = async () => {
    const ids = pending.value.splice(0, batchSize)
    timeout = null

    if (ids.length === 0) return

    const results = await $fetch<Record<string, T>>('/api/batch', {
      method: 'POST',
      body: { ids },
    })

    for (const [id, data] of Object.entries(results)) {
      cache.set(id, data)
    }
  }

  return { fetch }
}
```

---

## Monitoring and Metrics

### Performance Monitoring

```typescript
// plugins/performance.client.ts
export default defineNuxtPlugin(() => {
  if (process.env.NODE_ENV !== 'production') return

  // Web Vitals
  const reportWebVital = (metric: any) => {
    console.log(metric)
    
    // Send to analytics
    $fetch('/api/metrics', {
      method: 'POST',
      body: {
        name: metric.name,
        value: metric.value,
        id: metric.id,
        page: window.location.pathname,
      },
    })
  }

  // Observe Core Web Vitals
  if ('PerformanceObserver' in window) {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        reportWebVital({
          name: entry.name,
          value: entry.startTime,
          id: crypto.randomUUID(),
        })
      }
    })

    observer.observe({ type: 'largest-contentful-paint', buffered: true })
    observer.observe({ type: 'first-input', buffered: true })
    observer.observe({ type: 'layout-shift', buffered: true })
  }
})
```

### Error Tracking

```typescript
// plugins/error-tracking.client.ts
export default defineNuxtPlugin((nuxtApp) => {
  // Global error handler
  nuxtApp.vueApp.config.errorHandler = (error, instance, info) => {
    console.error('Vue Error:', error)
    
    // Send to error tracking service
    trackError({
      error: error as Error,
      component: instance?.$options?.name,
      info,
      url: window.location.href,
      userAgent: navigator.userAgent,
    })
  }

  // Unhandled promise rejections
  window.addEventListener('unhandledrejection', (event) => {
    console.error('Unhandled Rejection:', event.reason)
    
    trackError({
      error: event.reason,
      type: 'unhandledrejection',
      url: window.location.href,
    })
  })
})

function trackError(data: any) {
  // Send to your error tracking service (Sentry, Bugsnag, etc.)
  $fetch('/api/errors', {
    method: 'POST',
    body: data,
  }).catch(console.error)
}
```

---

## Best Practices Checklist

### Security

- [ ] Enable HTTPS in production
- [ ] Configure Content Security Policy
- [ ] Implement CSRF protection
- [ ] Sanitize user input
- [ ] Validate and escape output
- [ ] Use httpOnly cookies for tokens
- [ ] Implement rate limiting
- [ ] Set security headers
- [ ] Regular dependency updates
- [ ] Security audits with `pnpm audit`

### Performance

- [ ] Optimize images (WebP, lazy loading)
- [ ] Implement code splitting
- [ ] Use virtual scrolling for large lists
- [ ] Debounce/throttle expensive operations
- [ ] Configure asset caching
- [ ] Minimize JavaScript bundle
- [ ] Preload critical resources
- [ ] Monitor Core Web Vitals
- [ ] Use CDN for static assets
- [ ] Enable compression (gzip/brotli)

---

## Next Steps

Continue to [Deployment Guide](./12-deployment-guide.md) to learn about deployment strategies for SPA and SSR applications.