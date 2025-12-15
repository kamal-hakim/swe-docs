# Styling Architecture

## Overview

This document covers the CSS architecture for Nuxt applications using **UnoCSS** as the primary styling framework. It includes theming strategies, CSS organization, component styling patterns, and best practices for maintainable, performant styles.

## Technology Stack

| Technology | Purpose |
|------------|---------|
| **UnoCSS** | Atomic CSS framework |
| **CSS Custom Properties** | Theming and dynamic values |
| **PostCSS** | CSS processing (nesting, autoprefixer) |
| **@nuxtjs/color-mode** | Dark/light mode support |

---

## UnoCSS Setup

### Installation

```bash
pnpm add -D @unocss/nuxt
```

### Nuxt Configuration

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@unocss/nuxt'],
  
  css: [
    '@unocss/reset/tailwind.css',
    '~/styles/variables.css',
    '~/styles/base.css',
    '~/styles/utilities.css',
  ],
})
```

### UnoCSS Configuration

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
  // ==========================================
  // Shortcuts (Reusable utility combinations)
  // ==========================================
  shortcuts: {
    // Layout utilities
    'flex-center': 'flex items-center justify-center',
    'flex-between': 'flex items-center justify-between',
    'flex-col-center': 'flex flex-col items-center justify-center',
    'absolute-center': 'absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2',
    'absolute-fill': 'absolute inset-0',
    
    // Container
    'container-base': 'w-full max-w-7xl mx-auto px-4 sm:px-6 lg:px-8',
    
    // Buttons
    'btn': 'inline-flex items-center justify-center px-4 py-2 rounded-lg font-medium transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed focus:outline-none focus:ring-2 focus:ring-offset-2',
    'btn-primary': 'btn bg-primary-500 text-white hover:bg-primary-600 focus:ring-primary-500',
    'btn-secondary': 'btn bg-gray-200 text-gray-800 hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600',
    'btn-outline': 'btn border-2 border-primary-500 text-primary-500 hover:bg-primary-500 hover:text-white',
    'btn-ghost': 'btn hover:bg-gray-100 dark:hover:bg-gray-800',
    'btn-danger': 'btn bg-red-500 text-white hover:bg-red-600 focus:ring-red-500',
    'btn-sm': 'px-3 py-1.5 text-sm',
    'btn-lg': 'px-6 py-3 text-lg',
    'btn-icon': 'p-2 rounded-full',
    
    // Form elements
    'input-base': 'w-full px-3 py-2 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 rounded-lg transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-primary-500 focus:border-transparent disabled:opacity-50 disabled:cursor-not-allowed',
    'input-error': 'border-red-500 focus:ring-red-500',
    'label-base': 'block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1',
    'form-group': 'space-y-1 mb-4',
    
    // Cards
    'card': 'bg-white dark:bg-gray-800 rounded-lg shadow-md',
    'card-header': 'px-4 py-3 border-b border-gray-200 dark:border-gray-700',
    'card-body': 'p-4',
    'card-footer': 'px-4 py-3 border-t border-gray-200 dark:border-gray-700',
    
    // Text
    'text-heading': 'text-gray-900 dark:text-gray-100 font-bold',
    'text-body': 'text-gray-700 dark:text-gray-300',
    'text-muted': 'text-gray-500 dark:text-gray-400',
    'text-link': 'text-primary-500 hover:text-primary-600 hover:underline cursor-pointer',
    
    // Borders
    'border-base': 'border border-gray-200 dark:border-gray-700',
    'divide-base': 'divide-y divide-gray-200 dark:divide-gray-700',
    
    // Backgrounds
    'bg-base': 'bg-white dark:bg-gray-900',
    'bg-muted': 'bg-gray-50 dark:bg-gray-800',
    'bg-overlay': 'bg-black/50',
    
    // Transitions
    'transition-base': 'transition-all duration-200 ease-in-out',
    'transition-fast': 'transition-all duration-100 ease-in-out',
    'transition-slow': 'transition-all duration-300 ease-in-out',
    
    // Scrollbar
    'scrollbar-hide': 'scrollbar-width-none [-ms-overflow-style:none] [&::-webkit-scrollbar]:hidden',
    'scrollbar-thin': 'scrollbar-width-thin scrollbar-color-gray-400 scrollbar-track-transparent',
  },

  // ==========================================
  // Theme Configuration
  // ==========================================
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
        950: '#082f49',
      },
      // Add more custom colors as needed
    },
    fontFamily: {
      sans: ['Inter', 'system-ui', 'sans-serif'],
      mono: ['Fira Code', 'monospace'],
    },
    breakpoints: {
      xs: '475px',
      sm: '640px',
      md: '768px',
      lg: '1024px',
      xl: '1280px',
      '2xl': '1536px',
    },
  },

  // ==========================================
  // Presets
  // ==========================================
  presets: [
    presetUno(),
    presetAttributify(),
    presetIcons({
      scale: 1.2,
      extraProperties: {
        'display': 'inline-block',
        'vertical-align': 'middle',
      },
    }),
    presetTypography({
      cssExtend: {
        'code': {
          'background-color': 'var(--color-code-bg)',
          'padding': '0.125rem 0.25rem',
          'border-radius': '0.25rem',
        },
      },
    }),
    presetWebFonts({
      provider: 'google',
      fonts: {
        sans: 'Inter:400,500,600,700',
        mono: 'Fira Code:400,500',
      },
    }),
  ],

  // ==========================================
  // Transformers
  // ==========================================
  transformers: [
    transformerDirectives(),     // @apply, @screen, etc.
    transformerVariantGroup(),   // hover:(bg-gray-100 text-gray-900)
  ],

  // ==========================================
  // Safelist (Include dynamic classes)
  // ==========================================
  safelist: [
    // Grid columns
    ...Array.from({ length: 12 }, (_, i) => `col-span-${i + 1}`),
    ...Array.from({ length: 12 }, (_, i) => `grid-cols-${i + 1}`),
    
    // Colors that might be dynamic
    ...['primary', 'secondary', 'success', 'warning', 'error', 'info'].flatMap(color => [
      `text-${color}-500`,
      `bg-${color}-500`,
      `border-${color}-500`,
    ]),
  ],

  // ==========================================
  // Content Sources
  // ==========================================
  content: {
    pipeline: {
      include: [
        /\.(vue|svelte|[jt]sx|mdx?|astro|elm|php|phtml|html)($|\?)/,
        'app/**/*.{vue,ts}',
        'components/**/*.{vue,ts}',
      ],
    },
  },
})
```

---

## CSS File Structure

```
app/styles/
├── variables.css          # CSS custom properties
├── base.css               # Base/reset styles
├── utilities.css          # Custom utility classes
├── typography.css         # Typography styles
├── animations.css         # Animation definitions
└── components/            # Component-specific styles (if needed)
    ├── buttons.css
    └── forms.css
```

### variables.css

```css
/* app/styles/variables.css */

:root {
  /* ==========================================
   * Color Palette
   * ========================================== */
  
  /* Primary */
  --color-primary-50: #f0f9ff;
  --color-primary-100: #e0f2fe;
  --color-primary-200: #bae6fd;
  --color-primary-300: #7dd3fc;
  --color-primary-400: #38bdf8;
  --color-primary-500: #0ea5e9;
  --color-primary-600: #0284c7;
  --color-primary-700: #0369a1;
  --color-primary-800: #075985;
  --color-primary-900: #0c4a6e;

  /* Semantic colors */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-error: #ef4444;
  --color-info: #3b82f6;

  /* ==========================================
   * Light Theme
   * ========================================== */
  --color-bg-base: #ffffff;
  --color-bg-muted: #f9fafb;
  --color-bg-elevated: #ffffff;
  --color-bg-overlay: rgba(0, 0, 0, 0.5);
  
  --color-text-base: #111827;
  --color-text-muted: #6b7280;
  --color-text-inverted: #ffffff;
  
  --color-border-base: #e5e7eb;
  --color-border-muted: #f3f4f6;
  
  --color-code-bg: #f3f4f6;
  --color-code-text: #1f2937;

  /* ==========================================
   * Layout
   * ========================================== */
  --header-height: 64px;
  --sidebar-width: 280px;
  --sidebar-collapsed-width: 64px;
  --container-max-width: 1280px;

  /* ==========================================
   * Typography
   * ========================================== */
  --font-family-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-family-mono: 'Fira Code', monospace;
  
  --font-size-xs: 0.75rem;    /* 12px */
  --font-size-sm: 0.875rem;   /* 14px */
  --font-size-base: 1rem;     /* 16px */
  --font-size-lg: 1.125rem;   /* 18px */
  --font-size-xl: 1.25rem;    /* 20px */
  --font-size-2xl: 1.5rem;    /* 24px */
  --font-size-3xl: 1.875rem;  /* 30px */
  --font-size-4xl: 2.25rem;   /* 36px */

  /* ==========================================
   * Spacing
   * ========================================== */
  --spacing-unit: 0.25rem;    /* 4px */

  /* ==========================================
   * Borders
   * ========================================== */
  --radius-sm: 0.25rem;
  --radius-base: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;

  /* ==========================================
   * Shadows
   * ========================================== */
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-base: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px -1px rgba(0, 0, 0, 0.1);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);

  /* ==========================================
   * Transitions
   * ========================================== */
  --transition-fast: 100ms ease-in-out;
  --transition-base: 200ms ease-in-out;
  --transition-slow: 300ms ease-in-out;

  /* ==========================================
   * Z-Index
   * ========================================== */
  --z-dropdown: 1000;
  --z-sticky: 1100;
  --z-fixed: 1200;
  --z-modal-backdrop: 1300;
  --z-modal: 1400;
  --z-popover: 1500;
  --z-tooltip: 1600;
  --z-toast: 1700;
}

/* ==========================================
 * Dark Theme
 * ========================================== */
.dark {
  --color-bg-base: #111827;
  --color-bg-muted: #1f2937;
  --color-bg-elevated: #1f2937;
  --color-bg-overlay: rgba(0, 0, 0, 0.7);
  
  --color-text-base: #f9fafb;
  --color-text-muted: #9ca3af;
  --color-text-inverted: #111827;
  
  --color-border-base: #374151;
  --color-border-muted: #4b5563;
  
  --color-code-bg: #1f2937;
  --color-code-text: #e5e7eb;
  
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.3);
  --shadow-base: 0 1px 3px 0 rgba(0, 0, 0, 0.4), 0 1px 2px -1px rgba(0, 0, 0, 0.4);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.4), 0 2px 4px -2px rgba(0, 0, 0, 0.4);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.4), 0 4px 6px -4px rgba(0, 0, 0, 0.4);
}
```

### base.css

```css
/* app/styles/base.css */

/* ==========================================
 * Reset & Base Styles
 * ========================================== */

*,
*::before,
*::after {
  box-sizing: border-box;
}

html {
  font-family: var(--font-family-sans);
  font-size: var(--font-size-base);
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
}

body {
  margin: 0;
  padding: 0;
  background-color: var(--color-bg-base);
  color: var(--color-text-base);
  min-height: 100vh;
  overflow-x: hidden;
}

/* ==========================================
 * Selection
 * ========================================== */

::selection {
  background-color: var(--color-primary-200);
  color: var(--color-primary-900);
}

.dark ::selection {
  background-color: var(--color-primary-700);
  color: var(--color-primary-100);
}

/* ==========================================
 * Focus Styles
 * ========================================== */

:focus-visible {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
}

:focus:not(:focus-visible) {
  outline: none;
}

/* ==========================================
 * Links
 * ========================================== */

a {
  color: inherit;
  text-decoration: none;
}

/* ==========================================
 * Images
 * ========================================== */

img,
video {
  max-width: 100%;
  height: auto;
}

/* ==========================================
 * Code
 * ========================================== */

code,
pre {
  font-family: var(--font-family-mono);
}

code {
  background-color: var(--color-code-bg);
  color: var(--color-code-text);
  padding: 0.125rem 0.25rem;
  border-radius: var(--radius-sm);
  font-size: 0.875em;
}

pre {
  background-color: var(--color-code-bg);
  padding: 1rem;
  border-radius: var(--radius-base);
  overflow-x: auto;
}

pre code {
  background: none;
  padding: 0;
}

/* ==========================================
 * Scrollbar Styling
 * ========================================== */

::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background-color: var(--color-border-base);
  border-radius: var(--radius-full);
}

::-webkit-scrollbar-thumb:hover {
  background-color: var(--color-text-muted);
}

/* Firefox */
* {
  scrollbar-width: thin;
  scrollbar-color: var(--color-border-base) transparent;
}

/* ==========================================
 * Form Elements
 * ========================================== */

input,
textarea,
select,
button {
  font-family: inherit;
  font-size: inherit;
}

button {
  cursor: pointer;
  border: none;
  background: none;
}

/* ==========================================
 * Accessibility
 * ========================================== */

/* Visually hidden but accessible */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* Skip link */
.skip-link {
  position: absolute;
  left: -9999px;
  z-index: 9999;
  padding: 1rem;
  background: var(--color-bg-base);
  color: var(--color-text-base);
  text-decoration: underline;
}

.skip-link:focus {
  left: 0;
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### animations.css

```css
/* app/styles/animations.css */

/* ==========================================
 * Keyframes
 * ========================================== */

@keyframes fade-in {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes fade-out {
  from {
    opacity: 1;
  }
  to {
    opacity: 0;
  }
}

@keyframes slide-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slide-down {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slide-in-right {
  from {
    opacity: 0;
    transform: translateX(20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes slide-in-left {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes scale-in {
  from {
    opacity: 0;
    transform: scale(0.95);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-25%);
  }
}

@keyframes shake {
  0%, 100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-5px);
  }
  75% {
    transform: translateX(5px);
  }
}

/* ==========================================
 * Animation Classes
 * ========================================== */

.animate-fade-in {
  animation: fade-in var(--transition-base) forwards;
}

.animate-slide-up {
  animation: slide-up var(--transition-base) forwards;
}

.animate-slide-down {
  animation: slide-down var(--transition-base) forwards;
}

.animate-scale-in {
  animation: scale-in var(--transition-base) forwards;
}

.animate-spin {
  animation: spin 1s linear infinite;
}

.animate-pulse {
  animation: pulse 2s ease-in-out infinite;
}

.animate-bounce {
  animation: bounce 1s ease-in-out infinite;
}

.animate-shake {
  animation: shake 0.5s ease-in-out;
}

/* ==========================================
 * Page Transitions
 * ========================================== */

.page-enter-active,
.page-leave-active {
  transition: opacity var(--transition-base), transform var(--transition-base);
}

.page-enter-from,
.page-leave-to {
  opacity: 0;
  transform: translateY(10px);
}

/* Slide transition */
.slide-enter-active,
.slide-leave-active {
  transition: transform var(--transition-slow);
}

.slide-enter-from {
  transform: translateX(100%);
}

.slide-leave-to {
  transform: translateX(-100%);
}

/* Fade transition */
.fade-enter-active,
.fade-leave-active {
  transition: opacity var(--transition-base);
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
```

---

## Component Styling Patterns

### Using UnoCSS Classes

```vue
<template>
  <!-- Utility-first approach -->
  <div class="flex items-center gap-4 p-4 bg-white dark:bg-gray-800 rounded-lg shadow-md">
    <img
      :src="user.avatar"
      :alt="user.name"
      class="w-12 h-12 rounded-full object-cover"
    />
    <div class="flex-1 min-w-0">
      <h3 class="text-lg font-semibold text-gray-900 dark:text-gray-100 truncate">
        {{ user.name }}
      </h3>
      <p class="text-sm text-gray-500 dark:text-gray-400">
        {{ user.email }}
      </p>
    </div>
    <button class="btn btn-primary">
      Follow
    </button>
  </div>
</template>
```

### Using Shortcuts

```vue
<template>
  <!-- Using predefined shortcuts -->
  <div class="card">
    <div class="card-header">
      <h2 class="text-heading">Card Title</h2>
    </div>
    <div class="card-body">
      <p class="text-body">Card content goes here.</p>
    </div>
    <div class="card-footer flex-between">
      <span class="text-muted">Footer text</span>
      <button class="btn btn-primary">Action</button>
    </div>
  </div>
</template>
```

### Dynamic Classes with Computed

```vue
<script setup lang="ts">
interface Props {
  variant?: 'primary' | 'secondary' | 'danger'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
  loading?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
  disabled: false,
  loading: false,
})

const buttonClass = computed(() => [
  'btn',
  {
    // Variants
    'btn-primary': props.variant === 'primary',
    'btn-secondary': props.variant === 'secondary',
    'btn-danger': props.variant === 'danger',
    
    // Sizes
    'btn-sm': props.size === 'sm',
    'btn-lg': props.size === 'lg',
    
    // States
    'opacity-50 cursor-not-allowed': props.disabled || props.loading,
  },
])
</script>

<template>
  <button
    :class="buttonClass"
    :disabled="disabled || loading"
  >
    <span v-if="loading" class="i-carbon-loading animate-spin mr-2" />
    <slot />
  </button>
</template>
```

### Scoped Styles with CSS Variables

```vue
<script setup lang="ts">
interface Props {
  color?: string
  size?: number
}

const props = withDefaults(defineProps<Props>(), {
  color: 'var(--color-primary-500)',
  size: 48,
})

const cssVars = computed(() => ({
  '--avatar-color': props.color,
  '--avatar-size': `${props.size}px`,
}))
</script>

<template>
  <div class="avatar" :style="cssVars">
    <slot />
  </div>
</template>

<style scoped>
.avatar {
  width: var(--avatar-size);
  height: var(--avatar-size);
  border-radius: 50%;
  background-color: var(--avatar-color);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: calc(var(--avatar-size) * 0.4);
  color: white;
  font-weight: 600;
}
</style>
```

---

## Dark Mode Implementation

### Color Mode Configuration

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/color-mode'],
  
  colorMode: {
    classSuffix: '',          // No suffix: 'dark' instead of 'dark-mode'
    preference: 'system',     // Default to system preference
    fallback: 'light',        // Fallback if system can't be detected
    storageKey: 'color-mode', // localStorage key
  },
})
```

### Using Color Mode

```vue
<script setup lang="ts">
const colorMode = useColorMode()

// Get current mode
const isDark = computed(() => colorMode.value === 'dark')

// Toggle mode
function toggleDarkMode() {
  colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark'
}

// Set specific mode
function setMode(mode: 'light' | 'dark' | 'system') {
  colorMode.preference = mode
}
</script>

<template>
  <button @click="toggleDarkMode" class="btn btn-ghost btn-icon">
    <span v-if="isDark" class="i-carbon-sun" />
    <span v-else class="i-carbon-moon" />
  </button>
</template>
```

### Color Mode Dropdown

```vue
<script setup lang="ts">
const colorMode = useColorMode()

const modes = [
  { value: 'system', label: 'System', icon: 'i-carbon-laptop' },
  { value: 'light', label: 'Light', icon: 'i-carbon-sun' },
  { value: 'dark', label: 'Dark', icon: 'i-carbon-moon' },
]
</script>

<template>
  <select
    v-model="colorMode.preference"
    class="input-base"
  >
    <option
      v-for="mode in modes"
      :key="mode.value"
      :value="mode.value"
    >
      {{ mode.label }}
    </option>
  </select>
</template>
```

---

## Icons with UnoCSS

### Using Iconify Icons

```vue
<template>
  <!-- Carbon icons -->
  <span class="i-carbon-home" />
  <span class="i-carbon-settings" />
  <span class="i-carbon-user" />
  
  <!-- Heroicons -->
  <span class="i-heroicons-home" />
  <span class="i-heroicons-cog-6-tooth" />
  
  <!-- Custom sizing -->
  <span class="i-carbon-home w-6 h-6" />
  <span class="i-carbon-home text-2xl" />
  
  <!-- Custom color -->
  <span class="i-carbon-home text-primary-500" />
  <span class="i-carbon-home text-red-500" />
  
  <!-- Interactive icons -->
  <button class="btn btn-icon hover:text-primary-500">
    <span class="i-carbon-favorite transition-colors" />
  </button>
</template>
```

### Icon Component

```vue
<!-- components/ui/UiIcon.vue -->
<script setup lang="ts">
interface Props {
  name: string
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl'
}

const props = withDefaults(defineProps<Props>(), {
  size: 'md',
})

const sizeClass = computed(() => ({
  xs: 'text-xs',
  sm: 'text-sm',
  md: 'text-base',
  lg: 'text-lg',
  xl: 'text-xl',
}[props.size]))
</script>

<template>
  <span :class="[name, sizeClass]" aria-hidden="true" />
</template>

<!-- Usage -->
<UiIcon name="i-carbon-home" size="lg" />
```

---

## Responsive Design

### Breakpoint Utilities

```vue
<template>
  <!-- Mobile first approach -->
  <div class="
    grid grid-cols-1
    sm:grid-cols-2
    md:grid-cols-3
    lg:grid-cols-4
    gap-4
  ">
    <div v-for="item in items" :key="item.id" class="card">
      {{ item.name }}
    </div>
  </div>

  <!-- Responsive visibility -->
  <nav class="hidden md:flex">Desktop nav</nav>
  <nav class="flex md:hidden">Mobile nav</nav>

  <!-- Responsive typography -->
  <h1 class="text-2xl md:text-3xl lg:text-4xl">
    Responsive Heading
  </h1>

  <!-- Responsive spacing -->
  <div class="p-4 md:p-6 lg:p-8">
    Content with responsive padding
  </div>
</template>
```

### Container Queries (UnoCSS)

```vue
<template>
  <div class="@container">
    <div class="@md:flex @md:gap-4">
      <div class="@md:w-1/3">Sidebar</div>
      <div class="@md:w-2/3">Content</div>
    </div>
  </div>
</template>
```

---

## Performance Best Practices

### 1. Avoid Dynamic Class Strings

```vue
<!-- ❌ Bad: Dynamic strings are not extractable -->
<template>
  <div :class="`bg-${color}-500`">Content</div>
</template>

<!-- ✅ Good: Use conditional objects -->
<template>
  <div :class="{
    'bg-red-500': color === 'red',
    'bg-blue-500': color === 'blue',
    'bg-green-500': color === 'green',
  }">Content</div>
</template>

<!-- ✅ Good: Use safelist for dynamic values -->
<!-- Add to unocss.config.ts safelist -->
```

### 2. Use Shortcuts for Repeated Patterns

```typescript
// unocss.config.ts
shortcuts: {
  // Instead of repeating this everywhere
  'card-base': 'bg-white dark:bg-gray-800 rounded-lg shadow-md p-4 border border-gray-200 dark:border-gray-700',
}
```

### 3. Minimize Scoped Styles

```vue
<!-- ✅ Good: Use utility classes -->
<template>
  <div class="flex items-center gap-4 p-4">
    <slot />
  </div>
</template>

<!-- 🆗 When needed: Scoped styles -->
<style scoped>
.complex-component {
  /* Only for truly complex cases */
}
</style>
```

### 4. Critical CSS

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  features: {
    inlineStyles: true, // Inline critical CSS
  },
})
```

---

## Accessibility in Styles

### Focus States

```css
/* Ensure visible focus states */
:focus-visible {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
}

/* Remove focus outline for mouse users */
:focus:not(:focus-visible) {
  outline: none;
}
```

### High Contrast Mode

```css
@media (prefers-contrast: high) {
  :root {
    --color-bg-base: #ffffff;
    --color-text-base: #000000;
    --color-border-base: #000000;
  }
  
  .dark {
    --color-bg-base: #000000;
    --color-text-base: #ffffff;
    --color-border-base: #ffffff;
  }
}
```

### Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Best Practices Summary

| Practice | Description |
|----------|-------------|
| **Utility-first** | Use UnoCSS utilities before custom CSS |
| **CSS Variables** | Use custom properties for theming |
| **Mobile-first** | Start with mobile styles, add breakpoints up |
| **Dark mode** | Design both modes from the start |
| **Shortcuts** | Define shortcuts for repeated patterns |
| **Performance** | Avoid dynamic class strings |
| **Accessibility** | Include focus states and reduced motion |
| **Consistency** | Use design tokens from variables |

---

## Next Steps

Continue to [Testing Strategy](./09-testing-strategy.md) to learn about testing configuration and patterns for Nuxt applications.