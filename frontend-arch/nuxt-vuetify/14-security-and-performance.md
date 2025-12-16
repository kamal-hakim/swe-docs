# Security and Performance

## Overview

This document covers security best practices and performance optimization strategies for Nuxt applications with Vuetify. Security is implemented through the **nuxt-security** module, Content Security Policy (CSP), and various security headers. Performance optimization includes Vuetify tree-shaking, bundle optimization, caching strategies, and rendering optimization.

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
  modules: ["nuxt-security"],

  security: {
    // Enable in all environments
    enabled: true,

    // Security headers
    headers: {
      // Content Security Policy
      contentSecurityPolicy: {
        "base-uri": ["'self'"],
        "default-src": ["'self'"],
        "connect-src": ["'self'", "https:", "wss:"],
        "font-src": ["'self'", "https:", "data:"],
        "form-action": ["'self'"],
        "frame-ancestors": ["'none'"],
        "frame-src": ["'self'", "https:"],
        "img-src": ["'self'", "https:", "data:", "blob:"],
        "manifest-src": ["'self'"],
        "media-src": ["'self'", "https:"],
        "object-src": ["'none'"],
        // Vuetify requires unsafe-inline for styles
        "script-src": ["'self'", "'unsafe-inline'", "'wasm-unsafe-eval'"],
        "script-src-attr": ["'none'"],
        "style-src": ["'self'", "'unsafe-inline'"],
        "upgrade-insecure-requests": true,
        "worker-src": ["'self'", "blob:"],
      },

      // Cross-Origin settings
      crossOriginEmbedderPolicy: false,
      crossOriginOpenerPolicy: "same-origin",
      crossOriginResourcePolicy: "same-origin",

      // Other security headers
      originAgentCluster: "?1",
      referrerPolicy: "strict-origin-when-cross-origin",
      strictTransportSecurity: {
        maxAge: 31536000,
        includeSubdomains: true,
        preload: true,
      },
      xContentTypeOptions: "nosniff",
      xDNSPrefetchControl: "off",
      xDownloadOptions: "noopen",
      xFrameOptions: "DENY",
      xPermittedCrossDomainPolicies: "none",
      xXSSProtection: "1; mode=block",

      // Permissions Policy
      permissionsPolicy: {
        accelerometer: [],
        "ambient-light-sensor": [],
        autoplay: ["self"],
        camera: [],
        "display-capture": [],
        "document-domain": [],
        "encrypted-media": ["self"],
        fullscreen: ["self"],
        geolocation: [],
        gyroscope: [],
        magnetometer: [],
        microphone: [],
        midi: [],
        payment: [],
        "picture-in-picture": ["self"],
        "publickey-credentials-get": [],
        "screen-wake-lock": [],
        usb: [],
        "xr-spatial-tracking": [],
      },
    },

    // Rate limiting
    rateLimiter: {
      tokensPerInterval: 150,
      interval: 300000, // 5 minutes
      headers: true,
      driver: {
        name: "memory",
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
      stripIgnoreTagBody: ["script"],
    },

    // CORS configuration
    corsHandler: {
      origin: "*",
      methods: ["GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"],
      allowHeaders: ["Content-Type", "Authorization"],
      exposeHeaders: ["Content-Length", "X-Request-Id"],
      credentials: false,
      maxAge: 86400,
    },

    // CSRF protection
    csrf: {
      enabled: true,
      cookie: {
        name: "__csrf",
        sameSite: "strict",
      },
      methodsToProtect: ["POST", "PUT", "PATCH", "DELETE"],
    },

    // Nonce for inline scripts
    nonce: true,
  },
});
```

---

## XSS Prevention

### HTML Sanitization

```typescript
// composables/useSanitize.ts
import DOMPurify from "isomorphic-dompurify";

export function useSanitize() {
  const sanitizeHtml = (dirty: string, options?: DOMPurify.Config): string => {
    return DOMPurify.sanitize(dirty, {
      ALLOWED_TAGS: [
        "b",
        "i",
        "em",
        "strong",
        "a",
        "p",
        "br",
        "ul",
        "ol",
        "li",
      ],
      ALLOWED_ATTR: ["href", "target", "rel"],
      ALLOW_DATA_ATTR: false,
      ...options,
    });
  };

  const sanitizeUrl = (url: string): string => {
    try {
      const parsed = new URL(url);
      if (!["http:", "https:", "mailto:"].includes(parsed.protocol)) {
        return "";
      }
      return url;
    } catch {
      return "";
    }
  };

  const escapeHtml = (text: string): string => {
    const map: Record<string, string> = {
      "&": "&amp;",
      "<": "&lt;",
      ">": "&gt;",
      '"': "&quot;",
      "'": "&#039;",
    };
    return text.replace(/[&<>"']/g, (m) => map[m]);
  };

  return {
    sanitizeHtml,
    sanitizeUrl,
    escapeHtml,
  };
}
```

### Safe Content Component

```vue
<!-- components/common/SafeHtml.vue -->
<script setup lang="ts">
interface Props {
  content: string;
  tag?: string;
}

const props = withDefaults(defineProps<Props>(), {
  tag: "div",
});

const { sanitizeHtml } = useSanitize();

const sanitizedContent = computed(() => sanitizeHtml(props.content));
</script>

<template>
  <component :is="tag" v-html="sanitizedContent" />
</template>
```

---

## Authentication Security

### Secure Token Storage

```typescript
// composables/useAuth.ts
export function useAuth() {
  // Store tokens in httpOnly cookies (set by server)
  const accessToken = useCookie("access_token", {
    httpOnly: true,
    secure: true,
    sameSite: "strict",
    maxAge: 60 * 15, // 15 minutes
  });

  const refreshToken = useCookie("refresh_token", {
    httpOnly: true,
    secure: true,
    sameSite: "strict",
    maxAge: 60 * 60 * 24 * 7, // 7 days
  });

  const csrf = useCookie("csrf_token", {
    secure: true,
    sameSite: "strict",
  });

  return {
    accessToken: readonly(accessToken),
    refreshToken: readonly(refreshToken),
    csrf: readonly(csrf),
  };
}
```

### API Request Security

```typescript
// composables/api/useSecureFetch.ts
export function useSecureFetch() {
  const csrf = useCookie("csrf_token");

  const secureFetch = async <T>(
    url: string,
    options: RequestInit = {}
  ): Promise<T> => {
    const headers = new Headers(options.headers);

    // Add CSRF token for mutating requests
    if (["POST", "PUT", "PATCH", "DELETE"].includes(options.method || "")) {
      headers.set("X-CSRF-Token", csrf.value || "");
    }

    // Add security headers
    headers.set("X-Requested-With", "XMLHttpRequest");
    headers.set("X-Content-Type-Options", "nosniff");

    const response = await $fetch<T>(url, {
      ...options,
      headers,
      credentials: "include",
    });

    return response;
  };

  return { secureFetch };
}
```

---

## Performance Optimization

### Vuetify Tree-Shaking

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  build: {
    transpile: ["vuetify"],
  },

  vite: {
    ssr: {
      noExternal: ["vuetify"],
    },
  },
});
```

```typescript
// plugins/vuetify.ts
import { createVuetify } from "vuetify";

// Import only needed components
import {
  VApp,
  VAppBar,
  VBtn,
  VCard,
  VCardText,
  VCardTitle,
  VContainer,
  VDialog,
  VForm,
  VIcon,
  VList,
  VListItem,
  VMain,
  VNavigationDrawer,
  VSelect,
  VSnackbar,
  VTextField,
} from "vuetify/components";

// Import only needed directives
import { Ripple, Scroll } from "vuetify/directives";

export default defineNuxtPlugin((nuxtApp) => {
  const vuetify = createVuetify({
    components: {
      VApp,
      VAppBar,
      VBtn,
      VCard,
      VCardText,
      VCardTitle,
      VContainer,
      VDialog,
      VForm,
      VIcon,
      VList,
      VListItem,
      VMain,
      VNavigationDrawer,
      VSelect,
      VSnackbar,
      VTextField,
    },
    directives: {
      Ripple,
      Scroll,
    },
  });

  nuxtApp.vueApp.use(vuetify);
});
```

### Bundle Optimization

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  vite: {
    build: {
      // Target modern browsers
      target: "esnext",

      // Chunk splitting
      rollupOptions: {
        output: {
          manualChunks: {
            "vue-vendor": ["vue", "vue-router", "pinia"],
            vuetify: ["vuetify"],
            utils: ["@vueuse/core"],
          },
        },
      },

      // Minimize CSS
      cssMinify: "lightningcss",

      // Report bundle size
      reportCompressedSize: true,
    },

    // Optimize dependencies
    optimizeDeps: {
      include: ["vue", "vue-router", "pinia", "@vueuse/core", "vuetify"],
    },
  },

  // Experimental features
  experimental: {
    viewTransition: true,
    renderJsonPayloads: true,
    asyncContext: true,
  },

  // Build options
  build: {
    analyze: process.env.ANALYZE === "true",
  },
});
```

### Lazy Loading Components

```vue
<script setup lang="ts">
// Lazy load heavy components
const HeavyChart = defineAsyncComponent(
  () => import("~/components/charts/HeavyChart.vue")
);

const LazyDialog = defineAsyncComponent({
  loader: () => import("~/components/common/LazyDialog.vue"),
  loadingComponent: () => import("~/components/common/LoadingSkeleton.vue"),
  delay: 200,
  timeout: 10000,
});
</script>

<template>
  <div>
    <Suspense>
      <template #default>
        <HeavyChart :data="chartData" />
      </template>
      <template #fallback>
        <v-skeleton-loader type="image" />
      </template>
    </Suspense>
  </div>
</template>
```

### Vuetify Lazy Components

```vue
<template>
  <!-- Lazy load images -->
  <v-img :src="imageUrl" lazy-src="/placeholder.jpg" aspect-ratio="16/9">
    <template #placeholder>
      <v-row class="fill-height ma-0" align="center" justify="center">
        <v-progress-circular indeterminate color="primary" />
      </v-row>
    </template>
  </v-img>

  <!-- Virtual scroll for large lists -->
  <v-virtual-scroll :items="largeList" :item-height="50" height="400">
    <template #default="{ item }">
      <v-list-item :title="item.title" />
    </template>
  </v-virtual-scroll>
</template>
```

---

## Caching Strategies

### Static Asset Caching

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    publicAssets: [
      {
        dir: "public/images",
        baseURL: "/images",
        maxAge: 60 * 60 * 24 * 365, // 1 year
      },
      {
        dir: "public/fonts",
        baseURL: "/fonts",
        maxAge: 60 * 60 * 24 * 365, // 1 year
      },
    ],

    routeRules: {
      // Static pages
      "/": { prerender: true, cache: { maxAge: 60 * 60 } },
      "/about": { prerender: true, cache: { maxAge: 60 * 60 * 24 } },

      // API routes
      "/api/public/**": { cache: { maxAge: 60 } },

      // Dynamic content with SWR
      "/api/posts/**": {
        swr: true,
        cache: {
          maxAge: 60,
          staleMaxAge: 60 * 60,
        },
      },

      // No cache for authenticated
      "/api/user/**": { cache: false },
    },
  },
});
```

### API Response Caching

```typescript
// server/api/posts/[id].get.ts
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, "id");

  // Set cache headers
  setHeaders(event, {
    "Cache-Control":
      "public, max-age=60, s-maxage=300, stale-while-revalidate=600",
    "CDN-Cache-Control": "public, max-age=300",
    Vary: "Accept-Encoding",
  });

  const post = await fetchPost(id);

  // Set ETag
  const etag = `"${hashContent(post)}"`;
  setHeaders(event, { ETag: etag });

  // Check if client has cached version
  const ifNoneMatch = getHeader(event, "if-none-match");
  if (ifNoneMatch === etag) {
    setResponseStatus(event, 304);
    return null;
  }

  return post;
});
```

---

## Rendering Performance

### Virtual Scrolling with Vuetify

```vue
<script setup lang="ts">
interface Item {
  id: string;
  title: string;
  subtitle: string;
}

const props = defineProps<{
  items: Item[];
}>();
</script>

<template>
  <v-virtual-scroll :items="items" :item-height="72" height="600">
    <template #default="{ item }">
      <v-list-item :key="item.id" :title="item.title" :subtitle="item.subtitle">
        <template #prepend>
          <v-avatar color="primary">
            {{ item.title[0] }}
          </v-avatar>
        </template>
        <template #append>
          <v-btn icon="mdi-chevron-right" variant="text" />
        </template>
      </v-list-item>
    </template>
  </v-virtual-scroll>
</template>
```

### Debounced Search

```vue
<script setup lang="ts">
const searchQuery = ref("");
const debouncedQuery = refDebounced(searchQuery, 300);

const { data: results, pending } = await useFetch("/api/search", {
  query: { q: debouncedQuery },
  watch: [debouncedQuery],
});
</script>

<template>
  <v-text-field
    v-model="searchQuery"
    label="Search"
    prepend-inner-icon="mdi-magnify"
    :loading="pending"
    clearable
  />

  <v-list v-if="results?.length">
    <v-list-item
      v-for="result in results"
      :key="result.id"
      :title="result.title"
    />
  </v-list>
</template>
```

### Computed Properties Optimization

```vue
<script setup lang="ts">
// ✅ Good: Memoized with computed
const filteredItems = computed(() => {
  return items.value
    .filter((i) => i.active)
    .sort((a, b) => a.name.localeCompare(b.name));
});

// ✅ Use shallowRef for large objects
const largeData = shallowRef<LargeObject[]>([]);

// ✅ Lazy computed
const expensiveData = computedAsync(async () => {
  return await fetchExpensiveData();
});
</script>
```

---

## Network Performance

### Prefetching

```vue
<script setup lang="ts">
// Prefetch on hover
const prefetchPost = (id: string) => {
  $fetch(`/api/posts/${id}`, { method: "HEAD" });
};

// Preload critical resources
useHead({
  link: [
    { rel: "preload", as: "font", href: "/fonts/main.woff2", crossorigin: "" },
    { rel: "preconnect", href: "https://api.example.com" },
    { rel: "dns-prefetch", href: "https://cdn.example.com" },
  ],
});
</script>

<template>
  <v-list>
    <v-list-item
      v-for="post in posts"
      :key="post.id"
      :to="`/posts/${post.id}`"
      @mouseenter="prefetchPost(post.id)"
    >
      {{ post.title }}
    </v-list-item>
  </v-list>
</template>
```

### Image Optimization

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ["@nuxt/image"],

  image: {
    provider: "ipx",
    quality: 80,
    format: ["avif", "webp"],
    screens: {
      xs: 320,
      sm: 640,
      md: 768,
      lg: 1024,
      xl: 1280,
    },
    presets: {
      avatar: {
        modifiers: {
          format: "webp",
          width: 80,
          height: 80,
          fit: "cover",
        },
      },
      thumbnail: {
        modifiers: {
          format: "webp",
          width: 200,
          height: 200,
          fit: "cover",
        },
      },
    },
  },
});
```

---

## Monitoring and Metrics

### Performance Monitoring

```typescript
// plugins/performance.client.ts
export default defineNuxtPlugin(() => {
  if (process.env.NODE_ENV !== "production") return;

  // Web Vitals
  const reportWebVital = (metric: any) => {
    $fetch("/api/metrics", {
      method: "POST",
      body: {
        name: metric.name,
        value: metric.value,
        id: metric.id,
        page: window.location.pathname,
      },
    });
  };

  // Observe Core Web Vitals
  if ("PerformanceObserver" in window) {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        reportWebVital({
          name: entry.name,
          value: entry.startTime,
          id: crypto.randomUUID(),
        });
      }
    });

    observer.observe({ type: "largest-contentful-paint", buffered: true });
    observer.observe({ type: "first-input", buffered: true });
    observer.observe({ type: "layout-shift", buffered: true });
  }
});
```

### Error Tracking

```typescript
// plugins/error-tracking.client.ts
export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.config.errorHandler = (error, instance, info) => {
    console.error("Vue Error:", error);

    trackError({
      error: error as Error,
      component: instance?.$options?.name,
      info,
      url: window.location.href,
      userAgent: navigator.userAgent,
    });
  };

  window.addEventListener("unhandledrejection", (event) => {
    console.error("Unhandled Rejection:", event.reason);

    trackError({
      error: event.reason,
      type: "unhandledrejection",
      url: window.location.href,
    });
  });
});

function trackError(data: any) {
  $fetch("/api/errors", {
    method: "POST",
    body: data,
  }).catch(console.error);
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

- [ ] Enable Vuetify tree-shaking
- [ ] Optimize images (WebP, lazy loading)
- [ ] Implement code splitting
- [ ] Use virtual scrolling for large lists
- [ ] Debounce/throttle expensive operations
- [ ] Configure asset caching
- [ ] Minimize JavaScript bundle
- [ ] Preload critical resources
- [ ] Monitor Core Web Vitals
- [ ] Use CDN for static assets

---

## Next Steps

Continue to [Deployment Guide](./15-deployment-guide.md) to learn about deployment strategies for SPA and SSR applications.
