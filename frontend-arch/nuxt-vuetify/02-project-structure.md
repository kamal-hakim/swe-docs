# Project Structure

## Overview

A well-organized project structure is fundamental to maintainability and scalability. This document defines the standard folder organization for Nuxt applications with Vuetify 3, following domain-driven design principles while leveraging Nuxt's conventions and incorporating a three-tier testing architecture.

## Root Directory Structure

```
project-root/
├── .github/                    # GitHub workflows and templates
├── .nuxt/                      # Nuxt build output (generated, gitignored)
├── .output/                    # Production build output (generated, gitignored)
├── .vscode/                    # VS Code workspace settings
├── app/                        # Main application source code
├── config/                     # Application configuration files
├── locales/                    # Internationalization files
├── mocks/                      # Mock data and implementations
├── modules/                    # Custom Nuxt modules
├── public/                     # Static assets (served at root)
├── scripts/                    # Build and utility scripts
├── server/                     # Server-side code and API routes
├── shared/                     # Shared types and utilities
├── tests/                      # Test files and utilities (three-tier)
│   ├── unit/                  # Tier 1: Unit tests
│   ├── integration/           # Tier 2: Integration tests
│   ├── e2e/                   # Tier 3: End-to-end tests
│   ├── fixtures/              # Test fixtures and factories
│   └── utils/                 # Test utilities and helpers
├── .env                        # Environment variables (gitignored)
├── .env.example                # Environment variables template
├── .gitignore                  # Git ignore patterns
├── eslint.config.js            # ESLint configuration
├── nuxt.config.ts              # Nuxt configuration
├── package.json                # Project dependencies and scripts
├── playwright.config.ts        # Playwright E2E configuration
├── pnpm-lock.yaml              # pnpm lockfile
├── tsconfig.json               # TypeScript configuration
├── vitest.config.ts            # Vitest configuration
└── vuetify.config.ts           # Vuetify configuration (optional)
```

---

## App Directory Structure

The `app/` directory contains all client-side application code. This is where the majority of your development work happens.

```
app/
├── assets/                     # Processed assets (images, fonts, etc.)
│   ├── fonts/                 # Custom fonts
│   ├── images/                # Images (processed by build)
│   └── styles/                # Global SCSS/CSS files
│       ├── _variables.scss    # SCSS variables (Vuetify overrides)
│       ├── _mixins.scss       # SCSS mixins
│       ├── main.scss          # Main stylesheet
│       └── vuetify/           # Vuetify customizations
│           └── settings.scss  # Vuetify SASS variables
├── components/                 # Vue components
│   ├── common/                # Reusable generic components
│   ├── layout/                # Layout-related components
│   ├── [domain]/              # Domain-specific components
│   └── vuetify/               # Custom Vuetify wrapper components
├── composables/                # Vue composables
│   ├── api/                   # API integration composables
│   ├── utils/                 # Utility composables
│   └── [domain]/              # Domain-specific composables
├── constants/                  # Application constants
├── layouts/                    # Nuxt layouts
├── middleware/                 # Route middleware
├── pages/                      # File-based routing pages
├── plugins/                    # Nuxt plugins
│   └── vuetify.ts             # Vuetify plugin configuration
├── stores/                     # Pinia stores
├── types/                      # TypeScript type definitions
├── utils/                      # Utility functions
├── app.config.ts              # App configuration (runtime)
├── app.vue                     # Root Vue component
└── error.vue                   # Error page component
```

---

## Directory Explanations

### Components Directory

Components are organized using a **domain-driven** approach, with a special `vuetify/` directory for custom Vuetify component wrappers.

```
app/components/
├── common/                     # Shared, reusable components
│   ├── AppButton.vue          # Custom button (wraps v-btn)
│   ├── AppInput.vue           # Custom input (wraps v-text-field)
│   ├── AppModal.vue           # Modal dialog (wraps v-dialog)
│   ├── AppToast.vue           # Toast notification (wraps v-snackbar)
│   ├── AppPaginator.vue       # Pagination (wraps v-pagination)
│   └── AppErrorBoundary.vue   # Error boundary wrapper
├── layout/                     # Layout components
│   ├── LayoutHeader.vue       # Application header (v-app-bar)
│   ├── LayoutSidebar.vue      # Sidebar navigation (v-navigation-drawer)
│   ├── LayoutFooter.vue       # Application footer (v-footer)
│   └── LayoutBreadcrumb.vue   # Breadcrumb navigation (v-breadcrumbs)
├── user/                       # User domain components
│   ├── UserAvatar.vue         # User avatar display (v-avatar)
│   ├── UserCard.vue           # User information card (v-card)
│   ├── UserProfile.vue        # User profile view
│   ├── UserSettings.vue       # User settings form
│   └── UserBadge.vue          # User role/status badge (v-chip)
├── product/                    # Product domain components
│   ├── ProductCard.vue        # Product card display
│   ├── ProductList.vue        # Product listing (v-list)
│   ├── ProductDetail.vue      # Product detail view
│   ├── ProductFilter.vue      # Product filtering
│   └── ProductSearch.vue      # Product search
├── order/                      # Order domain components
│   ├── OrderSummary.vue       # Order summary
│   ├── OrderItem.vue          # Single order item
│   ├── OrderHistory.vue       # Order history list
│   └── OrderStatus.vue        # Order status indicator
└── vuetify/                    # Custom Vuetify wrappers
    ├── VTextField.vue         # Extended text field
    ├── VSelect.vue            # Extended select
    ├── VDataTable.vue         # Extended data table
    ├── VCard.vue              # Extended card
    └── VDialog.vue            # Extended dialog
```

#### Component Naming Convention

| Category         | Prefix         | Example            |
| ---------------- | -------------- | ------------------ |
| Common/Shared    | `App`          | `AppButton.vue`    |
| Layout           | `Layout`       | `LayoutHeader.vue` |
| Vuetify Wrappers | `V` (extended) | `VTextField.vue`   |
| Domain-specific  | Domain name    | `UserCard.vue`     |

### Composables Directory

Composables contain reusable logic following Vue 3's Composition API patterns.

```
app/composables/
├── api/                        # API-related composables
│   ├── useApi.ts              # Base API client with error handling
│   ├── useFetch.ts            # Data fetching wrapper
│   └── useInfiniteScroll.ts   # Infinite scroll pagination
├── auth/                       # Authentication composables
│   ├── useAuth.ts             # Authentication state
│   ├── useSession.ts          # Session management
│   └── usePermissions.ts      # Permission checking
├── utils/                      # Utility composables
│   ├── useDebounce.ts         # Debounce functionality
│   ├── useLocalStorage.ts     # Local storage wrapper
│   ├── useClipboard.ts        # Clipboard operations
│   └── useMediaQuery.ts       # Responsive breakpoints
├── ui/                         # UI-related composables
│   ├── useModal.ts            # Modal state management
│   ├── useToast.ts            # Toast notifications (v-snackbar)
│   ├── useDialog.ts           # Dialog confirmations
│   └── useTheme.ts            # Vuetify theme management
├── form/                       # Form handling composables
│   ├── useForm.ts             # Form state management
│   ├── useValidation.ts       # Form validation (VeeValidate)
│   └── useFormField.ts        # Individual field logic
└── [domain]/                   # Domain-specific composables
    ├── useUser.ts             # User operations
    ├── useProduct.ts          # Product operations
    └── useOrder.ts            # Order operations
```

### Pages Directory

Pages follow Nuxt's file-based routing convention.

```
app/pages/
├── index.vue                   # Home page (/)
├── login.vue                   # Login page (/login)
├── register.vue                # Registration page (/register)
├── about.vue                   # About page (/about)
├── dashboard/                  # Dashboard routes
│   ├── index.vue              # Dashboard home (/dashboard)
│   ├── analytics.vue          # Analytics page (/dashboard/analytics)
│   └── settings.vue           # Settings page (/dashboard/settings)
├── users/                      # User routes
│   ├── index.vue              # User list (/users)
│   └── [id].vue               # User detail (/users/:id)
├── products/                   # Product routes
│   ├── index.vue              # Product list (/products)
│   ├── [id].vue               # Product detail (/products/:id)
│   └── [id]/
│       └── edit.vue           # Product edit (/products/:id/edit)
├── orders/                     # Order routes
│   ├── index.vue              # Order list (/orders)
│   └── [id].vue               # Order detail (/orders/:id)
├── settings/                   # Settings routes
│   ├── index.vue              # Settings home (/settings)
│   ├── profile.vue            # Profile settings (/settings/profile)
│   ├── notifications.vue      # Notification settings (/settings/notifications)
│   └── security.vue           # Security settings (/settings/security)
└── [...slug].vue               # Catch-all 404 page
```

### Stores Directory

Pinia stores organized by domain.

```
app/stores/
├── auth.ts                     # Authentication store
├── user.ts                     # User store
├── app.ts                      # Application state store
├── notification.ts             # Notifications store
├── theme.ts                    # Theme preferences store
└── [domain].ts                 # Domain-specific stores
```

### Layouts Directory

Application layouts for different page structures, utilizing Vuetify layout components.

```
app/layouts/
├── default.vue                 # Default layout (v-app with navigation)
├── auth.vue                    # Auth pages layout (minimal)
├── dashboard.vue               # Dashboard layout (with v-navigation-drawer)
├── blank.vue                   # Blank layout (no chrome)
└── error.vue                   # Error page layout
```

### Middleware Directory

Route middleware for navigation guards.

```
app/middleware/
├── auth.ts                     # Authentication check
├── guest.ts                    # Guest-only routes
├── admin.ts                    # Admin permission check
└── setup.ts                    # Initial app setup
```

### Plugins Directory

Nuxt plugins for third-party integrations and global setup.

```
app/plugins/
├── vuetify.ts                  # Vuetify initialization
├── api.ts                      # API client initialization
├── auth.client.ts              # Client-side auth setup
├── analytics.client.ts         # Analytics integration
├── error-handler.ts            # Global error handling
├── i18n.ts                     # i18n configuration
└── vee-validate.ts             # VeeValidate setup
```

---

## Tests Directory Structure (Three-Tier Architecture)

```
tests/
├── setup.ts                    # Global test setup
├── utils/                      # Shared test utilities
│   ├── index.ts               # Utility exports
│   ├── mocks.ts               # Common mocks
│   ├── render.ts              # Custom render functions
│   └── vuetify.ts             # Vuetify test setup
├── fixtures/                   # Test fixtures and factories
│   ├── index.ts               # Fixture exports
│   ├── users.ts               # User fixtures
│   ├── products.ts            # Product fixtures
│   └── factories/             # Factory functions
│       ├── userFactory.ts     # User factory
│       ├── productFactory.ts  # Product factory
│       └── index.ts           # Factory exports
├── unit/                       # Tier 1: Unit tests
│   ├── utils/                 # Utility function tests
│   │   ├── formatDate.test.ts
│   │   └── validation.test.ts
│   ├── composables/           # Composable tests
│   │   ├── useAuth.test.ts
│   │   ├── useForm.test.ts
│   │   └── useApi.test.ts
│   └── stores/                # Store unit tests
│       ├── auth.test.ts
│       └── user.test.ts
├── integration/                # Tier 2: Integration tests
│   ├── components/            # Component tests
│   │   ├── common/
│   │   │   ├── AppButton.test.ts
│   │   │   └── AppModal.test.ts
│   │   ├── user/
│   │   │   └── UserCard.test.ts
│   │   └── vuetify/
│   │       └── VTextField.test.ts
│   ├── pages/                 # Page tests
│   │   ├── home.test.ts
│   │   └── login.test.ts
│   └── api/                   # API integration tests
│       └── users.test.ts
└── e2e/                        # Tier 3: End-to-end tests
    ├── pages/                 # Page object models
    │   ├── BasePage.ts        # Base page object
    │   ├── LoginPage.ts       # Login page object
    │   ├── DashboardPage.ts   # Dashboard page object
    │   └── index.ts           # Page exports
    ├── fixtures/              # E2E fixtures
    │   ├── auth.ts            # Authentication fixtures
    │   └── data.ts            # Test data
    ├── helpers/               # E2E helpers
    │   ├── assertions.ts      # Custom assertions
    │   ├── actions.ts         # Common actions
    │   └── index.ts           # Helper exports
    ├── specs/                  # Test specifications
    │   ├── auth/
    │   │   ├── login.spec.ts
    │   │   └── logout.spec.ts
    │   ├── user/
    │   │   ├── profile.spec.ts
    │   │   └── settings.spec.ts
    │   └── smoke.spec.ts      # Smoke tests
    └── global-setup.ts        # Playwright global setup
```

---

## Server Directory Structure

The `server/` directory contains all server-side code.

```
server/
├── api/                        # API routes
│   ├── auth/                  # Authentication endpoints
│   │   ├── login.post.ts     # POST /api/auth/login
│   │   ├── logout.post.ts    # POST /api/auth/logout
│   │   └── me.get.ts         # GET /api/auth/me
│   ├── users/                 # User endpoints
│   │   ├── index.get.ts      # GET /api/users
│   │   └── [id].get.ts       # GET /api/users/:id
│   └── [...].ts               # Catch-all API route
├── middleware/                 # Server middleware
│   ├── auth.ts                # Authentication middleware
│   ├── cors.ts                # CORS configuration
│   └── rate-limit.ts          # Rate limiting
├── plugins/                    # Server plugins
│   └── database.ts            # Database connection
├── utils/                      # Server utilities
│   ├── jwt.ts                 # JWT utilities
│   └── validation.ts          # Input validation
└── routes/                     # Server routes (non-API)
    └── health.get.ts          # Health check endpoint
```

---

## Locales Directory Structure

```
locales/
├── en/                         # English translations
│   ├── common.json            # Common translations
│   ├── auth.json              # Authentication translations
│   ├── validation.json        # Validation messages
│   ├── components/            # Per-component translations
│   │   ├── user.json
│   │   └── product.json
│   └── pages/                 # Per-page translations
│       ├── home.json
│       └── dashboard.json
├── es/                         # Spanish translations
│   └── ...                    # Same structure as en/
├── fr/                         # French translations
│   └── ...                    # Same structure as en/
└── index.ts                    # Locale configuration
```

---

## Configuration Files

### config/ Directory

```
config/
├── app.ts                      # Application settings
├── api.ts                      # API configuration
├── i18n.ts                     # Internationalization config
├── vuetify.ts                  # Vuetify theme configuration
├── pwa.ts                      # PWA configuration
└── security.ts                 # Security settings
```

### Root Configuration Files

| File                   | Purpose                        |
| ---------------------- | ------------------------------ |
| `nuxt.config.ts`       | Main Nuxt configuration        |
| `tsconfig.json`        | TypeScript configuration       |
| `eslint.config.js`     | ESLint rules and settings      |
| `vitest.config.ts`     | Vitest configuration           |
| `playwright.config.ts` | Playwright E2E configuration   |
| `.env.example`         | Environment variables template |

---

## Shared Directory

Code shared between client and server.

```
shared/
├── types/                      # Shared TypeScript types
│   ├── api.ts                 # API request/response types
│   ├── models.ts              # Domain model types
│   └── utils.ts               # Utility types
├── constants/                  # Shared constants
│   ├── errors.ts              # Error codes
│   └── config.ts              # Configuration constants
└── utils/                      # Shared utility functions
    ├── validation.ts          # Validation functions
    └── formatting.ts          # Formatting utilities
```

---

## Naming Conventions Summary

### Files and Directories

| Type              | Convention                 | Example              |
| ----------------- | -------------------------- | -------------------- |
| Directories       | kebab-case                 | `user-profile/`      |
| Vue Components    | PascalCase                 | `UserProfile.vue`    |
| Composables       | camelCase, "use" prefix    | `useUser.ts`         |
| Utilities         | camelCase                  | `formatDate.ts`      |
| Types             | PascalCase                 | `UserTypes.ts`       |
| Constants         | camelCase or UPPER_SNAKE   | `apiEndpoints.ts`    |
| Unit Tests        | Same as source + `.test`   | `useUser.test.ts`    |
| Integration Tests | Same as source + `.test`   | `UserCard.test.ts`   |
| E2E Tests         | kebab-case + `.spec`       | `user-login.spec.ts` |
| Page Objects      | PascalCase + `Page`        | `LoginPage.ts`       |
| Fixtures          | camelCase                  | `userFixtures.ts`    |
| Factories         | camelCase + `Factory`      | `userFactory.ts`     |
| API Routes        | kebab-case + method suffix | `users.get.ts`       |

### Component Naming by Category

```typescript
// Common/Generic components - "App" prefix
AppButton.vue;
AppModal.vue;
AppInput.vue;

// Layout components - "Layout" prefix
LayoutHeader.vue;
LayoutSidebar.vue;

// Vuetify Wrappers - "V" prefix (extended)
VTextField.vue;
VDataTable.vue;

// Domain components - No prefix, domain directory
user / UserCard.vue;
product / ProductList.vue;

// Page components - No prefix, describes view
user / UserProfile.vue;
product / ProductDetail.vue;
```

---

## Auto-Imports

Nuxt automatically imports certain files. Understand what is auto-imported:

### Auto-Imported by Default

| Directory      | Import Type                     |
| -------------- | ------------------------------- |
| `components/`  | Vue components                  |
| `composables/` | Composable functions            |
| `utils/`       | Utility functions               |
| `stores/`      | Pinia stores (with @pinia/nuxt) |

### Configure Additional Auto-Imports

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  imports: {
    dirs: ["composables/**", "utils/**", "stores/**"],
  },
});
```

---

## Best Practices

### 1. Keep Components Focused

Each component should have a single responsibility:

```
✅ Good: UserAvatar.vue (displays user avatar)
✅ Good: UserCard.vue (displays user summary)
❌ Bad: UserEverything.vue (does multiple things)
```

### 2. Colocate Related Files

Keep related files close together:

```
user/
├── UserCard.vue
├── UserCard.test.ts        # Test next to component
├── useUserCard.ts          # Related composable
└── types.ts                # Component-specific types
```

### 3. Avoid Deep Nesting

Limit directory depth to 3-4 levels:

```
✅ Good: components/user/UserCard.vue
❌ Bad: components/users/profile/cards/avatar/small/UserSmallAvatarCard.vue
```

### 4. Group by Feature, Not Type

```
✅ Good: components/user/ (domain-driven)
❌ Bad: components/cards/, components/buttons/ (type-driven)
```

### 5. Separate Test Tiers Clearly

```
tests/
├── unit/           # Fast, isolated tests
├── integration/    # Component and API tests
└── e2e/            # Full user journey tests
```

### 6. Use Index Files Sparingly

Only use `index.ts` when re-exporting:

```typescript
// composables/user/index.ts
export { useUser } from "./useUser";
export { useUserProfile } from "./useUserProfile";
export { useUserSettings } from "./useUserSettings";
```

---

## Next Steps

Continue to [Nuxt Configuration](./03-nuxt-configuration.md) for comprehensive configuration setup covering Vuetify 3, both SPA and SSR modes.
