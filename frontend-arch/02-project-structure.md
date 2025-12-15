# Project Structure

## Overview

A well-organized project structure is fundamental to maintainability and scalability. This document defines the standard folder organization for Nuxt applications, following domain-driven design principles while leveraging Nuxt's conventions.

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
├── tests/                      # Test files and utilities
├── .env                        # Environment variables (gitignored)
├── .env.example                # Environment variables template
├── .gitignore                  # Git ignore patterns
├── eslint.config.js            # ESLint configuration
├── nuxt.config.ts              # Nuxt configuration
├── package.json                # Project dependencies and scripts
├── pnpm-lock.yaml              # pnpm lockfile
├── tsconfig.json               # TypeScript configuration
├── unocss.config.ts            # UnoCSS configuration
└── vitest.config.ts            # Vitest configuration
```

---

## App Directory Structure

The `app/` directory contains all client-side application code. This is where the majority of your development work happens.

```
app/
├── assets/                     # Processed assets (images, fonts, etc.)
│   ├── fonts/                 # Custom fonts
│   ├── images/                # Images (processed by build)
│   └── icons/                 # SVG icons
├── components/                 # Vue components
│   ├── common/                # Reusable generic components
│   ├── layout/                # Layout-related components
│   ├── [domain]/              # Domain-specific components
│   └── ui/                    # Base UI components
├── composables/                # Vue composables
│   ├── api/                   # API integration composables
│   ├── utils/                 # Utility composables
│   └── [domain]/              # Domain-specific composables
├── constants/                  # Application constants
├── layouts/                    # Nuxt layouts
├── middleware/                 # Route middleware
├── pages/                      # File-based routing pages
├── plugins/                    # Nuxt plugins
├── stores/                     # Pinia stores
├── styles/                     # Global styles
│   ├── base.css               # Base/reset styles
│   ├── variables.css          # CSS custom properties
│   └── utilities.css          # Utility classes
├── types/                      # TypeScript type definitions
├── utils/                      # Utility functions
├── app.config.ts              # App configuration (runtime)
├── app.vue                     # Root Vue component
└── error.vue                   # Error page component
```

---

## Directory Explanations

### Components Directory

Components are organized using a **domain-driven** approach, where related components are grouped by their business function.

```
app/components/
├── common/                     # Shared, reusable components
│   ├── AppButton.vue          # Generic button component
│   ├── AppInput.vue           # Generic input component
│   ├── AppModal.vue           # Modal dialog component
│   ├── AppToast.vue           # Toast notification
│   ├── AppPaginator.vue       # Pagination component
│   └── AppErrorBoundary.vue   # Error boundary wrapper
├── layout/                     # Layout components
│   ├── LayoutHeader.vue       # Application header
│   ├── LayoutSidebar.vue      # Sidebar navigation
│   ├── LayoutFooter.vue       # Application footer
│   └── LayoutBreadcrumb.vue   # Breadcrumb navigation
├── user/                       # User domain components
│   ├── UserAvatar.vue         # User avatar display
│   ├── UserCard.vue           # User information card
│   ├── UserProfile.vue        # User profile view
│   ├── UserSettings.vue       # User settings form
│   └── UserBadge.vue          # User role/status badge
├── product/                    # Product domain components
│   ├── ProductCard.vue        # Product card display
│   ├── ProductList.vue        # Product listing
│   ├── ProductDetail.vue      # Product detail view
│   ├── ProductFilter.vue      # Product filtering
│   └── ProductSearch.vue      # Product search
├── order/                      # Order domain components
│   ├── OrderSummary.vue       # Order summary
│   ├── OrderItem.vue          # Single order item
│   ├── OrderHistory.vue       # Order history list
│   └── OrderStatus.vue        # Order status indicator
└── ui/                         # Base UI primitives
    ├── UiIcon.vue             # Icon component
    ├── UiSpinner.vue          # Loading spinner
    ├── UiBadge.vue            # Badge/tag component
    └── UiTooltip.vue          # Tooltip component
```

#### Component Naming Convention

| Category | Prefix | Example |
|----------|--------|---------|
| Common/Shared | `App` | `AppButton.vue` |
| Layout | `Layout` | `LayoutHeader.vue` |
| UI Primitives | `Ui` | `UiIcon.vue` |
| Domain-specific | Domain name | `UserCard.vue` |

### Composables Directory

Composables contain reusable logic following Vue 3's Composition API patterns.

```
app/composables/
├── api/                        # API-related composables
│   ├── useApi.ts              # Base API client
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
│   ├── useToast.ts            # Toast notifications
│   └── useDialog.ts           # Dialog confirmations
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
└── [domain].ts                 # Domain-specific stores
```

### Layouts Directory

Application layouts for different page structures.

```
app/layouts/
├── default.vue                 # Default layout (with navigation)
├── auth.vue                    # Auth pages layout (minimal)
├── dashboard.vue               # Dashboard layout (with sidebar)
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
├── api.ts                      # API client initialization
├── auth.client.ts              # Client-side auth setup
├── analytics.client.ts         # Analytics integration
├── error-handler.ts            # Global error handling
└── directives.ts               # Custom Vue directives
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

### API Route Naming Convention

| Method | File Suffix | Example | Route |
|--------|-------------|---------|-------|
| GET | `.get.ts` | `users.get.ts` | GET /api/users |
| POST | `.post.ts` | `users.post.ts` | POST /api/users |
| PUT | `.put.ts` | `[id].put.ts` | PUT /api/users/:id |
| PATCH | `.patch.ts` | `[id].patch.ts` | PATCH /api/users/:id |
| DELETE | `.delete.ts` | `[id].delete.ts` | DELETE /api/users/:id |

---

## Tests Directory Structure

```
tests/
├── setup.ts                    # Global test setup
├── utils/                      # Test utilities
│   ├── mocks.ts               # Common mocks
│   ├── factories.ts           # Test data factories
│   └── helpers.ts             # Test helper functions
├── unit/                       # Unit tests
│   ├── utils/                 # Utility function tests
│   │   └── formatDate.test.ts
│   └── composables/           # Composable tests
│       └── useAuth.test.ts
├── integration/                # Integration tests
│   ├── api/                   # API integration tests
│   │   └── auth.test.ts
│   └── stores/                # Store tests
│       └── user.test.ts
└── components/                 # Component tests
    ├── common/
    │   └── AppButton.test.ts
    └── user/
        └── UserCard.test.ts
```

---

## Configuration Files

### config/ Directory

```
config/
├── app.ts                      # Application settings
├── api.ts                      # API configuration
├── i18n.ts                     # Internationalization config
├── pwa.ts                      # PWA configuration
└── security.ts                 # Security settings
```

### Root Configuration Files

| File | Purpose |
|------|---------|
| `nuxt.config.ts` | Main Nuxt configuration |
| `tsconfig.json` | TypeScript configuration |
| `eslint.config.js` | ESLint rules and settings |
| `unocss.config.ts` | UnoCSS configuration |
| `vitest.config.ts` | Test runner configuration |
| `.env.example` | Environment variables template |

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

| Type | Convention | Example |
|------|------------|---------|
| Directories | kebab-case | `user-profile/` |
| Vue Components | PascalCase | `UserProfile.vue` |
| Composables | camelCase, "use" prefix | `useUser.ts` |
| Utilities | camelCase | `formatDate.ts` |
| Types | PascalCase | `UserTypes.ts` |
| Constants | camelCase or UPPER_SNAKE | `apiEndpoints.ts` |
| Tests | Same as source + `.test` | `useUser.test.ts` |
| API Routes | kebab-case + method suffix | `users.get.ts` |

### Component Naming by Category

```typescript
// Common/Generic components - "App" prefix
AppButton.vue
AppModal.vue
AppInput.vue

// Layout components - "Layout" prefix
LayoutHeader.vue
LayoutSidebar.vue

// UI Primitives - "Ui" prefix
UiIcon.vue
UiTooltip.vue

// Domain components - No prefix, domain directory
user/UserCard.vue
product/ProductList.vue

// Page components - No prefix, describes view
user/UserProfile.vue
product/ProductDetail.vue
```

---

## Auto-Imports

Nuxt automatically imports certain files. Understand what is auto-imported:

### Auto-Imported by Default

| Directory | Import Type |
|-----------|-------------|
| `components/` | Vue components |
| `composables/` | Composable functions |
| `utils/` | Utility functions |
| `stores/` | Pinia stores (with @pinia/nuxt) |

### Configure Additional Auto-Imports

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  imports: {
    dirs: [
      'composables/**',
      'utils/**',
      'stores/**',
    ],
  },
})
```

---

## Module Resolution

### Path Aliases

```typescript
// tsconfig.json (generated by Nuxt)
{
  "compilerOptions": {
    "paths": {
      "~/*": ["./app/*"],
      "@/*": ["./app/*"],
      "~~/*": ["./*"],
      "@@/*": ["./*"]
    }
  }
}
```

### Import Examples

```typescript
// Import from app directory
import UserCard from '~/components/user/UserCard.vue'
import { useUser } from '~/composables/user/useUser'
import type { User } from '~/types/user'

// Import from root (shared)
import { ApiError } from '~~/shared/types/api'
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

### 5. Use Index Files Sparingly

Only use `index.ts` when re-exporting:

```typescript
// composables/user/index.ts
export { useUser } from './useUser'
export { useUserProfile } from './useUserProfile'
export { useUserSettings } from './useUserSettings'
```

---

## Next Steps

Continue to [Nuxt Configuration](./03-nuxt-configuration.md) for comprehensive configuration setup covering both SPA and SSR modes.