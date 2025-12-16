# Composables Patterns

## Overview

Composables are the primary mechanism for logic reuse in Vue 3 applications. They encapsulate reactive state, computed properties, watchers, and lifecycle hooks into reusable functions. This document covers patterns, best practices, and guidelines for creating effective composables with Vuetify integration.

## Composables Organization

### Directory Structure

```
app/composables/
├── api/                        # API integration composables
│   ├── useApi.ts              # Base API client with error handling
│   ├── useFetch.ts            # Data fetching wrapper
│   └── useInfiniteScroll.ts   # Infinite pagination
├── auth/                       # Authentication composables
│   ├── useAuth.ts             # Authentication state
│   ├── useSession.ts          # Session management
│   └── usePermissions.ts      # Permission checking
├── ui/                         # UI-related composables
│   ├── useToast.ts            # Toast notifications (Vuetify snackbar)
│   ├── useDialog.ts           # Dialog confirmations
│   ├── useDrawer.ts           # Drawer state
│   └── useTheme.ts            # Theme management
├── form/                       # Form handling composables
│   ├── useForm.ts             # Form state management
│   ├── useValidation.ts       # Form validation
│   └── useFormField.ts        # Individual field logic
├── utils/                      # Utility composables
│   ├── useDebounce.ts         # Debounce functionality
│   ├── useLocalStorage.ts     # Local storage wrapper
│   └── useClipboard.ts        # Clipboard operations
└── [domain]/                   # Domain-specific composables
    ├── useUser.ts             # User operations
    ├── useProduct.ts          # Product operations
    └── useOrder.ts            # Order operations
```

---

## Core Patterns

### 1. Return Object Pattern

Always return an object with named properties for maximum flexibility:

```typescript
// composables/useCounter.ts
export function useCounter(initialValue = 0) {
  // State
  const count = ref(initialValue);
  const history = ref<number[]>([initialValue]);

  // Computed
  const isPositive = computed(() => count.value > 0);
  const isNegative = computed(() => count.value < 0);
  const isZero = computed(() => count.value === 0);

  // Methods
  function increment(amount = 1) {
    count.value += amount;
    history.value.push(count.value);
  }

  function decrement(amount = 1) {
    count.value -= amount;
    history.value.push(count.value);
  }

  function reset() {
    count.value = initialValue;
    history.value = [initialValue];
  }

  // Return object with readonly state
  return {
    // State (readonly to prevent external mutations)
    count: readonly(count),
    history: readonly(history),

    // Computed
    isPositive,
    isNegative,
    isZero,

    // Methods
    increment,
    decrement,
    reset,
  };
}
```

### 2. Options Pattern

For composables with complex configuration:

```typescript
// composables/usePagination.ts
interface UsePaginationOptions {
  initialPage?: number;
  pageSize?: number;
  total?: Ref<number> | number;
  onPageChange?: (page: number) => void | Promise<void>;
}

export function usePagination(options: UsePaginationOptions = {}) {
  const { initialPage = 1, pageSize = 10, total = 0, onPageChange } = options;

  // State
  const currentPage = ref(initialPage);
  const itemsPerPage = ref(pageSize);

  // Computed
  const totalRef = isRef(total) ? total : ref(total);

  const totalPages = computed(() =>
    Math.ceil(totalRef.value / itemsPerPage.value)
  );

  const offset = computed(() => (currentPage.value - 1) * itemsPerPage.value);

  const hasNextPage = computed(() => currentPage.value < totalPages.value);

  const hasPrevPage = computed(() => currentPage.value > 1);

  // Methods
  async function goToPage(page: number) {
    if (page < 1 || page > totalPages.value) return;
    currentPage.value = page;
    await onPageChange?.(page);
  }

  async function nextPage() {
    if (hasNextPage.value) {
      await goToPage(currentPage.value + 1);
    }
  }

  async function prevPage() {
    if (hasPrevPage.value) {
      await goToPage(currentPage.value - 1);
    }
  }

  function setPageSize(size: number) {
    itemsPerPage.value = size;
    currentPage.value = 1;
  }

  return {
    currentPage: readonly(currentPage),
    itemsPerPage: readonly(itemsPerPage),
    totalPages,
    offset,
    hasNextPage,
    hasPrevPage,
    goToPage,
    nextPage,
    prevPage,
    setPageSize,
  };
}
```

---

## API Composables with Error Handling

### Base API Composable

```typescript
// composables/api/useApi.ts
interface ApiOptions {
  baseURL?: string;
  headers?: Record<string, string>;
  timeout?: number;
}

interface ApiError {
  status: number;
  message: string;
  code?: string;
  details?: Record<string, any>;
}

interface RequestOptions extends RequestInit {
  params?: Record<string, string | number | boolean>;
  timeout?: number;
}

export function useApi(options: ApiOptions = {}) {
  const config = useRuntimeConfig();
  const baseURL = options.baseURL || config.public.apiBaseUrl;
  const defaultTimeout = options.timeout || 30000;

  const loading = ref(false);
  const error = ref<ApiError | null>(null);

  async function request<T>(
    endpoint: string,
    requestOptions: RequestOptions = {}
  ): Promise<T> {
    const {
      params,
      timeout = defaultTimeout,
      ...fetchOptions
    } = requestOptions;

    // Build URL with query params
    let url = `${baseURL}${endpoint}`;
    if (params) {
      const searchParams = new URLSearchParams();
      Object.entries(params).forEach(([key, value]) => {
        if (value !== undefined && value !== null) {
          searchParams.append(key, String(value));
        }
      });
      const queryString = searchParams.toString();
      if (queryString) {
        url += `?${queryString}`;
      }
    }

    loading.value = true;
    error.value = null;

    try {
      const response = await $fetch<T>(url, {
        ...fetchOptions,
        headers: {
          "Content-Type": "application/json",
          ...options.headers,
          ...fetchOptions.headers,
        },
        timeout,
      });

      return response;
    } catch (err: any) {
      const apiError: ApiError = {
        status: err.statusCode || err.status || 500,
        message: err.statusMessage || err.message || "An error occurred",
        code: err.data?.code,
        details: err.data?.details,
      };

      error.value = apiError;
      throw apiError;
    } finally {
      loading.value = false;
    }
  }

  return {
    loading: readonly(loading),
    error: readonly(error),

    get: <T>(endpoint: string, params?: Record<string, any>) =>
      request<T>(endpoint, { method: "GET", params }),

    post: <T>(endpoint: string, body: unknown) =>
      request<T>(endpoint, { method: "POST", body: JSON.stringify(body) }),

    put: <T>(endpoint: string, body: unknown) =>
      request<T>(endpoint, { method: "PUT", body: JSON.stringify(body) }),

    patch: <T>(endpoint: string, body: unknown) =>
      request<T>(endpoint, { method: "PATCH", body: JSON.stringify(body) }),

    delete: <T>(endpoint: string) => request<T>(endpoint, { method: "DELETE" }),
  };
}
```

### Async Data Composable

```typescript
// composables/api/useAsyncData.ts
interface UseAsyncDataOptions<T> {
  immediate?: boolean;
  default?: T;
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
  transform?: (data: any) => T;
}

export function useAsyncData<T>(
  fetcher: () => Promise<T>,
  options: UseAsyncDataOptions<T> = {}
) {
  const {
    immediate = true,
    default: defaultValue,
    onSuccess,
    onError,
    transform,
  } = options;

  const data = ref<T | undefined>(defaultValue) as Ref<T | undefined>;
  const error = ref<Error | null>(null);
  const loading = ref(false);
  const loaded = ref(false);

  async function execute(): Promise<T | undefined> {
    loading.value = true;
    error.value = null;

    try {
      let result = await fetcher();

      if (transform) {
        result = transform(result);
      }

      data.value = result;
      loaded.value = true;
      onSuccess?.(result);
      return result;
    } catch (err) {
      error.value = err as Error;
      onError?.(err as Error);
    } finally {
      loading.value = false;
    }
  }

  function reset() {
    data.value = defaultValue;
    error.value = null;
    loading.value = false;
    loaded.value = false;
  }

  // Execute immediately if option is set
  if (immediate) {
    execute();
  }

  return {
    data,
    error: readonly(error),
    loading: readonly(loading),
    loaded: readonly(loaded),
    execute,
    refresh: execute,
    reset,
  };
}
```

### Domain-Specific API Composable

```typescript
// composables/api/useUsers.ts
import type { User, CreateUserInput, UpdateUserInput } from "~/types";

export function useUsers() {
  const api = useApi();

  // State
  const users = ref<User[]>([]);
  const currentUser = ref<User | null>(null);

  // Computed
  const admins = computed(() => users.value.filter((u) => u.role === "admin"));
  const regularUsers = computed(() =>
    users.value.filter((u) => u.role === "user")
  );
  const userCount = computed(() => users.value.length);

  // Methods
  async function fetchUsers(params?: { role?: string; search?: string }) {
    const result = await api.get<User[]>("/users", params);
    users.value = result;
    return result;
  }

  async function fetchUser(id: string) {
    const result = await api.get<User>(`/users/${id}`);
    currentUser.value = result;
    return result;
  }

  async function createUser(input: CreateUserInput) {
    const result = await api.post<User>("/users", input);
    users.value.push(result);
    return result;
  }

  async function updateUser(id: string, input: UpdateUserInput) {
    const result = await api.patch<User>(`/users/${id}`, input);
    const index = users.value.findIndex((u) => u.id === id);
    if (index !== -1) {
      users.value[index] = result;
    }
    if (currentUser.value?.id === id) {
      currentUser.value = result;
    }
    return result;
  }

  async function deleteUser(id: string) {
    await api.delete(`/users/${id}`);
    users.value = users.value.filter((u) => u.id !== id);
    if (currentUser.value?.id === id) {
      currentUser.value = null;
    }
  }

  function getUserById(id: string) {
    return users.value.find((u) => u.id === id);
  }

  return {
    // State
    users: readonly(users),
    currentUser: readonly(currentUser),
    loading: api.loading,
    error: api.error,

    // Computed
    admins,
    regularUsers,
    userCount,

    // Methods
    fetchUsers,
    fetchUser,
    createUser,
    updateUser,
    deleteUser,
    getUserById,
  };
}
```

---

## UI Composables with Vuetify

### Toast Notifications

```typescript
// composables/ui/useToast.ts
interface Toast {
  id: string;
  message: string;
  type: "success" | "error" | "warning" | "info";
  timeout: number;
}

const toasts = ref<Toast[]>([]);

export function useToast() {
  function show(
    message: string,
    type: Toast["type"] = "info",
    timeout = 5000
  ): string {
    const id = crypto.randomUUID();

    toasts.value.push({ id, message, type, timeout });

    if (timeout > 0) {
      setTimeout(() => {
        dismiss(id);
      }, timeout);
    }

    return id;
  }

  function dismiss(id: string) {
    const index = toasts.value.findIndex((t) => t.id === id);
    if (index !== -1) {
      toasts.value.splice(index, 1);
    }
  }

  function dismissAll() {
    toasts.value = [];
  }

  return {
    toasts: readonly(toasts),
    show,
    success: (message: string, timeout?: number) =>
      show(message, "success", timeout),
    error: (message: string, timeout?: number) =>
      show(message, "error", timeout ?? 0),
    warning: (message: string, timeout?: number) =>
      show(message, "warning", timeout),
    info: (message: string, timeout?: number) => show(message, "info", timeout),
    dismiss,
    dismissAll,
  };
}
```

### Confirm Dialog

```typescript
// composables/ui/useConfirm.ts
interface ConfirmOptions {
  title?: string;
  message: string;
  confirmText?: string;
  cancelText?: string;
  type?: "info" | "warning" | "danger" | "success";
  persistent?: boolean;
}

interface ConfirmState {
  isOpen: boolean;
  options: ConfirmOptions | null;
  resolve: ((value: boolean) => void) | null;
}

const state = reactive<ConfirmState>({
  isOpen: false,
  options: null,
  resolve: null,
});

export function useConfirm() {
  function confirm(options: ConfirmOptions): Promise<boolean> {
    return new Promise((resolve) => {
      state.isOpen = true;
      state.options = {
        title: "Confirm",
        confirmText: "Confirm",
        cancelText: "Cancel",
        type: "info",
        persistent: false,
        ...options,
      };
      state.resolve = resolve;
    });
  }

  function handleConfirm() {
    state.resolve?.(true);
    reset();
  }

  function handleCancel() {
    state.resolve?.(false);
    reset();
  }

  function reset() {
    state.isOpen = false;
    state.options = null;
    state.resolve = null;
  }

  return {
    isOpen: computed(() => state.isOpen),
    options: computed(() => state.options),
    confirm,
    handleConfirm,
    handleCancel,
  };
}
```

### Theme Management

```typescript
// composables/ui/useTheme.ts
import { useTheme as useVuetifyTheme } from "vuetify";

export function useAppTheme() {
  const vuetifyTheme = useVuetifyTheme();
  const colorMode = useColorMode();

  const isDark = computed(() => vuetifyTheme.global.current.value.dark);
  const currentTheme = computed(() => vuetifyTheme.global.name.value);

  function setTheme(theme: "light" | "dark" | "system") {
    if (theme === "system") {
      const prefersDark = window.matchMedia(
        "(prefers-color-scheme: dark)"
      ).matches;
      vuetifyTheme.global.name.value = prefersDark ? "dark" : "light";
    } else {
      vuetifyTheme.global.name.value = theme;
    }
    colorMode.preference = theme;
  }

  function toggleTheme() {
    vuetifyTheme.global.name.value = isDark.value ? "light" : "dark";
    colorMode.preference = isDark.value ? "light" : "dark";
  }

  // Watch for system preference changes
  if (import.meta.client) {
    const mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");
    mediaQuery.addEventListener("change", (e) => {
      if (colorMode.preference === "system") {
        vuetifyTheme.global.name.value = e.matches ? "dark" : "light";
      }
    });
  }

  return {
    isDark,
    currentTheme,
    setTheme,
    toggleTheme,
  };
}
```

### Drawer State

```typescript
// composables/ui/useDrawer.ts
interface DrawerState {
  isOpen: boolean;
  isRail: boolean;
  isMini: boolean;
}

const state = reactive<DrawerState>({
  isOpen: true,
  isRail: false,
  isMini: false,
});

export function useDrawer() {
  const { mobile } = useDisplay();

  // Auto-close on mobile
  watch(mobile, (isMobile) => {
    if (isMobile) {
      state.isOpen = false;
    }
  });

  function toggle() {
    state.isOpen = !state.isOpen;
  }

  function open() {
    state.isOpen = true;
  }

  function close() {
    state.isOpen = false;
  }

  function toggleRail() {
    state.isRail = !state.isRail;
  }

  function setRail(value: boolean) {
    state.isRail = value;
  }

  return {
    isOpen: computed(() => state.isOpen),
    isRail: computed(() => state.isRail),
    isMini: computed(() => state.isMini),
    toggle,
    open,
    close,
    toggleRail,
    setRail,
  };
}
```

---

## Form Composables

### Form State Management

```typescript
// composables/form/useForm.ts
interface UseFormOptions<T extends Record<string, any>> {
  initialValues: T;
  validate?: (
    values: T
  ) =>
    | Partial<Record<keyof T, string>>
    | Promise<Partial<Record<keyof T, string>>>;
  onSubmit: (values: T) => void | Promise<void>;
}

export function useForm<T extends Record<string, any>>(
  options: UseFormOptions<T>
) {
  const { initialValues, validate, onSubmit } = options;

  // State
  const values = reactive({ ...initialValues }) as T;
  const errors = reactive<Partial<Record<keyof T, string>>>({});
  const touched = reactive<Partial<Record<keyof T, boolean>>>({});
  const isSubmitting = ref(false);
  const isValidating = ref(false);

  // Computed
  const isDirty = computed(() => {
    return Object.keys(initialValues).some(
      (key) => values[key as keyof T] !== initialValues[key as keyof T]
    );
  });

  const isValid = computed(() => {
    return Object.values(errors).every((error) => !error);
  });

  const canSubmit = computed(() => {
    return isValid.value && isDirty.value && !isSubmitting.value;
  });

  // Methods
  function setFieldValue<K extends keyof T>(field: K, value: T[K]) {
    values[field] = value;
  }

  function setFieldError<K extends keyof T>(
    field: K,
    error: string | undefined
  ) {
    if (error) {
      errors[field] = error;
    } else {
      delete errors[field];
    }
  }

  function setFieldTouched<K extends keyof T>(field: K, isTouched = true) {
    touched[field] = isTouched;
  }

  async function validateForm(): Promise<boolean> {
    if (!validate) return true;

    isValidating.value = true;
    try {
      const validationErrors = await validate(values);
      Object.keys(errors).forEach((key) => delete errors[key as keyof T]);
      Object.assign(errors, validationErrors);
      return Object.values(validationErrors).every((error) => !error);
    } finally {
      isValidating.value = false;
    }
  }

  async function validateField<K extends keyof T>(field: K): Promise<boolean> {
    if (!validate) return true;

    const validationErrors = await validate(values);
    const fieldError = validationErrors[field];
    setFieldError(field, fieldError);
    return !fieldError;
  }

  async function handleSubmit() {
    // Touch all fields
    Object.keys(values).forEach((key) => {
      touched[key as keyof T] = true;
    });

    // Validate
    const isFormValid = await validateForm();
    if (!isFormValid) return;

    // Submit
    isSubmitting.value = true;
    try {
      await onSubmit(values);
    } finally {
      isSubmitting.value = false;
    }
  }

  function reset() {
    Object.assign(values, initialValues);
    Object.keys(errors).forEach((key) => delete errors[key as keyof T]);
    Object.keys(touched).forEach((key) => delete touched[key as keyof T]);
  }

  function resetField<K extends keyof T>(field: K) {
    values[field] = initialValues[field];
    delete errors[field];
    delete touched[field];
  }

  return {
    values,
    errors: readonly(errors) as Readonly<Partial<Record<keyof T, string>>>,
    touched: readonly(touched) as Readonly<Partial<Record<keyof T, boolean>>>,
    isSubmitting: readonly(isSubmitting),
    isValidating: readonly(isValidating),
    isDirty,
    isValid,
    canSubmit,
    setFieldValue,
    setFieldError,
    setFieldTouched,
    validateForm,
    validateField,
    handleSubmit,
    reset,
    resetField,
  };
}
```

---

## Utility Composables

### Debounce

```typescript
// composables/utils/useDebounce.ts
export function useDebounce<T>(value: Ref<T>, delay = 300): Ref<T> {
  const debouncedValue = ref(value.value) as Ref<T>;
  let timeout: ReturnType<typeof setTimeout>;

  watch(value, (newValue) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      debouncedValue.value = newValue;
    }, delay);
  });

  onUnmounted(() => {
    clearTimeout(timeout);
  });

  return readonly(debouncedValue);
}

export function useDebounceFn<T extends (...args: any[]) => any>(
  fn: T,
  delay = 300
): (...args: Parameters<T>) => void {
  let timeout: ReturnType<typeof setTimeout>;

  const debouncedFn = (...args: Parameters<T>) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => fn(...args), delay);
  };

  onUnmounted(() => {
    clearTimeout(timeout);
  });

  return debouncedFn;
}
```

### Local Storage

```typescript
// composables/utils/useLocalStorage.ts
interface UseLocalStorageOptions<T> {
  serializer?: {
    read: (value: string) => T;
    write: (value: T) => string;
  };
  onError?: (error: Error) => void;
}

export function useLocalStorage<T>(
  key: string,
  defaultValue: T,
  options: UseLocalStorageOptions<T> = {}
): Ref<T> {
  const {
    serializer = {
      read: JSON.parse,
      write: JSON.stringify,
    },
    onError = console.error,
  } = options;

  // Read initial value
  function read(): T {
    if (import.meta.server) {
      return defaultValue;
    }

    try {
      const stored = localStorage.getItem(key);
      return stored ? serializer.read(stored) : defaultValue;
    } catch (error) {
      onError(error as Error);
      return defaultValue;
    }
  }

  // Create reactive ref
  const data = ref(read()) as Ref<T>;

  // Write to storage on change
  watch(
    data,
    (value) => {
      if (import.meta.server) return;

      try {
        if (value === null || value === undefined) {
          localStorage.removeItem(key);
        } else {
          localStorage.setItem(key, serializer.write(value));
        }
      } catch (error) {
        onError(error as Error);
      }
    },
    { deep: true }
  );

  // Listen for storage events (cross-tab sync)
  if (import.meta.client) {
    window.addEventListener("storage", (event) => {
      if (event.key === key) {
        data.value = event.newValue
          ? serializer.read(event.newValue)
          : defaultValue;
      }
    });
  }

  return data;
}
```

### Clipboard

```typescript
// composables/utils/useClipboard.ts
export function useClipboard() {
  const copied = ref(false);
  const error = ref<Error | null>(null);
  let timeout: ReturnType<typeof setTimeout>;

  async function copy(text: string): Promise<boolean> {
    error.value = null;

    try {
      await navigator.clipboard.writeText(text);
      copied.value = true;

      clearTimeout(timeout);
      timeout = setTimeout(() => {
        copied.value = false;
      }, 2000);

      return true;
    } catch (err) {
      error.value = err as Error;
      copied.value = false;
      return false;
    }
  }

  async function paste(): Promise<string | null> {
    error.value = null;

    try {
      return await navigator.clipboard.readText();
    } catch (err) {
      error.value = err as Error;
      return null;
    }
  }

  onUnmounted(() => {
    clearTimeout(timeout);
  });

  return {
    copied: readonly(copied),
    error: readonly(error),
    copy,
    paste,
  };
}
```

---

## Vuetify-Specific Composables

### Display Breakpoints

```typescript
// composables/ui/useBreakpoints.ts
import { useDisplay } from "vuetify";

export function useBreakpoints() {
  const display = useDisplay();

  return {
    // Breakpoint checks
    xs: computed(() => display.xs.value),
    sm: computed(() => display.sm.value),
    md: computed(() => display.md.value),
    lg: computed(() => display.lg.value),
    xl: computed(() => display.xl.value),
    xxl: computed(() => display.xxl.value),

    // Range checks
    smAndDown: computed(() => display.smAndDown.value),
    smAndUp: computed(() => display.smAndUp.value),
    mdAndDown: computed(() => display.mdAndDown.value),
    mdAndUp: computed(() => display.mdAndUp.value),
    lgAndDown: computed(() => display.lgAndDown.value),
    lgAndUp: computed(() => display.lgAndUp.value),

    // Device type
    mobile: computed(() => display.mobile.value),

    // Current breakpoint name
    name: computed(() => display.name.value),

    // Screen dimensions
    width: computed(() => display.width.value),
    height: computed(() => display.height.value),
  };
}
```

### Data Table Helpers

```typescript
// composables/ui/useDataTable.ts
interface UseDataTableOptions<T> {
  items: Ref<T[]>;
  itemsPerPage?: number;
  sortBy?: { key: string; order: "asc" | "desc" }[];
}

export function useDataTable<T extends Record<string, any>>(
  options: UseDataTableOptions<T>
) {
  const { items, itemsPerPage = 10, sortBy: initialSortBy = [] } = options;

  const page = ref(1);
  const perPage = ref(itemsPerPage);
  const sortBy = ref(initialSortBy);
  const search = ref("");
  const selected = ref<T[]>([]);

  const filteredItems = computed(() => {
    if (!search.value) return items.value;

    const searchLower = search.value.toLowerCase();
    return items.value.filter((item) => {
      return Object.values(item).some((value) =>
        String(value).toLowerCase().includes(searchLower)
      );
    });
  });

  const sortedItems = computed(() => {
    if (sortBy.value.length === 0) return filteredItems.value;

    return [...filteredItems.value].sort((a, b) => {
      for (const sort of sortBy.value) {
        const aVal = a[sort.key];
        const bVal = b[sort.key];

        let comparison = 0;
        if (typeof aVal === "string" && typeof bVal === "string") {
          comparison = aVal.localeCompare(bVal);
        } else {
          comparison = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
        }

        if (comparison !== 0) {
          return sort.order === "asc" ? comparison : -comparison;
        }
      }
      return 0;
    });
  });

  const paginatedItems = computed(() => {
    const start = (page.value - 1) * perPage.value;
    return sortedItems.value.slice(start, start + perPage.value);
  });

  const totalPages = computed(() =>
    Math.ceil(filteredItems.value.length / perPage.value)
  );

  const totalItems = computed(() => filteredItems.value.length);

  // Reset page when search changes
  watch(search, () => {
    page.value = 1;
  });

  function clearSelection() {
    selected.value = [];
  }

  function selectAll() {
    selected.value = [...filteredItems.value];
  }

  function toggleSelection(item: T) {
    const index = selected.value.indexOf(item);
    if (index === -1) {
      selected.value.push(item);
    } else {
      selected.value.splice(index, 1);
    }
  }

  return {
    page,
    perPage,
    sortBy,
    search,
    selected,
    filteredItems,
    sortedItems,
    paginatedItems,
    totalPages,
    totalItems,
    clearSelection,
    selectAll,
    toggleSelection,
  };
}
```

---

## Testing Composables

### Unit Testing

```typescript
// tests/unit/composables/useCounter.test.ts
import { describe, it, expect } from "vitest";
import { useCounter } from "~/composables/useCounter";

describe("useCounter", () => {
  it("initializes with default value", () => {
    const { count } = useCounter();
    expect(count.value).toBe(0);
  });

  it("initializes with custom value", () => {
    const { count } = useCounter(10);
    expect(count.value).toBe(10);
  });

  it("increments count", () => {
    const { count, increment } = useCounter(0);
    increment();
    expect(count.value).toBe(1);
  });

  it("decrements count", () => {
    const { count, decrement } = useCounter(5);
    decrement();
    expect(count.value).toBe(4);
  });

  it("resets to initial value", () => {
    const { count, increment, reset } = useCounter(5);
    increment();
    increment();
    expect(count.value).toBe(7);
    reset();
    expect(count.value).toBe(5);
  });

  it("tracks history", () => {
    const { history, increment, decrement } = useCounter(0);
    increment();
    increment();
    decrement();
    expect(history.value).toEqual([0, 1, 2, 1]);
  });
});
```

### Testing with Vue Test Utils

```typescript
// tests/unit/composables/useForm.test.ts
import { describe, it, expect, vi } from "vitest";
import { mount } from "@vue/test-utils";
import { defineComponent } from "vue";
import { useForm } from "~/composables/form/useForm";

describe("useForm", () => {
  const TestComponent = defineComponent({
    setup() {
      const onSubmit = vi.fn();
      return useForm({
        initialValues: { email: "", password: "" },
        validate: (values) => ({
          email: !values.email ? "Email required" : undefined,
          password: values.password.length < 8 ? "Min 8 chars" : undefined,
        }),
        onSubmit,
      });
    },
    template: "<div></div>",
  });

  it("validates form fields", async () => {
    const wrapper = mount(TestComponent);
    const vm = wrapper.vm as any;

    await vm.validateForm();

    expect(vm.errors.email).toBe("Email required");
    expect(vm.errors.password).toBe("Min 8 chars");
  });

  it("tracks dirty state", () => {
    const wrapper = mount(TestComponent);
    const vm = wrapper.vm as any;

    expect(vm.isDirty).toBe(false);

    vm.values.email = "test@example.com";

    expect(vm.isDirty).toBe(true);
  });

  it("resets form", () => {
    const wrapper = mount(TestComponent);
    const vm = wrapper.vm as any;

    vm.values.email = "test@example.com";
    vm.reset();

    expect(vm.values.email).toBe("");
    expect(vm.isDirty).toBe(false);
  });
});
```

---

## Best Practices

### 1. Always Clean Up

```typescript
export function useInterval(callback: () => void, delay: number) {
  const intervalId = ref<ReturnType<typeof setInterval>>();

  function start() {
    stop();
    intervalId.value = setInterval(callback, delay);
  }

  function stop() {
    if (intervalId.value) {
      clearInterval(intervalId.value);
      intervalId.value = undefined;
    }
  }

  // Always clean up on unmount
  onUnmounted(stop);

  return { start, stop };
}
```

### 2. Return Readonly State

```typescript
export function useData() {
  const data = ref<Data | null>(null);

  // ✅ Good: Return readonly to prevent external mutations
  return {
    data: readonly(data),
    setData: (newData: Data) => {
      data.value = newData;
    },
  };
}
```

### 3. Handle SSR Properly

```typescript
export function useWindowSize() {
  // Default values for SSR
  const width = ref(0);
  const height = ref(0);

  // Only run on client
  if (import.meta.client) {
    width.value = window.innerWidth;
    height.value = window.innerHeight;

    window.addEventListener("resize", () => {
      width.value = window.innerWidth;
      height.value = window.innerHeight;
    });
  }

  return { width, height };
}
```

### 4. Type Everything

```typescript
// ✅ Good: Full type coverage
interface UseCounterReturn {
  count: Readonly<Ref<number>>;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

export function useCounter(initial = 0): UseCounterReturn {
  // ...implementation
}
```

---

## Next Steps

Continue to [State Management](./06-state-management.md) to learn about Pinia stores and global state management patterns with TypeScript.
