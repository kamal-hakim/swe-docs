# State Management

## Overview

This document covers state management patterns for Nuxt applications using **Pinia**, Vue's official state management library. It includes store architecture, TypeScript patterns, common use cases, and best practices for both SPA and SSR environments.

## State Management Options

### When to Use Each Approach

| Approach                       | Use Case                    |
| ------------------------------ | --------------------------- |
| **Local State (ref/reactive)** | Component-specific state    |
| **Props/Emits**                | Parent-child communication  |
| **Provide/Inject**             | Deep component tree sharing |
| **Composables**                | Reusable logic with state   |
| **useState (Nuxt)**            | SSR-safe shared state       |
| **Pinia Stores**               | Global application state    |

---

## Pinia Setup

### Installation and Configuration

```bash
# Install Pinia (usually included with @pinia/nuxt)
pnpm add pinia @pinia/nuxt
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ["@pinia/nuxt"],

  pinia: {
    storesDirs: ["./app/stores/**"],
  },
});
```

### Store Directory Structure

```
app/stores/
├── auth.ts                     # Authentication state
├── user.ts                     # User profile state
├── app.ts                      # Application settings
├── notification.ts             # Notifications
├── cart.ts                     # Shopping cart (example)
└── index.ts                    # Store exports (optional)
```

---

## Store Patterns with TypeScript

### Setup Store Pattern (Recommended)

The Setup Store pattern uses Vue's Composition API syntax, providing better TypeScript support and more flexibility.

```typescript
// stores/user.ts
import { defineStore } from "pinia";

// Types
interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  role: "admin" | "user" | "guest";
  createdAt: string;
}

interface UserPreferences {
  theme: "light" | "dark" | "system";
  language: string;
  notifications: boolean;
  itemsPerPage: number;
}

interface UserState {
  currentUser: User | null;
  preferences: UserPreferences;
  loading: boolean;
  error: Error | null;
}

// Default values
const defaultPreferences: UserPreferences = {
  theme: "system",
  language: "en",
  notifications: true,
  itemsPerPage: 10,
};

export const useUserStore = defineStore("user", () => {
  // ==========================================
  // State
  // ==========================================
  const currentUser = ref<User | null>(null);
  const preferences = ref<UserPreferences>({ ...defaultPreferences });
  const loading = ref(false);
  const error = ref<Error | null>(null);

  // ==========================================
  // Getters (Computed)
  // ==========================================
  const isAuthenticated = computed(() => !!currentUser.value);

  const isAdmin = computed(() => currentUser.value?.role === "admin");

  const displayName = computed(() => currentUser.value?.name || "Guest");

  const userInitials = computed(() => {
    const name = currentUser.value?.name || "";
    return name
      .split(" ")
      .map((n) => n[0])
      .join("")
      .toUpperCase()
      .slice(0, 2);
  });

  const avatarUrl = computed(() => {
    if (currentUser.value?.avatar) {
      return currentUser.value.avatar;
    }
    // Generate gravatar or placeholder
    return `https://ui-avatars.com/api/?name=${encodeURIComponent(
      displayName.value
    )}&background=random`;
  });

  // ==========================================
  // Actions
  // ==========================================
  async function fetchUser(): Promise<User | null> {
    if (currentUser.value) return currentUser.value;

    loading.value = true;
    error.value = null;

    try {
      const response = await $fetch<User>("/api/auth/me");
      currentUser.value = response;
      return response;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function updateProfile(updates: Partial<User>): Promise<User> {
    if (!currentUser.value) throw new Error("Not authenticated");

    loading.value = true;
    error.value = null;

    try {
      const response = await $fetch<User>("/api/user/profile", {
        method: "PATCH",
        body: updates,
      });
      currentUser.value = response;
      return response;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function updateAvatar(file: File): Promise<string> {
    if (!currentUser.value) throw new Error("Not authenticated");

    const formData = new FormData();
    formData.append("avatar", file);

    loading.value = true;
    error.value = null;

    try {
      const response = await $fetch<{ url: string }>("/api/user/avatar", {
        method: "POST",
        body: formData,
      });

      if (currentUser.value) {
        currentUser.value.avatar = response.url;
      }

      return response.url;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  function updatePreferences(updates: Partial<UserPreferences>): void {
    preferences.value = { ...preferences.value, ...updates };

    // Persist to localStorage
    if (import.meta.client) {
      localStorage.setItem(
        "user-preferences",
        JSON.stringify(preferences.value)
      );
    }
  }

  function loadPreferences(): void {
    if (import.meta.client) {
      const stored = localStorage.getItem("user-preferences");
      if (stored) {
        try {
          preferences.value = { ...defaultPreferences, ...JSON.parse(stored) };
        } catch {
          // Invalid JSON, use defaults
        }
      }
    }
  }

  function setUser(user: User | null): void {
    currentUser.value = user;
  }

  function clearUser(): void {
    currentUser.value = null;
    preferences.value = { ...defaultPreferences };
    error.value = null;
  }

  // ==========================================
  // Return
  // ==========================================
  return {
    // State
    currentUser: readonly(currentUser),
    preferences,
    loading: readonly(loading),
    error: readonly(error),

    // Getters
    isAuthenticated,
    isAdmin,
    displayName,
    userInitials,
    avatarUrl,

    // Actions
    fetchUser,
    updateProfile,
    updateAvatar,
    updatePreferences,
    loadPreferences,
    setUser,
    clearUser,
  };
});

// Type export for use in components
export type UserStore = ReturnType<typeof useUserStore>;
```

---

## Authentication Store

```typescript
// stores/auth.ts
import { defineStore } from "pinia";

// Types
interface AuthCredentials {
  email: string;
  password: string;
  remember?: boolean;
}

interface RegisterData {
  name: string;
  email: string;
  password: string;
  passwordConfirmation: string;
}

interface AuthTokens {
  accessToken: string;
  refreshToken: string;
  expiresAt: number;
}

interface AuthResponse {
  tokens: AuthTokens;
  user: {
    id: string;
    name: string;
    email: string;
    role: "admin" | "user" | "guest";
  };
}

export const useAuthStore = defineStore("auth", () => {
  // Dependencies
  const userStore = useUserStore();
  const router = useRouter();

  // State
  const tokens = ref<AuthTokens | null>(null);
  const isInitialized = ref(false);
  const loading = ref(false);
  const error = ref<string | null>(null);

  // Computed
  const isAuthenticated = computed(() => {
    if (!tokens.value) return false;
    return tokens.value.expiresAt > Date.now();
  });

  const accessToken = computed(() => tokens.value?.accessToken);

  const isTokenExpiringSoon = computed(() => {
    if (!tokens.value) return false;
    // Token expires in less than 5 minutes
    return tokens.value.expiresAt - Date.now() < 5 * 60 * 1000;
  });

  // Actions
  async function login(credentials: AuthCredentials): Promise<AuthResponse> {
    loading.value = true;
    error.value = null;

    try {
      const response = await $fetch<AuthResponse>("/api/auth/login", {
        method: "POST",
        body: credentials,
      });

      tokens.value = response.tokens;
      userStore.setUser(response.user);

      // Persist tokens
      if (import.meta.client) {
        const storage = credentials.remember ? localStorage : sessionStorage;
        storage.setItem("auth_tokens", JSON.stringify(response.tokens));
      }

      return response;
    } catch (err: any) {
      error.value = err.data?.message || "Login failed";
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function register(data: RegisterData): Promise<AuthResponse> {
    loading.value = true;
    error.value = null;

    try {
      const response = await $fetch<AuthResponse>("/api/auth/register", {
        method: "POST",
        body: data,
      });

      tokens.value = response.tokens;
      userStore.setUser(response.user);

      if (import.meta.client) {
        localStorage.setItem("auth_tokens", JSON.stringify(response.tokens));
      }

      return response;
    } catch (err: any) {
      error.value = err.data?.message || "Registration failed";
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function logout(redirect = true): Promise<void> {
    try {
      if (tokens.value) {
        await $fetch("/api/auth/logout", { method: "POST" });
      }
    } catch {
      // Ignore logout errors
    } finally {
      tokens.value = null;
      userStore.clearUser();

      if (import.meta.client) {
        localStorage.removeItem("auth_tokens");
        sessionStorage.removeItem("auth_tokens");
      }

      if (redirect) {
        await router.push("/login");
      }
    }
  }

  async function refreshTokens(): Promise<AuthTokens> {
    if (!tokens.value?.refreshToken) {
      throw new Error("No refresh token available");
    }

    try {
      const response = await $fetch<AuthTokens>("/api/auth/refresh", {
        method: "POST",
        body: { refreshToken: tokens.value.refreshToken },
      });

      tokens.value = response;

      if (import.meta.client) {
        // Update in whichever storage has the tokens
        if (localStorage.getItem("auth_tokens")) {
          localStorage.setItem("auth_tokens", JSON.stringify(response));
        } else if (sessionStorage.getItem("auth_tokens")) {
          sessionStorage.setItem("auth_tokens", JSON.stringify(response));
        }
      }

      return response;
    } catch (err) {
      // Refresh failed, logout user
      await logout();
      throw err;
    }
  }

  async function forgotPassword(email: string): Promise<void> {
    loading.value = true;
    error.value = null;

    try {
      await $fetch("/api/auth/forgot-password", {
        method: "POST",
        body: { email },
      });
    } catch (err: any) {
      error.value = err.data?.message || "Failed to send reset email";
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function resetPassword(token: string, password: string): Promise<void> {
    loading.value = true;
    error.value = null;

    try {
      await $fetch("/api/auth/reset-password", {
        method: "POST",
        body: { token, password },
      });
    } catch (err: any) {
      error.value = err.data?.message || "Failed to reset password";
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function initialize(): Promise<void> {
    if (isInitialized.value) return;

    if (import.meta.client) {
      // Try localStorage first, then sessionStorage
      const stored =
        localStorage.getItem("auth_tokens") ||
        sessionStorage.getItem("auth_tokens");

      if (stored) {
        try {
          const parsed = JSON.parse(stored) as AuthTokens;

          if (parsed.expiresAt > Date.now()) {
            tokens.value = parsed;
            await userStore.fetchUser();
          } else if (parsed.refreshToken) {
            // Token expired, try to refresh
            tokens.value = parsed;
            await refreshTokens();
            await userStore.fetchUser();
          } else {
            // No valid tokens, clean up
            localStorage.removeItem("auth_tokens");
            sessionStorage.removeItem("auth_tokens");
          }
        } catch {
          localStorage.removeItem("auth_tokens");
          sessionStorage.removeItem("auth_tokens");
        }
      }
    }

    isInitialized.value = true;
  }

  // Auto-refresh tokens before expiry
  let refreshInterval: ReturnType<typeof setInterval> | null = null;

  function startTokenRefresh(): void {
    if (import.meta.server) return;

    stopTokenRefresh();

    refreshInterval = setInterval(async () => {
      if (isTokenExpiringSoon.value && tokens.value?.refreshToken) {
        try {
          await refreshTokens();
        } catch {
          // Refresh failed, will be handled by logout
        }
      }
    }, 60 * 1000); // Check every minute
  }

  function stopTokenRefresh(): void {
    if (refreshInterval) {
      clearInterval(refreshInterval);
      refreshInterval = null;
    }
  }

  // Watch for authentication changes
  watch(isAuthenticated, (authenticated) => {
    if (authenticated) {
      startTokenRefresh();
    } else {
      stopTokenRefresh();
    }
  });

  return {
    // State
    tokens: readonly(tokens),
    loading: readonly(loading),
    error: readonly(error),
    isInitialized: readonly(isInitialized),

    // Computed
    isAuthenticated,
    accessToken,
    isTokenExpiringSoon,

    // Actions
    login,
    register,
    logout,
    refreshTokens,
    forgotPassword,
    resetPassword,
    initialize,
  };
});

export type AuthStore = ReturnType<typeof useAuthStore>;
```

---

## Application Settings Store

```typescript
// stores/app.ts
import { defineStore } from "pinia";

// Types
interface AppSettings {
  sidebarCollapsed: boolean;
  sidebarRail: boolean;
  language: string;
  timezone: string;
  dateFormat: string;
  itemsPerPage: number;
  density: "default" | "comfortable" | "compact";
}

interface AppState {
  settings: AppSettings;
  isLoading: boolean;
  isMobile: boolean;
  isOnline: boolean;
}

const defaultSettings: AppSettings = {
  sidebarCollapsed: false,
  sidebarRail: false,
  language: "en",
  timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
  dateFormat: "YYYY-MM-DD",
  itemsPerPage: 10,
  density: "default",
};

export const useAppStore = defineStore("app", () => {
  // State
  const settings = ref<AppSettings>({ ...defaultSettings });
  const isLoading = ref(false);
  const isMobile = ref(false);
  const isOnline = ref(true);
  const pageTitle = ref("");
  const breadcrumbs = ref<Array<{ title: string; to?: string }>>([]);

  // Persist settings
  const persistedSettings = import.meta.client
    ? useLocalStorage<AppSettings>("app_settings", defaultSettings)
    : ref(defaultSettings);

  // Initialize from persisted
  if (import.meta.client) {
    settings.value = { ...defaultSettings, ...persistedSettings.value };
  }

  // Computed
  const sidebarOpen = computed(
    () => !settings.value.sidebarCollapsed && !isMobile.value
  );

  const currentDensity = computed(() => settings.value.density);

  // Actions
  function updateSettings(updates: Partial<AppSettings>): void {
    settings.value = { ...settings.value, ...updates };
    if (import.meta.client) {
      persistedSettings.value = settings.value;
    }
  }

  function toggleSidebar(): void {
    updateSettings({ sidebarCollapsed: !settings.value.sidebarCollapsed });
  }

  function toggleSidebarRail(): void {
    updateSettings({ sidebarRail: !settings.value.sidebarRail });
  }

  function setMobile(mobile: boolean): void {
    isMobile.value = mobile;
    if (mobile) {
      updateSettings({ sidebarCollapsed: true });
    }
  }

  function setOnline(online: boolean): void {
    isOnline.value = online;
  }

  function setLoading(loading: boolean): void {
    isLoading.value = loading;
  }

  function setPageTitle(title: string): void {
    pageTitle.value = title;
    if (import.meta.client) {
      document.title = title ? `${title} | App` : "App";
    }
  }

  function setBreadcrumbs(items: Array<{ title: string; to?: string }>): void {
    breadcrumbs.value = items;
  }

  function resetSettings(): void {
    settings.value = { ...defaultSettings };
    if (import.meta.client) {
      persistedSettings.value = defaultSettings;
    }
  }

  // Initialize online status listener
  if (import.meta.client) {
    window.addEventListener("online", () => setOnline(true));
    window.addEventListener("offline", () => setOnline(false));
    isOnline.value = navigator.onLine;
  }

  return {
    // State
    settings: readonly(settings),
    isLoading: readonly(isLoading),
    isMobile: readonly(isMobile),
    isOnline: readonly(isOnline),
    pageTitle: readonly(pageTitle),
    breadcrumbs: readonly(breadcrumbs),

    // Computed
    sidebarOpen,
    currentDensity,

    // Actions
    updateSettings,
    toggleSidebar,
    toggleSidebarRail,
    setMobile,
    setOnline,
    setLoading,
    setPageTitle,
    setBreadcrumbs,
    resetSettings,
  };
});

export type AppStore = ReturnType<typeof useAppStore>;
```

---

## Notification Store

```typescript
// stores/notification.ts
import { defineStore } from "pinia";

// Types
interface Notification {
  id: string;
  type: "success" | "error" | "warning" | "info";
  title: string;
  message?: string;
  timeout: number;
  dismissible: boolean;
  action?: {
    label: string;
    handler: () => void;
  };
}

type NotificationInput = Omit<Notification, "id">;

export const useNotificationStore = defineStore("notification", () => {
  // State
  const notifications = ref<Notification[]>([]);
  const maxNotifications = ref(5);

  // Computed
  const hasNotifications = computed(() => notifications.value.length > 0);

  const latestNotification = computed(
    () => notifications.value[notifications.value.length - 1]
  );

  const errorNotifications = computed(() =>
    notifications.value.filter((n) => n.type === "error")
  );

  // Actions
  function add(
    notification: Partial<NotificationInput> & { title: string }
  ): string {
    const id = crypto.randomUUID();
    const timeout =
      notification.timeout ?? (notification.type === "error" ? 0 : 5000);
    const dismissible = notification.dismissible ?? true;

    const newNotification: Notification = {
      id,
      type: notification.type || "info",
      title: notification.title,
      message: notification.message,
      timeout,
      dismissible,
      action: notification.action,
    };

    // Remove oldest if at max
    if (notifications.value.length >= maxNotifications.value) {
      notifications.value.shift();
    }

    notifications.value.push(newNotification);

    // Auto-dismiss
    if (timeout > 0) {
      setTimeout(() => {
        dismiss(id);
      }, timeout);
    }

    return id;
  }

  function dismiss(id: string): void {
    const index = notifications.value.findIndex((n) => n.id === id);
    if (index !== -1) {
      notifications.value.splice(index, 1);
    }
  }

  function dismissAll(): void {
    notifications.value = [];
  }

  function dismissByType(type: Notification["type"]): void {
    notifications.value = notifications.value.filter((n) => n.type !== type);
  }

  // Convenience methods
  function success(
    title: string,
    message?: string,
    options?: Partial<NotificationInput>
  ): string {
    return add({ type: "success", title, message, ...options });
  }

  function error(
    title: string,
    message?: string,
    options?: Partial<NotificationInput>
  ): string {
    return add({ type: "error", title, message, timeout: 0, ...options });
  }

  function warning(
    title: string,
    message?: string,
    options?: Partial<NotificationInput>
  ): string {
    return add({ type: "warning", title, message, ...options });
  }

  function info(
    title: string,
    message?: string,
    options?: Partial<NotificationInput>
  ): string {
    return add({ type: "info", title, message, ...options });
  }

  // API error handler
  function handleApiError(
    err: any,
    fallbackMessage = "An error occurred"
  ): string {
    const message = err.data?.message || err.message || fallbackMessage;
    return error("Error", message);
  }

  return {
    // State
    notifications: readonly(notifications),
    maxNotifications,

    // Computed
    hasNotifications,
    latestNotification,
    errorNotifications,

    // Actions
    add,
    dismiss,
    dismissAll,
    dismissByType,
    success,
    error,
    warning,
    info,
    handleApiError,
  };
});

export type NotificationStore = ReturnType<typeof useNotificationStore>;
```

---

## Data Collection Store Pattern

```typescript
// stores/products.ts
import { defineStore } from "pinia";

// Types
interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  category: string;
  inStock: boolean;
  imageUrl?: string;
  createdAt: string;
}

interface ProductFilters {
  search: string;
  category: string | null;
  inStockOnly: boolean;
  priceRange: [number, number] | null;
}

interface ProductSort {
  key: keyof Product;
  order: "asc" | "desc";
}

interface Pagination {
  page: number;
  perPage: number;
  total: number;
}

export const useProductsStore = defineStore("products", () => {
  // State
  const items = ref<Product[]>([]);
  const loading = ref(false);
  const error = ref<Error | null>(null);

  const filters = ref<ProductFilters>({
    search: "",
    category: null,
    inStockOnly: false,
    priceRange: null,
  });

  const sort = ref<ProductSort>({
    key: "name",
    order: "asc",
  });

  const pagination = ref<Pagination>({
    page: 1,
    perPage: 10,
    total: 0,
  });

  // Computed
  const filteredItems = computed(() => {
    let result = [...items.value];

    // Apply search filter
    if (filters.value.search) {
      const search = filters.value.search.toLowerCase();
      result = result.filter(
        (item) =>
          item.name.toLowerCase().includes(search) ||
          item.description.toLowerCase().includes(search) ||
          item.category.toLowerCase().includes(search)
      );
    }

    // Apply category filter
    if (filters.value.category) {
      result = result.filter(
        (item) => item.category === filters.value.category
      );
    }

    // Apply stock filter
    if (filters.value.inStockOnly) {
      result = result.filter((item) => item.inStock);
    }

    // Apply price range filter
    if (filters.value.priceRange) {
      const [min, max] = filters.value.priceRange;
      result = result.filter((item) => item.price >= min && item.price <= max);
    }

    return result;
  });

  const sortedItems = computed(() => {
    const sorted = [...filteredItems.value];

    sorted.sort((a, b) => {
      const aVal = a[sort.value.key];
      const bVal = b[sort.value.key];

      let comparison = 0;
      if (typeof aVal === "string" && typeof bVal === "string") {
        comparison = aVal.localeCompare(bVal);
      } else if (typeof aVal === "number" && typeof bVal === "number") {
        comparison = aVal - bVal;
      } else if (typeof aVal === "boolean" && typeof bVal === "boolean") {
        comparison = aVal === bVal ? 0 : aVal ? -1 : 1;
      }

      return sort.value.order === "asc" ? comparison : -comparison;
    });

    return sorted;
  });

  const paginatedItems = computed(() => {
    const start = (pagination.value.page - 1) * pagination.value.perPage;
    const end = start + pagination.value.perPage;
    return sortedItems.value.slice(start, end);
  });

  const totalPages = computed(() =>
    Math.ceil(filteredItems.value.length / pagination.value.perPage)
  );

  const categories = computed(() => {
    const cats = new Set(items.value.map((item) => item.category));
    return Array.from(cats).sort();
  });

  const priceRange = computed(() => {
    if (items.value.length === 0) return [0, 0] as [number, number];
    const prices = items.value.map((item) => item.price);
    return [Math.min(...prices), Math.max(...prices)] as [number, number];
  });

  const isEmpty = computed(() => items.value.length === 0);

  const isFiltered = computed(
    () =>
      filters.value.search !== "" ||
      filters.value.category !== null ||
      filters.value.inStockOnly ||
      filters.value.priceRange !== null
  );

  // Actions
  async function fetchProducts(
    params?: Record<string, any>
  ): Promise<Product[]> {
    loading.value = true;
    error.value = null;

    try {
      const response = await $fetch<{ items: Product[]; total: number }>(
        "/api/products",
        {
          params,
        }
      );
      items.value = response.items;
      pagination.value.total = response.total;
      return response.items;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function fetchProduct(id: string): Promise<Product> {
    loading.value = true;
    error.value = null;

    try {
      const product = await $fetch<Product>(`/api/products/${id}`);

      // Update in list if exists
      const index = items.value.findIndex((p) => p.id === id);
      if (index !== -1) {
        items.value[index] = product;
      }

      return product;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function createProduct(
    data: Omit<Product, "id" | "createdAt">
  ): Promise<Product> {
    loading.value = true;
    error.value = null;

    try {
      const product = await $fetch<Product>("/api/products", {
        method: "POST",
        body: data,
      });
      items.value.push(product);
      pagination.value.total++;
      return product;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function updateProduct(
    id: string,
    data: Partial<Product>
  ): Promise<Product> {
    loading.value = true;
    error.value = null;

    try {
      const product = await $fetch<Product>(`/api/products/${id}`, {
        method: "PATCH",
        body: data,
      });

      const index = items.value.findIndex((p) => p.id === id);
      if (index !== -1) {
        items.value[index] = product;
      }

      return product;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  async function deleteProduct(id: string): Promise<void> {
    loading.value = true;
    error.value = null;

    try {
      await $fetch(`/api/products/${id}`, { method: "DELETE" });
      items.value = items.value.filter((p) => p.id !== id);
      pagination.value.total--;
    } catch (err) {
      error.value = err as Error;
      throw err;
    } finally {
      loading.value = false;
    }
  }

  function setFilter<K extends keyof ProductFilters>(
    key: K,
    value: ProductFilters[K]
  ): void {
    filters.value[key] = value;
    pagination.value.page = 1; // Reset to first page on filter change
  }

  function clearFilters(): void {
    filters.value = {
      search: "",
      category: null,
      inStockOnly: false,
      priceRange: null,
    };
    pagination.value.page = 1;
  }

  function setSort(key: keyof Product, order?: "asc" | "desc"): void {
    if (sort.value.key === key && !order) {
      sort.value.order = sort.value.order === "asc" ? "desc" : "asc";
    } else {
      sort.value.key = key;
      sort.value.order = order || "asc";
    }
  }

  function setPage(page: number): void {
    pagination.value.page = Math.max(1, Math.min(page, totalPages.value));
  }

  function setPerPage(perPage: number): void {
    pagination.value.perPage = perPage;
    pagination.value.page = 1;
  }

  function getById(id: string): Product | undefined {
    return items.value.find((item) => item.id === id);
  }

  function reset(): void {
    items.value = [];
    clearFilters();
    sort.value = { key: "name", order: "asc" };
    pagination.value = { page: 1, perPage: 10, total: 0 };
    error.value = null;
  }

  return {
    // State
    items: readonly(items),
    loading: readonly(loading),
    error: readonly(error),
    filters,
    sort: readonly(sort),
    pagination,

    // Computed
    filteredItems,
    sortedItems,
    paginatedItems,
    totalPages,
    categories,
    priceRange,
    isEmpty,
    isFiltered,

    // Actions
    fetchProducts,
    fetchProduct,
    createProduct,
    updateProduct,
    deleteProduct,
    setFilter,
    clearFilters,
    setSort,
    setPage,
    setPerPage,
    getById,
    reset,
  };
});

export type ProductsStore = ReturnType<typeof useProductsStore>;
```

---

## Store Composition

### Using Stores Together

```typescript
// stores/checkout.ts
import { defineStore } from "pinia";

interface ShippingAddress {
  street: string;
  city: string;
  state: string;
  zipCode: string;
  country: string;
}

interface PaymentMethod {
  type: "card" | "paypal" | "bank";
  details: Record<string, any>;
}

export const useCheckoutStore = defineStore("checkout", () => {
  // Use other stores
  const authStore = useAuthStore();
  const cartStore = useCartStore();
  const notificationStore = useNotificationStore();

  // State
  const step = ref<"cart" | "shipping" | "payment" | "confirmation">("cart");
  const shippingAddress = ref<ShippingAddress | null>(null);
  const paymentMethod = ref<PaymentMethod | null>(null);
  const processing = ref(false);
  const orderId = ref<string | null>(null);

  // Computed
  const canProceed = computed(() => {
    switch (step.value) {
      case "cart":
        return cartStore.itemCount > 0;
      case "shipping":
        return !!shippingAddress.value;
      case "payment":
        return !!paymentMethod.value;
      default:
        return false;
    }
  });

  const steps = computed(() => [
    { key: "cart", title: "Cart", completed: step.value !== "cart" },
    {
      key: "shipping",
      title: "Shipping",
      completed: ["payment", "confirmation"].includes(step.value),
    },
    {
      key: "payment",
      title: "Payment",
      completed: step.value === "confirmation",
    },
    { key: "confirmation", title: "Confirmation", completed: false },
  ]);

  // Actions
  function nextStep(): void {
    const stepOrder = ["cart", "shipping", "payment", "confirmation"] as const;
    const currentIndex = stepOrder.indexOf(step.value);
    if (currentIndex < stepOrder.length - 1 && canProceed.value) {
      step.value = stepOrder[currentIndex + 1];
    }
  }

  function prevStep(): void {
    const stepOrder = ["cart", "shipping", "payment", "confirmation"] as const;
    const currentIndex = stepOrder.indexOf(step.value);
    if (currentIndex > 0) {
      step.value = stepOrder[currentIndex - 1];
    }
  }

  function goToStep(targetStep: typeof step.value): void {
    step.value = targetStep;
  }

  function setShippingAddress(address: ShippingAddress): void {
    shippingAddress.value = address;
  }

  function setPaymentMethod(method: PaymentMethod): void {
    paymentMethod.value = method;
  }

  async function processOrder(): Promise<string> {
    if (!authStore.isAuthenticated) {
      throw new Error("Must be authenticated to checkout");
    }

    if (!shippingAddress.value || !paymentMethod.value) {
      throw new Error("Missing required information");
    }

    processing.value = true;

    try {
      const order = await $fetch<{ id: string }>("/api/orders", {
        method: "POST",
        body: {
          items: cartStore.items,
          shippingAddress: shippingAddress.value,
          paymentMethod: paymentMethod.value,
        },
      });

      orderId.value = order.id;
      cartStore.clearCart();
      notificationStore.success("Order placed successfully!");
      step.value = "confirmation";

      return order.id;
    } catch (error) {
      notificationStore.error("Failed to process order");
      throw error;
    } finally {
      processing.value = false;
    }
  }

  function reset(): void {
    step.value = "cart";
    shippingAddress.value = null;
    paymentMethod.value = null;
    orderId.value = null;
  }

  return {
    // State
    step: readonly(step),
    shippingAddress: readonly(shippingAddress),
    paymentMethod: readonly(paymentMethod),
    processing: readonly(processing),
    orderId: readonly(orderId),

    // Computed
    canProceed,
    steps,

    // Actions
    nextStep,
    prevStep,
    goToStep,
    setShippingAddress,
    setPaymentMethod,
    processOrder,
    reset,
  };
});
```

---

## SSR Considerations

### useState for Simple State

For simple SSR-safe state, use Nuxt's `useState`:

```typescript
// composables/useCount.ts
export function useCount() {
  // SSR-safe state
  const count = useState<number>("count", () => 0);

  function increment() {
    count.value++;
  }

  return { count, increment };
}
```

### Hydration-Safe Stores

```typescript
// stores/ssr-safe.ts
import { defineStore, skipHydrate } from "pinia";

export const useSSRStore = defineStore("ssr", () => {
  // State that should be hydrated from server
  const serverData = ref<string | null>(null);

  // State that should NOT be hydrated (client-only)
  const clientOnlyState = ref({
    mousePosition: { x: 0, y: 0 },
    windowSize: { width: 0, height: 0 },
  });

  return {
    serverData,
    // Skip hydration for client-only state
    clientOnlyState: skipHydrate(clientOnlyState),
  };
});
```

### Store Initialization Plugin

```typescript
// plugins/store-init.ts
export default defineNuxtPlugin(async () => {
  const authStore = useAuthStore();
  const appStore = useAppStore();
  const userStore = useUserStore();

  // Initialize stores on app start
  if (import.meta.client) {
    await authStore.initialize();
    userStore.loadPreferences();
  }

  // Watch for auth changes
  watch(
    () => authStore.isAuthenticated,
    (isAuth) => {
      if (!isAuth) {
        // Reset other stores on logout
        userStore.clearUser();
      }
    }
  );
});
```

---

## Testing Stores

### Unit Testing with Arrange-Act-Assert

```typescript
// tests/unit/stores/user.test.ts
import { setActivePinia, createPinia } from "pinia";
import { describe, it, expect, beforeEach, vi } from "vitest";
import { useUserStore } from "~/stores/user";

// Mock $fetch
vi.stubGlobal("$fetch", vi.fn());

describe("useUserStore", () => {
  beforeEach(() => {
    // Arrange: Create fresh Pinia instance
    setActivePinia(createPinia());
    vi.clearAllMocks();
  });

  describe("initial state", () => {
    it("initializes with null user", () => {
      // Arrange
      const store = useUserStore();

      // Assert
      expect(store.currentUser).toBeNull();
      expect(store.isAuthenticated).toBe(false);
    });
  });

  describe("setUser", () => {
    it("sets user correctly", () => {
      // Arrange
      const store = useUserStore();
      const user = {
        id: "1",
        name: "Test User",
        email: "test@example.com",
        role: "user" as const,
        createdAt: new Date().toISOString(),
      };

      // Act
      store.setUser(user);

      // Assert
      expect(store.currentUser).toEqual(user);
      expect(store.isAuthenticated).toBe(true);
      expect(store.displayName).toBe("Test User");
    });
  });

  describe("clearUser", () => {
    it("clears user on logout", () => {
      // Arrange
      const store = useUserStore();
      store.setUser({
        id: "1",
        name: "Test",
        email: "test@example.com",
        role: "user",
        createdAt: new Date().toISOString(),
      });

      // Act
      store.clearUser();

      // Assert
      expect(store.currentUser).toBeNull();
      expect(store.isAuthenticated).toBe(false);
    });
  });

  describe("fetchUser", () => {
    it("fetches user from API", async () => {
      // Arrange
      const mockUser = {
        id: "1",
        name: "API User",
        email: "api@example.com",
        role: "user" as const,
        createdAt: new Date().toISOString(),
      };
      vi.mocked($fetch).mockResolvedValueOnce(mockUser);
      const store = useUserStore();

      // Act
      await store.fetchUser();

      // Assert
      expect($fetch).toHaveBeenCalledWith("/api/auth/me");
      expect(store.currentUser).toEqual(mockUser);
    });

    it("handles fetch error", async () => {
      // Arrange
      const error = new Error("Network error");
      vi.mocked($fetch).mockRejectedValueOnce(error);
      const store = useUserStore();

      // Act & Assert
      await expect(store.fetchUser()).rejects.toThrow("Network error");
      expect(store.error).toBe(error);
    });
  });

  describe("userInitials", () => {
    it("computes initials correctly", () => {
      // Arrange
      const store = useUserStore();
      store.setUser({
        id: "1",
        name: "John Doe",
        email: "john@example.com",
        role: "user",
        createdAt: new Date().toISOString(),
      });

      // Assert
      expect(store.userInitials).toBe("JD");
    });

    it("handles single name", () => {
      // Arrange
      const store = useUserStore();
      store.setUser({
        id: "1",
        name: "John",
        email: "john@example.com",
        role: "user",
        createdAt: new Date().toISOString(),
      });

      // Assert
      expect(store.userInitials).toBe("J");
    });
  });
});
```

### Integration Testing

```typescript
// tests/integration/stores/auth.test.ts
import { setActivePinia, createPinia } from "pinia";
import { describe, it, expect, beforeEach, vi } from "vitest";
import { useAuthStore } from "~/stores/auth";
import { useUserStore } from "~/stores/user";

vi.stubGlobal("$fetch", vi.fn());

describe("Auth Store Integration", () => {
  beforeEach(() => {
    setActivePinia(createPinia());
    vi.clearAllMocks();

    // Clear storage
    if (typeof localStorage !== "undefined") {
      localStorage.clear();
      sessionStorage.clear();
    }
  });

  describe("login flow", () => {
    it("authenticates user and updates user store", async () => {
      // Arrange
      const mockResponse = {
        tokens: {
          accessToken: "token",
          refreshToken: "refresh",
          expiresAt: Date.now() + 3600000,
        },
        user: {
          id: "1",
          name: "Test User",
          email: "test@example.com",
          role: "user" as const,
        },
      };
      vi.mocked($fetch).mockResolvedValueOnce(mockResponse);

      const authStore = useAuthStore();
      const userStore = useUserStore();

      // Act
      await authStore.login({
        email: "test@example.com",
        password: "password",
      });

      // Assert
      expect(authStore.isAuthenticated).toBe(true);
      expect(authStore.accessToken).toBe("token");
      expect(userStore.currentUser).toEqual(mockResponse.user);
    });
  });

  describe("logout flow", () => {
    it("clears auth and user stores", async () => {
      // Arrange
      const authStore = useAuthStore();
      const userStore = useUserStore();

      // Setup authenticated state
      vi.mocked($fetch).mockResolvedValueOnce({
        tokens: {
          accessToken: "token",
          refreshToken: "refresh",
          expiresAt: Date.now() + 3600000,
        },
        user: {
          id: "1",
          name: "Test",
          email: "test@example.com",
          role: "user",
        },
      });
      await authStore.login({
        email: "test@example.com",
        password: "password",
      });

      vi.mocked($fetch).mockResolvedValueOnce({});

      // Act
      await authStore.logout(false);

      // Assert
      expect(authStore.isAuthenticated).toBe(false);
      expect(userStore.currentUser).toBeNull();
    });
  });
});
```

---

## Best Practices

### 1. Store Organization

```typescript
// ✅ Good: Focused, single-responsibility stores
const useAuthStore = defineStore("auth", () => {
  /* auth logic */
});
const useUserStore = defineStore("user", () => {
  /* user logic */
});

// ❌ Bad: Monolithic store with everything
const useAppStore = defineStore("app", () => {
  /* auth + user + settings + notifications + ... */
});
```

### 2. Readonly State

```typescript
// ✅ Good: Expose readonly refs
return {
  users: readonly(users),
  loading: readonly(loading),
  fetchUsers,
};

// ❌ Bad: Allow direct mutations
return {
  users,
  loading,
};
```

### 3. Async Actions

```typescript
// ✅ Good: Proper loading and error handling
async function fetchData() {
  loading.value = true;
  error.value = null;

  try {
    data.value = await $fetch("/api/data");
  } catch (err) {
    error.value = err as Error;
    throw err;
  } finally {
    loading.value = false;
  }
}

// ❌ Bad: No error handling
async function fetchData() {
  data.value = await $fetch("/api/data");
}
```

### 4. Type Exports

```typescript
// ✅ Good: Export store type for use in components
export const useUserStore = defineStore("user", () => {
  /* ... */
});
export type UserStore = ReturnType<typeof useUserStore>;

// Usage in component
const userStore: UserStore = useUserStore();
```

### 5. SSR Safety

```typescript
// ✅ Good: Check for client
if (import.meta.client) {
  localStorage.setItem("key", value);
}

// ❌ Bad: Direct browser API access
localStorage.setItem("key", value); // Fails on server
```

---

## Next Steps

Continue to [Routing & Navigation](./07-routing-and-navigation.md) to learn about Nuxt's file-based routing, middleware guards, and authentication flow patterns.
