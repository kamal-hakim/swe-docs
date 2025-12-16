# E2E Testing with Playwright

## Overview

This document covers **Tier 3** of the three-tier testing architecture: end-to-end testing using **Playwright**. It includes configuration, page object models, cross-browser support, custom assertions, test helpers, and visual regression testing.

## Technology Stack

| Tool                  | Purpose                    |
| --------------------- | -------------------------- |
| **Playwright**        | E2E testing framework      |
| **@playwright/test**  | Test runner and assertions |
| **Page Object Model** | Test organization pattern  |
| **Axe-core**          | Accessibility testing      |

---

## Installation and Setup

### Install Dependencies

```bash
# Install Playwright
pnpm add -D @playwright/test

# Install browsers
pnpm exec playwright install

# Install accessibility testing
pnpm add -D @axe-core/playwright
```

### Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

const isCI = process.env.CI === "true";
const baseURL = process.env.PLAYWRIGHT_BASE_URL || "http://localhost:3000";

export default defineConfig({
  // Test directory
  testDir: "./tests/e2e/specs",

  // Test file pattern
  testMatch: "**/*.spec.ts",

  // Timeout settings
  timeout: 30000,
  expect: {
    timeout: 5000,
  },

  // Parallel execution
  fullyParallel: true,
  workers: isCI ? 1 : undefined,

  // Retry on failure
  retries: isCI ? 2 : 0,

  // Reporter configuration
  reporter: isCI
    ? [
        ["html", { outputFolder: "playwright-report" }],
        ["junit", { outputFile: "test-results/e2e-results.xml" }],
        ["github"],
      ]
    : [["html", { open: "never" }], ["list"]],

  // Global setup/teardown
  globalSetup: "./tests/e2e/global-setup.ts",
  globalTeardown: "./tests/e2e/global-teardown.ts",

  // Shared settings
  use: {
    baseURL,

    // Browser settings
    headless: isCI,
    viewport: { width: 1280, height: 720 },

    // Artifacts
    screenshot: "only-on-failure",
    video: isCI ? "retain-on-failure" : "off",
    trace: isCI ? "retain-on-failure" : "on-first-retry",

    // Navigation
    actionTimeout: 10000,
    navigationTimeout: 30000,

    // Locale
    locale: "en-US",
    timezoneId: "America/New_York",
  },

  // Browser projects
  projects: [
    // Desktop browsers
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
    },
    {
      name: "webkit",
      use: { ...devices["Desktop Safari"] },
    },

    // Mobile browsers
    {
      name: "mobile-chrome",
      use: { ...devices["Pixel 5"] },
    },
    {
      name: "mobile-safari",
      use: { ...devices["iPhone 12"] },
    },

    // Tablet
    {
      name: "tablet",
      use: { ...devices["iPad Pro 11"] },
    },
  ],

  // Development server
  webServer: {
    command: "pnpm dev",
    url: baseURL,
    reuseExistingServer: !isCI,
    timeout: 120000,
  },
});
```

### Global Setup

```typescript
// tests/e2e/global-setup.ts
import { chromium, FullConfig } from "@playwright/test";

async function globalSetup(config: FullConfig) {
  const { baseURL } = config.projects[0].use;

  // Create authenticated state
  const browser = await chromium.launch();
  const page = await browser.newPage();

  // Login and save state
  await page.goto(`${baseURL}/login`);
  await page.fill('[data-testid="email-input"]', "test@example.com");
  await page.fill('[data-testid="password-input"]', "password123");
  await page.click('[data-testid="login-button"]');
  await page.waitForURL("**/dashboard");

  // Save authentication state
  await page.context().storageState({ path: "./tests/e2e/.auth/user.json" });

  await browser.close();
}

export default globalSetup;
```

### Global Teardown

```typescript
// tests/e2e/global-teardown.ts
import { FullConfig } from "@playwright/test";
import fs from "fs";
import path from "path";

async function globalTeardown(config: FullConfig) {
  // Clean up auth state
  const authPath = "./tests/e2e/.auth";
  if (fs.existsSync(authPath)) {
    fs.rmSync(authPath, { recursive: true });
  }

  // Clean up any test data
  // await cleanupTestData()
}

export default globalTeardown;
```

---

## Directory Structure

```
tests/e2e/
├── .auth/                      # Authentication state (gitignored)
│   └── user.json
├── fixtures/                   # Test fixtures
│   ├── index.ts               # Fixture exports
│   ├── auth.fixture.ts        # Authentication fixture
│   └── test-data.fixture.ts   # Test data fixture
├── pages/                      # Page Object Models
│   ├── base.page.ts           # Base page class
│   ├── login.page.ts          # Login page
│   ├── dashboard.page.ts      # Dashboard page
│   ├── profile.page.ts        # Profile page
│   └── components/            # Component page objects
│       ├── header.component.ts
│       ├── sidebar.component.ts
│       └── dialog.component.ts
├── helpers/                    # Test helpers
│   ├── assertions.ts          # Custom assertions
│   ├── actions.ts             # Common actions
│   └── selectors.ts           # Selector utilities
├── specs/                      # Test specifications
│   ├── auth/
│   │   ├── login.spec.ts
│   │   └── register.spec.ts
│   ├── dashboard/
│   │   └── dashboard.spec.ts
│   └── user/
│       └── profile.spec.ts
├── global-setup.ts            # Global setup
└── global-teardown.ts         # Global teardown
```

---

## Page Object Models

### Base Page

```typescript
// tests/e2e/pages/base.page.ts
import { Page, Locator, expect } from "@playwright/test";

export abstract class BasePage {
  readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Common selectors
  get loadingSpinner(): Locator {
    return this.page.locator(".v-progress-circular");
  }

  get snackbar(): Locator {
    return this.page.locator(".v-snackbar");
  }

  get dialog(): Locator {
    return this.page.locator(".v-dialog");
  }

  // Navigation
  abstract goto(): Promise<void>;

  // Common actions
  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState("networkidle");
  }

  async waitForLoading(): Promise<void> {
    await this.loadingSpinner.waitFor({ state: "hidden", timeout: 30000 });
  }

  async getSnackbarMessage(): Promise<string> {
    await this.snackbar.waitFor({ state: "visible" });
    return this.snackbar.textContent() ?? "";
  }

  async closeSnackbar(): Promise<void> {
    await this.snackbar.locator("button").click();
    await this.snackbar.waitFor({ state: "hidden" });
  }

  async closeDialog(): Promise<void> {
    await this.page.keyboard.press("Escape");
    await this.dialog.waitFor({ state: "hidden" });
  }

  // Assertions
  async expectUrl(path: string): Promise<void> {
    await expect(this.page).toHaveURL(new RegExp(path));
  }

  async expectTitle(title: string): Promise<void> {
    await expect(this.page).toHaveTitle(title);
  }

  // Screenshot
  async takeScreenshot(name: string): Promise<void> {
    await this.page.screenshot({
      path: `screenshots/${name}.png`,
      fullPage: true,
    });
  }
}
```

### Login Page

```typescript
// tests/e2e/pages/login.page.ts
import { Page, Locator, expect } from "@playwright/test";
import { BasePage } from "./base.page";

export class LoginPage extends BasePage {
  // Selectors
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly rememberMeCheckbox: Locator;
  readonly loginButton: Locator;
  readonly forgotPasswordLink: Locator;
  readonly registerLink: Locator;
  readonly errorAlert: Locator;

  constructor(page: Page) {
    super(page);

    this.emailInput = page.locator('[data-testid="email-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.rememberMeCheckbox = page.locator('[data-testid="remember-me"]');
    this.loginButton = page.locator('[data-testid="login-button"]');
    this.forgotPasswordLink = page.locator(
      '[data-testid="forgot-password-link"]'
    );
    this.registerLink = page.locator('[data-testid="register-link"]');
    this.errorAlert = page.locator('.v-alert[type="error"]');
  }

  // Navigation
  async goto(): Promise<void> {
    await this.page.goto("/login");
    await this.waitForPageLoad();
  }

  // Actions
  async fillEmail(email: string): Promise<void> {
    await this.emailInput.fill(email);
  }

  async fillPassword(password: string): Promise<void> {
    await this.passwordInput.fill(password);
  }

  async toggleRememberMe(): Promise<void> {
    await this.rememberMeCheckbox.click();
  }

  async clickLogin(): Promise<void> {
    await this.loginButton.click();
  }

  async login(email: string, password: string): Promise<void> {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.clickLogin();
  }

  async loginAndWaitForDashboard(
    email: string,
    password: string
  ): Promise<void> {
    await this.login(email, password);
    await this.page.waitForURL("**/dashboard");
  }

  async clickForgotPassword(): Promise<void> {
    await this.forgotPasswordLink.click();
  }

  async clickRegister(): Promise<void> {
    await this.registerLink.click();
  }

  // Assertions
  async expectEmailError(message: string): Promise<void> {
    const errorMessage = this.emailInput
      .locator("..")
      .locator(".v-messages__message");
    await expect(errorMessage).toContainText(message);
  }

  async expectPasswordError(message: string): Promise<void> {
    const errorMessage = this.passwordInput
      .locator("..")
      .locator(".v-messages__message");
    await expect(errorMessage).toContainText(message);
  }

  async expectLoginError(message: string): Promise<void> {
    await expect(this.errorAlert).toBeVisible();
    await expect(this.errorAlert).toContainText(message);
  }

  async expectLoginButtonDisabled(): Promise<void> {
    await expect(this.loginButton).toBeDisabled();
  }

  async expectLoginButtonEnabled(): Promise<void> {
    await expect(this.loginButton).toBeEnabled();
  }
}
```

### Dashboard Page

```typescript
// tests/e2e/pages/dashboard.page.ts
import { Page, Locator, expect } from "@playwright/test";
import { BasePage } from "./base.page";
import { HeaderComponent } from "./components/header.component";
import { SidebarComponent } from "./components/sidebar.component";

export class DashboardPage extends BasePage {
  // Components
  readonly header: HeaderComponent;
  readonly sidebar: SidebarComponent;

  // Selectors
  readonly welcomeMessage: Locator;
  readonly statsCards: Locator;
  readonly recentActivityList: Locator;
  readonly quickActionsMenu: Locator;

  constructor(page: Page) {
    super(page);

    this.header = new HeaderComponent(page);
    this.sidebar = new SidebarComponent(page);

    this.welcomeMessage = page.locator('[data-testid="welcome-message"]');
    this.statsCards = page.locator('[data-testid="stats-card"]');
    this.recentActivityList = page.locator('[data-testid="recent-activity"]');
    this.quickActionsMenu = page.locator('[data-testid="quick-actions"]');
  }

  // Navigation
  async goto(): Promise<void> {
    await this.page.goto("/dashboard");
    await this.waitForPageLoad();
    await this.waitForLoading();
  }

  // Actions
  async clickStatsCard(index: number): Promise<void> {
    await this.statsCards.nth(index).click();
  }

  async openQuickActions(): Promise<void> {
    await this.quickActionsMenu.click();
  }

  // Assertions
  async expectWelcomeMessage(name: string): Promise<void> {
    await expect(this.welcomeMessage).toContainText(name);
  }

  async expectStatsCardsCount(count: number): Promise<void> {
    await expect(this.statsCards).toHaveCount(count);
  }

  async expectRecentActivityVisible(): Promise<void> {
    await expect(this.recentActivityList).toBeVisible();
  }
}
```

### Component Page Objects

```typescript
// tests/e2e/pages/components/header.component.ts
import { Page, Locator, expect } from "@playwright/test";

export class HeaderComponent {
  readonly page: Page;
  readonly root: Locator;
  readonly logo: Locator;
  readonly searchInput: Locator;
  readonly notificationButton: Locator;
  readonly notificationBadge: Locator;
  readonly userMenu: Locator;
  readonly themeToggle: Locator;
  readonly languageSelector: Locator;

  constructor(page: Page) {
    this.page = page;
    this.root = page.locator('[data-testid="app-header"]');
    this.logo = this.root.locator('[data-testid="logo"]');
    this.searchInput = this.root.locator('[data-testid="search-input"]');
    this.notificationButton = this.root.locator(
      '[data-testid="notifications-button"]'
    );
    this.notificationBadge = this.notificationButton.locator(".v-badge__badge");
    this.userMenu = this.root.locator('[data-testid="user-menu"]');
    this.themeToggle = this.root.locator('[data-testid="theme-toggle"]');
    this.languageSelector = this.root.locator(
      '[data-testid="language-selector"]'
    );
  }

  // Actions
  async clickLogo(): Promise<void> {
    await this.logo.click();
  }

  async search(query: string): Promise<void> {
    await this.searchInput.fill(query);
    await this.page.keyboard.press("Enter");
  }

  async openNotifications(): Promise<void> {
    await this.notificationButton.click();
  }

  async openUserMenu(): Promise<void> {
    await this.userMenu.click();
  }

  async toggleTheme(): Promise<void> {
    await this.themeToggle.click();
  }

  async selectLanguage(code: string): Promise<void> {
    await this.languageSelector.click();
    await this.page.locator(`[data-testid="language-${code}"]`).click();
  }

  async logout(): Promise<void> {
    await this.openUserMenu();
    await this.page.locator('[data-testid="logout-button"]').click();
  }

  // Assertions
  async expectNotificationCount(count: number): Promise<void> {
    if (count === 0) {
      await expect(this.notificationBadge).not.toBeVisible();
    } else {
      await expect(this.notificationBadge).toContainText(String(count));
    }
  }

  async expectUserName(name: string): Promise<void> {
    await expect(this.userMenu).toContainText(name);
  }
}
```

```typescript
// tests/e2e/pages/components/sidebar.component.ts
import { Page, Locator, expect } from "@playwright/test";

export class SidebarComponent {
  readonly page: Page;
  readonly root: Locator;
  readonly menuItems: Locator;
  readonly collapseButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.root = page.locator('[data-testid="app-sidebar"]');
    this.menuItems = this.root.locator(".v-list-item");
    this.collapseButton = this.root.locator('[data-testid="collapse-button"]');
  }

  // Actions
  async navigateTo(itemText: string): Promise<void> {
    await this.menuItems.filter({ hasText: itemText }).click();
  }

  async toggleCollapse(): Promise<void> {
    await this.collapseButton.click();
  }

  // Assertions
  async expectActiveItem(itemText: string): Promise<void> {
    const activeItem = this.menuItems.filter({ hasText: itemText });
    await expect(activeItem).toHaveClass(/v-list-item--active/);
  }

  async expectCollapsed(): Promise<void> {
    await expect(this.root).toHaveClass(/collapsed/);
  }

  async expectExpanded(): Promise<void> {
    await expect(this.root).not.toHaveClass(/collapsed/);
  }
}
```

---

## Test Fixtures

### Authentication Fixture

```typescript
// tests/e2e/fixtures/auth.fixture.ts
import { test as base, Page } from "@playwright/test";
import { LoginPage } from "../pages/login.page";
import { DashboardPage } from "../pages/dashboard.page";

// Extend base test with fixtures
export const test = base.extend<{
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
  authenticatedPage: Page;
}>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },

  dashboardPage: async ({ page }, use) => {
    const dashboardPage = new DashboardPage(page);
    await use(dashboardPage);
  },

  authenticatedPage: async ({ browser }, use) => {
    // Create context with saved auth state
    const context = await browser.newContext({
      storageState: "./tests/e2e/.auth/user.json",
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  },
});

export { expect } from "@playwright/test";
```

### Test Data Fixture

```typescript
// tests/e2e/fixtures/test-data.fixture.ts
import { test as base } from "@playwright/test";
import { faker } from "@faker-js/faker";

interface TestUser {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
}

interface TestProduct {
  name: string;
  price: number;
  description: string;
}

export const test = base.extend<{
  testUser: TestUser;
  testProduct: TestProduct;
}>({
  testUser: async ({}, use) => {
    const user: TestUser = {
      email: faker.internet.email(),
      password: faker.internet.password({ length: 12 }),
      firstName: faker.person.firstName(),
      lastName: faker.person.lastName(),
    };
    await use(user);
  },

  testProduct: async ({}, use) => {
    const product: TestProduct = {
      name: faker.commerce.productName(),
      price: parseFloat(faker.commerce.price()),
      description: faker.commerce.productDescription(),
    };
    await use(product);
  },
});

export { expect } from "@playwright/test";
```

### Combined Fixtures

```typescript
// tests/e2e/fixtures/index.ts
import { mergeTests } from "@playwright/test";
import { test as authTest } from "./auth.fixture";
import { test as dataTest } from "./test-data.fixture";

export const test = mergeTests(authTest, dataTest);
export { expect } from "@playwright/test";
```

---

## Custom Assertions

```typescript
// tests/e2e/helpers/assertions.ts
import { expect, Locator, Page } from "@playwright/test";

// Extend Playwright's expect
export const customExpect = {
  // Assert element has Vuetify error state
  async toHaveVuetifyError(locator: Locator, message?: string): Promise<void> {
    const input = locator.locator("..");
    await expect(input).toHaveClass(/v-input--error/);

    if (message) {
      const errorMessage = input.locator(".v-messages__message");
      await expect(errorMessage).toContainText(message);
    }
  },

  // Assert Vuetify snackbar shows message
  async toShowSnackbar(
    page: Page,
    message: string,
    type?: "success" | "error" | "warning" | "info"
  ): Promise<void> {
    const snackbar = page.locator(".v-snackbar");
    await expect(snackbar).toBeVisible();
    await expect(snackbar).toContainText(message);

    if (type) {
      await expect(snackbar).toHaveClass(new RegExp(`bg-${type}`));
    }
  },

  // Assert Vuetify dialog is open
  async toHaveOpenDialog(page: Page, title?: string): Promise<void> {
    const dialog = page.locator(".v-dialog");
    await expect(dialog).toBeVisible();

    if (title) {
      await expect(dialog.locator(".v-card-title")).toContainText(title);
    }
  },

  // Assert table has rows
  async toHaveTableRows(locator: Locator, count: number): Promise<void> {
    const rows = locator.locator("tbody tr");
    await expect(rows).toHaveCount(count);
  },

  // Assert element is loading
  async toBeLoading(locator: Locator): Promise<void> {
    await expect(locator.locator(".v-progress-circular")).toBeVisible();
  },

  // Assert element finished loading
  async toFinishLoading(locator: Locator): Promise<void> {
    await expect(locator.locator(".v-progress-circular")).not.toBeVisible();
  },

  // Assert form is valid
  async toBeValidForm(page: Page): Promise<void> {
    const errors = page.locator(".v-input--error");
    await expect(errors).toHaveCount(0);
  },

  // Assert URL contains path
  async toHaveUrlPath(page: Page, path: string): Promise<void> {
    await expect(page).toHaveURL(new RegExp(path));
  },

  // Assert page has no accessibility violations
  async toHaveNoA11yViolations(page: Page): Promise<void> {
    const AxeBuilder = (await import("@axe-core/playwright")).default;
    const results = await new AxeBuilder({ page }).analyze();
    expect(results.violations).toEqual([]);
  },
};
```

---

## Test Helpers

### Common Actions

```typescript
// tests/e2e/helpers/actions.ts
import { Page, Locator } from "@playwright/test";

// Wait for network idle
export async function waitForNetworkIdle(page: Page): Promise<void> {
  await page.waitForLoadState("networkidle");
}

// Fill Vuetify text field
export async function fillVuetifyTextField(
  locator: Locator,
  value: string
): Promise<void> {
  await locator.click();
  await locator.fill(value);
  await locator.press("Tab");
}

// Select Vuetify dropdown option
export async function selectVuetifyOption(
  page: Page,
  selectLocator: Locator,
  optionText: string
): Promise<void> {
  await selectLocator.click();
  await page.locator(".v-list-item").filter({ hasText: optionText }).click();
}

// Toggle Vuetify checkbox
export async function toggleVuetifyCheckbox(locator: Locator): Promise<void> {
  await locator.locator(".v-selection-control__input").click();
}

// Click Vuetify button and wait
export async function clickAndWait(
  page: Page,
  buttonLocator: Locator
): Promise<void> {
  await buttonLocator.click();
  await waitForNetworkIdle(page);
}

// Upload file
export async function uploadFile(
  page: Page,
  inputLocator: Locator,
  filePath: string
): Promise<void> {
  await inputLocator.setInputFiles(filePath);
}

// Scroll to element
export async function scrollToElement(locator: Locator): Promise<void> {
  await locator.scrollIntoViewIfNeeded();
}

// Wait for toast/snackbar to disappear
export async function waitForSnackbarToDisappear(page: Page): Promise<void> {
  await page
    .locator(".v-snackbar")
    .waitFor({ state: "hidden", timeout: 10000 });
}

// Confirm dialog
export async function confirmDialog(page: Page): Promise<void> {
  await page.locator('.v-dialog [data-testid="confirm-button"]').click();
  await page.locator(".v-dialog").waitFor({ state: "hidden" });
}

// Cancel dialog
export async function cancelDialog(page: Page): Promise<void> {
  await page.locator('.v-dialog [data-testid="cancel-button"]').click();
  await page.locator(".v-dialog").waitFor({ state: "hidden" });
}
```

### Selector Utilities

```typescript
// tests/e2e/helpers/selectors.ts

// Get data-testid selector
export function testId(id: string): string {
  return `[data-testid="${id}"]`;
}

// Get Vuetify component selectors
export const vuetifySelectors = {
  // Form inputs
  textField: (name: string) => `[data-testid="${name}-input"] input`,
  select: (name: string) => `[data-testid="${name}-select"]`,
  checkbox: (name: string) => `[data-testid="${name}-checkbox"]`,
  radio: (name: string) => `[data-testid="${name}-radio"]`,

  // Buttons
  button: (name: string) => `[data-testid="${name}-button"]`,
  iconButton: (name: string) => `[data-testid="${name}-icon-button"]`,

  // Navigation
  navItem: (name: string) => `.v-list-item[data-testid="${name}-nav"]`,
  tab: (name: string) => `.v-tab[data-testid="${name}-tab"]`,

  // Data display
  card: (name: string) => `.v-card[data-testid="${name}-card"]`,
  table: (name: string) => `.v-data-table[data-testid="${name}-table"]`,
  tableRow: (index: number) => `tbody tr:nth-child(${index + 1})`,

  // Feedback
  alert: (type: string) => `.v-alert[type="${type}"]`,
  snackbar: ".v-snackbar",
  dialog: ".v-dialog",

  // Loading
  progressCircular: ".v-progress-circular",
  progressLinear: ".v-progress-linear",
  skeleton: ".v-skeleton-loader",
};
```

---

## Test Specifications

### Login Tests

```typescript
// tests/e2e/specs/auth/login.spec.ts
import { test, expect } from "../../fixtures";
import { LoginPage } from "../../pages/login.page";

test.describe("Login Page", () => {
  let loginPage: LoginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.goto();
  });

  test("should display login form", async () => {
    // Assert
    await expect(loginPage.emailInput).toBeVisible();
    await expect(loginPage.passwordInput).toBeVisible();
    await expect(loginPage.loginButton).toBeVisible();
  });

  test("should show validation errors for empty fields", async () => {
    // Act
    await loginPage.clickLogin();

    // Assert
    await loginPage.expectEmailError("required");
    await loginPage.expectPasswordError("required");
  });

  test("should show error for invalid email format", async () => {
    // Act
    await loginPage.fillEmail("invalid-email");
    await loginPage.fillPassword("password123");
    await loginPage.clickLogin();

    // Assert
    await loginPage.expectEmailError("valid email");
  });

  test("should show error for invalid credentials", async () => {
    // Act
    await loginPage.login("wrong@example.com", "wrongpassword");

    // Assert
    await loginPage.expectLoginError("Invalid credentials");
  });

  test("should login successfully with valid credentials", async ({ page }) => {
    // Act
    await loginPage.loginAndWaitForDashboard("test@example.com", "password123");

    // Assert
    await expect(page).toHaveURL(/dashboard/);
  });

  test("should navigate to register page", async ({ page }) => {
    // Act
    await loginPage.clickRegister();

    // Assert
    await expect(page).toHaveURL(/register/);
  });

  test("should navigate to forgot password page", async ({ page }) => {
    // Act
    await loginPage.clickForgotPassword();

    // Assert
    await expect(page).toHaveURL(/forgot-password/);
  });

  test("should persist login with remember me", async ({ page, context }) => {
    // Act
    await loginPage.toggleRememberMe();
    await loginPage.loginAndWaitForDashboard("test@example.com", "password123");

    // Get cookies
    const cookies = await context.cookies();
    const authCookie = cookies.find((c) => c.name === "auth_token");

    // Assert
    expect(authCookie).toBeDefined();
    expect(authCookie?.expires).toBeGreaterThan(Date.now() / 1000 + 86400); // > 1 day
  });
});
```

### Dashboard Tests

```typescript
// tests/e2e/specs/dashboard/dashboard.spec.ts
import { test, expect } from "../../fixtures";
import { DashboardPage } from "../../pages/dashboard.page";

test.describe("Dashboard", () => {
  test.use({ storageState: "./tests/e2e/.auth/user.json" });

  let dashboardPage: DashboardPage;

  test.beforeEach(async ({ page }) => {
    dashboardPage = new DashboardPage(page);
    await dashboardPage.goto();
  });

  test("should display welcome message", async () => {
    // Assert
    await dashboardPage.expectWelcomeMessage("John");
  });

  test("should display stats cards", async () => {
    // Assert
    await dashboardPage.expectStatsCardsCount(4);
  });

  test("should display recent activity", async () => {
    // Assert
    await dashboardPage.expectRecentActivityVisible();
  });

  test("should navigate via sidebar", async ({ page }) => {
    // Act
    await dashboardPage.sidebar.navigateTo("Profile");

    // Assert
    await expect(page).toHaveURL(/profile/);
  });

  test("should toggle theme", async ({ page }) => {
    // Get initial theme
    const initialTheme = await page.evaluate(() =>
      document.documentElement.classList.contains("v-theme--dark")
    );

    // Act
    await dashboardPage.header.toggleTheme();

    // Assert
    const newTheme = await page.evaluate(() =>
      document.documentElement.classList.contains("v-theme--dark")
    );
    expect(newTheme).not.toBe(initialTheme);
  });

  test("should logout successfully", async ({ page }) => {
    // Act
    await dashboardPage.header.logout();

    // Assert
    await expect(page).toHaveURL(/login/);
  });

  test("should be responsive on mobile", async ({ page }) => {
    // Resize to mobile
    await page.setViewportSize({ width: 375, height: 667 });

    // Assert sidebar is hidden
    await expect(dashboardPage.sidebar.root).not.toBeVisible();

    // Open mobile menu
    await page.locator('[data-testid="mobile-menu-button"]').click();

    // Assert sidebar is visible
    await expect(dashboardPage.sidebar.root).toBeVisible();
  });
});
```

### Accessibility Tests

```typescript
// tests/e2e/specs/accessibility.spec.ts
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

test.describe("Accessibility", () => {
  test("login page should have no accessibility violations", async ({
    page,
  }) => {
    // Arrange
    await page.goto("/login");

    // Act
    const results = await new AxeBuilder({ page })
      .withTags(["wcag2a", "wcag2aa", "wcag21a", "wcag21aa"])
      .analyze();

    // Assert
    expect(results.violations).toEqual([]);
  });

  test("dashboard should have no accessibility violations", async ({
    page,
  }) => {
    // Arrange
    await page.goto("/dashboard");

    // Act
    const results = await new AxeBuilder({ page })
      .exclude(".v-progress-circular") // Exclude loading spinners
      .analyze();

    // Assert
    expect(results.violations).toEqual([]);
  });

  test("should be keyboard navigable", async ({ page }) => {
    // Arrange
    await page.goto("/login");

    // Act - Tab through form
    await page.keyboard.press("Tab");
    await expect(
      page.locator('[data-testid="email-input"] input')
    ).toBeFocused();

    await page.keyboard.press("Tab");
    await expect(
      page.locator('[data-testid="password-input"] input')
    ).toBeFocused();

    await page.keyboard.press("Tab");
    await expect(page.locator('[data-testid="remember-me"]')).toBeFocused();

    await page.keyboard.press("Tab");
    await expect(page.locator('[data-testid="login-button"]')).toBeFocused();
  });
});
```

### Visual Regression Tests

```typescript
// tests/e2e/specs/visual/visual.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Visual Regression", () => {
  test("login page matches snapshot", async ({ page }) => {
    // Arrange
    await page.goto("/login");
    await page.waitForLoadState("networkidle");

    // Assert
    await expect(page).toHaveScreenshot("login-page.png", {
      fullPage: true,
      animations: "disabled",
    });
  });

  test("dashboard matches snapshot", async ({ page }) => {
    // Arrange
    await page.goto("/dashboard");
    await page.waitForLoadState("networkidle");

    // Wait for all images to load
    await page.waitForFunction(() => {
      const images = document.querySelectorAll("img");
      return Array.from(images).every((img) => img.complete);
    });

    // Assert
    await expect(page).toHaveScreenshot("dashboard-page.png", {
      fullPage: true,
      animations: "disabled",
      mask: [page.locator('[data-testid="dynamic-content"]')],
    });
  });

  test("dark theme matches snapshot", async ({ page }) => {
    // Arrange
    await page.goto("/login");
    await page.emulateMedia({ colorScheme: "dark" });
    await page.waitForLoadState("networkidle");

    // Assert
    await expect(page).toHaveScreenshot("login-page-dark.png", {
      fullPage: true,
      animations: "disabled",
    });
  });
});
```

---

## Package Scripts

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:headed": "playwright test --headed",
    "test:e2e:debug": "playwright test --debug",
    "test:e2e:chromium": "playwright test --project=chromium",
    "test:e2e:firefox": "playwright test --project=firefox",
    "test:e2e:webkit": "playwright test --project=webkit",
    "test:e2e:mobile": "playwright test --project=mobile-chrome --project=mobile-safari",
    "test:e2e:report": "playwright show-report",
    "test:e2e:update-snapshots": "playwright test --update-snapshots"
  }
}
```

---

## Best Practices

### 1. Use Page Object Model

```typescript
// ✅ Good: Use page objects
const loginPage = new LoginPage(page);
await loginPage.login("test@example.com", "password");

// ❌ Bad: Direct selectors in tests
await page.fill('[data-testid="email"]', "test@example.com");
await page.fill('[data-testid="password"]', "password");
await page.click('[data-testid="submit"]');
```

### 2. Use Data-TestId Attributes

```vue
<!-- ✅ Good: Stable test selectors -->
<v-btn data-testid="submit-button">Submit</v-btn>

<!-- ❌ Bad: Fragile selectors -->
<v-btn class="submit-btn">Submit</v-btn>
```

### 3. Wait for Stability

```typescript
// ✅ Good: Wait for specific conditions
await page.waitForLoadState("networkidle");
await expect(element).toBeVisible();

// ❌ Bad: Arbitrary waits
await page.waitForTimeout(2000);
```

### 4. Isolate Tests

```typescript
// ✅ Good: Each test is independent
test.beforeEach(async ({ page }) => {
  await page.goto("/login");
});

// ❌ Bad: Tests depend on each other
```

### 5. Use Fixtures for Common Setup

```typescript
// ✅ Good: Use fixtures
test("authenticated user can access dashboard", async ({
  authenticatedPage,
}) => {
  await authenticatedPage.goto("/dashboard");
});
```

---

## Next Steps

Continue to [Code Quality](./13-code-quality.md) to learn about ESLint configuration, TypeScript settings, and git hooks.
