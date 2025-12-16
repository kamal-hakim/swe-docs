# Nuxt Configuration

## Overview

This document provides comprehensive Nuxt configuration patterns for both **SPA (Single Page Application)** and **SSR (Server-Side Rendering)** deployments. Each configuration section includes explanations and best practices.

## Base Configuration

### Complete nuxt.config.ts Template

```typescript
// nuxt.config.ts
import { resolve } from 'node:path'

// Environment detection
const isDevelopment = process.env.NODE_ENV === 'development'
const isProduction = process.env.NODE_ENV === 'production'
const isCI = process.env.CI === 'true'

export default defineNuxtConfig({
  // ==========================================
  // Core Configuration
  // ==========================================
  
  // Nuxt 4 compatibility date
  compatibilityDate: '2025-01-01',
  
  // Enable/disable SSR (true = SSR, false = SPA)
  ssr: true,
  
  // Source directory for Nuxt app
  srcDir: 'app/',
  
  // Enable Nuxt DevTools
  devtools: { enabled: true },

  // ==========================================
  // TypeScript Configuration
  // ==========================================
  typescript: {
    strict: true,
    typeCheck: true,
    shim: false,
    tsConfig: {
      compilerOptions: {
        strict: true,
        noUncheckedIndexedAccess: true,
        forceConsistentCasingInFileNames: true,
      },
      vueCompilerOptions: {
        target: 3.5,
      },
    },
  },

  // ==========================================
  // Modules
  // ==========================================
  modules: [
    // Core modules
    '@vueuse/nuxt',
    '@unocss/nuxt',
    '@pinia/nuxt',
    '@nuxtjs/color-mode',
    '@nuxt/test-utils/module',
    
    // Optional modules (uncomment as needed)
    // '@nuxtjs/i18n',
    // '@vite-pwa/nuxt',
    // 'nuxt-security',
  ],

  // ==========================================
  // Auto-imports Configuration
  // ==========================================
  imports: {
    dirs: [
      'composables/**',
      'utils/**',
      'stores/**',
    ],
  },

  // ==========================================
  // Components Configuration
  // ==========================================
  components: [
    {
      path: '~/components',
      pathPrefix: false,
      extensions: ['.vue'],
    },
  ],

  // ==========================================
  // CSS Configuration
  // ==========================================
  css: [
    '@unocss/reset/tailwind.css',
    '~/styles/variables.css',
    '~/styles/base.css',
    '~/styles/utilities.css',
  ],

  // ==========================================
  // PostCSS Configuration
  // ==========================================
  postcss: {
    plugins: {
      'postcss-nested': {},
    },
  },

  // ==========================================
  // Runtime Configuration
  // ==========================================
  runtimeConfig: {
    // Private keys (server-only)
    apiSecret: process.env.API_SECRET || '',
    
    // Public keys (exposed to client)
    public: {
      apiBaseUrl: process.env.NUXT_PUBLIC_API_BASE_URL || '/api',
      appName: process.env.NUXT_PUBLIC_APP_NAME || 'My App',
      appVersion: process.env.npm_package_version || '0.0.0',
    },
  },

  // ==========================================
  // App Configuration
  // ==========================================
  app: {
    // Keep-alive for pages
    keepalive: true,
    
    // Head configuration
    head: {
      viewport: 'width=device-width, initial-scale=1, viewport-fit=cover',
      charset: 'utf-8',
      htmlAttrs: {
        lang: 'en',
      },
      bodyAttrs: {
        class: 'antialiased',
      },
      link: [
        { rel: 'icon', href: '/favicon.ico', sizes: 'any' },
        { rel: 'icon', type: 'image/svg+xml', href: '/icon.svg' },
        { rel: 'apple-touch-icon', href: '/apple-touch-icon.png' },
      ],
      meta: [
        { name: 'theme-color', content: '#ffffff' },
        { name: 'format-detection', content: 'telephone=no' },
        { property: 'og:type', content: 'website' },
        { name: 'twitter:card', content: 'summary_large_image' },
      ],
    },
    
    // Page transition
    pageTransition: { name: 'page', mode: 'out-in' },
    layoutTransition: { name: 'layout', mode: 'out-in' },
  },

  // ==========================================
  // Route Rules
  // ==========================================
  routeRules: {
    // Static generation
    '/': { prerender: true },
    '/about': { prerender: true },
    
    // SPA routes (client-side only)
    '/dashboard/**': { ssr: false },
    '/admin/**': { ssr: false },
    
    // API caching
    '/api/**': { 
      cors: true,
      headers: { 'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE' },
    },
    
    // Static asset caching
    '/_nuxt/**': { headers: { 'Cache-Control': 'public, max-age=31536000, immutable' } },
  },

  // ==========================================
  // Vite Configuration
  // ==========================================
  vite: {
    build: {
      target: 'esnext',
      sourcemap: isDevelopment,
    },
    optimizeDeps: {
      include: [
        // Pre-bundle frequently used dependencies
        'vue',
        'vue-router',
        'pinia',
        '@vueuse/core',
      ],
    },
    css: {
      devSourcemap: true,
    },
  },

  // ==========================================
  // Nitro Server Configuration
  // ==========================================
  nitro: {
    preset: 'node-server', // Change based on deployment target
    compressPublicAssets: true,
    minify: true,
    
    // Public assets with cache headers
    publicAssets: [
      {
        dir: 'public/images',
        maxAge: 60 * 60 * 24 * 30, // 30 days
        baseURL: '/images',
      },
      {
        dir: 'public/fonts',
        maxAge: 60 * 60 * 24 * 365, // 1 year
        baseURL: '/fonts',
      },
    ],
    
    // Route rules for API
    routeRules: {
      '/api/**': { cors: true },
    },
  },

  // ==========================================
  // Experimental Features
  // ==========================================
  experimental: {
    payloadExtraction: true,
    renderJsonPayloads: true,
    typedPages: true,
    viewTransition: true,
  },

  // ==========================================
  // Features
  // ==========================================
  features: {
    inlineStyles: true,
    devLogs: true,
  },

  // ==========================================
  // Module Configurations
  // ==========================================
  
  // Color Mode
  colorMode: {
    classSuffix: '',
    preference: 'system',
    fallback: 'light',
  },

  // Pinia
  pinia: {
    storesDirs: ['./app/stores/**'],
  },
})
```

---

## SPA-Specific Configuration

For Single Page Application deployments with client-side only rendering:

```typescript
// nuxt.config.ts - SPA Mode
export default defineNuxtConfig({
  // Disable SSR for pure SPA
  ssr: false,
  
  // Compatibility date
  compatibilityDate: '2025-01-01',
  
  // Source directory
  srcDir: 'app/',
  
  // App configuration for SPA
  app: {
    // Base URL for deployment (e.g., GitHub Pages)
    baseURL: process.env.NUXT_APP_BASE_URL || '/',
    
    head: {
      viewport: 'width=device-width, initial-scale=1',
      meta: [
        { name: 'theme-color', content: '#ffffff' },
      ],
    },
  },
  
  // Route rules - all routes use client-side rendering
  routeRules: {
    '/**': { ssr: false },
  },
  
  // Nitro configuration for static hosting
  nitro: {
    preset: 'static', // Generates static files
    
    // For GitHub Pages or similar subdirectory deployments
    // preset: 'github-pages',
    
    // Static generation options
    prerender: {
      crawlLinks: true,
      routes: ['/'],
      failOnError: false,
    },
  },
  
  // Experimental features for SPA
  experimental: {
    payloadExtraction: true,
  },
  
  // Router options for SPA
  router: {
    options: {
      // Use hash mode for static hosting without server config
      // hashMode: true,
    },
  },
})
```

### SPA Build Command

```bash
# Generate static SPA
pnpm generate

# Preview generated site
pnpm preview
```

---

## SSR-Specific Configuration

For Server-Side Rendering with Node.js server:

```typescript
// nuxt.config.ts - SSR Mode
export default defineNuxtConfig({
  // Enable SSR
  ssr: true,
  
  // Compatibility date
  compatibilityDate: '2025-01-01',
  
  // Source directory
  srcDir: 'app/',
  
  // Route rules for hybrid rendering
  routeRules: {
    // Pre-render static pages
    '/': { prerender: true },
    '/about': { prerender: true },
    '/blog/**': { prerender: true },
    
    // ISR for dynamic content
    '/products/**': { swr: 3600 }, // Revalidate every hour
    
    // Client-only for authenticated routes
    '/dashboard/**': { ssr: false },
    '/account/**': { ssr: false },
    
    // API routes
    '/api/**': { cors: true },
  },
  
  // Nitro configuration for SSR
  nitro: {
    // Deployment presets:
    // - 'node-server': Standard Node.js server
    // - 'vercel': Vercel deployment
    // - 'netlify': Netlify deployment
    // - 'cloudflare-pages': Cloudflare Pages
    // - 'aws-lambda': AWS Lambda
    preset: 'node-server',
    
    // Server compression
    compressPublicAssets: true,
    
    // Static asset caching
    publicAssets: [
      {
        dir: 'public',
        maxAge: 60 * 60 * 24 * 30,
      },
    ],
  },
  
  // Experimental SSR features
  experimental: {
    payloadExtraction: true,
    renderJsonPayloads: true,
    componentIslands: true,
  },
})
```

### SSR Build Commands

```bash
# Build for production
pnpm build

# Start production server
node .output/server/index.mjs

# Or use the built-in command
pnpm preview
```

---

## Module Configurations

### VueUse Module

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@vueuse/nuxt'],
  
  // VueUse auto-imports composables automatically
})
```

### UnoCSS Module

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@unocss/nuxt'],
  
  // UnoCSS configuration is in unocss.config.ts
})
```

```typescript
// unocss.config.ts
import {
  defineConfig,
  presetAttributify,
  presetIcons,
  presetTypography,
  presetUno,
  presetWebFonts,
  transformerDirectives,
  transformerVariantGroup,
} from 'unocss'

export default defineConfig({
  shortcuts: {
    // Layout
    'flex-center': 'flex items-center justify-center',
    'flex-col-center': 'flex flex-col items-center justify-center',
    
    // Components
    'btn': 'px-4 py-2 rounded-lg font-medium transition-colors cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed',
    'btn-primary': 'btn bg-primary-500 text-white hover:bg-primary-600',
    'btn-secondary': 'btn bg-gray-200 text-gray-800 hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200',
    
    // Form elements
    'input-base': 'w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-primary-500 dark:border-gray-600 dark:bg-gray-800',
    
    // Cards
    'card': 'bg-white dark:bg-gray-800 rounded-lg shadow-md p-4',
  },
  
  theme: {
    colors: {
      primary: {
        50: '#f0f9ff',
        100: '#e0f2fe',
        200: '#bae6fd',
        300: '#7dd3fc',
        400: '#38bdf8',
        500: '#0ea5e9',
        600: '#0284c7',
        700: '#0369a1',
        800: '#075985',
        900: '#0c4a6e',
      },
    },
  },
  
  presets: [
    presetUno(),
    presetAttributify(),
    presetIcons({
      scale: 1.2,
      cdn: 'https://esm.sh/',
    }),
    presetTypography(),
    presetWebFonts({
      fonts: {
        sans: 'Inter:400,500,600,700',
        mono: 'Fira Code:400,500',
      },
    }),
  ],
  
  transformers: [
    transformerDirectives(),
    transformerVariantGroup(),
  ],
  
  safelist: [
    // Safelist dynamic classes
    ...Array.from({ length: 10 }, (_, i) => `col-span-${i + 1}`),
  ],
})
```

### Pinia Module

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@pinia/nuxt'],
  
  pinia: {
    storesDirs: ['./app/stores/**'],
  },
})
```

### i18n Module

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/i18n'],
  
  i18n: {
    locales: [
      { code: 'en', name: 'English', file: 'en.json' },
      { code: 'es', name: 'Español', file: 'es.json' },
      { code: 'fr', name: 'Français', file: 'fr.json' },
    ],
    defaultLocale: 'en',
    strategy: 'prefix_except_default',
    lazy: true,
    langDir: '../locales',
    detectBrowserLanguage: {
      useCookie: true,
      cookieKey: 'i18n_redirected',
      redirectOn: 'root',
    },
  },
})
```

### Color Mode Module

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/color-mode'],
  
  colorMode: {
    classSuffix: '',
    preference: 'system',
    fallback: 'light',
    storageKey: 'color-mode',
  },
})
```

### PWA Module

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@vite-pwa/nuxt'],
  
  pwa: {
    registerType: 'autoUpdate',
    manifest: {
      name: 'My App',
      short_name: 'MyApp',
      description: 'My awesome Nuxt app',
      theme_color: '#ffffff',
      background_color: '#ffffff',
      display: 'standalone',
      orientation: 'portrait',
      icons: [
        {
          src: 'icons/icon-192x192.png',
          sizes: '192x192',
          type: 'image/png',
        },
        {
          src: 'icons/icon-512x512.png',
          sizes: '512x512',
          type: 'image/png',
        },
        {
          src: 'icons/icon-512x512.png',
          sizes: '512x512',
          type: 'image/png',
          purpose: 'maskable',
        },
      ],
    },
    workbox: {
      navigateFallback: '/',
      globPatterns: ['**/*.{js,css,html,png,svg,ico,woff2}'],
      runtimeCaching: [
        {
          urlPattern: /^https:\/\/api\..*/i,
          handler: 'NetworkFirst',
          options: {
            cacheName: 'api-cache',
            expiration: {
              maxEntries: 50,
              maxAgeSeconds: 60 * 60 * 24, // 24 hours
            },
          },
        },
      ],
    },
    devOptions: {
      enabled: true,
      type: 'module',
    },
  },
})
```

### Security Module

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['nuxt-security'],
  
  security: {
    headers: {
      crossOriginEmbedderPolicy: false,
      contentSecurityPolicy: {
        'base-uri': ["'self'"],
        'default-src': ["'self'"],
        'connect-src': ["'self'", 'https:', 'wss:'],
        'font-src': ["'self'", 'https:', 'data:'],
        'form-action': ["'self'"],
        'frame-ancestors': ["'self'"],
        'frame-src': ["'self'", 'https:'],
        'img-src': ["'self'", 'https:', 'data:', 'blob:'],
        'manifest-src': ["'self'"],
        'media-src': ["'self'", 'https:'],
        'object-src': ["'none'"],
        'script-src': ["'self'", "'unsafe-inline'"],
        'style-src': ["'self'", "'unsafe-inline'"],
        'upgrade-insecure-requests': true,
      },
      permissionsPolicy: {
        camera: [],
        microphone: [],
        geolocation: [],
      },
    },
    rateLimiter: {
      tokensPerInterval: 150,
      interval: 300000, // 5 minutes
    },
  },
})
```

---

## Environment Variables

### .env.example Template

```bash
# ==============================================
# Application
# ==============================================
NODE_ENV=development
NUXT_PUBLIC_APP_NAME="My Application"
NUXT_PUBLIC_APP_VERSION="1.0.0"

# ==============================================
# API Configuration
# ==============================================
NUXT_PUBLIC_API_BASE_URL=http://localhost:3001/api
API_SECRET=your-secret-key-here

# ==============================================
# Authentication (if applicable)
# ==============================================
AUTH_SECRET=your-auth-secret-here
AUTH_ORIGIN=http://localhost:3000

# ==============================================
# Database (if applicable)
# ==============================================
DATABASE_URL=postgresql://user:password@localhost:5432/myapp

# ==============================================
# External Services
# ==============================================
ANALYTICS_ID=
SENTRY_DSN=

# ==============================================
# Feature Flags
# ==============================================
NUXT_PUBLIC_ENABLE_ANALYTICS=false
NUXT_PUBLIC_ENABLE_PWA=false

# ==============================================
# Deployment
# ==============================================
NUXT_APP_BASE_URL=/
```

### Accessing Environment Variables

```typescript
// In nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // Server-only (not exposed to client)
    apiSecret: process.env.API_SECRET,
    
    // Public (exposed to client)
    public: {
      apiBaseUrl: process.env.NUXT_PUBLIC_API_BASE_URL,
      appName: process.env.NUXT_PUBLIC_APP_NAME,
    },
  },
})

// In composables/components
const config = useRuntimeConfig()
console.log(config.public.apiBaseUrl) // Available on client and server
// config.apiSecret // Only available on server
```

---

## TypeScript Configuration

### tsconfig.json

```json
{
  "extends": "./.nuxt/tsconfig.json",
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "forceConsistentCasingInFileNames": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": false,
    "skipLibCheck": true
  },
  "include": [
    "app/**/*",
    "server/**/*",
    "shared/**/*",
    "tests/**/*"
  ],
  "exclude": [
    "node_modules",
    ".nuxt",
    ".output"
  ]
}
```

---

## Deployment Presets

### Nitro Presets by Platform

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    // Choose based on deployment target:
    
    // Node.js server
    preset: 'node-server',
    
    // Static hosting (SPA)
    // preset: 'static',
    
    // Vercel
    // preset: 'vercel',
    
    // Netlify
    // preset: 'netlify',
    
    // Cloudflare Pages
    // preset: 'cloudflare-pages',
    
    // AWS Lambda
    // preset: 'aws-lambda',
    
    // Docker
    // preset: 'node-server',
    
    // Deno
    // preset: 'deno-server',
    
    // Bun
    // preset: 'bun',
  },
})
```

---

## Performance Configuration

### Build Optimization

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Vite build optimization
  vite: {
    build: {
      target: 'esnext',
      minify: 'esbuild',
      cssMinify: true,
      rollupOptions: {
        output: {
          manualChunks: {
            'vendor': ['vue', 'vue-router', 'pinia'],
            'utils': ['@vueuse/core'],
          },
        },
      },
    },
  },
  
  // Nuxt optimization
  experimental: {
    payloadExtraction: true,
    treeshakeClientOnly: true,
  },
  
  // Source maps (disable in production)
  sourcemap: {
    server: false,
    client: false,
  },
})
```

### Route Rules for Caching

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // Static pages - prerender at build time
    '/': { prerender: true },
    '/about': { prerender: true },
    
    // ISR - regenerate in background
    '/blog/**': { swr: 3600 },
    '/products/**': { isr: 600 },
    
    // CDN caching headers
    '/_nuxt/**': {
      headers: {
        'Cache-Control': 'public, max-age=31536000, immutable',
      },
    },
    
    // API caching
    '/api/public/**': {
      cache: {
        maxAge: 60,
        staleMaxAge: 300,
      },
    },
  },
})
```

---

## Best Practices

### 1. Environment-Aware Configuration

```typescript
// nuxt.config.ts
const isDevelopment = process.env.NODE_ENV === 'development'
const isProduction = process.env.NODE_ENV === 'production'

export default defineNuxtConfig({
  devtools: { enabled: isDevelopment },
  sourcemap: isDevelopment,
  
  vite: {
    build: {
      sourcemap: isDevelopment,
    },
  },
})
```

### 2. Conditional Module Loading

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@vueuse/nuxt',
    '@pinia/nuxt',
    // Only load security module in production
    ...(process.env.NODE_ENV === 'production' ? ['nuxt-security'] : []),
  ],
})
```

### 3. Separate Configuration Files

```typescript
// config/app.ts
export const appConfig = {
  name: 'My App',
  version: '1.0.0',
}

// config/security.ts
export const securityConfig = {
  headers: {
    // ...
  },
}

// nuxt.config.ts
import { appConfig } from './config/app'
import { securityConfig } from './config/security'

export default defineNuxtConfig({
  runtimeConfig: {
    public: {
      appName: appConfig.name,
    },
  },
  security: securityConfig,
})
```

---

## Next Steps

Continue to [Component Architecture](./04-component-architecture.md) to learn about organizing and building Vue components in a Nuxt application.