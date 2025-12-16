# Routing & Navigation

## Overview

Nuxt provides a powerful file-based routing system that automatically generates routes based on the directory structure in the `pages/` directory. This document covers routing patterns, middleware guards, authentication flow, and navigation patterns using Vuetify components.

## File-Based Routing

### Basic Route Structure

```
app/pages/
├── index.vue              # /
├── about.vue              # /about
├── contact.vue            # /contact
├── login.vue              # /login
├── register.vue           # /register
├── dashboard/
│   ├── index.vue          # /dashboard
│   ├── settings.vue       # /dashboard/settings
│   └── analytics.vue      # /dashboard/analytics
├── users/
│   ├── index.vue          # /users
│   └── [id].vue           # /users/:id (dynamic)
├── products/
│   ├── index.vue          # /products
│   ├── [id].vue           # /products/:id
│   └── [id]/
│       └── edit.vue       # /products/:id/edit
└── [...slug].vue          # Catch-all route (404)
```

### Route Naming Conventions

| Pattern     | File            | Route         | Example URL      |
| ----------- | --------------- | ------------- | ---------------- |
| Static      | `about.vue`     | `/about`      | `/about`         |
| Index       | `index.vue`     | `/`           | `/`              |
| Dynamic     | `[id].vue`      | `/:id`        | `/123`           |
| Optional    | `[[id]].vue`    | `/:id?`       | `/` or `/123`    |
| Catch-all   | `[...slug].vue` | `/:slug(.*)*` | `/any/path/here` |
| Named param | `user-[id].vue` | `/user-:id`   | `/user-123`      |

---

## Dynamic Routes

### Single Parameter

```vue
<!-- pages/users/[id].vue -->
<script setup lang="ts">
const route = useRoute();

// Type-safe parameter access
const userId = computed(() => route.params.id as string);

// Fetch user data
const {
  data: user,
  pending,
  error,
} = await useAsyncData(`user-${userId.value}`, () =>
  $fetch(`/api/users/${userId.value}`)
);
</script>

<template>
  <v-container>
    <v-skeleton-loader v-if="pending" type="card" />

    <v-alert v-else-if="error" type="error" variant="tonal">
      {{ error.message }}
    </v-alert>

    <v-card v-else>
      <v-card-title>{{ user?.name }}</v-card-title>
      <v-card-subtitle>{{ user?.email }}</v-card-subtitle>
      <v-card-text>
        <p>Role: {{ user?.role }}</p>
      </v-card-text>
    </v-card>
  </v-container>
</template>
```

### Multiple Parameters

```vue
<!-- pages/[category]/[product].vue -->
<script setup lang="ts">
const route = useRoute();

const category = computed(() => route.params.category as string);
const productSlug = computed(() => route.params.product as string);

const { data: product } = await useAsyncData(
  `product-${category.value}-${productSlug.value}`,
  () => $fetch(`/api/products/${category.value}/${productSlug.value}`)
);
</script>

<template>
  <v-container>
    <v-breadcrumbs
      :items="[
        { title: 'Home', to: '/' },
        { title: category, to: `/${category}` },
        { title: product?.name || '', disabled: true },
      ]"
    />

    <v-card v-if="product">
      <v-card-title>{{ product.name }}</v-card-title>
      <v-card-text>{{ product.description }}</v-card-text>
    </v-card>
  </v-container>
</template>
```

### Catch-All Routes (404)

```vue
<!-- pages/[...slug].vue -->
<script setup lang="ts">
const route = useRoute();

const segments = computed(() => route.params.slug as string[]);
const fullPath = computed(() => segments.value?.join("/") || "");

definePageMeta({
  layout: "default",
});
</script>

<template>
  <v-container class="fill-height">
    <v-row justify="center" align="center">
      <v-col cols="12" sm="8" md="6" class="text-center">
        <v-icon size="120" color="grey-lighten-1" class="mb-4">
          mdi-alert-circle-outline
        </v-icon>

        <h1 class="text-h3 mb-4">Page Not Found</h1>

        <p class="text-body-1 text-grey mb-6">
          The path "{{ fullPath }}" does not exist.
        </p>

        <v-btn color="primary" to="/" size="large">
          <v-icon start>mdi-home</v-icon>
          Go Home
        </v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>
```

---

## Page Metadata

### definePageMeta

```vue
<script setup lang="ts">
definePageMeta({
  // Layout to use
  layout: "dashboard",

  // Middleware to run
  middleware: ["auth", "admin"],

  // Custom page key for caching
  key: (route) => route.fullPath,

  // Keep-alive configuration
  keepalive: true,

  // Page transition
  pageTransition: {
    name: "slide",
    mode: "out-in",
  },

  // Custom meta data
  title: "Dashboard",
  requiresAuth: true,
  roles: ["admin", "manager"],
});
</script>
```

### SEO Metadata

```vue
<script setup lang="ts">
const route = useRoute();
const { data: product } = await useAsyncData(() =>
  fetchProduct(route.params.id)
);

useHead({
  title: () => product.value?.name || "Product",
  meta: [
    { name: "description", content: () => product.value?.description },
    { property: "og:title", content: () => product.value?.name },
    { property: "og:description", content: () => product.value?.description },
    { property: "og:image", content: () => product.value?.image },
  ],
});

useSeoMeta({
  title: () => product.value?.name,
  ogTitle: () => product.value?.name,
  description: () => product.value?.description,
  ogDescription: () => product.value?.description,
  ogImage: () => product.value?.image,
  twitterCard: "summary_large_image",
});
</script>
```

---

## Layouts

### Default Layout with Vuetify

```vue
<!-- layouts/default.vue -->
<script setup lang="ts">
const appStore = useAppStore();
const route = useRoute();

const drawer = ref(true);
</script>

<template>
  <v-app>
    <!-- App Bar -->
    <v-app-bar flat border>
      <v-app-bar-nav-icon @click="drawer = !drawer" />

      <v-app-bar-title>
        <NuxtLink to="/" class="text-decoration-none text-inherit">
          My App
        </NuxtLink>
      </v-app-bar-title>

      <v-spacer />

      <ThemeToggle />
      <UserMenu />
    </v-app-bar>

    <!-- Navigation Drawer -->
    <v-navigation-drawer v-model="drawer" :rail="appStore.settings.sidebarRail">
      <NavigationMenu />
    </v-navigation-drawer>

    <!-- Main Content -->
    <v-main>
      <slot />
    </v-main>

    <!-- Footer -->
    <v-footer app>
      <span>&copy; {{ new Date().getFullYear() }} My App</span>
    </v-footer>
  </v-app>
</template>
```

### Auth Layout (Minimal)

```vue
<!-- layouts/auth.vue -->
<template>
  <v-app>
    <v-main class="bg-grey-lighten-4">
      <v-container class="fill-height">
        <v-row justify="center" align="center">
          <v-col cols="12" sm="8" md="6" lg="4">
            <div class="text-center mb-8">
              <NuxtLink to="/">
                <v-img
                  src="/logo.svg"
                  alt="Logo"
                  max-width="120"
                  class="mx-auto"
                />
              </NuxtLink>
            </div>

            <slot />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>
```

### Dashboard Layout

```vue
<!-- layouts/dashboard.vue -->
<script setup lang="ts">
const appStore = useAppStore();
const userStore = useUserStore();

const drawer = ref(true);
const rail = computed({
  get: () => appStore.settings.sidebarRail,
  set: (value) => appStore.updateSettings({ sidebarRail: value }),
});
</script>

<template>
  <v-app>
    <!-- App Bar -->
    <v-app-bar flat border>
      <v-app-bar-nav-icon @click="drawer = !drawer" />

      <v-breadcrumbs :items="appStore.breadcrumbs" />

      <v-spacer />

      <v-btn icon variant="text">
        <v-badge color="error" :content="3" :model-value="true">
          <v-icon>mdi-bell-outline</v-icon>
        </v-badge>
      </v-btn>

      <ThemeToggle />
      <UserMenu />
    </v-app-bar>

    <!-- Navigation Drawer -->
    <v-navigation-drawer v-model="drawer" :rail="rail">
      <v-list nav>
        <v-list-item
          prepend-avatar="https://randomuser.me/api/portraits/men/1.jpg"
          :title="userStore.displayName"
          :subtitle="userStore.currentUser?.email"
        />
      </v-list>

      <v-divider />

      <DashboardNavigation />

      <template #append>
        <v-list nav density="compact">
          <v-list-item
            prepend-icon="mdi-chevron-left"
            title="Collapse"
            @click="rail = !rail"
          />
        </v-list>
      </template>
    </v-navigation-drawer>

    <!-- Main Content -->
    <v-main>
      <v-container fluid>
        <slot />
      </v-container>
    </v-main>
  </v-app>
</template>
```

### Using Layouts in Pages

```vue
<script setup lang="ts">
// Specify layout in definePageMeta
definePageMeta({
  layout: "dashboard",
});
</script>
```

---

## Middleware

### Middleware Directory Structure

```
app/middleware/
├── auth.ts                 # Authentication check
├── guest.ts                # Guest-only routes
├── admin.ts                # Admin permission check
├── role.ts                 # Role-based access
├── setup.global.ts         # Global middleware (automatic)
└── analytics.global.ts     # Analytics tracking (automatic)
```

### Authentication Middleware

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const authStore = useAuthStore();

  // Check authentication
  if (!authStore.isAuthenticated) {
    // Save intended destination
    const redirectPath = to.fullPath;

    return navigateTo({
      path: "/login",
      query: { redirect: redirectPath },
    });
  }
});
```

### Guest-Only Middleware

```typescript
// middleware/guest.ts
export default defineNuxtRouteMiddleware(() => {
  const authStore = useAuthStore();

  // Redirect authenticated users away from guest pages
  if (authStore.isAuthenticated) {
    return navigateTo("/dashboard");
  }
});
```

### Admin Middleware

```typescript
// middleware/admin.ts
export default defineNuxtRouteMiddleware((to) => {
  const userStore = useUserStore();

  if (!userStore.isAdmin) {
    // Show access denied or redirect
    return abortNavigation({
      statusCode: 403,
      statusMessage: "Access Denied",
      message: "You do not have permission to access this page.",
    });
  }
});
```

### Role-Based Middleware

```typescript
// middleware/role.ts
export default defineNuxtRouteMiddleware((to) => {
  const userStore = useUserStore();
  const requiredRoles = to.meta.roles as string[] | undefined;

  if (requiredRoles && requiredRoles.length > 0) {
    const userRole = userStore.currentUser?.role || "";
    const hasRole = requiredRoles.includes(userRole);

    if (!hasRole) {
      return abortNavigation({
        statusCode: 403,
        statusMessage: "Access Denied",
        message: `This page requires one of the following roles: ${requiredRoles.join(
          ", "
        )}`,
      });
    }
  }
});
```

### Global Setup Middleware

```typescript
// middleware/setup.global.ts
export default defineNuxtRouteMiddleware(async () => {
  const authStore = useAuthStore();

  // Initialize auth state on first load
  if (!authStore.isInitialized) {
    await authStore.initialize();
  }
});
```

### Analytics Middleware

```typescript
// middleware/analytics.global.ts
export default defineNuxtRouteMiddleware((to, from) => {
  // Track page views (client-side only)
  if (import.meta.client && from.path !== to.path) {
    // Send to analytics service
    trackPageView({
      path: to.path,
      title: to.meta.title as string,
      referrer: from.fullPath,
    });
  }
});
```

### Using Middleware in Pages

```vue
<script setup lang="ts">
// Single middleware
definePageMeta({
  middleware: "auth",
});

// Multiple middleware (executed in order)
definePageMeta({
  middleware: ["auth", "admin"],
});

// Inline middleware
definePageMeta({
  middleware: [
    function (to, from) {
      // Custom logic
      if (someCondition) {
        return navigateTo("/other-page");
      }
    },
  ],
});

// With role requirements
definePageMeta({
  middleware: ["auth", "role"],
  roles: ["admin", "manager"],
});
</script>
```

---

## Authentication Flow

### Login Page

```vue
<!-- pages/login.vue -->
<script setup lang="ts">
definePageMeta({
  layout: "auth",
  middleware: "guest",
});

const authStore = useAuthStore();
const route = useRoute();
const router = useRouter();

const form = ref({
  email: "",
  password: "",
  remember: false,
});

const loading = ref(false);
const error = ref<string | null>(null);

async function handleLogin() {
  loading.value = true;
  error.value = null;

  try {
    await authStore.login({
      email: form.value.email,
      password: form.value.password,
      remember: form.value.remember,
    });

    // Redirect to intended destination or dashboard
    const redirect = (route.query.redirect as string) || "/dashboard";
    await router.push(redirect);
  } catch (err: any) {
    error.value = err.data?.message || "Invalid credentials";
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <v-card>
    <v-card-title class="text-h5 text-center py-4"> Sign In </v-card-title>

    <v-card-text>
      <v-alert
        v-if="error"
        type="error"
        variant="tonal"
        closable
        class="mb-4"
        @click:close="error = null"
      >
        {{ error }}
      </v-alert>

      <v-form @submit.prevent="handleLogin">
        <v-text-field
          v-model="form.email"
          label="Email"
          type="email"
          prepend-inner-icon="mdi-email-outline"
          variant="outlined"
          required
          class="mb-4"
        />

        <v-text-field
          v-model="form.password"
          label="Password"
          type="password"
          prepend-inner-icon="mdi-lock-outline"
          variant="outlined"
          required
          class="mb-4"
        />

        <div class="d-flex justify-space-between align-center mb-4">
          <v-checkbox
            v-model="form.remember"
            label="Remember me"
            hide-details
          />

          <NuxtLink to="/forgot-password" class="text-primary">
            Forgot password?
          </NuxtLink>
        </div>

        <v-btn
          type="submit"
          color="primary"
          size="large"
          block
          :loading="loading"
        >
          Sign In
        </v-btn>
      </v-form>
    </v-card-text>

    <v-divider />

    <v-card-text class="text-center">
      Don't have an account?
      <NuxtLink to="/register" class="text-primary font-weight-medium">
        Sign Up
      </NuxtLink>
    </v-card-text>
  </v-card>
</template>
```

### Register Page

```vue
<!-- pages/register.vue -->
<script setup lang="ts">
definePageMeta({
  layout: "auth",
  middleware: "guest",
});

const authStore = useAuthStore();
const router = useRouter();

const form = ref({
  name: "",
  email: "",
  password: "",
  passwordConfirmation: "",
});

const loading = ref(false);
const error = ref<string | null>(null);

async function handleRegister() {
  if (form.value.password !== form.value.passwordConfirmation) {
    error.value = "Passwords do not match";
    return;
  }

  loading.value = true;
  error.value = null;

  try {
    await authStore.register(form.value);
    await router.push("/dashboard");
  } catch (err: any) {
    error.value = err.data?.message || "Registration failed";
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <v-card>
    <v-card-title class="text-h5 text-center py-4">
      Create Account
    </v-card-title>

    <v-card-text>
      <v-alert
        v-if="error"
        type="error"
        variant="tonal"
        closable
        class="mb-4"
        @click:close="error = null"
      >
        {{ error }}
      </v-alert>

      <v-form @submit.prevent="handleRegister">
        <v-text-field
          v-model="form.name"
          label="Full Name"
          prepend-inner-icon="mdi-account-outline"
          variant="outlined"
          required
          class="mb-4"
        />

        <v-text-field
          v-model="form.email"
          label="Email"
          type="email"
          prepend-inner-icon="mdi-email-outline"
          variant="outlined"
          required
          class="mb-4"
        />

        <v-text-field
          v-model="form.password"
          label="Password"
          type="password"
          prepend-inner-icon="mdi-lock-outline"
          variant="outlined"
          required
          class="mb-4"
        />

        <v-text-field
          v-model="form.passwordConfirmation"
          label="Confirm Password"
          type="password"
          prepend-inner-icon="mdi-lock-check-outline"
          variant="outlined"
          required
          class="mb-4"
        />

        <v-btn
          type="submit"
          color="primary"
          size="large"
          block
          :loading="loading"
        >
          Create Account
        </v-btn>
      </v-form>
    </v-card-text>

    <v-divider />

    <v-card-text class="text-center">
      Already have an account?
      <NuxtLink to="/login" class="text-primary font-weight-medium">
        Sign In
      </NuxtLink>
    </v-card-text>
  </v-card>
</template>
```

### Protected Dashboard Page

```vue
<!-- pages/dashboard/index.vue -->
<script setup lang="ts">
definePageMeta({
  layout: "dashboard",
  middleware: "auth",
});

const userStore = useUserStore();
const appStore = useAppStore();

// Set page metadata
appStore.setPageTitle("Dashboard");
appStore.setBreadcrumbs([{ title: "Dashboard", to: "/dashboard" }]);

// Fetch dashboard data
const { data: stats, pending } = await useAsyncData("dashboard-stats", () =>
  $fetch("/api/dashboard/stats")
);
</script>

<template>
  <div>
    <h1 class="text-h4 mb-6">Welcome back, {{ userStore.displayName }}!</h1>

    <v-row v-if="pending">
      <v-col v-for="i in 4" :key="i" cols="12" sm="6" md="3">
        <v-skeleton-loader type="card" />
      </v-col>
    </v-row>

    <v-row v-else>
      <v-col cols="12" sm="6" md="3">
        <v-card>
          <v-card-text>
            <div class="text-overline">Total Users</div>
            <div class="text-h4">{{ stats?.users }}</div>
          </v-card-text>
        </v-card>
      </v-col>

      <!-- More stat cards... -->
    </v-row>
  </div>
</template>
```

---

## Navigation Components

### Navigation Menu

```vue
<!-- components/layout/NavigationMenu.vue -->
<script setup lang="ts">
const route = useRoute();
const userStore = useUserStore();

interface NavItem {
  title: string;
  icon: string;
  to: string;
  roles?: string[];
  children?: NavItem[];
}

const navItems: NavItem[] = [
  { title: "Dashboard", icon: "mdi-view-dashboard", to: "/dashboard" },
  { title: "Users", icon: "mdi-account-group", to: "/users", roles: ["admin"] },
  { title: "Products", icon: "mdi-package-variant", to: "/products" },
  { title: "Orders", icon: "mdi-cart", to: "/orders" },
  {
    title: "Settings",
    icon: "mdi-cog",
    to: "/settings",
    children: [
      { title: "Profile", icon: "mdi-account", to: "/settings/profile" },
      { title: "Security", icon: "mdi-shield", to: "/settings/security" },
      {
        title: "Notifications",
        icon: "mdi-bell",
        to: "/settings/notifications",
      },
    ],
  },
];

// Filter items based on user role
const filteredNavItems = computed(() => {
  return navItems.filter((item) => {
    if (!item.roles) return true;
    return item.roles.includes(userStore.currentUser?.role || "");
  });
});

function isActive(item: NavItem): boolean {
  if (item.children) {
    return item.children.some((child) => route.path.startsWith(child.to));
  }
  return route.path.startsWith(item.to);
}
</script>

<template>
  <v-list nav density="compact">
    <template v-for="item in filteredNavItems" :key="item.to">
      <!-- Item with children -->
      <v-list-group v-if="item.children" :value="item.title">
        <template #activator="{ props }">
          <v-list-item
            v-bind="props"
            :prepend-icon="item.icon"
            :title="item.title"
          />
        </template>

        <v-list-item
          v-for="child in item.children"
          :key="child.to"
          :to="child.to"
          :prepend-icon="child.icon"
          :title="child.title"
          :active="route.path === child.to"
        />
      </v-list-group>

      <!-- Simple item -->
      <v-list-item
        v-else
        :to="item.to"
        :prepend-icon="item.icon"
        :title="item.title"
        :active="isActive(item)"
      />
    </template>
  </v-list>
</template>
```

### User Menu

```vue
<!-- components/layout/UserMenu.vue -->
<script setup lang="ts">
const authStore = useAuthStore();
const userStore = useUserStore();
const router = useRouter();

async function handleLogout() {
  await authStore.logout();
}
</script>

<template>
  <v-menu>
    <template #activator="{ props }">
      <v-btn v-bind="props" icon variant="text">
        <v-avatar size="32" color="primary">
          <v-img
            v-if="userStore.currentUser?.avatar"
            :src="userStore.currentUser.avatar"
            :alt="userStore.displayName"
          />
          <span v-else class="text-body-2">
            {{ userStore.userInitials }}
          </span>
        </v-avatar>
      </v-btn>
    </template>

    <v-list min-width="200">
      <v-list-item
        :title="userStore.displayName"
        :subtitle="userStore.currentUser?.email"
      >
        <template #prepend>
          <v-avatar color="primary" size="40">
            <v-img
              v-if="userStore.currentUser?.avatar"
              :src="userStore.currentUser.avatar"
            />
            <span v-else>{{ userStore.userInitials }}</span>
          </v-avatar>
        </template>
      </v-list-item>

      <v-divider />

      <v-list-item
        to="/settings/profile"
        prepend-icon="mdi-account-outline"
        title="Profile"
      />

      <v-list-item
        to="/settings"
        prepend-icon="mdi-cog-outline"
        title="Settings"
      />

      <v-divider />

      <v-list-item
        prepend-icon="mdi-logout"
        title="Sign Out"
        @click="handleLogout"
      />
    </v-list>
  </v-menu>
</template>
```

### Breadcrumbs Component

```vue
<!-- components/layout/AppBreadcrumbs.vue -->
<script setup lang="ts">
const route = useRoute();

interface Breadcrumb {
  title: string;
  to?: string;
  disabled?: boolean;
}

const breadcrumbs = computed<Breadcrumb[]>(() => {
  const paths = route.path.split("/").filter(Boolean);

  const items: Breadcrumb[] = [{ title: "Home", to: "/" }];

  paths.forEach((segment, index) => {
    const path = "/" + paths.slice(0, index + 1).join("/");
    const title =
      segment.charAt(0).toUpperCase() + segment.slice(1).replace(/-/g, " ");

    items.push({
      title,
      to: index < paths.length - 1 ? path : undefined,
      disabled: index === paths.length - 1,
    });
  });

  return items;
});
</script>

<template>
  <v-breadcrumbs :items="breadcrumbs">
    <template #item="{ item }">
      <v-breadcrumbs-item :to="item.to" :disabled="item.disabled">
        {{ item.title }}
      </v-breadcrumbs-item>
    </template>
  </v-breadcrumbs>
</template>
```

---

## Programmatic Navigation

### Using navigateTo

```typescript
// Using navigateTo
async function goToUser(id: string) {
  await navigateTo(`/users/${id}`);
}

async function goToSearch(query: string) {
  await navigateTo({
    path: "/search",
    query: { q: query },
  });
}

// With options
async function goToDashboard() {
  await navigateTo("/dashboard", {
    replace: true, // Replace current history entry
    redirectCode: 301, // For SSR redirects
    external: false, // External URL
    open: {
      // Window.open options
      target: "_blank",
    },
  });
}
```

### Using useRouter

```typescript
const router = useRouter();

function goBack() {
  router.back();
}

function goForward() {
  router.forward();
}

async function navigate() {
  await router.push("/new-page");
}

async function replaceRoute() {
  await router.replace("/new-page");
}
```

### Navigation Guards in Components

```vue
<script setup lang="ts">
import { onBeforeRouteLeave, onBeforeRouteUpdate } from "vue-router";

const hasUnsavedChanges = ref(false);

// Confirm before leaving with unsaved changes
onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges.value) {
    const answer = window.confirm("You have unsaved changes. Leave anyway?");
    if (!answer) {
      return false; // Cancel navigation
    }
  }
});

// Handle route param changes
onBeforeRouteUpdate(async (to, from) => {
  if (to.params.id !== from.params.id) {
    await fetchData(to.params.id as string);
  }
});
</script>
```

---

## Route Validation

### validate Function

```vue
<script setup lang="ts">
definePageMeta({
  validate: async (route) => {
    const id = route.params.id as string;

    // Check if valid format
    if (!/^\d+$/.test(id)) {
      return false; // Returns 404
    }

    // Check if resource exists
    try {
      await $fetch(`/api/users/${id}`);
      return true;
    } catch {
      return {
        statusCode: 404,
        statusMessage: "User not found",
      };
    }
  },
});
</script>
```

### Custom Error Page

```vue
<!-- error.vue (root level) -->
<script setup lang="ts">
interface Props {
  error: {
    statusCode: number;
    statusMessage: string;
    message?: string;
  };
}

const props = defineProps<Props>();

const handleError = () => {
  clearError({ redirect: "/" });
};

const errorContent = computed(() => {
  switch (props.error.statusCode) {
    case 404:
      return {
        title: "Page Not Found",
        message: "The page you are looking for does not exist.",
        icon: "mdi-file-search-outline",
        color: "warning",
      };
    case 403:
      return {
        title: "Access Denied",
        message: "You do not have permission to view this page.",
        icon: "mdi-shield-alert-outline",
        color: "error",
      };
    case 500:
      return {
        title: "Server Error",
        message: "Something went wrong on our end.",
        icon: "mdi-server-off",
        color: "error",
      };
    default:
      return {
        title: "Error",
        message: props.error.message || "An unexpected error occurred.",
        icon: "mdi-alert-circle-outline",
        color: "error",
      };
  }
});
</script>

<template>
  <v-app>
    <v-main class="bg-grey-lighten-4">
      <v-container class="fill-height">
        <v-row justify="center" align="center">
          <v-col cols="12" sm="8" md="6" class="text-center">
            <v-icon
              :icon="errorContent.icon"
              :color="errorContent.color"
              size="120"
              class="mb-4"
            />

            <h1 class="text-h3 mb-2">{{ error.statusCode }}</h1>
            <h2 class="text-h5 mb-4">{{ errorContent.title }}</h2>
            <p class="text-body-1 text-grey mb-6">
              {{ errorContent.message }}
            </p>

            <v-btn color="primary" size="large" @click="handleError">
              <v-icon start>mdi-home</v-icon>
              Go Home
            </v-btn>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
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
    "/": { prerender: true },
    "/about": { prerender: true },
    "/blog/**": { prerender: true },

    // Client-side only (SPA)
    "/dashboard/**": { ssr: false },
    "/admin/**": { ssr: false },

    // ISR (Incremental Static Regeneration)
    "/products/**": { swr: 3600 },
    "/news/**": { isr: 600 },

    // Redirects
    "/old-page": { redirect: "/new-page" },

    // Headers
    "/api/**": {
      cors: true,
      headers: {
        "Access-Control-Allow-Methods": "GET,POST,PUT,DELETE",
      },
    },

    // Cache control
    "/_nuxt/**": {
      headers: {
        "Cache-Control": "public, max-age=31536000, immutable",
      },
    },
  },
});
```

---

## Page Transitions

### Configure Transitions

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    pageTransition: { name: "page", mode: "out-in" },
    layoutTransition: { name: "layout", mode: "out-in" },
  },
});
```

### Transition Styles

```css
/* assets/styles/transitions.css */

/* Page transition */
.page-enter-active,
.page-leave-active {
  transition: all 0.3s;
}

.page-enter-from,
.page-leave-to {
  opacity: 0;
  transform: translateY(20px);
}

/* Slide transition */
.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
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
  transition: opacity 0.2s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
```

### Per-Page Transitions

```vue
<script setup lang="ts">
definePageMeta({
  pageTransition: {
    name: "slide",
    mode: "out-in",
  },
});
</script>
```

---

## Testing Routes and Middleware

### Middleware Testing

```typescript
// tests/unit/middleware/auth.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { setActivePinia, createPinia } from "pinia";

describe("auth middleware", () => {
  beforeEach(() => {
    setActivePinia(createPinia());
  });

  it("redirects unauthenticated users to login", async () => {
    // Arrange
    const authStore = useAuthStore();
    const navigateTo = vi.fn();
    vi.stubGlobal("navigateTo", navigateTo);

    const to = { fullPath: "/dashboard" };
    const from = { fullPath: "/" };

    // Act
    const middleware = await import("~/middleware/auth").then((m) => m.default);
    await middleware(to as any, from as any);

    // Assert
    expect(navigateTo).toHaveBeenCalledWith({
      path: "/login",
      query: { redirect: "/dashboard" },
    });
  });

  it("allows authenticated users", async () => {
    // Arrange
    const authStore = useAuthStore();
    // Mock authenticated state
    vi.mocked($fetch).mockResolvedValueOnce({
      tokens: {
        accessToken: "token",
        refreshToken: "refresh",
        expiresAt: Date.now() + 3600000,
      },
      user: { id: "1", name: "Test", email: "test@example.com", role: "user" },
    });
    await authStore.login({ email: "test@example.com", password: "password" });

    const navigateTo = vi.fn();
    vi.stubGlobal("navigateTo", navigateTo);

    const to = { fullPath: "/dashboard" };
    const from = { fullPath: "/" };

    // Act
    const middleware = await import("~/middleware/auth").then((m) => m.default);
    const result = await middleware(to as any, from as any);

    // Assert
    expect(navigateTo).not.toHaveBeenCalled();
    expect(result).toBeUndefined();
  });
});
```

### Navigation Testing

```typescript
// tests/integration/navigation.test.ts
import { describe, it, expect } from "vitest";
import { mountSuspended } from "@nuxt/test-utils/runtime";

describe("Navigation", () => {
  it("navigates to dashboard after login", async () => {
    // Arrange
    const wrapper = await mountSuspended(LoginPage);
    const router = useRouter();

    // Act
    await wrapper
      .find('[data-testid="email-input"]')
      .setValue("test@example.com");
    await wrapper.find('[data-testid="password-input"]').setValue("password");
    await wrapper.find('[data-testid="login-button"]').trigger("click");

    // Assert
    await vi.waitFor(() => {
      expect(router.currentRoute.value.path).toBe("/dashboard");
    });
  });
});
```

---

## Best Practices

### 1. Use NuxtLink for Internal Links

```vue
<!-- ✅ Good: Use NuxtLink -->
<NuxtLink to="/about">About</NuxtLink>

<!-- ❌ Bad: Use anchor tags -->
<a href="/about">About</a>
```

### 2. Protect Routes at Multiple Levels

```typescript
// 1. Middleware for route protection
definePageMeta({
  middleware: ['auth', 'admin'],
})

// 2. Component-level checks for UI
<template>
  <v-btn v-if="userStore.isAdmin" to="/admin">
    Admin Panel
  </v-btn>
</template>

// 3. API-level validation
export default defineEventHandler(async (event) => {
  const user = await requireAuth(event)
  if (user.role !== 'admin') {
    throw createError({ statusCode: 403 })
  }
})
```

### 3. Handle Loading States

```vue
<script setup lang="ts">
const nuxtApp = useNuxtApp();
const isLoading = ref(false);

nuxtApp.hook("page:start", () => {
  isLoading.value = true;
});

nuxtApp.hook("page:finish", () => {
  isLoading.value = false;
});
</script>

<template>
  <v-app>
    <v-progress-linear
      v-if="isLoading"
      indeterminate
      color="primary"
      class="position-fixed"
      style="top: 0; z-index: 9999;"
    />
    <NuxtPage />
  </v-app>
</template>
```

---

## Next Steps

Continue to [Vuetify Theming](./08-vuetify-theming.md) to learn about theme customization, dark mode implementation, and dynamic theming with Vuetify 3.
