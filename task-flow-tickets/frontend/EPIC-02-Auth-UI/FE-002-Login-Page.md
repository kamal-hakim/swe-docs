# Ticket: FE-002 - Login Page Implementation

**Epic**: Authentication Flow
**Type**: Story
**Priority**: High
**Dependencies**: FE-001, BE-004

## 📝 Story
As a user, I need a login page so that I can authenticate and access the dashboard.

## ✅ Acceptance Criteria
- [ ] Login page created at `/login`.
- [ ] `useAuth` composable implemented for API interaction.
- [ ] `auth` store (Pinia) implemented to manage session state.
- [ ] Form validation using VeeValidate + Zod (Email/Password requirements).
- [ ] Redirect to `/dashboard` on success.
- [ ] Display error toast on failure.

## 🛠️ Solution Approach

### Component Hierarchy
- `pages/login.vue` (Layout: `auth.vue`)
  - `components/auth/LoginForm.vue`
    - `VTextField` (with validation)
    - `AppButton`

### State Management
1. **Composable**: `useAuth()` calls the generic `useApi()`.
2. **Store**: `useAuthStore` saves the JWT token (in Cookie via `useCookie`) and User object.

```typescript
// stores/auth.ts
const token = useCookie('auth_token')
const user = useState('user')

async function login(credentials) {
  const { data } = await useAuth().login(credentials)
  token.value = data.token
  user.value = data.user
}
```

### Best Practices Explained
> **Validation**: Never rely solely on backend validation. Use VeeValidate schemas to give instant feedback.
> **Cookies**: Store tokens in `HttpOnly` cookies if possible, or at least `useCookie` in Nuxt for SSR compatibility.

## 🔗 References
- [Authentication Architecture](../docs/frontend/07-routing-and-navigation.md)
