# Ticket: FE-003 - Task Dashboard and Kanban View

**Epic**: Dashboard & Task Management
**Type**: Story
**Priority**: High
**Dependencies**: FE-001, BE-007

## 📝 Story
As a user, I want to see my tasks on a dashboard board so that I can visualize my workload.

## ✅ Acceptance Criteria
- [ ] Dashboard page created at `/dashboard`.
- [ ] `TaskCard` component implemented (displays title, priority, status).
- [ ] `useTask` composable implemented (fetch list).
- [ ] Grid layout using Vuetify (`v-row`/`v-col`) to show tasks.
- [ ] Filter bar (filter by Priority).

## 🛠️ Solution Approach

### Components
```
components/
└── task/
    ├── TaskCard.vue      # The visual card
    ├── TaskList.vue      # Container for cards
    └── TaskFilter.vue    # Search/Filter inputs
```

### Data Fetching
Use Nuxt's `useAsyncData` or `useFetch` for SSR-friendly data loading.

```typescript
// pages/dashboard.vue
const { data: tasks } = await useAsyncData('tasks', () => useTask().list())
```

### Best Practices Explained
> **Smart vs Dumb Components**: `TaskList` (Smart) handles the fetching/store interaction. `TaskCard` (Dumb) just takes props and emits events.
> **Vuetify Grid**: Use `v-container`, `v-row` and `v-col` for responsive layouts that work on mobile.

## 🔗 References
- [Component Architecture](../docs/frontend/04-component-architecture.md)
