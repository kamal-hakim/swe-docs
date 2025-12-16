# Nuxt Configuration

## Overview

This document provides comprehensive Nuxt configuration patterns for both **SPA (Single Page Application)** and **SSR (Server-Side Rendering)** deployments with **Vuetify 3** integration. Each configuration section includes explanations and best practices.

## Base Configuration

### Complete nuxt.config.ts Template

```typescript
// nuxt.config.ts
import vuetify, { transformAssetUrls } from "vite-plugin-vuetify";

// Environment detection
const isDevelopment = process.env.NODE_ENV === "development";
const isProduction = process.env.NODE_ENV === "production";

export default defineNuxtConfig({
  // ==========================================
  // Core Configuration
  // ==========================================

  // Nuxt 4 compatibility date
  compatibilityDate: "2025-01-01",

  // Enable/disable SSR (true = SSR, false = SPA)
  ssr: true,

  // Source directory for Nuxt app
  srcDir: "app/",

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
  // Build Configuration for Vuetify
  // ==========================================
  build: {
    transpile: ["vuetify"],
  },

  // ==========================================
  // Modules
  // ==========================================
  modules: [
    // Core modules
    "@vueuse/nuxt",
    "@pinia/nuxt",
    "@nuxtjs/i18n",
    "@nuxt/test-utils/module",

    // Vuetify module (inline configuration)
    (_options, nuxt) => {
      nuxt.hooks.hook("vite:extendConfig", (config) => {
        config.plugins?.push(vuetify({ autoImport: true }));
      });
    },

    // Optional modules (uncomment as needed)
    // '@vite-pwa/nuxt',
    // 'nuxt-security',
  ],

  // ==========================================
  // Vite Configuration for Vuetify
  // ==========================================
  vite: {
    vue: {
      template: {
        transformAssetUrls,
      },
    },
    build: {
      target: "esnext",
      sourcemap: isDevelopment,
    },
    optimizeDeps: {
      include: ["vue", "vue-router", "pinia", "@vueuse/core"],
    },
    css: {
      devSourcemap: true,
      preprocessorOptions: {
        scss: {
          additionalData: '@use "~/assets/styles/vuetify/settings.scss" as *;',
        },
      },
    },
  },

  // ==========================================
  // Auto-imports Configuration
  // ==========================================
  imports: {
    dirs: ["composables/**", "utils/**", "stores/**"],
  },

  // ==========================================
  // Components Configuration
  // ==========================================
  components: [
    {
      path: "~/components",
      pathPrefix: false,
      extensions: [".vue"],
    },
  ],

  // ==========================================
  // CSS Configuration
  // ==========================================
  css: [
    "vuetify/styles",
    "@mdi/font/css/materialdesignicons.css",
    "~/assets/styles/main.scss",
  ],

  // ==========================================
  // Runtime Configuration
  // ==========================================
  runtimeConfig: {
    // Private keys (server-only)
    apiSecret: process.env.API_SECRET || "",

    // Public keys (exposed to client)
    public: {
      apiBaseUrl: process.env.NUXT_PUBLIC_API_BASE_URL || "/api",
      appName: process.env.NUXT_PUBLIC_APP_NAME || "My App",
      appVersion: process.env.npm_package_version || "0.0.0",
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
      viewport: "width=device-width, initial-scale=1, viewport-fit=cover",
      charset: "utf-8",
      htmlAttrs: {
        lang: "en",
      },
      link: [
        { rel: "icon", href: "/favicon.ico", sizes: "any" },
        { rel: "icon", type: "image/svg+xml", href: "/icon.svg" },
        { rel: "apple-touch-icon", href: "/apple-touch-icon.png" },
      ],
      meta: [
        { name: "theme-color", content: "#1867C0" }, // Vuetify primary color
        { name: "format-detection", content: "telephone=no" },
        { property: "og:type", content: "website" },
        { name: "twitter:card", content: "summary_large_image" },
      ],
    },

    // Page transition
    pageTransition: { name: "page", mode: "out-in" },
    layoutTransition: { name: "layout", mode: "out-in" },
  },

  // ==========================================
  // Route Rules
  // ==========================================
  routeRules: {
    // Static generation
    "/": { prerender: true },
    "/about": { prerender: true },

    // SPA routes (client-side only)
    "/dashboard/**": { ssr: false },
    "/admin/**": { ssr: false },

    // API caching
    "/api/**": {
      cors: true,
      headers: { "Access-Control-Allow-Methods": "GET,POST,PUT,DELETE" },
    },

    // Static asset caching
    "/_nuxt/**": {
      headers: { "Cache-Control": "public, max-age=31536000, immutable" },
    },
  },

  // ==========================================
  // Nitro Server Configuration
  // ==========================================
  nitro: {
    preset: "node-server", // Change based on deployment target
    compressPublicAssets: true,
    minify: true,
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
  // Module Configurations
  // ==========================================

  // Pinia
  pinia: {
    storesDirs: ["./app/stores/**"],
  },

  // i18n
  i18n: {
    locales: [
      { code: "en", name: "English", file: "en.json" },
      { code: "es", name: "Español", file: "es.json" },
      { code: "fr", name: "Français", file: "fr.json" },
    ],
    defaultLocale: "en",
    strategy: "prefix_except_default",
    lazy: true,
    langDir: "../locales",
    detectBrowserLanguage: {
      useCookie: true,
      cookieKey: "i18n_redirected",
      redirectOn: "root",
    },
  },
});
```

---

## Vuetify Plugin Configuration

### Basic Vuetify Plugin

```typescript
// app/plugins/vuetify.ts
import { createVuetify } from "vuetify";
import * as components from "vuetify/components";
import * as directives from "vuetify/directives";
import { aliases, mdi } from "vuetify/iconsets/mdi";

export default defineNuxtPlugin((nuxtApp) => {
  const vuetify = createVuetify({
    ssr: true,
    components,
    directives,
    icons: {
      defaultSet: "mdi",
      aliases,
      sets: { mdi },
    },
    theme: {
      defaultTheme: "light",
      themes: {
        light: {
          dark: false,
          colors: {
            primary: "#1867C0",
            secondary: "#5CBBF6",
            accent: "#4CAF50",
            error: "#FF5252",
            info: "#2196F3",
            success: "#4CAF50",
            warning: "#FFC107",
            background: "#FFFFFF",
            surface: "#FFFFFF",
          },
        },
        dark: {
          dark: true,
          colors: {
            primary: "#2196F3",
            secondary: "#424242",
            accent: "#FF4081",
            error: "#FF5252",
            info: "#2196F3",
            success: "#4CAF50",
            warning: "#FFC107",
            background: "#121212",
            surface: "#212121",
          },
        },
      },
    },
    defaults: {
      VBtn: {
        variant: "flat",
        rounded: "lg",
      },
      VCard: {
        rounded: "lg",
        elevation: 2,
      },
      VTextField: {
        variant: "outlined",
        density: "comfortable",
      },
      VSelect: {
        variant: "outlined",
        density: "comfortable",
      },
    },
  });

  nuxtApp.vueApp.use(vuetify);
});
```

### Advanced Vuetify Plugin with i18n Integration

```typescript
// app/plugins/vuetify.ts
import { createVuetify, type ThemeDefinition } from "vuetify";
import * as components from "vuetify/components";
import * as directives from "vuetify/directives";
import { aliases, mdi } from "vuetify/iconsets/mdi";
import { createVueI18nAdapter } from "vuetify/locale/adapters/vue-i18n";
import { useI18n } from "vue-i18n";

// Custom light theme
const lightTheme: ThemeDefinition = {
  dark: false,
  colors: {
    primary: "#1867C0",
    "primary-darken-1": "#1558A8",
    secondary: "#5CBBF6",
    "secondary-darken-1": "#4AA8E3",
    accent: "#4CAF50",
    error: "#FF5252",
    info: "#2196F3",
    success: "#4CAF50",
    warning: "#FFC107",
    background: "#F5F5F5",
    surface: "#FFFFFF",
    "surface-bright": "#FFFFFF",
    "surface-light": "#EEEEEE",
    "surface-variant": "#424242",
    "on-surface-variant": "#EEEEEE",
  },
  variables: {
    "border-color": "#E0E0E0",
    "border-opacity": 0.12,
    "high-emphasis-opacity": 0.87,
    "medium-emphasis-opacity": 0.6,
    "disabled-opacity": 0.38,
    "idle-opacity": 0.04,
    "hover-opacity": 0.04,
    "focus-opacity": 0.12,
    "selected-opacity": 0.08,
    "activated-opacity": 0.12,
    "pressed-opacity": 0.12,
    "dragged-opacity": 0.08,
    "theme-kbd": "#212529",
    "theme-on-kbd": "#FFFFFF",
    "theme-code": "#F5F5F5",
    "theme-on-code": "#000000",
  },
};

// Custom dark theme
const darkTheme: ThemeDefinition = {
  dark: true,
  colors: {
    primary: "#2196F3",
    "primary-darken-1": "#1976D2",
    secondary: "#424242",
    "secondary-darken-1": "#212121",
    accent: "#FF4081",
    error: "#FF5252",
    info: "#2196F3",
    success: "#4CAF50",
    warning: "#FFC107",
    background: "#121212",
    surface: "#212121",
    "surface-bright": "#CCBFD6",
    "surface-light": "#424242",
    "surface-variant": "#A3A3A3",
    "on-surface-variant": "#424242",
  },
  variables: {
    "border-color": "#FFFFFF",
    "border-opacity": 0.12,
    "high-emphasis-opacity": 1,
    "medium-emphasis-opacity": 0.7,
    "disabled-opacity": 0.5,
    "idle-opacity": 0.1,
    "hover-opacity": 0.04,
    "focus-opacity": 0.12,
    "selected-opacity": 0.08,
    "activated-opacity": 0.12,
    "pressed-opacity": 0.16,
    "dragged-opacity": 0.08,
    "theme-kbd": "#212529",
    "theme-on-kbd": "#FFFFFF",
    "theme-code": "#343434",
    "theme-on-code": "#CCCCCC",
  },
};

export default defineNuxtPlugin((nuxtApp) => {
  const i18n = nuxtApp.$i18n;

  const vuetify = createVuetify({
    ssr: true,
    components,
    directives,
    icons: {
      defaultSet: "mdi",
      aliases,
      sets: { mdi },
    },
    locale: {
      adapter: createVueI18nAdapter({ i18n, useI18n }),
    },
    theme: {
      defaultTheme: "light",
      themes: {
        light: lightTheme,
        dark: darkTheme,
      },
    },
    defaults: {
      global: {
        ripple: true,
      },
      VAppBar: {
        elevation: 1,
      },
      VBtn: {
        variant: "flat",
        rounded: "lg",
      },
      VCard: {
        rounded: "lg",
        elevation: 2,
      },
      VTextField: {
        variant: "outlined",
        density: "comfortable",
        color: "primary",
      },
      VSelect: {
        variant: "outlined",
        density: "comfortable",
        color: "primary",
      },
      VTextarea: {
        variant: "outlined",
        density: "comfortable",
        color: "primary",
      },
      VCheckbox: {
        color: "primary",
      },
      VRadio: {
        color: "primary",
      },
      VSwitch: {
        color: "primary",
      },
      VTabs: {
        color: "primary",
      },
      VChip: {
        rounded: "lg",
      },
      VAlert: {
        rounded: "lg",
      },
      VDialog: {
        rounded: "lg",
      },
      VSnackbar: {
        rounded: "lg",
      },
      VDataTable: {
        hover: true,
      },
    },
  });

  nuxtApp.vueApp.use(vuetify);
});
```

---

## SPA-Specific Configuration

For Single Page Application deployments with client-side only rendering:

```typescript
// nuxt.config.ts - SPA Mode
import vuetify, { transformAssetUrls } from "vite-plugin-vuetify";

export default defineNuxtConfig({
  // Disable SSR for pure SPA
  ssr: false,

  // Compatibility date
  compatibilityDate: "2025-01-01",

  // Source directory
  srcDir: "app/",

  // Build configuration
  build: {
    transpile: ["vuetify"],
  },

  // Vuetify module
  modules: [
    "@vueuse/nuxt",
    "@pinia/nuxt",
    "@nuxtjs/i18n",
    (_options, nuxt) => {
      nuxt.hooks.hook("vite:extendConfig", (config) => {
        config.plugins?.push(vuetify({ autoImport: true }));
      });
    },
  ],

  // Vite configuration
  vite: {
    vue: {
      template: {
        transformAssetUrls,
      },
    },
  },

  // CSS
  css: [
    "vuetify/styles",
    "@mdi/font/css/materialdesignicons.css",
    "~/assets/styles/main.scss",
  ],

  // App configuration for SPA
  app: {
    // Base URL for deployment (e.g., GitHub Pages)
    baseURL: process.env.NUXT_APP_BASE_URL || "/",

    head: {
      viewport: "width=device-width, initial-scale=1",
      meta: [{ name: "theme-color", content: "#1867C0" }],
    },
  },

  // Route rules - all routes use client-side rendering
  routeRules: {
    "/**": { ssr: false },
  },

  // Nitro configuration for static hosting
  nitro: {
    preset: "static",
    prerender: {
      crawlLinks: true,
      routes: ["/"],
      failOnError: false,
    },
  },

  // Experimental features for SPA
  experimental: {
    payloadExtraction: true,
  },
});
```

---

## SSR-Specific Configuration

For Server-Side Rendering with Node.js server:

```typescript
// nuxt.config.ts - SSR Mode
import vuetify, { transformAssetUrls } from "vite-plugin-vuetify";

export default defineNuxtConfig({
  // Enable SSR
  ssr: true,

  // Compatibility date
  compatibilityDate: "2025-01-01",

  // Source directory
  srcDir: "app/",

  // Build configuration
  build: {
    transpile: ["vuetify"],
  },

  // Modules
  modules: [
    "@vueuse/nuxt",
    "@pinia/nuxt",
    "@nuxtjs/i18n",
    (_options, nuxt) => {
      nuxt.hooks.hook("vite:extendConfig", (config) => {
        config.plugins?.push(
          vuetify({
            autoImport: true,
            styles: { configFile: "app/assets/styles/vuetify/settings.scss" },
          })
        );
      });
    },
  ],

  // Vite configuration
  vite: {
    vue: {
      template: {
        transformAssetUrls,
      },
    },
    ssr: {
      noExternal: ["vuetify"],
    },
  },

  // CSS
  css: [
    "vuetify/styles",
    "@mdi/font/css/materialdesignicons.css",
    "~/assets/styles/main.scss",
  ],

  // Route rules for hybrid rendering
  routeRules: {
    // Pre-render static pages
    "/": { prerender: true },
    "/about": { prerender: true },
    "/blog/**": { prerender: true },

    // ISR for dynamic content
    "/products/**": { swr: 3600 },

    // Client-only for authenticated routes
    "/dashboard/**": { ssr: false },
    "/account/**": { ssr: false },

    // API routes
    "/api/**": { cors: true },
  },

  // Nitro configuration for SSR
  nitro: {
    preset: "node-server",
    compressPublicAssets: true,
  },

  // Experimental SSR features
  experimental: {
    payloadExtraction: true,
    renderJsonPayloads: true,
    componentIslands: true,
  },
});
```

---

## Vuetify SASS Configuration

### SASS Variables Override

```scss
// app/assets/styles/vuetify/settings.scss

// ==========================================
// Vuetify SASS Variables
// ==========================================

// Enable CSS variables
$vuetify-enable-css-variables: true;

// Border radius
$border-radius-root: 8px;

// Typography
$body-font-family: "Inter", "Roboto", sans-serif;
$heading-font-family: "Inter", "Roboto", sans-serif;

// Spacing
$spacer: 4px;

// Grid breakpoints
$grid-breakpoints: (
  "xs": 0,
  "sm": 600px,
  "md": 960px,
  "lg": 1280px,
  "xl": 1920px,
  "xxl": 2560px,
);

// Container max widths
$container-max-widths: (
  "sm": 540px,
  "md": 720px,
  "lg": 960px,
  "xl": 1140px,
  "xxl": 1320px,
);

// Elevation
$shadow-key-umbra-opacity: 0.2;
$shadow-key-penumbra-opacity: 0.14;
$shadow-key-ambient-opacity: 0.12;

// Component-specific variables
$btn-border-radius: 8px;
$btn-font-weight: 500;
$btn-letter-spacing: 0.0892857143em;
$btn-text-transform: none;

$card-border-radius: 12px;

$text-field-border-radius: 8px;

$chip-border-radius: 16px;

$alert-border-radius: 8px;

$dialog-border-radius: 12px;

$snackbar-border-radius: 8px;
```

### Main Stylesheet

```scss
// app/assets/styles/main.scss

// ==========================================
// Global Styles
// ==========================================

// Variables
@use "variables" as *;

// Mixins
@use "mixins" as *;

// Base styles
html {
  font-family: $font-family-base;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  margin: 0;
  padding: 0;
}

// ==========================================
// Vuetify Overrides
// ==========================================

// App bar customization
.v-app-bar {
  &.v-toolbar {
    border-bottom: 1px solid rgba(var(--v-border-color), var(--v-border-opacity));
  }
}

// Navigation drawer customization
.v-navigation-drawer {
  border-right: 1px solid rgba(var(--v-border-color), var(--v-border-opacity)) !important;
}

// Card hover effect
.v-card {
  transition: box-shadow 0.2s ease-in-out, transform 0.2s ease-in-out;

  &.v-card--hover:hover {
    transform: translateY(-2px);
  }
}

// Button customization
.v-btn {
  text-transform: none;
  letter-spacing: normal;
}

// Form field customization
.v-field {
  &--focused {
    .v-field__outline {
      --v-field-border-width: 2px;
    }
  }
}

// Data table customization
.v-data-table {
  .v-data-table__tr {
    &:hover {
      background-color: rgba(var(--v-theme-primary), 0.04);
    }
  }
}

// ==========================================
// Page Transitions
// ==========================================

.page-enter-active,
.page-leave-active {
  transition: opacity 0.2s ease-in-out;
}

.page-enter-from,
.page-leave-to {
  opacity: 0;
}

// ==========================================
// Utility Classes
// ==========================================

.text-truncate-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.text-truncate-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

// Scrollbar styling
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background-color: rgba(var(--v-border-color), 0.5);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background-color: rgba(var(--v-border-color), 0.7);
}
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
# Vuetify Theme
# ==============================================
NUXT_PUBLIC_DEFAULT_THEME=light

# ==============================================
# i18n
# ==============================================
NUXT_PUBLIC_DEFAULT_LOCALE=en

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
    "skipLibCheck": true,
    "types": ["vuetify"]
  },
  "include": ["app/**/*", "server/**/*", "shared/**/*", "tests/**/*"],
  "exclude": ["node_modules", ".nuxt", ".output"]
}
```

### Vuetify Type Augmentation

```typescript
// app/types/vuetify.d.ts
import "vuetify";

declare module "vuetify" {
  export interface ThemeDefinition {
    dark: boolean;
    colors: {
      primary: string;
      secondary: string;
      accent: string;
      error: string;
      info: string;
      success: string;
      warning: string;
      background: string;
      surface: string;
      [key: string]: string;
    };
  }
}

// Global component type augmentation
declare module "@vue/runtime-core" {
  export interface GlobalComponents {
    // Add custom global components here
  }
}
```

---

## Package.json Scripts

```json
{
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    "postinstall": "nuxt prepare",

    "lint": "eslint .",
    "lint:fix": "eslint . --fix",

    "typecheck": "nuxt typecheck",

    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui",

    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:headed": "playwright test --headed",
    "test:e2e:debug": "playwright test --debug",

    "prepare": "simple-git-hooks"
  }
}
```

---

## Next Steps

Continue to [Component Architecture](./04-component-architecture.md) to learn about organizing and building Vue components with Vuetify in a Nuxt application.
