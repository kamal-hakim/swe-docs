# Testing Strategy

## Overview

This document covers the comprehensive testing strategy for Nuxt applications using a **three-tier testing architecture**: unit testing with Vitest, component and integration testing with @nuxt/test-utils and happy-dom, and end-to-end testing with Playwright. All tests follow the **Arrange-Act-Assert** pattern with proper TypeScript typing.

## Three-Tier Testing Architecture

| Tier       | Tool                         | Purpose                                       | Speed  | Coverage       |
| ---------- | ---------------------------- | --------------------------------------------- | ------ | -------------- |
| **Tier 1** | Vitest                       | Unit tests for functions, composables, stores | Fast   | High           |
| **Tier 2** | @nuxt/test-utils + happy-dom | Component and integration tests               | Medium | Medium         |
| **Tier 3** | Playwright                   | End-to-end tests with real browser            | Slow   | Critical paths |

---

## Installation and Setup

### Install Dependencies

```bash
# Tier 1 & 2: Unit and Integration Testing
pnpm add -D vitest @vue/test-utils @nuxt/test-utils happy-dom @vitest/coverage-v8

# Tier 3: E2E Testing (covered in next chapter)
pnpm add -D @playwright/test

# Testing utilities
pnpm add -D @faker-js/faker msw
```

### Vitest Configuration

```typescript
// vitest.config.ts
import { defineVitestConfig } from "@nuxt/test-utils/config";

export default defineVitestConfig({
  test: {
    // Global settings
    globals: true,
    environment: "happy-dom",

    // Setup files
    setupFiles: ["./tests/setup.ts"],

    // Test file patterns
    include: ["tests/unit/**/*.test.ts", "tests/integration/**/*.test.ts"],
    exclude: ["tests/e2e/**", "node_modules", ".nuxt"],

    // Coverage configuration
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html", "lcov"],
      reportsDirectory: "./coverage",
      exclude: [
        "node_modules/",
        ".nuxt/",
        ".output/",
        "tests/",
        "**/*.d.ts",
        "**/*.config.*",
        "**/types/**",
      ],
      thresholds: {
        lines: 80,
        branches: 80,
        functions: 80,
        statements: 80,
      },
    },

    // Reporter configuration
    reporters: process.env.CI ? ["default", "junit"] : ["default"],
    outputFile: {
      junit: "./test-results/junit.xml",
    },

    // Timeout
    testTimeout: 10000,
    hookTimeout: 10000,

    // Type checking
    typecheck: {
      enabled: true,
      tsconfig: "./tsconfig.json",
    },
  },
});
```

### Test Setup File

```typescript
// tests/setup.ts
import { vi, beforeEach, afterEach } from "vitest";
import { config } from "@vue/test-utils";
import { createVuetify } from "vuetify";
import * as components from "vuetify/components";
import * as directives from "vuetify/directives";

// ==========================================
// Vuetify Setup
// ==========================================

const vuetify = createVuetify({
  components,
  directives,
});

config.global.plugins = [vuetify];

// ==========================================
// Global Mocks
// ==========================================

// Mock browser APIs
if (typeof window !== "undefined") {
  // matchMedia
  Object.defineProperty(window, "matchMedia", {
    writable: true,
    value: vi.fn().mockImplementation((query: string) => ({
      matches: false,
      media: query,
      onchange: null,
      addListener: vi.fn(),
      removeListener: vi.fn(),
      addEventListener: vi.fn(),
      removeEventListener: vi.fn(),
      dispatchEvent: vi.fn(),
    })),
  });

  // IntersectionObserver
  class MockIntersectionObserver {
    observe = vi.fn();
    disconnect = vi.fn();
    unobserve = vi.fn();
  }
  window.IntersectionObserver = MockIntersectionObserver as any;

  // ResizeObserver
  class MockResizeObserver {
    observe = vi.fn();
    disconnect = vi.fn();
    unobserve = vi.fn();
  }
  window.ResizeObserver = MockResizeObserver as any;

  // scrollTo
  window.scrollTo = vi.fn();
}

// ==========================================
// Vue Router Mock
// ==========================================

vi.mock("vue-router", async () => {
  const actual = await vi.importActual<typeof import("vue-router")>(
    "vue-router"
  );
  return {
    ...actual,
    useRouter: vi.fn(() => ({
      push: vi.fn(),
      replace: vi.fn(),
      go: vi.fn(),
      back: vi.fn(),
      forward: vi.fn(),
      currentRoute: { value: { path: "/", params: {}, query: {} } },
    })),
    useRoute: vi.fn(() => ({
      params: {},
      query: {},
      path: "/",
      name: "index",
      meta: {},
      fullPath: "/",
    })),
  };
});

// ==========================================
// Nuxt Composables Mock
// ==========================================

vi.mock("#app", () => ({
  useNuxtApp: vi.fn(() => ({
    $i18n: { locale: { value: "en" } },
  })),
  useRuntimeConfig: vi.fn(() => ({
    public: {
      apiBaseUrl: "http://localhost:3000/api",
    },
  })),
  navigateTo: vi.fn(),
  useState: vi.fn((key: string, init?: () => any) => ref(init?.())),
}));

// ==========================================
// i18n Mock
// ==========================================

vi.mock("vue-i18n", () => ({
  useI18n: vi.fn(() => ({
    t: (key: string, params?: any) => key,
    n: (value: number) => String(value),
    d: (date: Date) => date.toISOString(),
    locale: ref("en"),
  })),
}));

// ==========================================
// Cleanup
// ==========================================

beforeEach(() => {
  vi.clearAllMocks();
});

afterEach(() => {
  vi.restoreAllMocks();
});

// ==========================================
// Global Test Utilities
// ==========================================

export const flushPromises = () =>
  new Promise((resolve) => setTimeout(resolve, 0));
export const wait = (ms: number) =>
  new Promise((resolve) => setTimeout(resolve, ms));
```

---

## Test Directory Structure

```
tests/
├── setup.ts                    # Global test setup
├── fixtures/                   # Test data factories
│   ├── index.ts               # Factory exports
│   ├── user.factory.ts        # User factory
│   ├── product.factory.ts     # Product factory
│   └── order.factory.ts       # Order factory
├── utils/                      # Test utilities
│   ├── index.ts               # Utility exports
│   ├── mocks.ts               # Mock implementations
│   ├── helpers.ts             # Helper functions
│   └── vuetify.ts             # Vuetify test helpers
├── unit/                       # Tier 1: Unit tests
│   ├── utils/                 # Utility function tests
│   │   ├── format.test.ts
│   │   └── validation.test.ts
│   ├── composables/           # Composable tests
│   │   ├── useApi.test.ts
│   │   └── useForm.test.ts
│   └── stores/                # Store tests
│       ├── auth.test.ts
│       └── user.test.ts
├── integration/                # Tier 2: Integration tests
│   ├── components/            # Component tests
│   │   ├── common/
│   │   │   ├── AppButton.test.ts
│   │   │   └── AppTextField.test.ts
│   │   └── user/
│   │       └── UserCard.test.ts
│   └── pages/                 # Page tests
│       ├── login.test.ts
│       └── dashboard.test.ts
└── e2e/                        # Tier 3: E2E tests (Playwright)
    ├── pages/                 # Page object models
    ├── fixtures/              # E2E fixtures
    └── specs/                 # Test specifications
```

---

## Test Fixtures and Factories

### Factory Pattern

```typescript
// tests/fixtures/user.factory.ts
import { faker } from "@faker-js/faker";

export interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  avatar?: string;
  role: "admin" | "user" | "guest";
  createdAt: string;
  updatedAt: string;
}

export interface CreateUserOptions {
  id?: string;
  email?: string;
  firstName?: string;
  lastName?: string;
  avatar?: string;
  role?: User["role"];
  createdAt?: string;
  updatedAt?: string;
}

export function createUser(options: CreateUserOptions = {}): User {
  const now = new Date().toISOString();

  return {
    id: options.id ?? faker.string.uuid(),
    email: options.email ?? faker.internet.email(),
    firstName: options.firstName ?? faker.person.firstName(),
    lastName: options.lastName ?? faker.person.lastName(),
    avatar: options.avatar ?? faker.image.avatar(),
    role: options.role ?? "user",
    createdAt: options.createdAt ?? now,
    updatedAt: options.updatedAt ?? now,
  };
}

export function createUsers(
  count: number,
  options: CreateUserOptions = {}
): User[] {
  return Array.from({ length: count }, () => createUser(options));
}

export function createAdmin(options: CreateUserOptions = {}): User {
  return createUser({ ...options, role: "admin" });
}

export function createGuest(options: CreateUserOptions = {}): User {
  return createUser({ ...options, role: "guest" });
}
```

### Product Factory

```typescript
// tests/fixtures/product.factory.ts
import { faker } from "@faker-js/faker";

export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  salePrice?: number;
  category: string;
  stock: number;
  images: string[];
  createdAt: string;
}

export interface CreateProductOptions {
  id?: string;
  name?: string;
  description?: string;
  price?: number;
  salePrice?: number;
  category?: string;
  stock?: number;
  images?: string[];
  createdAt?: string;
}

export function createProduct(options: CreateProductOptions = {}): Product {
  return {
    id: options.id ?? faker.string.uuid(),
    name: options.name ?? faker.commerce.productName(),
    description: options.description ?? faker.commerce.productDescription(),
    price:
      options.price ?? parseFloat(faker.commerce.price({ min: 10, max: 1000 })),
    salePrice: options.salePrice,
    category: options.category ?? faker.commerce.department(),
    stock: options.stock ?? faker.number.int({ min: 0, max: 100 }),
    images: options.images ?? [faker.image.url()],
    createdAt: options.createdAt ?? new Date().toISOString(),
  };
}

export function createProducts(
  count: number,
  options: CreateProductOptions = {}
): Product[] {
  return Array.from({ length: count }, () => createProduct(options));
}

export function createOutOfStockProduct(
  options: CreateProductOptions = {}
): Product {
  return createProduct({ ...options, stock: 0 });
}

export function createSaleProduct(options: CreateProductOptions = {}): Product {
  const price =
    options.price ?? parseFloat(faker.commerce.price({ min: 50, max: 500 }));
  return createProduct({
    ...options,
    price,
    salePrice: price * 0.8,
  });
}
```

### Factory Index

```typescript
// tests/fixtures/index.ts
export * from "./user.factory";
export * from "./product.factory";
export * from "./order.factory";

// Re-export faker for custom data
export { faker } from "@faker-js/faker";
```

---

## Test Utilities

### Vuetify Test Helpers

```typescript
// tests/utils/vuetify.ts
import { mount, VueWrapper } from "@vue/test-utils";
import { createVuetify } from "vuetify";
import * as components from "vuetify/components";
import * as directives from "vuetify/directives";
import { createPinia, setActivePinia } from "pinia";
import type { Component } from "vue";

// Create Vuetify instance for tests
export function createTestVuetify() {
  return createVuetify({
    components,
    directives,
  });
}

// Mount options interface
export interface MountWithVuetifyOptions {
  props?: Record<string, any>;
  slots?: Record<string, any>;
  stubs?: Record<string, any>;
  mocks?: Record<string, any>;
  attachTo?: HTMLElement | string;
  shallow?: boolean;
}

// Mount component with Vuetify
export function mountWithVuetify<T extends Component>(
  component: T,
  options: MountWithVuetifyOptions = {}
): VueWrapper<InstanceType<T>> {
  const vuetify = createTestVuetify();
  const pinia = createPinia();
  setActivePinia(pinia);

  return mount(component, {
    props: options.props,
    slots: options.slots,
    attachTo: options.attachTo,
    shallow: options.shallow,
    global: {
      plugins: [vuetify, pinia],
      stubs: {
        NuxtLink: {
          template: '<a :href="to"><slot /></a>',
          props: ["to"],
        },
        ...options.stubs,
      },
      mocks: {
        $t: (key: string) => key,
        $n: (value: number) => String(value),
        $d: (date: Date) => date.toISOString(),
        ...options.mocks,
      },
    },
  }) as VueWrapper<InstanceType<T>>;
}

// Find element by data-testid
export function findByTestId(wrapper: VueWrapper<any>, testId: string) {
  return wrapper.find(`[data-testid="${testId}"]`);
}

// Find all elements by data-testid
export function findAllByTestId(wrapper: VueWrapper<any>, testId: string) {
  return wrapper.findAll(`[data-testid="${testId}"]`);
}

// Wait for Vuetify transitions
export async function waitForVuetifyTransition(wrapper: VueWrapper<any>) {
  await wrapper.vm.$nextTick();
  await new Promise((resolve) => setTimeout(resolve, 300));
}

// Trigger Vuetify menu open
export async function openVuetifyMenu(
  wrapper: VueWrapper<any>,
  activator: string
) {
  await wrapper.find(activator).trigger("click");
  await waitForVuetifyTransition(wrapper);
}

// Close Vuetify dialog
export async function closeVuetifyDialog(wrapper: VueWrapper<any>) {
  const overlay = document.querySelector(".v-overlay");
  if (overlay) {
    overlay.dispatchEvent(new Event("click"));
  }
  await waitForVuetifyTransition(wrapper);
}
```

### Mock Implementations

```typescript
// tests/utils/mocks.ts
import { vi } from "vitest";

// API mock
export function createApiMock() {
  return {
    get: vi.fn(),
    post: vi.fn(),
    put: vi.fn(),
    patch: vi.fn(),
    delete: vi.fn(),
  };
}

// $fetch mock
export function mockFetch<T>(response: T) {
  return vi.fn().mockResolvedValue(response);
}

export function mockFetchError(error: Error | string) {
  const err = typeof error === "string" ? new Error(error) : error;
  return vi.fn().mockRejectedValue(err);
}

// Router mock
export function createRouterMock() {
  return {
    push: vi.fn(),
    replace: vi.fn(),
    go: vi.fn(),
    back: vi.fn(),
    forward: vi.fn(),
    currentRoute: {
      value: {
        path: "/",
        params: {},
        query: {},
        meta: {},
      },
    },
  };
}

// Auth store mock
export function createAuthStoreMock(overrides = {}) {
  return {
    isAuthenticated: false,
    user: null,
    tokens: null,
    login: vi.fn(),
    logout: vi.fn(),
    refreshTokens: vi.fn(),
    ...overrides,
  };
}

// Notification store mock
export function createNotificationStoreMock() {
  return {
    notifications: [],
    add: vi.fn(),
    dismiss: vi.fn(),
    dismissAll: vi.fn(),
    success: vi.fn(),
    error: vi.fn(),
    warning: vi.fn(),
    info: vi.fn(),
  };
}
```

### Helper Functions

```typescript
// tests/utils/helpers.ts
import { VueWrapper } from "@vue/test-utils";

// Wait for promises to resolve
export function flushPromises(): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, 0));
}

// Wait for specific time
export function wait(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

// Fill form field
export async function fillField(
  wrapper: VueWrapper<any>,
  selector: string,
  value: string
): Promise<void> {
  const input = wrapper.find(selector);
  await input.setValue(value);
  await input.trigger("blur");
}

// Submit form
export async function submitForm(wrapper: VueWrapper<any>): Promise<void> {
  await wrapper.find("form").trigger("submit");
  await flushPromises();
}

// Check if element is visible
export function isVisible(wrapper: VueWrapper<any>, selector: string): boolean {
  const element = wrapper.find(selector);
  return element.exists() && element.isVisible();
}

// Get text content
export function getTextContent(
  wrapper: VueWrapper<any>,
  selector: string
): string {
  return wrapper.find(selector).text().trim();
}

// Assert error message
export function assertErrorMessage(
  wrapper: VueWrapper<any>,
  expectedMessage: string
): void {
  const errorMessages = wrapper.findAll(".v-messages__message");
  const messages = errorMessages.map((el) => el.text());
  expect(messages).toContain(expectedMessage);
}

// Assert no error messages
export function assertNoErrors(wrapper: VueWrapper<any>): void {
  const errorMessages = wrapper.findAll(".v-messages__message");
  expect(errorMessages.length).toBe(0);
}
```

---

## Tier 1: Unit Tests

### Testing Utility Functions

```typescript
// tests/unit/utils/format.test.ts
import { describe, it, expect } from "vitest";
import { formatCurrency, formatDate, truncate } from "~/utils/format";

describe("formatCurrency", () => {
  it("formats USD currency correctly", () => {
    // Arrange
    const value = 1234.56;

    // Act
    const result = formatCurrency(value, "USD");

    // Assert
    expect(result).toBe("$1,234.56");
  });

  it("formats EUR currency correctly", () => {
    // Arrange
    const value = 1234.56;

    // Act
    const result = formatCurrency(value, "EUR", "de-DE");

    // Assert
    expect(result).toContain("€");
    expect(result).toContain("1.234,56");
  });

  it("handles zero value", () => {
    // Arrange
    const value = 0;

    // Act
    const result = formatCurrency(value);

    // Assert
    expect(result).toBe("$0.00");
  });

  it("handles negative values", () => {
    // Arrange
    const value = -100;

    // Act
    const result = formatCurrency(value);

    // Assert
    expect(result).toBe("-$100.00");
  });
});

describe("formatDate", () => {
  it("formats date with default options", () => {
    // Arrange
    const date = new Date("2024-01-15T10:30:00Z");

    // Act
    const result = formatDate(date);

    // Assert
    expect(result).toContain("2024");
    expect(result).toContain("Jan");
  });

  it("formats date with custom format", () => {
    // Arrange
    const date = new Date("2024-01-15");

    // Act
    const result = formatDate(date, { dateStyle: "full" });

    // Assert
    expect(result).toContain("Monday");
    expect(result).toContain("January");
  });

  it("returns fallback for invalid date", () => {
    // Arrange
    const invalidDate = new Date("invalid");

    // Act
    const result = formatDate(invalidDate);

    // Assert
    expect(result).toBe("Invalid date");
  });
});

describe("truncate", () => {
  it("truncates long strings", () => {
    // Arrange
    const text = "This is a very long string that should be truncated";

    // Act
    const result = truncate(text, 20);

    // Assert
    expect(result).toBe("This is a very long...");
    expect(result.length).toBeLessThanOrEqual(23);
  });

  it("does not truncate short strings", () => {
    // Arrange
    const text = "Short";

    // Act
    const result = truncate(text, 20);

    // Assert
    expect(result).toBe("Short");
  });

  it("uses custom suffix", () => {
    // Arrange
    const text = "Long text here";

    // Act
    const result = truncate(text, 8, "…");

    // Assert
    expect(result).toBe("Long tex…");
  });
});
```

### Testing Composables

```typescript
// tests/unit/composables/useApi.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { useApi } from "~/composables/api/useApi";

// Mock $fetch
const mockFetch = vi.fn();
vi.stubGlobal("$fetch", mockFetch);

describe("useApi", () => {
  beforeEach(() => {
    mockFetch.mockReset();
  });

  describe("get", () => {
    it("fetches data successfully", async () => {
      // Arrange
      const mockData = { id: 1, name: "Test" };
      mockFetch.mockResolvedValueOnce(mockData);
      const { get, data, loading, error } = useApi();

      // Act
      await get("/api/test");

      // Assert
      expect(data.value).toEqual(mockData);
      expect(loading.value).toBe(false);
      expect(error.value).toBeNull();
    });

    it("handles fetch error", async () => {
      // Arrange
      const mockError = new Error("Network error");
      mockFetch.mockRejectedValueOnce(mockError);
      const { get, data, loading, error } = useApi();

      // Act
      await expect(get("/api/test")).rejects.toThrow("Network error");

      // Assert
      expect(data.value).toBeNull();
      expect(loading.value).toBe(false);
      expect(error.value).toBe(mockError);
    });

    it("sets loading state during fetch", async () => {
      // Arrange
      mockFetch.mockImplementation(
        () => new Promise((resolve) => setTimeout(() => resolve({}), 100))
      );
      const { get, loading } = useApi();

      // Act
      const promise = get("/api/test");

      // Assert - loading should be true during fetch
      expect(loading.value).toBe(true);

      await promise;
      expect(loading.value).toBe(false);
    });
  });

  describe("post", () => {
    it("posts data successfully", async () => {
      // Arrange
      const requestData = { name: "New Item" };
      const responseData = { id: 1, name: "New Item" };
      mockFetch.mockResolvedValueOnce(responseData);
      const { post } = useApi();

      // Act
      const result = await post("/api/items", requestData);

      // Assert
      expect(result).toEqual(responseData);
      expect(mockFetch).toHaveBeenCalledWith("/api/items", {
        method: "POST",
        body: requestData,
      });
    });
  });
});
```

### Testing Stores

```typescript
// tests/unit/stores/auth.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { setActivePinia, createPinia } from "pinia";
import { useAuthStore } from "~/stores/auth";
import { createUser } from "../../fixtures";

// Mock $fetch
const mockFetch = vi.fn();
vi.stubGlobal("$fetch", mockFetch);

describe("useAuthStore", () => {
  beforeEach(() => {
    setActivePinia(createPinia());
    mockFetch.mockReset();
    localStorage.clear();
  });

  describe("initial state", () => {
    it("starts with unauthenticated state", () => {
      // Arrange & Act
      const store = useAuthStore();

      // Assert
      expect(store.isAuthenticated).toBe(false);
      expect(store.user).toBeNull();
      expect(store.tokens).toBeNull();
    });
  });

  describe("login", () => {
    it("authenticates user successfully", async () => {
      // Arrange
      const mockUser = createUser();
      const mockTokens = {
        accessToken: "access-token",
        refreshToken: "refresh-token",
        expiresAt: Date.now() + 3600000,
      };
      mockFetch.mockResolvedValueOnce({ user: mockUser, tokens: mockTokens });
      const store = useAuthStore();

      // Act
      await store.login({ email: "test@example.com", password: "password" });

      // Assert
      expect(store.isAuthenticated).toBe(true);
      expect(store.user).toEqual(mockUser);
      expect(store.tokens).toEqual(mockTokens);
    });

    it("handles login failure", async () => {
      // Arrange
      mockFetch.mockRejectedValueOnce(new Error("Invalid credentials"));
      const store = useAuthStore();

      // Act & Assert
      await expect(
        store.login({ email: "test@example.com", password: "wrong" })
      ).rejects.toThrow("Invalid credentials");

      expect(store.isAuthenticated).toBe(false);
      expect(store.user).toBeNull();
    });
  });

  describe("logout", () => {
    it("clears authentication state", async () => {
      // Arrange
      const store = useAuthStore();
      const mockUser = createUser();
      mockFetch.mockResolvedValueOnce({
        user: mockUser,
        tokens: {
          accessToken: "token",
          refreshToken: "refresh",
          expiresAt: Date.now() + 3600000,
        },
      });
      await store.login({ email: "test@example.com", password: "password" });
      expect(store.isAuthenticated).toBe(true);

      // Act
      mockFetch.mockResolvedValueOnce({});
      await store.logout();

      // Assert
      expect(store.isAuthenticated).toBe(false);
      expect(store.user).toBeNull();
      expect(store.tokens).toBeNull();
    });
  });

  describe("isAuthenticated", () => {
    it("returns false when token is expired", async () => {
      // Arrange
      const store = useAuthStore();
      mockFetch.mockResolvedValueOnce({
        user: createUser(),
        tokens: {
          accessToken: "token",
          refreshToken: "refresh",
          expiresAt: Date.now() - 1000, // Expired
        },
      });
      await store.login({ email: "test@example.com", password: "password" });

      // Act & Assert
      expect(store.isAuthenticated).toBe(false);
    });
  });
});
```

---

## Tier 2: Integration Tests

### Testing Components

```typescript
// tests/integration/components/common/AppTextField.test.ts
import { describe, it, expect } from "vitest";
import { mountWithVuetify, findByTestId } from "../../../utils/vuetify";
import AppTextField from "~/components/common/AppTextField.vue";

describe("AppTextField", () => {
  it("renders with label", () => {
    // Arrange & Act
    const wrapper = mountWithVuetify(AppTextField, {
      props: {
        name: "email",
        label: "Email Address",
      },
    });

    // Assert
    expect(wrapper.text()).toContain("Email Address");
  });

  it("displays error message", async () => {
    // Arrange
    const wrapper = mountWithVuetify(AppTextField, {
      props: {
        name: "email",
        label: "Email",
        errorMessage: "Invalid email format",
      },
    });

    // Act
    await wrapper.vm.$nextTick();

    // Assert
    expect(wrapper.text()).toContain("Invalid email format");
  });

  it("emits update event on input", async () => {
    // Arrange
    const wrapper = mountWithVuetify(AppTextField, {
      props: {
        name: "email",
        modelValue: "",
      },
    });

    // Act
    const input = wrapper.find("input");
    await input.setValue("test@example.com");

    // Assert
    expect(wrapper.emitted("update:modelValue")).toBeTruthy();
    expect(wrapper.emitted("update:modelValue")![0]).toEqual([
      "test@example.com",
    ]);
  });

  it("applies disabled state", () => {
    // Arrange & Act
    const wrapper = mountWithVuetify(AppTextField, {
      props: {
        name: "email",
        disabled: true,
      },
    });

    // Assert
    const input = wrapper.find("input");
    expect(input.attributes("disabled")).toBeDefined();
  });

  it("shows prepend icon", () => {
    // Arrange & Act
    const wrapper = mountWithVuetify(AppTextField, {
      props: {
        name: "email",
        prependInnerIcon: "mdi-email",
      },
    });

    // Assert
    expect(wrapper.find(".mdi-email").exists()).toBe(true);
  });
});
```

### Testing User Components

```typescript
// tests/integration/components/user/UserCard.test.ts
import { describe, it, expect, vi } from "vitest";
import { mountWithVuetify, findByTestId } from "../../../utils/vuetify";
import UserCard from "~/components/user/UserCard.vue";
import { createUser, createAdmin } from "../../../fixtures";

describe("UserCard", () => {
  const defaultUser = createUser({
    firstName: "John",
    lastName: "Doe",
    email: "john@example.com",
  });

  it("renders user information", () => {
    // Arrange & Act
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: defaultUser },
    });

    // Assert
    expect(wrapper.text()).toContain("John");
    expect(wrapper.text()).toContain("Doe");
    expect(wrapper.text()).toContain("john@example.com");
  });

  it("displays user avatar", () => {
    // Arrange & Act
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: defaultUser },
    });

    // Assert
    const avatar = wrapper.find(".v-avatar img");
    expect(avatar.exists()).toBe(true);
    expect(avatar.attributes("src")).toBe(defaultUser.avatar);
  });

  it("shows admin badge for admin users", () => {
    // Arrange
    const adminUser = createAdmin();

    // Act
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: adminUser },
    });

    // Assert
    expect(wrapper.find(".v-chip").exists()).toBe(true);
    expect(wrapper.text()).toContain("Admin");
  });

  it("hides admin badge for regular users", () => {
    // Arrange & Act
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: defaultUser },
    });

    // Assert
    const adminBadge = wrapper
      .findAll(".v-chip")
      .filter((c) => c.text().includes("Admin"));
    expect(adminBadge.length).toBe(0);
  });

  it("emits select event on click", async () => {
    // Arrange
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: defaultUser },
    });

    // Act
    await wrapper.find(".v-card").trigger("click");

    // Assert
    expect(wrapper.emitted("select")).toBeTruthy();
    expect(wrapper.emitted("select")![0]).toEqual([defaultUser]);
  });

  it("emits edit event when edit button clicked", async () => {
    // Arrange
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: defaultUser, showActions: true },
    });

    // Act
    await findByTestId(wrapper, "edit-button").trigger("click");

    // Assert
    expect(wrapper.emitted("edit")).toBeTruthy();
    expect(wrapper.emitted("edit")![0]).toEqual([defaultUser]);
  });

  it("emits delete event when delete button clicked", async () => {
    // Arrange
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: defaultUser, showActions: true },
    });

    // Act
    await findByTestId(wrapper, "delete-button").trigger("click");

    // Assert
    expect(wrapper.emitted("delete")).toBeTruthy();
    expect(wrapper.emitted("delete")![0]).toEqual([defaultUser]);
  });
});
```

### Testing Pages

```typescript
// tests/integration/pages/login.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { mountSuspended } from "@nuxt/test-utils/runtime";
import LoginPage from "~/pages/login.vue";
import { flushPromises, fillField, submitForm } from "../../utils/helpers";

// Mock stores
const mockLogin = vi.fn();
vi.mock("~/stores/auth", () => ({
  useAuthStore: () => ({
    login: mockLogin,
    isAuthenticated: false,
  }),
}));

// Mock navigation
const mockNavigateTo = vi.fn();
vi.mock("#app", () => ({
  navigateTo: mockNavigateTo,
}));

describe("LoginPage", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("renders login form", async () => {
    // Arrange & Act
    const wrapper = await mountSuspended(LoginPage);

    // Assert
    expect(wrapper.find("form").exists()).toBe(true);
    expect(wrapper.find('input[type="email"]').exists()).toBe(true);
    expect(wrapper.find('input[type="password"]').exists()).toBe(true);
  });

  it("shows validation errors for empty fields", async () => {
    // Arrange
    const wrapper = await mountSuspended(LoginPage);

    // Act
    await submitForm(wrapper);

    // Assert
    expect(wrapper.text()).toContain("required");
  });

  it("calls login on valid submission", async () => {
    // Arrange
    mockLogin.mockResolvedValueOnce({});
    const wrapper = await mountSuspended(LoginPage);

    // Act
    await fillField(wrapper, 'input[type="email"]', "test@example.com");
    await fillField(wrapper, 'input[type="password"]', "password123");
    await submitForm(wrapper);

    // Assert
    expect(mockLogin).toHaveBeenCalledWith({
      email: "test@example.com",
      password: "password123",
    });
  });

  it("navigates to dashboard on successful login", async () => {
    // Arrange
    mockLogin.mockResolvedValueOnce({});
    const wrapper = await mountSuspended(LoginPage);

    // Act
    await fillField(wrapper, 'input[type="email"]', "test@example.com");
    await fillField(wrapper, 'input[type="password"]', "password123");
    await submitForm(wrapper);
    await flushPromises();

    // Assert
    expect(mockNavigateTo).toHaveBeenCalledWith("/dashboard");
  });

  it("displays error message on login failure", async () => {
    // Arrange
    mockLogin.mockRejectedValueOnce(new Error("Invalid credentials"));
    const wrapper = await mountSuspended(LoginPage);

    // Act
    await fillField(wrapper, 'input[type="email"]', "test@example.com");
    await fillField(wrapper, 'input[type="password"]', "wrongpassword");
    await submitForm(wrapper);
    await flushPromises();

    // Assert
    expect(wrapper.text()).toContain("Invalid credentials");
  });
});
```

---

## Package Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui",
    "test:unit": "vitest run tests/unit",
    "test:integration": "vitest run tests/integration",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:all": "pnpm test:run && pnpm test:e2e"
  }
}
```

---

## Best Practices

### 1. Follow Arrange-Act-Assert Pattern

```typescript
it("should do something", () => {
  // Arrange - Set up test data and conditions
  const input = "test";

  // Act - Execute the code being tested
  const result = myFunction(input);

  // Assert - Verify the results
  expect(result).toBe("expected");
});
```

### 2. Use Descriptive Test Names

```typescript
// ✅ Good: Describes behavior
it("displays error message when email is invalid", () => {});
it("disables submit button while form is submitting", () => {});

// ❌ Bad: Vague descriptions
it("works correctly", () => {});
it("handles error", () => {});
```

### 3. Test One Thing Per Test

```typescript
// ✅ Good: Single assertion focus
it("validates email format", () => {
  expect(isValidEmail("invalid")).toBe(false);
});

it("accepts valid email", () => {
  expect(isValidEmail("test@example.com")).toBe(true);
});

// ❌ Bad: Multiple unrelated assertions
it("validates email", () => {
  expect(isValidEmail("invalid")).toBe(false);
  expect(isValidEmail("test@example.com")).toBe(true);
  expect(isValidEmail("")).toBe(false);
});
```

### 4. Use Factories for Test Data

```typescript
// ✅ Good: Use factories
const user = createUser({ role: "admin" });

// ❌ Bad: Inline test data
const user = {
  id: "123",
  email: "test@example.com",
  firstName: "John",
  // ... many more fields
};
```

### 5. Isolate Tests

```typescript
beforeEach(() => {
  vi.clearAllMocks();
  localStorage.clear();
});

afterEach(() => {
  vi.restoreAllMocks();
});
```

---

## Next Steps

Continue to [E2E Testing](./12-e2e-testing.md) to learn about Playwright configuration, page object models, and cross-browser testing.
