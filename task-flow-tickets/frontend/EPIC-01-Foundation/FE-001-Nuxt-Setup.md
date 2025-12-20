# Ticket: FE-001 - Nuxt 3 Project Setup & Configuration

**Epic**: Foundation & Design System
**Type**: Task
**Priority**: Highest

## 📝 Story
As a frontend developer, I need the base Nuxt 3 project set up with Vuetify 3 and our folder structure so that I can begin building UI components.

## ✅ Acceptance Criteria
- [ ] Nuxt 3 project initialized.
- [ ] Vuetify 3 configured (with treeshaking).
- [ ] Folder structure matches reference architecture (`app/components/[domain]`, `app/stores`, etc.).
- [ ] Pinia installed.
- [ ] ESLint + Prettier configured.
- [ ] `layouts/default.vue` implemented with a basic App Bar.

## 🛠️ Solution Approach

### Structure
Move the generated `app.vue` into `app/` directory and configure `nuxt.config.ts` to scan `app/`.

```
app/
├── components/
│   ├── common/
│   ├── layout/
│   └── vuetify/
├── composables/
├── pages/
├── plugins/
└── assets/styles/settings.scss
```

### Vuetify Setup
- Use `vite-plugin-vuetify` for auto-imports.
- Define a custom theme in `plugins/vuetify.ts` (Primary: Deep Blue, Secondary: Amber).

### Best Practices Explained
> **Domain Organization**: We don't dump everything in `components/`. We use `components/user`, `components/task` etc.
> **Strict Types**: Ensure `tsconfig.json` has `strict: true`.

## 🔗 References
- [Project Structure](../docs/frontend/02-project-structure.md)
