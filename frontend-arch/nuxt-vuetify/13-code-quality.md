# Code Quality

## Overview

This document covers code quality tooling and configuration for Nuxt applications with Vuetify, including ESLint for linting, TypeScript for type checking, git hooks for pre-commit validation, and best practices for maintaining code consistency.

## Tooling Stack

| Tool                     | Purpose                          |
| ------------------------ | -------------------------------- |
| **ESLint**               | Code linting and formatting      |
| **@antfu/eslint-config** | Opinionated ESLint configuration |
| **TypeScript**           | Static type checking             |
| **vue-tsc**              | Vue TypeScript checking          |
| **simple-git-hooks**     | Git hooks management             |
| **lint-staged**          | Run linters on staged files      |

---

## ESLint Configuration

### Installation

```bash
# Install ESLint with Anthony Fu's config
pnpm add -D eslint @antfu/eslint-config

# Optional: Additional plugins
pnpm add -D eslint-plugin-vuetify
```

### Basic Configuration

```javascript
// eslint.config.js
// @ts-check
import antfu from "@antfu/eslint-config";

export default antfu({
  // Enable Vue support
  vue: true,

  // Enable TypeScript support
  typescript: true,

  // Stylistic formatting rules
  stylistic: {
    indent: 2,
    quotes: "single",
    semi: false,
  },

  // Ignore patterns
  ignores: [
    "**/node_modules/**",
    "**/.nuxt/**",
    "**/.output/**",
    "**/dist/**",
    "**/public/**",
    "**/*.min.*",
    "**/coverage/**",
    "**/playwright-report/**",
  ],
});
```

### Extended Configuration

```javascript
// eslint.config.js
// @ts-check
import antfu from "@antfu/eslint-config";

export default antfu(
  {
    vue: true,
    typescript: true,
    stylistic: {
      indent: 2,
      quotes: "single",
      semi: false,
    },
    ignores: [
      "**/node_modules/**",
      "**/.nuxt/**",
      "**/.output/**",
      "**/dist/**",
      "**/public/**",
    ],
  },
  // Vue-specific overrides
  {
    files: ["**/*.vue"],
    rules: {
      // Component naming
      "vue/component-name-in-template-casing": ["error", "PascalCase"],
      "vue/multi-word-component-names": "off",

      // Custom event naming
      "vue/custom-event-name-casing": ["error", "camelCase"],

      // Prop naming
      "vue/prop-name-casing": ["error", "camelCase"],

      // Maximum attributes per line
      "vue/max-attributes-per-line": [
        "error",
        {
          singleline: { max: 3 },
          multiline: { max: 1 },
        },
      ],

      // First attribute on new line
      "vue/first-attribute-linebreak": [
        "error",
        {
          singleline: "ignore",
          multiline: "below",
        },
      ],

      // Component tags order
      "vue/component-tags-order": [
        "error",
        {
          order: ["script", "template", "style"],
        },
      ],

      // Define macros order
      "vue/define-macros-order": [
        "error",
        {
          order: ["defineProps", "defineEmits", "defineSlots"],
        },
      ],

      // No unused refs
      "vue/no-unused-refs": "error",

      // Self-closing
      "vue/html-self-closing": [
        "error",
        {
          html: {
            void: "always",
            normal: "never",
            component: "always",
          },
        },
      ],
    },
  },
  // TypeScript-specific overrides
  {
    files: ["**/*.ts", "**/*.tsx", "**/*.mts"],
    rules: {
      // Explicit return types
      "@typescript-eslint/explicit-function-return-type": "off",

      // Unused variables
      "@typescript-eslint/no-unused-vars": [
        "error",
        {
          argsIgnorePattern: "^_",
          varsIgnorePattern: "^_",
        },
      ],

      // Any usage
      "@typescript-eslint/no-explicit-any": "warn",

      // Non-null assertion
      "@typescript-eslint/no-non-null-assertion": "warn",

      // Consistent type imports
      "@typescript-eslint/consistent-type-imports": [
        "error",
        {
          prefer: "type-imports",
          disallowTypeAnnotations: false,
        },
      ],
    },
  },
  // Test file overrides
  {
    files: ["**/tests/**/*.ts", "**/*.test.ts", "**/*.spec.ts"],
    rules: {
      "@typescript-eslint/no-explicit-any": "off",
      "no-console": "off",
    },
  },
  // JSON file sorting
  {
    files: ["**/locales/**/*.json"],
    rules: {
      "jsonc/sort-keys": "error",
    },
  },
  // Nuxt auto-imports
  {
    rules: {
      // Allow Nuxt auto-imports
      "no-undef": "off",
    },
  }
);
```

### Vuetify-Specific Rules

```javascript
// eslint.config.js (additional rules for Vuetify)
{
  files: ['**/*.vue'],
  rules: {
    // Prefer Vuetify components over native HTML
    'vue/no-restricted-html-elements': ['error',
      {
        element: 'button',
        message: 'Use <v-btn> instead of <button>',
      },
      {
        element: 'input',
        message: 'Use Vuetify form components instead of <input>',
      },
      {
        element: 'select',
        message: 'Use <v-select> instead of <select>',
      },
      {
        element: 'textarea',
        message: 'Use <v-textarea> instead of <textarea>',
      },
    ],

    // Enforce NuxtLink over anchor tags
    'vue/no-restricted-syntax': ['error', {
      selector: 'VElement[name=\'a\']',
      message: 'Use <NuxtLink> instead of <a> tags.',
    }],
  },
},
```

---

## TypeScript Configuration

### Base Configuration

```json
// tsconfig.json
{
  "extends": "./.nuxt/tsconfig.json",
  "compilerOptions": {
    // Strict type checking
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,

    // Module resolution
    "moduleResolution": "bundler",
    "module": "ESNext",
    "target": "ESNext",

    // Emit
    "noEmit": true,

    // Interop
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,

    // Skip lib check for faster builds
    "skipLibCheck": true,

    // Paths (Nuxt auto-generates these)
    "paths": {
      "~/*": ["./app/*"],
      "@/*": ["./*"]
    }
  },
  "include": ["app/**/*", "server/**/*", "shared/**/*", "tests/**/*"],
  "exclude": ["node_modules", ".nuxt", ".output", "dist"]
}
```

### Nuxt TypeScript Settings

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  typescript: {
    // Enable type checking on build
    typeCheck: true,

    // Strict mode
    strict: true,

    // Shim d.ts generation
    shim: false,

    // Include paths in tsconfig
    tsConfig: {
      compilerOptions: {
        noUncheckedIndexedAccess: true,
        forceConsistentCasingInFileNames: true,
      },
      include: ["../tests/**/*"],
      exclude: ["../playwright-report/**/*"],
    },
  },
});
```

### Type Declaration Files

```typescript
// app/types/index.d.ts

// Global type augmentation
declare global {
  // Extend Window interface
  interface Window {
    __APP_VERSION__: string;
  }
}

// Component props helpers
export type RequiredProps<T, K extends keyof T> = Omit<T, K> &
  Required<Pick<T, K>>;
export type OptionalProps<T, K extends keyof T> = Omit<T, K> &
  Partial<Pick<T, K>>;

// API response types
export interface ApiResponse<T> {
  data: T;
  meta?: {
    total: number;
    page: number;
    limit: number;
  };
  error?: {
    code: string;
    message: string;
  };
}

// Utility types
export type Nullable<T> = T | null;
export type Optional<T> = T | undefined;
export type VoidFunction = () => void;
export type AsyncFunction<T = void> = () => Promise<T>;
```

### Vuetify Type Augmentation

```typescript
// app/types/vuetify.d.ts
import "vuetify";

declare module "vuetify" {
  export interface ThemeDefinition {
    // Add custom theme properties
    variables?: {
      "border-color"?: string;
      "border-opacity"?: number;
      "high-emphasis-opacity"?: number;
      "medium-emphasis-opacity"?: number;
      "disabled-opacity"?: number;
      "idle-opacity"?: number;
      "hover-opacity"?: number;
      "focus-opacity"?: number;
      "selected-opacity"?: number;
      "activated-opacity"?: number;
      "pressed-opacity"?: number;
      "dragged-opacity"?: number;
      "theme-kbd"?: string;
      "theme-on-kbd"?: string;
      "theme-code"?: string;
      "theme-on-code"?: string;
    };
  }
}

// Extend component props
declare module "vuetify/components" {
  export interface VBtnProps {
    // Add custom props
    loading?: boolean;
  }
}
```

---

## Git Hooks

### Installation

```bash
# Install git hooks tools
pnpm add -D simple-git-hooks lint-staged
```

### Configuration

```json
// package.json
{
  "simple-git-hooks": {
    "pre-commit": "pnpm lint-staged",
    "commit-msg": "node scripts/verify-commit-msg.js"
  },
  "lint-staged": {
    "*.{js,ts,vue}": ["eslint --fix"],
    "*.{json,md,yaml,yml}": ["prettier --write"]
  }
}
```

### Setup Command

```bash
# Add to package.json scripts
{
  "scripts": {
    "prepare": "simple-git-hooks"
  }
}

# Run once after install
pnpm prepare
```

### Commit Message Validation

```javascript
// scripts/verify-commit-msg.js
import { readFileSync } from "node:fs";

const msgPath = process.argv[2];
const msg = readFileSync(msgPath, "utf-8").trim();

// Conventional commit pattern
const commitRE =
  /^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?: .{1,72}/;

if (!commitRE.test(msg)) {
  console.error(`
  Invalid commit message format.

  Proper commit message format is required:
    <type>(<scope>): <subject>

  Examples:
    feat(auth): add login form
    fix(api): handle network errors
    docs: update README
    style(vuetify): update theme colors

  Types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
  `);
  process.exit(1);
}
```

### Advanced Lint-Staged Configuration

```javascript
// lint-staged.config.js
export default {
  // JavaScript/TypeScript files
  "*.{js,ts,mjs,mts}": ["eslint --fix"],

  // Vue files
  "*.vue": ["eslint --fix"],

  // JSON files
  "*.json": ["prettier --write"],

  // Markdown files
  "*.md": ["prettier --write"],

  // YAML files
  "*.{yaml,yml}": ["prettier --write"],

  // Type checking for TypeScript files
  "*.{ts,tsx,vue}": () => "vue-tsc --noEmit",
};
```

---

## Package Scripts

```json
// package.json
{
  "scripts": {
    // Development
    "dev": "nuxt dev",
    "build": "nuxt build",
    "generate": "nuxt generate",
    "preview": "nuxt preview",

    // Linting
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",

    // Type checking
    "typecheck": "nuxt typecheck",
    "typecheck:vue": "vue-tsc --noEmit",

    // Testing
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",

    // All quality checks
    "check": "pnpm lint && pnpm typecheck && pnpm test:run",
    "check:fix": "pnpm lint:fix && pnpm typecheck",

    // Git hooks
    "prepare": "simple-git-hooks",

    // Clean
    "clean": "rm -rf .nuxt .output node_modules/.cache",
    "clean:all": "rm -rf .nuxt .output node_modules dist coverage playwright-report"
  }
}
```

---

## Editor Configuration

### VS Code Settings

```json
// .vscode/settings.json
{
  // Disable built-in formatting
  "editor.formatOnSave": false,

  // Enable ESLint formatting
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit",
    "source.organizeImports": "never"
  },

  // ESLint validation
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact",
    "vue",
    "json",
    "jsonc"
  ],

  // TypeScript settings
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,

  // Vue settings
  "vue.inlayHints.inlineHandlerLeading": true,
  "vue.inlayHints.missingProps": true,
  "vue.inlayHints.optionsWrapper": true,

  // Vuetify settings
  "vuetify.componentCase": "PascalCase",

  // File nesting
  "explorer.fileNesting.enabled": true,
  "explorer.fileNesting.patterns": {
    "package.json": "package-lock.json, pnpm-lock.yaml, yarn.lock, .npmrc, .nvmrc",
    "nuxt.config.ts": "nuxt.config.*, app.config.ts, app.vue, error.vue",
    "tsconfig.json": "tsconfig.*.json",
    "vitest.config.ts": "vitest.config.*, playwright.config.ts",
    "*.vue": "${capture}.test.ts, ${capture}.spec.ts"
  },

  // File associations
  "files.associations": {
    "*.css": "postcss"
  }
}
```

### VS Code Extensions

```json
// .vscode/extensions.json
{
  "recommendations": [
    "vue.volar",
    "dbaeumer.vscode-eslint",
    "vuetifyjs.vuetify-vscode",
    "bradlc.vscode-tailwindcss",
    "editorconfig.editorconfig",
    "ms-playwright.playwright"
  ]
}
```

### EditorConfig

```ini
# .editorconfig
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.{yml,yaml}]
indent_size = 2

[Makefile]
indent_style = tab
```

---

## Code Quality Rules

### Import Organization

```javascript
// ESLint import rules
{
  rules: {
    // Import order
    'import/order': ['error', {
      'groups': [
        'builtin',
        'external',
        'internal',
        'parent',
        'sibling',
        'index',
        'type',
      ],
      'pathGroups': [
        {
          pattern: '#*',
          group: 'internal',
          position: 'after',
        },
        {
          pattern: '~/**',
          group: 'internal',
          position: 'after',
        },
      ],
      'newlines-between': 'always',
      'alphabetize': {
        order: 'asc',
        caseInsensitive: true,
      },
    }],

    // No duplicate imports
    'import/no-duplicates': 'error',

    // Prefer named exports
    'import/prefer-default-export': 'off',
  },
}
```

### Naming Conventions

```javascript
// Naming convention rules
{
  rules: {
    '@typescript-eslint/naming-convention': [
      'error',
      // Variables
      {
        selector: 'variable',
        format: ['camelCase', 'UPPER_CASE', 'PascalCase'],
        leadingUnderscore: 'allow',
      },
      // Functions
      {
        selector: 'function',
        format: ['camelCase', 'PascalCase'],
      },
      // Types
      {
        selector: 'typeLike',
        format: ['PascalCase'],
      },
      // Interfaces (no I prefix)
      {
        selector: 'interface',
        format: ['PascalCase'],
        custom: {
          regex: '^I[A-Z]',
          match: false,
        },
      },
      // Enum members
      {
        selector: 'enumMember',
        format: ['UPPER_CASE', 'PascalCase'],
      },
    ],
  },
}
```

---

## CI/CD Integration

### GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm

      - name: Install dependencies
        run: pnpm install

      - name: Run ESLint
        run: pnpm lint

      - name: Run type check
        run: pnpm typecheck

  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm

      - name: Install dependencies
        run: pnpm install

      - name: Run unit tests
        run: pnpm test:run

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm

      - name: Install dependencies
        run: pnpm install

      - name: Install Playwright browsers
        run: pnpm exec playwright install --with-deps

      - name: Run E2E tests
        run: pnpm test:e2e

      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm

      - name: Install dependencies
        run: pnpm install

      - name: Build application
        run: pnpm build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: .output
```

### Pre-merge Checks

```yaml
# .github/workflows/pr-check.yml
name: PR Check

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  check:
    name: Quality Gate
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v2
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm

      - name: Install dependencies
        run: pnpm install

      - name: Check formatting
        run: pnpm lint

      - name: Check types
        run: pnpm typecheck

      - name: Run tests with coverage
        run: pnpm test:coverage

      - name: Check coverage threshold
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage is below 80%: $COVERAGE%"
            exit 1
          fi
```

---

## Best Practices

### 1. Consistent Code Style

- Use ESLint with a shared configuration
- Enable format-on-save in your editor
- Run linting in pre-commit hooks
- Enforce rules in CI/CD pipeline

### 2. Type Safety

```typescript
// ✅ Good: Full type coverage
interface User {
  id: string;
  name: string;
  email: string;
}

function getUser(id: string): Promise<User> {
  return $fetch(`/api/users/${id}`);
}

// ❌ Bad: Using any
function getUser(id: any): Promise<any> {
  return $fetch(`/api/users/${id}`);
}
```

### 3. Git Workflow

- Use conventional commits
- Run quality checks before committing
- Keep commits atomic and focused
- Write descriptive commit messages

### 4. Code Reviews

- Enforce linting in PR checks
- Review for code quality, not just functionality
- Use automated tools for consistency
- Document conventions in contribution guide

### 5. Documentation

- Document complex logic inline
- Keep README up to date
- Document API contracts
- Maintain changelog

---

## Next Steps

Continue to [Security and Performance](./14-security-and-performance.md) to learn about security headers, CSP, and performance optimization.
