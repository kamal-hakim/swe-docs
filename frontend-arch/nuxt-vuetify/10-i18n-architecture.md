# i18n Architecture

## Overview

This document covers comprehensive internationalization (i18n) implementation for Nuxt applications using **@nuxtjs/i18n** module integrated with Vuetify 3. It includes lazy-loaded locale files, SEO-friendly URL strategies, pluralization rules, datetime and number formatting, per-component translations, and locale switching with persistence.

## Technology Stack

| Tool               | Purpose                           |
| ------------------ | --------------------------------- |
| **@nuxtjs/i18n**   | Nuxt i18n module                  |
| **vue-i18n**       | Core i18n library                 |
| **Vuetify Locale** | Vuetify component translations    |
| **Intl API**       | Native datetime/number formatting |

---

## Installation and Setup

### Install Dependencies

```bash
pnpm add @nuxtjs/i18n
```

### Module Configuration

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ["@nuxtjs/i18n"],

  i18n: {
    // Locale configuration
    locales: [
      {
        code: "en",
        name: "English",
        file: "en.json",
        iso: "en-US",
        dir: "ltr",
      },
      {
        code: "es",
        name: "Español",
        file: "es.json",
        iso: "es-ES",
        dir: "ltr",
      },
      {
        code: "fr",
        name: "Français",
        file: "fr.json",
        iso: "fr-FR",
        dir: "ltr",
      },
      {
        code: "de",
        name: "Deutsch",
        file: "de.json",
        iso: "de-DE",
        dir: "ltr",
      },
      {
        code: "ar",
        name: "العربية",
        file: "ar.json",
        iso: "ar-SA",
        dir: "rtl",
      },
      {
        code: "zh",
        name: "中文",
        file: "zh.json",
        iso: "zh-CN",
        dir: "ltr",
      },
    ],

    // Default locale
    defaultLocale: "en",

    // Lazy loading
    lazy: true,
    langDir: "locales",

    // URL strategy
    strategy: "prefix_except_default",

    // Browser language detection
    detectBrowserLanguage: {
      useCookie: true,
      cookieKey: "i18n_locale",
      cookieSecure: true,
      redirectOn: "root",
      alwaysRedirect: false,
      fallbackLocale: "en",
    },

    // SEO
    baseUrl: "https://example.com",

    // Vue I18n options
    vueI18n: "./config/i18n.config.ts",
  },
});
```

### Vue I18n Configuration

```typescript
// config/i18n.config.ts
export default defineI18nConfig(() => ({
  legacy: false,
  locale: "en",
  fallbackLocale: "en",

  // Missing key handling
  missingWarn: process.env.NODE_ENV === "development",
  fallbackWarn: process.env.NODE_ENV === "development",

  // Number formats
  numberFormats: {
    en: {
      currency: {
        style: "currency",
        currency: "USD",
        notation: "standard",
      },
      decimal: {
        style: "decimal",
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      },
      percent: {
        style: "percent",
        useGrouping: false,
      },
      compact: {
        notation: "compact",
        compactDisplay: "short",
      },
    },
    es: {
      currency: {
        style: "currency",
        currency: "EUR",
        notation: "standard",
      },
      decimal: {
        style: "decimal",
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      },
      percent: {
        style: "percent",
        useGrouping: false,
      },
    },
    de: {
      currency: {
        style: "currency",
        currency: "EUR",
        notation: "standard",
      },
      decimal: {
        style: "decimal",
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      },
      percent: {
        style: "percent",
        useGrouping: false,
      },
    },
  },

  // Datetime formats
  datetimeFormats: {
    en: {
      short: {
        year: "numeric",
        month: "short",
        day: "numeric",
      },
      long: {
        year: "numeric",
        month: "long",
        day: "numeric",
        weekday: "long",
      },
      time: {
        hour: "numeric",
        minute: "numeric",
      },
      datetime: {
        year: "numeric",
        month: "short",
        day: "numeric",
        hour: "numeric",
        minute: "numeric",
      },
      relative: {
        style: "long",
        numeric: "auto",
      },
    },
    es: {
      short: {
        year: "numeric",
        month: "short",
        day: "numeric",
      },
      long: {
        year: "numeric",
        month: "long",
        day: "numeric",
        weekday: "long",
      },
      time: {
        hour: "numeric",
        minute: "numeric",
        hour12: false,
      },
      datetime: {
        year: "numeric",
        month: "short",
        day: "numeric",
        hour: "numeric",
        minute: "numeric",
        hour12: false,
      },
    },
    de: {
      short: {
        year: "numeric",
        month: "2-digit",
        day: "2-digit",
      },
      long: {
        year: "numeric",
        month: "long",
        day: "numeric",
        weekday: "long",
      },
      time: {
        hour: "2-digit",
        minute: "2-digit",
        hour12: false,
      },
      datetime: {
        year: "numeric",
        month: "2-digit",
        day: "2-digit",
        hour: "2-digit",
        minute: "2-digit",
        hour12: false,
      },
    },
  },

  // Pluralization rules
  pluralRules: {
    // Russian pluralization (example of complex rules)
    ru: (choice: number) => {
      const teen = choice > 10 && choice < 20;
      const endsWithOne = choice % 10 === 1;
      if (!teen && endsWithOne) return 0;
      if (!teen && choice % 10 >= 2 && choice % 10 <= 4) return 1;
      return 2;
    },
    // Arabic pluralization
    ar: (choice: number) => {
      if (choice === 0) return 0;
      if (choice === 1) return 1;
      if (choice === 2) return 2;
      if (choice >= 3 && choice <= 10) return 3;
      if (choice >= 11 && choice <= 99) return 4;
      return 5;
    },
  },
}));
```

---

## Locale Files Structure

### Directory Structure

```
locales/
├── en.json                 # English (default)
├── es.json                 # Spanish
├── fr.json                 # French
├── de.json                 # German
├── ar.json                 # Arabic (RTL)
├── zh.json                 # Chinese
└── components/             # Per-component translations
    ├── header.en.json
    ├── header.es.json
    ├── footer.en.json
    └── footer.es.json
```

### Main Locale File

```json
// locales/en.json
{
  "common": {
    "loading": "Loading...",
    "error": "An error occurred",
    "retry": "Retry",
    "cancel": "Cancel",
    "save": "Save",
    "delete": "Delete",
    "edit": "Edit",
    "create": "Create",
    "search": "Search",
    "filter": "Filter",
    "sort": "Sort",
    "noResults": "No results found",
    "confirm": "Confirm",
    "back": "Back",
    "next": "Next",
    "previous": "Previous",
    "close": "Close",
    "yes": "Yes",
    "no": "No"
  },

  "navigation": {
    "home": "Home",
    "dashboard": "Dashboard",
    "profile": "Profile",
    "settings": "Settings",
    "logout": "Logout",
    "login": "Login",
    "register": "Register"
  },

  "auth": {
    "login": {
      "title": "Sign In",
      "subtitle": "Welcome back! Please sign in to continue.",
      "email": "Email Address",
      "password": "Password",
      "rememberMe": "Remember me",
      "forgotPassword": "Forgot password?",
      "submit": "Sign In",
      "noAccount": "Don't have an account?",
      "signUp": "Sign up"
    },
    "register": {
      "title": "Create Account",
      "subtitle": "Join us today and get started.",
      "firstName": "First Name",
      "lastName": "Last Name",
      "email": "Email Address",
      "password": "Password",
      "confirmPassword": "Confirm Password",
      "acceptTerms": "I accept the {terms} and {privacy}",
      "terms": "Terms of Service",
      "privacy": "Privacy Policy",
      "submit": "Create Account",
      "hasAccount": "Already have an account?",
      "signIn": "Sign in"
    },
    "errors": {
      "invalidCredentials": "Invalid email or password",
      "emailTaken": "This email is already registered",
      "weakPassword": "Password is too weak",
      "sessionExpired": "Your session has expired. Please sign in again."
    }
  },

  "validation": {
    "required": "This field is required",
    "email": "Please enter a valid email address",
    "minLength": "Must be at least {min} characters",
    "maxLength": "Must not exceed {max} characters",
    "passwordMatch": "Passwords do not match",
    "invalidFormat": "Invalid format"
  },

  "user": {
    "profile": {
      "title": "Profile",
      "personalInfo": "Personal Information",
      "contactInfo": "Contact Information",
      "avatar": "Profile Picture",
      "changeAvatar": "Change Picture",
      "removeAvatar": "Remove Picture"
    },
    "settings": {
      "title": "Settings",
      "language": "Language",
      "theme": "Theme",
      "notifications": "Notifications",
      "privacy": "Privacy",
      "security": "Security"
    }
  },

  "notifications": {
    "success": {
      "saved": "Changes saved successfully",
      "deleted": "Item deleted successfully",
      "created": "Item created successfully"
    },
    "error": {
      "generic": "Something went wrong. Please try again.",
      "network": "Network error. Please check your connection.",
      "unauthorized": "You are not authorized to perform this action."
    }
  },

  "plurals": {
    "items": "no items | {count} item | {count} items",
    "users": "no users | {count} user | {count} users",
    "messages": "no messages | {count} message | {count} messages",
    "days": "{count} day | {count} days",
    "hours": "{count} hour | {count} hours",
    "minutes": "{count} minute | {count} minutes"
  },

  "time": {
    "justNow": "Just now",
    "minutesAgo": "{count} minute ago | {count} minutes ago",
    "hoursAgo": "{count} hour ago | {count} hours ago",
    "daysAgo": "{count} day ago | {count} days ago",
    "weeksAgo": "{count} week ago | {count} weeks ago"
  }
}
```

### Spanish Locale File

```json
// locales/es.json
{
  "common": {
    "loading": "Cargando...",
    "error": "Se produjo un error",
    "retry": "Reintentar",
    "cancel": "Cancelar",
    "save": "Guardar",
    "delete": "Eliminar",
    "edit": "Editar",
    "create": "Crear",
    "search": "Buscar",
    "filter": "Filtrar",
    "sort": "Ordenar",
    "noResults": "No se encontraron resultados",
    "confirm": "Confirmar",
    "back": "Atrás",
    "next": "Siguiente",
    "previous": "Anterior",
    "close": "Cerrar",
    "yes": "Sí",
    "no": "No"
  },

  "navigation": {
    "home": "Inicio",
    "dashboard": "Panel",
    "profile": "Perfil",
    "settings": "Configuración",
    "logout": "Cerrar sesión",
    "login": "Iniciar sesión",
    "register": "Registrarse"
  },

  "auth": {
    "login": {
      "title": "Iniciar Sesión",
      "subtitle": "¡Bienvenido de nuevo! Por favor, inicia sesión para continuar.",
      "email": "Correo Electrónico",
      "password": "Contraseña",
      "rememberMe": "Recordarme",
      "forgotPassword": "¿Olvidaste tu contraseña?",
      "submit": "Iniciar Sesión",
      "noAccount": "¿No tienes una cuenta?",
      "signUp": "Regístrate"
    },
    "register": {
      "title": "Crear Cuenta",
      "subtitle": "Únete hoy y comienza.",
      "firstName": "Nombre",
      "lastName": "Apellido",
      "email": "Correo Electrónico",
      "password": "Contraseña",
      "confirmPassword": "Confirmar Contraseña",
      "acceptTerms": "Acepto los {terms} y la {privacy}",
      "terms": "Términos de Servicio",
      "privacy": "Política de Privacidad",
      "submit": "Crear Cuenta",
      "hasAccount": "¿Ya tienes una cuenta?",
      "signIn": "Inicia sesión"
    }
  },

  "plurals": {
    "items": "ningún elemento | {count} elemento | {count} elementos",
    "users": "ningún usuario | {count} usuario | {count} usuarios",
    "messages": "ningún mensaje | {count} mensaje | {count} mensajes"
  }
}
```

---

## Vuetify Locale Integration

### Setup Vuetify with i18n

```typescript
// plugins/vuetify.ts
import { createVuetify } from "vuetify";
import { createVueI18nAdapter } from "vuetify/locale/adapters/vue-i18n";
import { useI18n } from "vue-i18n";

// Import Vuetify locale messages
import { en, es, fr, de, ar, zhHans } from "vuetify/locale";

export default defineNuxtPlugin((nuxtApp) => {
  const i18n = nuxtApp.$i18n;

  const vuetify = createVuetify({
    locale: {
      adapter: createVueI18nAdapter({ i18n, useI18n }),
    },
    // ... other options
  });

  nuxtApp.vueApp.use(vuetify);
});
```

### Vuetify Messages in Locale Files

```json
// locales/en.json
{
  "$vuetify": {
    "badge": "Badge",
    "close": "Close",
    "dataIterator": {
      "noResultsText": "No matching records found",
      "loadingText": "Loading items..."
    },
    "dataTable": {
      "itemsPerPageText": "Rows per page:",
      "ariaLabel": {
        "sortDescending": "Sorted descending.",
        "sortAscending": "Sorted ascending.",
        "sortNone": "Not sorted.",
        "activateNone": "Activate to remove sorting.",
        "activateDescending": "Activate to sort descending.",
        "activateAscending": "Activate to sort ascending."
      },
      "sortBy": "Sort by"
    },
    "dataFooter": {
      "itemsPerPageText": "Items per page:",
      "itemsPerPageAll": "All",
      "nextPage": "Next page",
      "prevPage": "Previous page",
      "firstPage": "First page",
      "lastPage": "Last page",
      "pageText": "{0}-{1} of {2}"
    },
    "datePicker": {
      "itemsSelected": "{0} selected",
      "nextMonthAriaLabel": "Next month",
      "nextYearAriaLabel": "Next year",
      "prevMonthAriaLabel": "Previous month",
      "prevYearAriaLabel": "Previous year"
    },
    "noDataText": "No data available",
    "carousel": {
      "prev": "Previous visual",
      "next": "Next visual",
      "ariaLabel": {
        "delimiter": "Carousel slide {0} of {1}"
      }
    },
    "calendar": {
      "moreEvents": "{0} more"
    },
    "input": {
      "clear": "Clear {0}",
      "prependAction": "{0} prepended action",
      "appendAction": "{0} appended action"
    },
    "fileInput": {
      "counter": "{0} files",
      "counterSize": "{0} files ({1} in total)"
    },
    "timePicker": {
      "am": "AM",
      "pm": "PM"
    },
    "pagination": {
      "ariaLabel": {
        "root": "Pagination Navigation",
        "next": "Next page",
        "previous": "Previous page",
        "page": "Go to page {0}",
        "currentPage": "Page {0}, Current page",
        "first": "First page",
        "last": "Last page"
      }
    },
    "rating": {
      "ariaLabel": {
        "item": "Rating {0} of {1}"
      }
    }
  }
}
```

---

## Using Translations

### In Templates

```vue
<script setup lang="ts">
const { t, n, d, locale } = useI18n();
</script>

<template>
  <div>
    <!-- Simple translation -->
    <h1>{{ $t("auth.login.title") }}</h1>

    <!-- With interpolation -->
    <p>{{ $t("validation.minLength", { min: 8 }) }}</p>

    <!-- Pluralization -->
    <span>{{ $t("plurals.items", { count: itemCount }) }}</span>

    <!-- Number formatting -->
    <span>{{ $n(price, "currency") }}</span>
    <span>{{ $n(percentage, "percent") }}</span>

    <!-- Date formatting -->
    <span>{{ $d(new Date(), "short") }}</span>
    <span>{{ $d(createdAt, "datetime") }}</span>

    <!-- With named slots for links -->
    <i18n-t keypath="auth.register.acceptTerms" tag="p">
      <template #terms>
        <NuxtLink to="/terms">{{ $t("auth.register.terms") }}</NuxtLink>
      </template>
      <template #privacy>
        <NuxtLink to="/privacy">{{ $t("auth.register.privacy") }}</NuxtLink>
      </template>
    </i18n-t>
  </div>
</template>
```

### In Composition API

```typescript
// composables/useTranslation.ts
export function useTranslation() {
  const { t, n, d, locale, locales, setLocale } = useI18n();

  // Format currency
  function formatCurrency(value: number, currency?: string): string {
    return n(value, "currency", { currency });
  }

  // Format date
  function formatDate(date: Date | string, format: string = "short"): string {
    const dateObj = typeof date === "string" ? new Date(date) : date;
    return d(dateObj, format);
  }

  // Format relative time
  function formatRelativeTime(date: Date | string): string {
    const dateObj = typeof date === "string" ? new Date(date) : date;
    const now = new Date();
    const diffMs = now.getTime() - dateObj.getTime();
    const diffMins = Math.floor(diffMs / 60000);
    const diffHours = Math.floor(diffMs / 3600000);
    const diffDays = Math.floor(diffMs / 86400000);

    if (diffMins < 1) return t("time.justNow");
    if (diffMins < 60) return t("time.minutesAgo", { count: diffMins });
    if (diffHours < 24) return t("time.hoursAgo", { count: diffHours });
    if (diffDays < 7) return t("time.daysAgo", { count: diffDays });

    return formatDate(dateObj, "short");
  }

  // Get available locales
  const availableLocales = computed(() => {
    return locales.value.map((l) => ({
      code: typeof l === "string" ? l : l.code,
      name: typeof l === "string" ? l : l.name,
    }));
  });

  return {
    t,
    n,
    d,
    locale,
    setLocale,
    formatCurrency,
    formatDate,
    formatRelativeTime,
    availableLocales,
  };
}
```

---

## Locale Switching

### Language Selector Component

```vue
<!-- components/common/LanguageSelector.vue -->
<script setup lang="ts">
const { locale, locales, setLocale } = useI18n();
const switchLocalePath = useSwitchLocalePath();

interface LocaleOption {
  code: string;
  name: string;
  flag?: string;
}

const availableLocales = computed<LocaleOption[]>(() => {
  return locales.value.map((l) => {
    const loc = typeof l === "string" ? { code: l, name: l } : l;
    return {
      code: loc.code,
      name: loc.name || loc.code,
      flag: getFlag(loc.code),
    };
  });
});

function getFlag(code: string): string {
  const flags: Record<string, string> = {
    en: "🇺🇸",
    es: "🇪🇸",
    fr: "🇫🇷",
    de: "🇩🇪",
    ar: "🇸🇦",
    zh: "🇨🇳",
  };
  return flags[code] || "🌐";
}

async function changeLocale(code: string) {
  await setLocale(code);
  // Navigate to localized path
  navigateTo(switchLocalePath(code));
}
</script>

<template>
  <v-menu>
    <template #activator="{ props }">
      <v-btn v-bind="props" variant="text">
        <span class="mr-2">{{ getFlag(locale) }}</span>
        <span class="d-none d-sm-inline">
          {{ availableLocales.find((l) => l.code === locale)?.name }}
        </span>
        <v-icon end>mdi-chevron-down</v-icon>
      </v-btn>
    </template>

    <v-list density="compact" min-width="180">
      <v-list-subheader>{{ $t("user.settings.language") }}</v-list-subheader>

      <v-list-item
        v-for="loc in availableLocales"
        :key="loc.code"
        :active="locale === loc.code"
        @click="changeLocale(loc.code)"
      >
        <template #prepend>
          <span class="mr-3">{{ loc.flag }}</span>
        </template>
        <v-list-item-title>{{ loc.name }}</v-list-item-title>
      </v-list-item>
    </v-list>
  </v-menu>
</template>
```

### Language Selector with Flags

```vue
<!-- components/common/LanguageSelectorFull.vue -->
<script setup lang="ts">
const { locale, locales, setLocale } = useI18n();
const switchLocalePath = useSwitchLocalePath();

const selectedLocale = ref(locale.value);

const localeOptions = computed(() => {
  return locales.value.map((l) => {
    const loc = typeof l === "string" ? { code: l, name: l } : l;
    return {
      value: loc.code,
      title: loc.name,
      subtitle: loc.iso,
    };
  });
});

watch(selectedLocale, async (newLocale) => {
  if (newLocale !== locale.value) {
    await setLocale(newLocale);
    navigateTo(switchLocalePath(newLocale));
  }
});
</script>

<template>
  <v-select
    v-model="selectedLocale"
    :items="localeOptions"
    item-value="value"
    item-title="title"
    :label="$t('user.settings.language')"
    variant="outlined"
    density="compact"
  >
    <template #item="{ item, props }">
      <v-list-item v-bind="props">
        <template #subtitle>
          {{ item.raw.subtitle }}
        </template>
      </v-list-item>
    </template>
  </v-select>
</template>
```

---

## Per-Component Translations

### Component-Scoped Messages

```vue
<!-- components/features/ProductCard.vue -->
<script setup lang="ts">
const { t } = useI18n({
  useScope: "local",
  messages: {
    en: {
      addToCart: "Add to Cart",
      outOfStock: "Out of Stock",
      inStock: "{count} in stock",
      price: "Price",
      sale: "Sale",
    },
    es: {
      addToCart: "Añadir al Carrito",
      outOfStock: "Agotado",
      inStock: "{count} en stock",
      price: "Precio",
      sale: "Oferta",
    },
  },
});

interface Props {
  product: {
    name: string;
    price: number;
    salePrice?: number;
    stock: number;
    image: string;
  };
}

const props = defineProps<Props>();
const { formatCurrency } = useTranslation();
</script>

<template>
  <v-card>
    <v-img :src="product.image" height="200" cover>
      <v-chip v-if="product.salePrice" color="error" class="ma-2" size="small">
        {{ t("sale") }}
      </v-chip>
    </v-img>

    <v-card-title>{{ product.name }}</v-card-title>

    <v-card-text>
      <div class="d-flex align-center ga-2">
        <span class="text-h6">
          {{ formatCurrency(product.salePrice || product.price) }}
        </span>
        <span
          v-if="product.salePrice"
          class="text-decoration-line-through text-medium-emphasis"
        >
          {{ formatCurrency(product.price) }}
        </span>
      </div>

      <p class="text-caption mt-2">
        {{
          product.stock > 0
            ? t("inStock", { count: product.stock })
            : t("outOfStock")
        }}
      </p>
    </v-card-text>

    <v-card-actions>
      <v-btn color="primary" block :disabled="product.stock === 0">
        {{ t("addToCart") }}
      </v-btn>
    </v-card-actions>
  </v-card>
</template>
```

### Loading External Component Messages

```typescript
// composables/useComponentI18n.ts
export function useComponentI18n(componentName: string) {
  const { locale } = useI18n();
  const messages = ref<Record<string, string>>({});
  const loading = ref(true);

  async function loadMessages() {
    loading.value = true;
    try {
      const module = await import(
        `~/locales/components/${componentName}.${locale.value}.json`
      );
      messages.value = module.default;
    } catch {
      // Fallback to English
      const fallback = await import(
        `~/locales/components/${componentName}.en.json`
      );
      messages.value = fallback.default;
    } finally {
      loading.value = false;
    }
  }

  watch(locale, loadMessages, { immediate: true });

  function t(key: string, params?: Record<string, any>): string {
    let message = messages.value[key] || key;
    if (params) {
      Object.entries(params).forEach(([k, v]) => {
        message = message.replace(`{${k}}`, String(v));
      });
    }
    return message;
  }

  return { t, loading, messages };
}
```

---

## SEO and Meta Tags

### Localized Meta Tags

```vue
<!-- pages/index.vue -->
<script setup lang="ts">
const { t, locale } = useI18n();
const localePath = useLocalePath();

// SEO meta tags
useHead({
  title: t("pages.home.title"),
  meta: [
    { name: "description", content: t("pages.home.description") },
    { property: "og:title", content: t("pages.home.title") },
    { property: "og:description", content: t("pages.home.description") },
    { property: "og:locale", content: locale.value },
  ],
  link: [
    // Alternate language links
    { rel: "alternate", hreflang: "en", href: localePath("/", "en") },
    { rel: "alternate", hreflang: "es", href: localePath("/", "es") },
    { rel: "alternate", hreflang: "fr", href: localePath("/", "fr") },
    { rel: "alternate", hreflang: "x-default", href: localePath("/", "en") },
  ],
});

// Structured data
useSchemaOrg([
  defineWebPage({
    name: t("pages.home.title"),
    description: t("pages.home.description"),
    inLanguage: locale.value,
  }),
]);
</script>
```

### Localized Sitemap

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  sitemap: {
    i18n: {
      locales: ["en", "es", "fr", "de"],
      defaultLocale: "en",
    },
  },
});
```

---

## RTL Support

### RTL Configuration

```typescript
// plugins/vuetify.ts
import { createVuetify } from "vuetify";

export default defineNuxtPlugin((nuxtApp) => {
  const { locale } = useI18n();

  const vuetify = createVuetify({
    // RTL support
    locale: {
      rtl: {
        ar: true,
        he: true,
        fa: true,
      },
    },
  });

  // Watch for locale changes
  watch(
    locale,
    (newLocale) => {
      const isRtl = ["ar", "he", "fa"].includes(newLocale);
      document.documentElement.dir = isRtl ? "rtl" : "ltr";
      document.documentElement.lang = newLocale;
    },
    { immediate: true }
  );

  nuxtApp.vueApp.use(vuetify);
});
```

### RTL-Aware Styles

```scss
// assets/styles/rtl.scss

// RTL-specific styles
[dir="rtl"] {
  // Flip margins and paddings
  .ml-auto {
    margin-left: unset !important;
    margin-right: auto !important;
  }

  .mr-auto {
    margin-right: unset !important;
    margin-left: auto !important;
  }

  // Flip text alignment
  .text-left {
    text-align: right !important;
  }

  .text-right {
    text-align: left !important;
  }

  // Flip icons
  .flip-rtl {
    transform: scaleX(-1);
  }
}
```

---

## Testing i18n

### Unit Tests

```typescript
// tests/unit/i18n/translations.test.ts
import { describe, it, expect } from "vitest";
import en from "~/locales/en.json";
import es from "~/locales/es.json";

describe("Translations", () => {
  it("has all required keys in English", () => {
    // Arrange
    const requiredKeys = [
      "common.loading",
      "common.error",
      "auth.login.title",
      "auth.register.title",
    ];

    // Act & Assert
    requiredKeys.forEach((key) => {
      const value = key.split(".").reduce((obj, k) => obj?.[k], en as any);
      expect(value, `Missing key: ${key}`).toBeDefined();
    });
  });

  it("has matching keys between English and Spanish", () => {
    // Arrange
    function getKeys(obj: any, prefix = ""): string[] {
      return Object.entries(obj).flatMap(([key, value]) => {
        const fullKey = prefix ? `${prefix}.${key}` : key;
        if (typeof value === "object" && value !== null) {
          return getKeys(value, fullKey);
        }
        return [fullKey];
      });
    }

    // Act
    const enKeys = getKeys(en);
    const esKeys = getKeys(es);

    // Assert
    const missingInEs = enKeys.filter((k) => !esKeys.includes(k));
    expect(missingInEs, "Missing keys in Spanish").toEqual([]);
  });

  it("has valid pluralization format", () => {
    // Arrange
    const pluralKeys = ["plurals.items", "plurals.users", "plurals.messages"];

    // Act & Assert
    pluralKeys.forEach((key) => {
      const value = key.split(".").reduce((obj, k) => obj?.[k], en as any);
      expect(value).toContain("|");
    });
  });
});
```

### Component Tests with i18n

```typescript
// tests/components/LanguageSelector.test.ts
import { describe, it, expect } from "vitest";
import { mount } from "@vue/test-utils";
import { createI18n } from "vue-i18n";
import { createVuetify } from "vuetify";
import LanguageSelector from "~/components/common/LanguageSelector.vue";

describe("LanguageSelector", () => {
  const i18n = createI18n({
    legacy: false,
    locale: "en",
    messages: {
      en: { "user.settings.language": "Language" },
      es: { "user.settings.language": "Idioma" },
    },
  });

  const vuetify = createVuetify();

  function mountComponent() {
    return mount(LanguageSelector, {
      global: {
        plugins: [i18n, vuetify],
      },
    });
  }

  it("displays current locale", () => {
    // Arrange & Act
    const wrapper = mountComponent();

    // Assert
    expect(wrapper.text()).toContain("English");
  });

  it("shows available locales in dropdown", async () => {
    // Arrange
    const wrapper = mountComponent();

    // Act
    await wrapper.find("button").trigger("click");

    // Assert
    expect(wrapper.text()).toContain("Español");
  });
});
```

---

## Best Practices

### 1. Use Namespaced Keys

```json
// ✅ Good: Namespaced keys
{
  "auth": {
    "login": {
      "title": "Sign In",
      "submit": "Sign In"
    }
  }
}

// ❌ Bad: Flat keys
{
  "loginTitle": "Sign In",
  "loginSubmit": "Sign In"
}
```

### 2. Avoid Concatenation

```vue
<!-- ✅ Good: Use interpolation -->
<p>{{ $t('greeting', { name: userName }) }}</p>

<!-- ❌ Bad: String concatenation -->
<p>{{ $t('hello') + ' ' + userName }}</p>
```

### 3. Use ICU Message Format for Complex Plurals

```json
{
  "items": "{count, plural, =0 {No items} one {# item} other {# items}}"
}
```

### 4. Keep Translations Consistent

```typescript
// Create a translation key checker script
// scripts/check-translations.ts
import en from "../locales/en.json";
import es from "../locales/es.json";

function checkMissingKeys(source: any, target: any, path = ""): string[] {
  const missing: string[] = [];

  for (const key of Object.keys(source)) {
    const fullPath = path ? `${path}.${key}` : key;

    if (!(key in target)) {
      missing.push(fullPath);
    } else if (typeof source[key] === "object") {
      missing.push(...checkMissingKeys(source[key], target[key], fullPath));
    }
  }

  return missing;
}

const missingInEs = checkMissingKeys(en, es);
if (missingInEs.length > 0) {
  console.error("Missing translations in Spanish:", missingInEs);
  process.exit(1);
}
```

### 5. Lazy Load Large Locale Files

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  i18n: {
    lazy: true,
    langDir: "locales",
    // Only load needed locale
    defaultLocale: "en",
  },
});
```

---

## Next Steps

Continue to [Testing Strategy](./11-testing-strategy.md) to learn about unit and integration testing patterns with Vitest and @nuxt/test-utils.
