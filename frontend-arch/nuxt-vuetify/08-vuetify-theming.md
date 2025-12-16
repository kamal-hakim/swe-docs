# Vuetify Theming

## Overview

Vuetify 3 provides a powerful theming system based on Material Design 3 that supports multiple themes, dark mode, custom color palettes, and dynamic theme switching. This document covers theme configuration, customization patterns, and best practices for implementing a robust theming system in your Nuxt application.

## Theme Architecture

### Core Concepts

| Concept            | Description                                                |
| ------------------ | ---------------------------------------------------------- |
| **Theme**          | A collection of colors, typography, and component defaults |
| **Light/Dark**     | Built-in theme variants with automatic color adjustments   |
| **CSS Variables**  | Runtime theme values exposed as CSS custom properties      |
| **useTheme**       | Composable for programmatic theme access and manipulation  |
| **SASS Variables** | Build-time customization of Vuetify's internal styles      |

---

## Basic Theme Configuration

### Plugin Setup

```typescript
// plugins/vuetify.ts
import { createVuetify, type ThemeDefinition } from "vuetify";
import * as components from "vuetify/components";
import * as directives from "vuetify/directives";
import { aliases, mdi } from "vuetify/iconsets/mdi";

// Define light theme
const lightTheme: ThemeDefinition = {
  dark: false,
  colors: {
    background: "#FFFFFF",
    surface: "#FFFFFF",
    "surface-bright": "#FFFFFF",
    "surface-light": "#EEEEEE",
    "surface-variant": "#424242",
    "on-surface-variant": "#EEEEEE",
    primary: "#1867C0",
    "primary-darken-1": "#1F5592",
    secondary: "#48A9A6",
    "secondary-darken-1": "#018786",
    error: "#B00020",
    info: "#2196F3",
    success: "#4CAF50",
    warning: "#FB8C00",
  },
  variables: {
    "border-color": "#000000",
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

// Define dark theme
const darkTheme: ThemeDefinition = {
  dark: true,
  colors: {
    background: "#121212",
    surface: "#212121",
    "surface-bright": "#ccbfd6",
    "surface-light": "#424242",
    "surface-variant": "#a3a3a3",
    "on-surface-variant": "#424242",
    primary: "#2196F3",
    "primary-darken-1": "#1976D2",
    secondary: "#54B6B2",
    "secondary-darken-1": "#48A9A6",
    error: "#CF6679",
    info: "#2196F3",
    success: "#4CAF50",
    warning: "#FB8C00",
  },
  variables: {
    "border-color": "#FFFFFF",
    "border-opacity": 0.12,
    "high-emphasis-opacity": 0.87,
    "medium-emphasis-opacity": 0.6,
    "disabled-opacity": 0.38,
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

export default defineNuxtPlugin((app) => {
  const vuetify = createVuetify({
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
        light: lightTheme,
        dark: darkTheme,
      },
    },
  });

  app.vueApp.use(vuetify);
});
```

---

## Custom Color Palettes

### Brand Colors

```typescript
// config/theme.ts
import type { ThemeDefinition } from "vuetify";

// Brand color palette
export const brandColors = {
  // Primary brand color with shades
  primary: {
    base: "#6366F1",
    lighten5: "#EEF2FF",
    lighten4: "#E0E7FF",
    lighten3: "#C7D2FE",
    lighten2: "#A5B4FC",
    lighten1: "#818CF8",
    darken1: "#4F46E5",
    darken2: "#4338CA",
    darken3: "#3730A3",
    darken4: "#312E81",
  },
  // Secondary brand color
  secondary: {
    base: "#10B981",
    lighten5: "#ECFDF5",
    lighten4: "#D1FAE5",
    lighten3: "#A7F3D0",
    lighten2: "#6EE7B7",
    lighten1: "#34D399",
    darken1: "#059669",
    darken2: "#047857",
    darken3: "#065F46",
    darken4: "#064E3B",
  },
  // Accent color
  accent: {
    base: "#F59E0B",
    lighten1: "#FBBF24",
    darken1: "#D97706",
  },
};

// Light theme with brand colors
export const lightTheme: ThemeDefinition = {
  dark: false,
  colors: {
    background: "#FAFAFA",
    surface: "#FFFFFF",
    "surface-variant": "#F5F5F5",
    "on-surface-variant": "#424242",

    primary: brandColors.primary.base,
    "primary-lighten-1": brandColors.primary.lighten1,
    "primary-darken-1": brandColors.primary.darken1,
    "on-primary": "#FFFFFF",

    secondary: brandColors.secondary.base,
    "secondary-lighten-1": brandColors.secondary.lighten1,
    "secondary-darken-1": brandColors.secondary.darken1,
    "on-secondary": "#FFFFFF",

    accent: brandColors.accent.base,
    "on-accent": "#FFFFFF",

    error: "#EF4444",
    "on-error": "#FFFFFF",
    info: "#3B82F6",
    "on-info": "#FFFFFF",
    success: "#22C55E",
    "on-success": "#FFFFFF",
    warning: "#F59E0B",
    "on-warning": "#000000",
  },
};

// Dark theme with brand colors
export const darkTheme: ThemeDefinition = {
  dark: true,
  colors: {
    background: "#0F172A",
    surface: "#1E293B",
    "surface-variant": "#334155",
    "on-surface-variant": "#CBD5E1",

    primary: brandColors.primary.lighten1,
    "primary-lighten-1": brandColors.primary.lighten2,
    "primary-darken-1": brandColors.primary.base,
    "on-primary": "#000000",

    secondary: brandColors.secondary.lighten1,
    "secondary-lighten-1": brandColors.secondary.lighten2,
    "secondary-darken-1": brandColors.secondary.base,
    "on-secondary": "#000000",

    accent: brandColors.accent.lighten1,
    "on-accent": "#000000",

    error: "#F87171",
    "on-error": "#000000",
    info: "#60A5FA",
    "on-info": "#000000",
    success: "#4ADE80",
    "on-success": "#000000",
    warning: "#FBBF24",
    "on-warning": "#000000",
  },
};
```

### Multiple Theme Variants

```typescript
// plugins/vuetify.ts
import { lightTheme, darkTheme } from "~/config/theme";

// Additional theme variants
const highContrastLight: ThemeDefinition = {
  dark: false,
  colors: {
    background: "#FFFFFF",
    surface: "#FFFFFF",
    primary: "#0000EE",
    secondary: "#006400",
    error: "#CC0000",
    // High contrast colors
  },
};

const highContrastDark: ThemeDefinition = {
  dark: true,
  colors: {
    background: "#000000",
    surface: "#000000",
    primary: "#00FFFF",
    secondary: "#00FF00",
    error: "#FF6666",
    // High contrast colors
  },
};

export default defineNuxtPlugin((app) => {
  const vuetify = createVuetify({
    theme: {
      defaultTheme: "light",
      themes: {
        light: lightTheme,
        dark: darkTheme,
        "high-contrast-light": highContrastLight,
        "high-contrast-dark": highContrastDark,
      },
    },
  });

  app.vueApp.use(vuetify);
});
```

---

## Dark Mode Implementation

### Theme Toggle Component

```vue
<!-- components/common/ThemeToggle.vue -->
<script setup lang="ts">
const theme = useTheme();
const appStore = useAppStore();

const isDark = computed({
  get: () => theme.global.current.value.dark,
  set: (value) => {
    theme.global.name.value = value ? "dark" : "light";
    appStore.updateSettings({ theme: value ? "dark" : "light" });
  },
});

function toggleTheme() {
  isDark.value = !isDark.value;
}
</script>

<template>
  <v-btn
    icon
    variant="text"
    @click="toggleTheme"
    :aria-label="isDark ? 'Switch to light mode' : 'Switch to dark mode'"
  >
    <v-icon>
      {{ isDark ? "mdi-weather-sunny" : "mdi-weather-night" }}
    </v-icon>

    <v-tooltip activator="parent" location="bottom">
      {{ isDark ? "Light mode" : "Dark mode" }}
    </v-tooltip>
  </v-btn>
</template>
```

### Theme Selector Dropdown

```vue
<!-- components/common/ThemeSelector.vue -->
<script setup lang="ts">
const theme = useTheme();
const appStore = useAppStore();

interface ThemeOption {
  value: string;
  title: string;
  icon: string;
}

const themeOptions: ThemeOption[] = [
  { value: "light", title: "Light", icon: "mdi-weather-sunny" },
  { value: "dark", title: "Dark", icon: "mdi-weather-night" },
  { value: "system", title: "System", icon: "mdi-laptop" },
];

const selectedTheme = ref(appStore.settings.themePreference || "system");

// Watch for system preference changes
const prefersDark = useMediaQuery("(prefers-color-scheme: dark)");

watch(
  [selectedTheme, prefersDark],
  ([newTheme, systemDark]) => {
    let actualTheme: string;

    if (newTheme === "system") {
      actualTheme = systemDark ? "dark" : "light";
    } else {
      actualTheme = newTheme;
    }

    theme.global.name.value = actualTheme;
    appStore.updateSettings({ themePreference: newTheme });
  },
  { immediate: true }
);
</script>

<template>
  <v-menu>
    <template #activator="{ props }">
      <v-btn v-bind="props" icon variant="text">
        <v-icon>
          {{ themeOptions.find((t) => t.value === selectedTheme)?.icon }}
        </v-icon>
      </v-btn>
    </template>

    <v-list density="compact" min-width="150">
      <v-list-subheader>Theme</v-list-subheader>

      <v-list-item
        v-for="option in themeOptions"
        :key="option.value"
        :value="option.value"
        :active="selectedTheme === option.value"
        @click="selectedTheme = option.value"
      >
        <template #prepend>
          <v-icon :icon="option.icon" />
        </template>
        <v-list-item-title>{{ option.title }}</v-list-item-title>
      </v-list-item>
    </v-list>
  </v-menu>
</template>
```

### Theme Persistence

```typescript
// composables/useThemePreference.ts
export function useThemePreference() {
  const theme = useTheme();
  const prefersDark = useMediaQuery("(prefers-color-scheme: dark)");

  // Persist theme preference
  const themePreference = useLocalStorage<"light" | "dark" | "system">(
    "theme-preference",
    "system"
  );

  // Apply theme based on preference
  const applyTheme = () => {
    let actualTheme: string;

    if (themePreference.value === "system") {
      actualTheme = prefersDark.value ? "dark" : "light";
    } else {
      actualTheme = themePreference.value;
    }

    theme.global.name.value = actualTheme;
  };

  // Watch for changes
  watch([themePreference, prefersDark], applyTheme, { immediate: true });

  // Set theme preference
  const setTheme = (preference: "light" | "dark" | "system") => {
    themePreference.value = preference;
  };

  // Toggle between light and dark
  const toggleTheme = () => {
    if (themePreference.value === "system") {
      themePreference.value = prefersDark.value ? "light" : "dark";
    } else {
      themePreference.value =
        themePreference.value === "dark" ? "light" : "dark";
    }
  };

  return {
    themePreference: readonly(themePreference),
    isDark: computed(() => theme.global.current.value.dark),
    setTheme,
    toggleTheme,
  };
}
```

### SSR-Safe Theme Initialization

```typescript
// plugins/theme-init.client.ts
export default defineNuxtPlugin(() => {
  const theme = useTheme();

  // Get stored preference
  const stored = localStorage.getItem("theme-preference");
  const prefersDark = window.matchMedia("(prefers-color-scheme: dark)").matches;

  let initialTheme: string;

  if (
    stored === "dark" ||
    (stored === "system" && prefersDark) ||
    (!stored && prefersDark)
  ) {
    initialTheme = "dark";
  } else {
    initialTheme = "light";
  }

  theme.global.name.value = initialTheme;

  // Prevent flash of wrong theme
  document.documentElement.classList.add(initialTheme);
});
```

---

## useTheme Composable

### Basic Usage

```vue
<script setup lang="ts">
const theme = useTheme();

// Get current theme name
const currentTheme = computed(() => theme.global.name.value);

// Check if dark mode
const isDark = computed(() => theme.global.current.value.dark);

// Get theme colors
const primaryColor = computed(() => theme.global.current.value.colors.primary);
const backgroundColor = computed(
  () => theme.global.current.value.colors.background
);

// Change theme
function setTheme(name: string) {
  theme.global.name.value = name;
}

// Get computed color (with opacity)
const primaryWithOpacity = computed(() => {
  return theme.global.current.value.colors.primary + "80"; // 50% opacity
});
</script>

<template>
  <div>
    <p>Current theme: {{ currentTheme }}</p>
    <p>Is dark: {{ isDark }}</p>
    <p>Primary color: {{ primaryColor }}</p>

    <v-btn @click="setTheme('dark')">Dark</v-btn>
    <v-btn @click="setTheme('light')">Light</v-btn>
  </div>
</template>
```

### Dynamic Theme Colors

```vue
<script setup lang="ts">
const theme = useTheme();

// Access all theme colors
const colors = computed(() => theme.global.current.value.colors);

// Get specific color with fallback
function getColor(name: string, fallback = "#000000"): string {
  return colors.value[name] || fallback;
}

// Create color variants
const colorVariants = computed(() => ({
  primary: getColor("primary"),
  primaryLight: getColor("primary-lighten-1", getColor("primary")),
  primaryDark: getColor("primary-darken-1", getColor("primary")),
}));
</script>

<template>
  <div>
    <div
      v-for="(color, name) in colorVariants"
      :key="name"
      :style="{ backgroundColor: color }"
      class="pa-4 mb-2"
    >
      {{ name }}: {{ color }}
    </div>
  </div>
</template>
```

### Theme-Aware Styles

```vue
<script setup lang="ts">
const theme = useTheme();

// Computed styles based on theme
const cardStyles = computed(() => ({
  backgroundColor: theme.global.current.value.dark
    ? "rgba(255, 255, 255, 0.05)"
    : "rgba(0, 0, 0, 0.02)",
  borderColor: theme.global.current.value.dark
    ? "rgba(255, 255, 255, 0.12)"
    : "rgba(0, 0, 0, 0.12)",
}));
</script>

<template>
  <div class="custom-card" :style="cardStyles">
    <slot />
  </div>
</template>

<style scoped>
.custom-card {
  border: 1px solid;
  border-radius: 8px;
  padding: 16px;
  transition: background-color 0.2s, border-color 0.2s;
}
</style>
```

---

## SASS Variable Customization

### Setup SASS Variables

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  css: ["vuetify/styles", "~/assets/styles/main.scss"],

  vite: {
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `@use "~/assets/styles/variables" as *;`,
        },
      },
    },
  },
});
```

### Custom Variables File

```scss
// assets/styles/variables.scss

// Override Vuetify SASS variables
@use "vuetify/settings" with (
  // Border radius
  $border-radius-root: 8px,

  // Typography
  $body-font-family: ("Inter", sans-serif),
  $heading-font-family: ("Inter", sans-serif),
  // Spacing
  $spacer: 4px,

  // Grid
  $grid-gutter: 24px,

  // Component-specific
  $button-height: 44px,
  $button-border-radius: 8px,
  $card-border-radius: 12px,
  $text-field-border-radius: 8px,

  // Elevation
  $shadow-key-umbra-opacity: 0.08,
  $shadow-key-penumbra-opacity: 0.06,
  $shadow-key-ambient-opacity: 0.04
);

// Custom SCSS variables
$header-height: 64px;
$sidebar-width: 280px;
$sidebar-collapsed-width: 64px;
$footer-height: 48px;

// Transition durations
$transition-fast: 150ms;
$transition-base: 250ms;
$transition-slow: 350ms;

// Z-index layers
$z-index-drawer: 1000;
$z-index-appbar: 1100;
$z-index-dialog: 1200;
$z-index-snackbar: 1300;
$z-index-tooltip: 1400;
```

### Component-Specific Overrides

```scss
// assets/styles/components/_buttons.scss

// Button customizations
.v-btn {
  text-transform: none;
  letter-spacing: 0.01em;
  font-weight: 500;

  &--size-small {
    font-size: 0.8125rem;
  }

  &--size-large {
    font-size: 1rem;
  }
}

// Outlined button hover
.v-btn--variant-outlined {
  &:hover {
    background-color: rgba(var(--v-theme-primary), 0.04);
  }
}
```

```scss
// assets/styles/components/_cards.scss

// Card customizations
.v-card {
  overflow: hidden;

  &--hover {
    transition: transform 0.2s, box-shadow 0.2s;

    &:hover {
      transform: translateY(-2px);
    }
  }
}

// Card title
.v-card-title {
  font-weight: 600;
  line-height: 1.4;
}
```

```scss
// assets/styles/components/_inputs.scss

// Text field customizations
.v-text-field {
  .v-field__outline {
    transition: border-color 0.2s;
  }

  &--focused {
    .v-field__outline {
      border-width: 2px;
    }
  }
}

// Input label
.v-label {
  font-size: 0.875rem;
  font-weight: 500;
}
```

### Main Styles Entry

```scss
// assets/styles/main.scss

// Import component overrides
@import "components/buttons";
@import "components/cards";
@import "components/inputs";

// Global styles
html {
  scroll-behavior: smooth;
}

body {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

// Custom utility classes
.text-gradient {
  background: linear-gradient(
    135deg,
    rgb(var(--v-theme-primary)) 0%,
    rgb(var(--v-theme-secondary)) 100%
  );
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
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
  background-color: rgba(var(--v-theme-on-surface), 0.2);
  border-radius: 4px;

  &:hover {
    background-color: rgba(var(--v-theme-on-surface), 0.3);
  }
}
```

---

## CSS Variables Integration

### Accessing Vuetify CSS Variables

```vue
<template>
  <div class="custom-component">
    <p class="primary-text">Primary colored text</p>
    <div class="surface-card">Surface background</div>
  </div>
</template>

<style scoped>
.primary-text {
  color: rgb(var(--v-theme-primary));
}

.surface-card {
  background-color: rgb(var(--v-theme-surface));
  border: 1px solid rgba(var(--v-border-color), var(--v-border-opacity));
  border-radius: 8px;
  padding: 16px;
}

/* Using theme variables for hover states */
.interactive-element {
  transition: background-color 0.2s;

  &:hover {
    background-color: rgba(var(--v-theme-primary), var(--v-hover-opacity));
  }

  &:focus {
    background-color: rgba(var(--v-theme-primary), var(--v-focus-opacity));
  }
}
</style>
```

### Custom CSS Variables

```typescript
// plugins/vuetify.ts
const lightTheme: ThemeDefinition = {
  dark: false,
  colors: {
    // ... colors
  },
  variables: {
    // Custom variables
    "header-height": "64px",
    "sidebar-width": "280px",
    "content-max-width": "1200px",
    "card-shadow": "0 2px 8px rgba(0, 0, 0, 0.08)",
    "transition-fast": "150ms",
    "transition-base": "250ms",
  },
};
```

```vue
<style scoped>
.layout-header {
  height: var(--v-header-height);
}

.layout-sidebar {
  width: var(--v-sidebar-width);
}

.content-container {
  max-width: var(--v-content-max-width);
  margin: 0 auto;
}

.card-elevated {
  box-shadow: var(--v-card-shadow);
}

.animated-element {
  transition: all var(--v-transition-base) ease;
}
</style>
```

---

## Dynamic Theme Switching

### Runtime Theme Creation

```typescript
// composables/useDynamicTheme.ts
import { useTheme, type ThemeDefinition } from "vuetify";

export function useDynamicTheme() {
  const theme = useTheme();

  // Create a new theme at runtime
  function createTheme(name: string, definition: ThemeDefinition) {
    theme.themes.value[name] = definition;
  }

  // Update existing theme colors
  function updateThemeColors(
    themeName: string,
    colors: Record<string, string>
  ) {
    const existingTheme = theme.themes.value[themeName];
    if (existingTheme) {
      theme.themes.value[themeName] = {
        ...existingTheme,
        colors: {
          ...existingTheme.colors,
          ...colors,
        },
      };
    }
  }

  // Set primary color dynamically
  function setPrimaryColor(color: string) {
    const currentThemeName = theme.global.name.value;
    updateThemeColors(currentThemeName, { primary: color });
  }

  // Generate theme from primary color
  function generateThemeFromColor(primaryColor: string) {
    // Use a color manipulation library like chroma.js or tinycolor2
    // to generate a full color palette from a single color
    const colors = generateColorPalette(primaryColor);

    updateThemeColors("light", {
      primary: colors.primary,
      "primary-lighten-1": colors.primaryLight,
      "primary-darken-1": colors.primaryDark,
    });

    updateThemeColors("dark", {
      primary: colors.primaryLight,
      "primary-lighten-1": colors.primaryLighter,
      "primary-darken-1": colors.primary,
    });
  }

  return {
    createTheme,
    updateThemeColors,
    setPrimaryColor,
    generateThemeFromColor,
  };
}

// Helper function to generate color palette
function generateColorPalette(baseColor: string) {
  // Simplified example - use a proper color library in production
  return {
    primary: baseColor,
    primaryLight: lightenColor(baseColor, 20),
    primaryLighter: lightenColor(baseColor, 40),
    primaryDark: darkenColor(baseColor, 20),
    primaryDarker: darkenColor(baseColor, 40),
  };
}
```

### Theme Customizer Component

```vue
<!-- components/settings/ThemeCustomizer.vue -->
<script setup lang="ts">
const theme = useTheme();
const { setPrimaryColor } = useDynamicTheme();

const presetColors = [
  { name: "Indigo", value: "#6366F1" },
  { name: "Blue", value: "#3B82F6" },
  { name: "Cyan", value: "#06B6D4" },
  { name: "Teal", value: "#14B8A6" },
  { name: "Green", value: "#22C55E" },
  { name: "Orange", value: "#F97316" },
  { name: "Red", value: "#EF4444" },
  { name: "Pink", value: "#EC4899" },
  { name: "Purple", value: "#A855F7" },
];

const customColor = ref("#6366F1");
const selectedPreset = ref(presetColors[0].value);

function applyPresetColor(color: string) {
  selectedPreset.value = color;
  customColor.value = color;
  setPrimaryColor(color);
}

function applyCustomColor() {
  selectedPreset.value = "";
  setPrimaryColor(customColor.value);
}
</script>

<template>
  <v-card>
    <v-card-title>Theme Customization</v-card-title>

    <v-card-text>
      <!-- Preset Colors -->
      <div class="mb-6">
        <p class="text-subtitle-2 mb-2">Preset Colors</p>
        <div class="d-flex flex-wrap ga-2">
          <v-btn
            v-for="color in presetColors"
            :key="color.value"
            :color="color.value"
            :variant="selectedPreset === color.value ? 'flat' : 'outlined'"
            size="small"
            rounded
            @click="applyPresetColor(color.value)"
          >
            {{ color.name }}
          </v-btn>
        </div>
      </div>

      <!-- Custom Color Picker -->
      <div class="mb-6">
        <p class="text-subtitle-2 mb-2">Custom Color</p>
        <div class="d-flex align-center ga-4">
          <v-color-picker
            v-model="customColor"
            hide-inputs
            hide-canvas
            show-swatches
            @update:model-value="applyCustomColor"
          />

          <v-text-field
            v-model="customColor"
            label="Hex Color"
            variant="outlined"
            density="compact"
            style="max-width: 150px;"
            @blur="applyCustomColor"
          />
        </div>
      </div>

      <!-- Preview -->
      <div>
        <p class="text-subtitle-2 mb-2">Preview</p>
        <div class="d-flex ga-2">
          <v-btn color="primary">Primary</v-btn>
          <v-btn color="primary" variant="outlined">Outlined</v-btn>
          <v-btn color="primary" variant="text">Text</v-btn>
        </div>
      </div>
    </v-card-text>
  </v-card>
</template>
```

---

## Theme-Aware Components

### Adaptive Card Component

```vue
<!-- components/common/AdaptiveCard.vue -->
<script setup lang="ts">
const theme = useTheme();

interface Props {
  variant?: "elevated" | "flat" | "outlined" | "tonal";
  hover?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  variant: "elevated",
  hover: false,
});

// Compute elevation based on theme
const elevation = computed(() => {
  if (props.variant !== "elevated") return 0;
  return theme.global.current.value.dark ? 2 : 4;
});

// Compute background based on theme
const backgroundColor = computed(() => {
  if (props.variant === "tonal") {
    return theme.global.current.value.dark
      ? "rgba(255, 255, 255, 0.05)"
      : "rgba(0, 0, 0, 0.02)";
  }
  return undefined;
});
</script>

<template>
  <v-card
    :variant="variant"
    :elevation="elevation"
    :style="{ backgroundColor }"
    :class="{ 'card-hover': hover }"
  >
    <slot />
  </v-card>
</template>

<style scoped>
.card-hover {
  transition: transform 0.2s, box-shadow 0.2s;
  cursor: pointer;
}

.card-hover:hover {
  transform: translateY(-2px);
}
</style>
```

### Theme-Aware Icon

```vue
<!-- components/common/ThemeIcon.vue -->
<script setup lang="ts">
const theme = useTheme();

interface Props {
  lightIcon: string;
  darkIcon: string;
  size?: string | number;
  color?: string;
}

const props = withDefaults(defineProps<Props>(), {
  size: 24,
});

const currentIcon = computed(() => {
  return theme.global.current.value.dark ? props.darkIcon : props.lightIcon;
});
</script>

<template>
  <v-icon :icon="currentIcon" :size="size" :color="color" />
</template>
```

---

## Testing Themes

### Theme Testing Utilities

```typescript
// tests/utils/theme-helpers.ts
import { createVuetify, type ThemeDefinition } from "vuetify";
import * as components from "vuetify/components";
import * as directives from "vuetify/directives";

export function createTestVuetify(
  options: {
    theme?: string;
    customThemes?: Record<string, ThemeDefinition>;
  } = {}
) {
  return createVuetify({
    components,
    directives,
    theme: {
      defaultTheme: options.theme || "light",
      themes: {
        light: {
          dark: false,
          colors: {
            primary: "#1867C0",
            secondary: "#48A9A6",
          },
        },
        dark: {
          dark: true,
          colors: {
            primary: "#2196F3",
            secondary: "#54B6B2",
          },
        },
        ...options.customThemes,
      },
    },
  });
}

export function mountWithTheme(
  component: any,
  options: {
    theme?: string;
    props?: Record<string, any>;
  } = {}
) {
  const vuetify = createTestVuetify({ theme: options.theme });

  return mount(component, {
    props: options.props,
    global: {
      plugins: [vuetify],
    },
  });
}
```

### Theme Component Tests

```typescript
// tests/components/ThemeToggle.test.ts
import { describe, it, expect, beforeEach } from "vitest";
import { mount } from "@vue/test-utils";
import { createTestVuetify } from "../utils/theme-helpers";
import ThemeToggle from "~/components/common/ThemeToggle.vue";

describe("ThemeToggle", () => {
  let vuetify: ReturnType<typeof createTestVuetify>;

  beforeEach(() => {
    vuetify = createTestVuetify({ theme: "light" });
  });

  it("renders sun icon in dark mode", () => {
    // Arrange
    vuetify = createTestVuetify({ theme: "dark" });

    // Act
    const wrapper = mount(ThemeToggle, {
      global: { plugins: [vuetify] },
    });

    // Assert
    expect(wrapper.find(".mdi-weather-sunny").exists()).toBe(true);
  });

  it("renders moon icon in light mode", () => {
    // Arrange & Act
    const wrapper = mount(ThemeToggle, {
      global: { plugins: [vuetify] },
    });

    // Assert
    expect(wrapper.find(".mdi-weather-night").exists()).toBe(true);
  });

  it("toggles theme on click", async () => {
    // Arrange
    const wrapper = mount(ThemeToggle, {
      global: { plugins: [vuetify] },
    });

    // Act
    await wrapper.find("button").trigger("click");

    // Assert
    expect(vuetify.theme.global.name.value).toBe("dark");
  });
});
```

---

## Best Practices

### 1. Use Semantic Color Names

```typescript
// ✅ Good: Semantic color names
colors: {
  primary: '#6366F1',
  secondary: '#10B981',
  error: '#EF4444',
  warning: '#F59E0B',
  info: '#3B82F6',
  success: '#22C55E',
}

// ❌ Bad: Non-semantic names
colors: {
  indigo: '#6366F1',
  green: '#10B981',
  red: '#EF4444',
}
```

### 2. Provide Sufficient Contrast

```typescript
// Ensure text is readable on colored backgrounds
colors: {
  primary: '#6366F1',
  'on-primary': '#FFFFFF', // White text on primary

  warning: '#F59E0B',
  'on-warning': '#000000', // Black text on warning (yellow)
}
```

### 3. Support System Preference

```typescript
// Always respect user's system preference as default
const prefersDark = useMediaQuery("(prefers-color-scheme: dark)");

const initialTheme = computed(() => {
  const stored = localStorage.getItem("theme-preference");
  if (stored && stored !== "system") return stored;
  return prefersDark.value ? "dark" : "light";
});
```

### 4. Avoid Flash of Wrong Theme

```html
<!-- Add to app.vue or index.html -->
<script>
  // Inline script to set theme before Vue loads
  (function () {
    const stored = localStorage.getItem("theme-preference");
    const prefersDark = window.matchMedia(
      "(prefers-color-scheme: dark)"
    ).matches;
    const theme =
      stored === "dark" || (stored !== "light" && prefersDark)
        ? "dark"
        : "light";
    document.documentElement.classList.add(theme);
  })();
</script>
```

### 5. Test Both Themes

```typescript
// Always test components in both light and dark modes
describe("MyComponent", () => {
  it.each(["light", "dark"])("renders correctly in %s mode", (theme) => {
    const wrapper = mountWithTheme(MyComponent, { theme });
    expect(wrapper.exists()).toBe(true);
  });
});
```

---

## Next Steps

Continue to [Form Validation](./09-form-validation.md) to learn about integrating VeeValidate with Vuetify form components for comprehensive form validation patterns.
