# Component Architecture

## Overview

This document outlines the component architecture patterns for Nuxt applications using Vuetify 3 and Vue 3's Composition API. It covers component organization, custom Vuetify wrappers, composition strategies, and best practices for building maintainable, reusable components.

## Component Categories

### 1. Vuetify Wrapper Components

Custom components that extend Vuetify components with application-specific defaults and functionality.

```
components/vuetify/
├── VTextField.vue         # Extended text field
├── VSelect.vue            # Extended select
├── VDataTable.vue         # Extended data table
├── VCard.vue              # Extended card
├── VDialog.vue            # Extended dialog
├── VBtn.vue               # Extended button
└── VSnackbar.vue          # Extended snackbar
```

### 2. Common Components

Shared components used across multiple features with application-specific logic.

```
components/common/
├── AppButton.vue          # Application button
├── AppModal.vue           # Modal dialog wrapper
├── AppToast.vue           # Toast notification
├── AppPaginator.vue       # Pagination wrapper
├── AppDropdown.vue        # Dropdown menu
├── AppErrorBoundary.vue   # Error boundary
└── AppConfirmDialog.vue   # Confirmation dialog
```

### 3. Layout Components

Components that define page structure and navigation using Vuetify layout components.

```
components/layout/
├── LayoutHeader.vue       # v-app-bar wrapper
├── LayoutSidebar.vue      # v-navigation-drawer wrapper
├── LayoutFooter.vue       # v-footer wrapper
├── LayoutBreadcrumb.vue   # v-breadcrumbs wrapper
└── LayoutNavigation.vue   # Navigation menu
```

### 4. Domain Components

Feature-specific components organized by business domain.

```
components/user/
├── UserCard.vue
├── UserAvatar.vue
├── UserProfile.vue
└── UserSettings.vue

components/product/
├── ProductCard.vue
├── ProductList.vue
├── ProductDetail.vue
└── ProductFilter.vue
```

---

## Custom Vuetify Wrapper Components

### Extended Text Field

```vue
<!-- components/vuetify/VTextField.vue -->
<script setup lang="ts">
import { computed } from "vue";
import type { VTextField } from "vuetify/components";

/**
 * Extended VTextField with application defaults
 * @description Wraps Vuetify's VTextField with consistent styling and validation
 */

interface Props {
  modelValue?: string | number;
  label?: string;
  placeholder?: string;
  hint?: string;
  errorMessages?: string | string[];
  rules?: ((value: any) => boolean | string)[];
  variant?:
    | "outlined"
    | "filled"
    | "underlined"
    | "plain"
    | "solo"
    | "solo-inverted"
    | "solo-filled";
  density?: "default" | "comfortable" | "compact";
  type?: string;
  disabled?: boolean;
  readonly?: boolean;
  loading?: boolean;
  clearable?: boolean;
  prependIcon?: string;
  appendIcon?: string;
  prependInnerIcon?: string;
  appendInnerIcon?: string;
  maxlength?: number;
  counter?: boolean | number | string;
  persistentHint?: boolean;
  persistentPlaceholder?: boolean;
  hideDetails?: boolean | "auto";
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: "",
  variant: "outlined",
  density: "comfortable",
  type: "text",
  disabled: false,
  readonly: false,
  loading: false,
  clearable: false,
  persistentHint: false,
  persistentPlaceholder: false,
  hideDetails: "auto",
});

const emit = defineEmits<{
  "update:modelValue": [value: string | number];
  "click:clear": [];
  "click:append": [];
  "click:prepend": [];
  "click:appendInner": [];
  "click:prependInner": [];
  focus: [event: FocusEvent];
  blur: [event: FocusEvent];
}>();

const localValue = computed({
  get: () => props.modelValue,
  set: (value) => emit("update:modelValue", value),
});

const hasError = computed(() => {
  if (!props.errorMessages) return false;
  if (Array.isArray(props.errorMessages)) return props.errorMessages.length > 0;
  return !!props.errorMessages;
});
</script>

<template>
  <v-text-field
    v-model="localValue"
    :label="label"
    :placeholder="placeholder"
    :hint="hint"
    :error-messages="errorMessages"
    :rules="rules"
    :variant="variant"
    :density="density"
    :type="type"
    :disabled="disabled"
    :readonly="readonly"
    :loading="loading"
    :clearable="clearable"
    :prepend-icon="prependIcon"
    :append-icon="appendIcon"
    :prepend-inner-icon="prependInnerIcon"
    :append-inner-icon="appendInnerIcon"
    :maxlength="maxlength"
    :counter="counter"
    :persistent-hint="persistentHint"
    :persistent-placeholder="persistentPlaceholder"
    :hide-details="hideDetails"
    :class="{ 'v-text-field--error': hasError }"
    color="primary"
    @click:clear="emit('click:clear')"
    @click:append="emit('click:append')"
    @click:prepend="emit('click:prepend')"
    @click:append-inner="emit('click:appendInner')"
    @click:prepend-inner="emit('click:prependInner')"
    @focus="emit('focus', $event)"
    @blur="emit('blur', $event)"
  >
    <template v-if="$slots.prepend" #prepend>
      <slot name="prepend" />
    </template>
    <template v-if="$slots.append" #append>
      <slot name="append" />
    </template>
    <template v-if="$slots.prependInner" #prepend-inner>
      <slot name="prepend-inner" />
    </template>
    <template v-if="$slots.appendInner" #append-inner>
      <slot name="append-inner" />
    </template>
    <template v-if="$slots.details" #details>
      <slot name="details" />
    </template>
  </v-text-field>
</template>
```

### Extended Select

```vue
<!-- components/vuetify/VSelect.vue -->
<script setup lang="ts">
import { computed } from "vue";

/**
 * Extended VSelect with application defaults
 */

interface SelectItem {
  title: string;
  value: string | number;
  disabled?: boolean;
  [key: string]: any;
}

interface Props {
  modelValue?: string | number | (string | number)[] | null;
  items: SelectItem[] | string[] | number[];
  label?: string;
  placeholder?: string;
  hint?: string;
  errorMessages?: string | string[];
  rules?: ((value: any) => boolean | string)[];
  variant?:
    | "outlined"
    | "filled"
    | "underlined"
    | "plain"
    | "solo"
    | "solo-inverted"
    | "solo-filled";
  density?: "default" | "comfortable" | "compact";
  disabled?: boolean;
  readonly?: boolean;
  loading?: boolean;
  clearable?: boolean;
  multiple?: boolean;
  chips?: boolean;
  closableChips?: boolean;
  itemTitle?: string;
  itemValue?: string;
  returnObject?: boolean;
  hideDetails?: boolean | "auto";
  noDataText?: string;
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: null,
  variant: "outlined",
  density: "comfortable",
  disabled: false,
  readonly: false,
  loading: false,
  clearable: false,
  multiple: false,
  chips: false,
  closableChips: false,
  itemTitle: "title",
  itemValue: "value",
  returnObject: false,
  hideDetails: "auto",
  noDataText: "No data available",
});

const emit = defineEmits<{
  "update:modelValue": [value: string | number | (string | number)[] | null];
  "update:search": [value: string];
}>();

const localValue = computed({
  get: () => props.modelValue,
  set: (value) => emit("update:modelValue", value),
});
</script>

<template>
  <v-select
    v-model="localValue"
    :items="items"
    :label="label"
    :placeholder="placeholder"
    :hint="hint"
    :error-messages="errorMessages"
    :rules="rules"
    :variant="variant"
    :density="density"
    :disabled="disabled"
    :readonly="readonly"
    :loading="loading"
    :clearable="clearable"
    :multiple="multiple"
    :chips="chips"
    :closable-chips="closableChips"
    :item-title="itemTitle"
    :item-value="itemValue"
    :return-object="returnObject"
    :hide-details="hideDetails"
    :no-data-text="noDataText"
    color="primary"
  >
    <template v-if="$slots.item" #item="slotProps">
      <slot name="item" v-bind="slotProps" />
    </template>
    <template v-if="$slots.selection" #selection="slotProps">
      <slot name="selection" v-bind="slotProps" />
    </template>
    <template v-if="$slots.prepend" #prepend>
      <slot name="prepend" />
    </template>
    <template v-if="$slots.append" #append>
      <slot name="append" />
    </template>
    <template v-if="$slots['no-data']" #no-data>
      <slot name="no-data" />
    </template>
  </v-select>
</template>
```

### Extended Data Table

```vue
<!-- components/vuetify/VDataTable.vue -->
<script setup lang="ts" generic="T extends Record<string, any>">
import { computed, ref } from "vue";

/**
 * Extended VDataTable with application defaults
 */

interface DataTableHeader {
  title: string;
  key: string;
  align?: "start" | "center" | "end";
  sortable?: boolean;
  filterable?: boolean;
  width?: string | number;
  minWidth?: string;
  maxWidth?: string;
}

interface Props {
  items: T[];
  headers: DataTableHeader[];
  loading?: boolean;
  search?: string;
  itemsPerPage?: number;
  itemsPerPageOptions?: number[];
  showSelect?: boolean;
  selectStrategy?: "single" | "page" | "all";
  modelValue?: T[];
  itemValue?: string;
  sortBy?: { key: string; order: "asc" | "desc" }[];
  hover?: boolean;
  density?: "default" | "comfortable" | "compact";
  fixedHeader?: boolean;
  fixedFooter?: boolean;
  height?: string | number;
  noDataText?: string;
  loadingText?: string;
  hideDefaultFooter?: boolean;
  hideDefaultHeader?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  loading: false,
  search: "",
  itemsPerPage: 10,
  itemsPerPageOptions: () => [5, 10, 25, 50, 100],
  showSelect: false,
  selectStrategy: "page",
  modelValue: () => [],
  itemValue: "id",
  sortBy: () => [],
  hover: true,
  density: "comfortable",
  fixedHeader: false,
  fixedFooter: false,
  noDataText: "No data available",
  loadingText: "Loading...",
  hideDefaultFooter: false,
  hideDefaultHeader: false,
});

const emit = defineEmits<{
  "update:modelValue": [value: T[]];
  "update:sortBy": [value: { key: string; order: "asc" | "desc" }[]];
  "update:itemsPerPage": [value: number];
  "update:page": [value: number];
  "click:row": [event: Event, item: { item: T }];
}>();

const selected = computed({
  get: () => props.modelValue,
  set: (value) => emit("update:modelValue", value),
});

const localSortBy = computed({
  get: () => props.sortBy,
  set: (value) => emit("update:sortBy", value),
});

const page = ref(1);
const localItemsPerPage = computed({
  get: () => props.itemsPerPage,
  set: (value) => emit("update:itemsPerPage", value),
});
</script>

<template>
  <v-data-table
    v-model="selected"
    v-model:sort-by="localSortBy"
    v-model:items-per-page="localItemsPerPage"
    v-model:page="page"
    :items="items"
    :headers="headers"
    :loading="loading"
    :search="search"
    :items-per-page-options="itemsPerPageOptions"
    :show-select="showSelect"
    :select-strategy="selectStrategy"
    :item-value="itemValue"
    :hover="hover"
    :density="density"
    :fixed-header="fixedHeader"
    :fixed-footer="fixedFooter"
    :height="height"
    :no-data-text="noDataText"
    :loading-text="loadingText"
    :hide-default-footer="hideDefaultFooter"
    :hide-default-header="hideDefaultHeader"
    class="elevation-1"
    @click:row="(event, item) => emit('click:row', event, item)"
  >
    <!-- Pass through all slots -->
    <template v-for="(_, name) in $slots" #[name]="slotProps">
      <slot :name="name" v-bind="slotProps || {}" />
    </template>
  </v-data-table>
</template>

<style scoped>
:deep(.v-data-table) {
  border-radius: 8px;
}

:deep(.v-data-table__tr:hover) {
  background-color: rgba(var(--v-theme-primary), 0.04) !important;
}
</style>
```

### Extended Dialog

```vue
<!-- components/vuetify/VDialog.vue -->
<script setup lang="ts">
import { computed, watch } from "vue";

/**
 * Extended VDialog with application defaults
 */

interface Props {
  modelValue: boolean;
  title?: string;
  subtitle?: string;
  width?: string | number;
  maxWidth?: string | number;
  fullscreen?: boolean;
  persistent?: boolean;
  scrollable?: boolean;
  closeOnBack?: boolean;
  closeOnContentClick?: boolean;
  noClickAnimation?: boolean;
  transition?: string;
  hideOverlay?: boolean;
  loading?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  width: "auto",
  maxWidth: 600,
  fullscreen: false,
  persistent: false,
  scrollable: false,
  closeOnBack: true,
  closeOnContentClick: false,
  noClickAnimation: false,
  transition: "dialog-transition",
  hideOverlay: false,
  loading: false,
});

const emit = defineEmits<{
  "update:modelValue": [value: boolean];
  close: [];
}>();

const isOpen = computed({
  get: () => props.modelValue,
  set: (value) => emit("update:modelValue", value),
});

function close() {
  isOpen.value = false;
  emit("close");
}

// Handle escape key
watch(isOpen, (value) => {
  if (value && !props.persistent) {
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === "Escape") {
        close();
      }
    };
    document.addEventListener("keydown", handleEscape);
    return () => document.removeEventListener("keydown", handleEscape);
  }
});

defineExpose({ close });
</script>

<template>
  <v-dialog
    v-model="isOpen"
    :width="width"
    :max-width="maxWidth"
    :fullscreen="fullscreen"
    :persistent="persistent"
    :scrollable="scrollable"
    :close-on-back="closeOnBack"
    :close-on-content-click="closeOnContentClick"
    :no-click-animation="noClickAnimation"
    :transition="transition"
    :scrim="!hideOverlay"
  >
    <v-card :loading="loading">
      <v-card-title v-if="title || $slots.title" class="d-flex align-center">
        <slot name="title">
          <span class="text-h6">{{ title }}</span>
        </slot>
        <v-spacer />
        <v-btn
          v-if="!persistent"
          icon="mdi-close"
          variant="text"
          size="small"
          @click="close"
        />
      </v-card-title>

      <v-card-subtitle v-if="subtitle || $slots.subtitle">
        <slot name="subtitle">
          {{ subtitle }}
        </slot>
      </v-card-subtitle>

      <v-divider v-if="title || subtitle || $slots.title || $slots.subtitle" />

      <v-card-text :class="{ 'pa-0': $slots['no-padding'] }">
        <slot />
      </v-card-text>

      <template v-if="$slots.actions">
        <v-divider />
        <v-card-actions>
          <slot name="actions" :close="close" />
        </v-card-actions>
      </template>
    </v-card>
  </v-dialog>
</template>
```

---

## Common Application Components

### App Button

```vue
<!-- components/common/AppButton.vue -->
<script setup lang="ts">
import { computed } from "vue";

/**
 * Application button with consistent styling
 */

interface Props {
  variant?: "flat" | "elevated" | "tonal" | "outlined" | "text" | "plain";
  color?: string;
  size?: "x-small" | "small" | "default" | "large" | "x-large";
  icon?: string;
  prependIcon?: string;
  appendIcon?: string;
  loading?: boolean;
  disabled?: boolean;
  block?: boolean;
  rounded?: string | number | boolean;
  type?: "button" | "submit" | "reset";
  to?: string | object;
  href?: string;
  target?: string;
}

const props = withDefaults(defineProps<Props>(), {
  variant: "flat",
  color: "primary",
  size: "default",
  loading: false,
  disabled: false,
  block: false,
  rounded: "lg",
  type: "button",
});

const emit = defineEmits<{
  click: [event: MouseEvent];
}>();

const isIconOnly = computed(
  () => props.icon && !props.prependIcon && !props.appendIcon
);
</script>

<template>
  <v-btn
    :variant="variant"
    :color="color"
    :size="size"
    :icon="isIconOnly ? icon : undefined"
    :prepend-icon="prependIcon"
    :append-icon="appendIcon"
    :loading="loading"
    :disabled="disabled"
    :block="block"
    :rounded="rounded"
    :type="type"
    :to="to"
    :href="href"
    :target="target"
    @click="emit('click', $event)"
  >
    <template v-if="!isIconOnly">
      <slot />
    </template>
  </v-btn>
</template>
```

### App Modal (Confirm Dialog)

```vue
<!-- components/common/AppConfirmDialog.vue -->
<script setup lang="ts">
import { ref, computed } from "vue";

/**
 * Confirmation dialog component
 */

interface Props {
  modelValue: boolean;
  title?: string;
  message?: string;
  confirmText?: string;
  cancelText?: string;
  confirmColor?: string;
  cancelColor?: string;
  type?: "info" | "warning" | "danger" | "success";
  persistent?: boolean;
  loading?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  title: "Confirm",
  message: "Are you sure you want to proceed?",
  confirmText: "Confirm",
  cancelText: "Cancel",
  confirmColor: "primary",
  cancelColor: "grey",
  type: "info",
  persistent: false,
  loading: false,
});

const emit = defineEmits<{
  "update:modelValue": [value: boolean];
  confirm: [];
  cancel: [];
}>();

const isOpen = computed({
  get: () => props.modelValue,
  set: (value) => emit("update:modelValue", value),
});

const iconMap = {
  info: { icon: "mdi-information", color: "info" },
  warning: { icon: "mdi-alert", color: "warning" },
  danger: { icon: "mdi-alert-circle", color: "error" },
  success: { icon: "mdi-check-circle", color: "success" },
};

const typeConfig = computed(() => iconMap[props.type]);

function handleConfirm() {
  emit("confirm");
  if (!props.loading) {
    isOpen.value = false;
  }
}

function handleCancel() {
  emit("cancel");
  isOpen.value = false;
}
</script>

<template>
  <v-dialog
    v-model="isOpen"
    max-width="400"
    :persistent="persistent || loading"
  >
    <v-card>
      <v-card-title class="d-flex align-center ga-2">
        <v-icon :icon="typeConfig.icon" :color="typeConfig.color" />
        <span>{{ title }}</span>
      </v-card-title>

      <v-card-text>
        <slot>
          {{ message }}
        </slot>
      </v-card-text>

      <v-card-actions>
        <v-spacer />
        <v-btn
          :color="cancelColor"
          variant="text"
          :disabled="loading"
          @click="handleCancel"
        >
          {{ cancelText }}
        </v-btn>
        <v-btn
          :color="type === 'danger' ? 'error' : confirmColor"
          variant="flat"
          :loading="loading"
          @click="handleConfirm"
        >
          {{ confirmText }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>
```

### App Toast (Snackbar)

```vue
<!-- components/common/AppToast.vue -->
<script setup lang="ts">
import { computed } from "vue";

/**
 * Toast notification component using v-snackbar
 */

interface Props {
  modelValue: boolean;
  message: string;
  type?: "success" | "error" | "warning" | "info";
  timeout?: number;
  location?:
    | "top"
    | "bottom"
    | "top start"
    | "top end"
    | "bottom start"
    | "bottom end";
  closable?: boolean;
  multiLine?: boolean;
  vertical?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  type: "info",
  timeout: 5000,
  location: "bottom",
  closable: true,
  multiLine: false,
  vertical: false,
});

const emit = defineEmits<{
  "update:modelValue": [value: boolean];
}>();

const isVisible = computed({
  get: () => props.modelValue,
  set: (value) => emit("update:modelValue", value),
});

const colorMap = {
  success: "success",
  error: "error",
  warning: "warning",
  info: "info",
};

const iconMap = {
  success: "mdi-check-circle",
  error: "mdi-alert-circle",
  warning: "mdi-alert",
  info: "mdi-information",
};

const color = computed(() => colorMap[props.type]);
const icon = computed(() => iconMap[props.type]);
</script>

<template>
  <v-snackbar
    v-model="isVisible"
    :color="color"
    :timeout="timeout"
    :location="location"
    :multi-line="multiLine"
    :vertical="vertical"
    rounded="lg"
  >
    <div class="d-flex align-center ga-2">
      <v-icon :icon="icon" />
      <span>{{ message }}</span>
    </div>

    <template v-if="closable" #actions>
      <v-btn
        variant="text"
        icon="mdi-close"
        size="small"
        @click="isVisible = false"
      />
    </template>
  </v-snackbar>
</template>
```

---

## Layout Components

### Layout Header

```vue
<!-- components/layout/LayoutHeader.vue -->
<script setup lang="ts">
import { useTheme } from "vuetify";

/**
 * Application header using v-app-bar
 */

interface Props {
  title?: string;
  showDrawerToggle?: boolean;
  showThemeToggle?: boolean;
  elevation?: number;
}

const props = withDefaults(defineProps<Props>(), {
  title: "My App",
  showDrawerToggle: true,
  showThemeToggle: true,
  elevation: 1,
});

const emit = defineEmits<{
  "toggle-drawer": [];
}>();

const theme = useTheme();

function toggleTheme() {
  theme.global.name.value = theme.global.current.value.dark ? "light" : "dark";
}

const isDark = computed(() => theme.global.current.value.dark);
</script>

<template>
  <v-app-bar :elevation="elevation" color="surface">
    <v-app-bar-nav-icon
      v-if="showDrawerToggle"
      @click="emit('toggle-drawer')"
    />

    <v-app-bar-title>
      <slot name="title">
        {{ title }}
      </slot>
    </v-app-bar-title>

    <template #append>
      <slot name="actions" />

      <v-btn
        v-if="showThemeToggle"
        :icon="isDark ? 'mdi-weather-sunny' : 'mdi-weather-night'"
        variant="text"
        @click="toggleTheme"
      />

      <slot name="append" />
    </template>
  </v-app-bar>
</template>
```

### Layout Sidebar

```vue
<!-- components/layout/LayoutSidebar.vue -->
<script setup lang="ts">
import { computed } from "vue";

/**
 * Application sidebar using v-navigation-drawer
 */

interface NavItem {
  title: string;
  icon?: string;
  to?: string;
  href?: string;
  children?: NavItem[];
  divider?: boolean;
  header?: string;
}

interface Props {
  modelValue: boolean;
  items: NavItem[];
  rail?: boolean;
  railWidth?: number;
  width?: number;
  permanent?: boolean;
  temporary?: boolean;
  location?: "start" | "end" | "left" | "right";
}

const props = withDefaults(defineProps<Props>(), {
  rail: false,
  railWidth: 56,
  width: 280,
  permanent: false,
  temporary: false,
  location: "start",
});

const emit = defineEmits<{
  "update:modelValue": [value: boolean];
  "update:rail": [value: boolean];
}>();

const isOpen = computed({
  get: () => props.modelValue,
  set: (value) => emit("update:modelValue", value),
});
</script>

<template>
  <v-navigation-drawer
    v-model="isOpen"
    :rail="rail"
    :rail-width="railWidth"
    :width="width"
    :permanent="permanent"
    :temporary="temporary"
    :location="location"
  >
    <slot name="prepend" />

    <v-list nav density="compact">
      <template v-for="(item, index) in items" :key="index">
        <!-- Divider -->
        <v-divider v-if="item.divider" class="my-2" />

        <!-- Header -->
        <v-list-subheader v-else-if="item.header">
          {{ item.header }}
        </v-list-subheader>

        <!-- Group with children -->
        <v-list-group v-else-if="item.children?.length" :value="item.title">
          <template #activator="{ props: activatorProps }">
            <v-list-item
              v-bind="activatorProps"
              :prepend-icon="item.icon"
              :title="item.title"
            />
          </template>

          <v-list-item
            v-for="(child, childIndex) in item.children"
            :key="childIndex"
            :to="child.to"
            :href="child.href"
            :prepend-icon="child.icon"
            :title="child.title"
          />
        </v-list-group>

        <!-- Single item -->
        <v-list-item
          v-else
          :to="item.to"
          :href="item.href"
          :prepend-icon="item.icon"
          :title="item.title"
        />
      </template>
    </v-list>

    <template #append>
      <slot name="append" />
    </template>
  </v-navigation-drawer>
</template>
```

---

## Domain Components

### User Card

```vue
<!-- components/user/UserCard.vue -->
<script setup lang="ts">
import type { User } from "~/types";

/**
 * User card component displaying user information
 */

interface Props {
  user: User;
  showActions?: boolean;
  loading?: boolean;
  clickable?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  showActions: false,
  loading: false,
  clickable: false,
});

const emit = defineEmits<{
  click: [user: User];
  edit: [user: User];
  delete: [user: User];
}>();

const initials = computed(() => {
  const names = props.user.name.split(" ");
  return names
    .map((n) => n[0])
    .join("")
    .toUpperCase()
    .slice(0, 2);
});

const roleColor = computed(() => {
  const colors: Record<string, string> = {
    admin: "error",
    user: "primary",
    guest: "grey",
  };
  return colors[props.user.role] || "grey";
});
</script>

<template>
  <v-card
    :loading="loading"
    :hover="clickable"
    :ripple="clickable"
    @click="clickable && emit('click', user)"
  >
    <v-card-item>
      <template #prepend>
        <v-avatar
          :image="user.avatar"
          :color="user.avatar ? undefined : 'primary'"
          size="48"
        >
          <span v-if="!user.avatar" class="text-h6">{{ initials }}</span>
        </v-avatar>
      </template>

      <v-card-title>{{ user.name }}</v-card-title>
      <v-card-subtitle>{{ user.email }}</v-card-subtitle>

      <template #append>
        <v-chip
          :color="roleColor"
          size="small"
          variant="tonal"
          data-testid="role-badge"
        >
          {{ user.role }}
        </v-chip>
      </template>
    </v-card-item>

    <v-card-actions v-if="showActions">
      <v-spacer />
      <v-btn
        icon="mdi-pencil"
        variant="text"
        size="small"
        data-testid="edit-button"
        @click.stop="emit('edit', user)"
      />
      <v-btn
        icon="mdi-delete"
        variant="text"
        size="small"
        color="error"
        data-testid="delete-button"
        @click.stop="emit('delete', user)"
      />
    </v-card-actions>
  </v-card>
</template>
```

### Product Card

```vue
<!-- components/product/ProductCard.vue -->
<script setup lang="ts">
import type { Product } from "~/types";

/**
 * Product card component
 */

interface Props {
  product: Product;
  showActions?: boolean;
  loading?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  showActions: true,
  loading: false,
});

const emit = defineEmits<{
  click: [product: Product];
  "add-to-cart": [product: Product];
  favorite: [product: Product];
}>();

const { n } = useI18n();

const formattedPrice = computed(() => {
  return n(props.product.price, "currency");
});

const stockColor = computed(() => {
  if (props.product.stock === 0) return "error";
  if (props.product.stock < 10) return "warning";
  return "success";
});
</script>

<template>
  <v-card :loading="loading" hover @click="emit('click', product)">
    <v-img :src="product.image" :alt="product.name" height="200" cover>
      <template #placeholder>
        <v-row class="fill-height ma-0" align="center" justify="center">
          <v-progress-circular indeterminate color="grey-lighten-5" />
        </v-row>
      </template>

      <v-chip v-if="product.discount" color="error" size="small" class="ma-2">
        -{{ product.discount }}%
      </v-chip>
    </v-img>

    <v-card-item>
      <v-card-title class="text-truncate">
        {{ product.name }}
      </v-card-title>
      <v-card-subtitle>
        {{ product.category }}
      </v-card-subtitle>
    </v-card-item>

    <v-card-text>
      <div class="d-flex align-center justify-space-between">
        <span class="text-h6 font-weight-bold">{{ formattedPrice }}</span>
        <v-chip :color="stockColor" size="small" variant="tonal">
          {{ product.stock }} in stock
        </v-chip>
      </div>
    </v-card-text>

    <v-card-actions v-if="showActions">
      <v-btn
        variant="text"
        icon="mdi-heart-outline"
        @click.stop="emit('favorite', product)"
      />
      <v-spacer />
      <v-btn
        color="primary"
        variant="flat"
        prepend-icon="mdi-cart-plus"
        :disabled="product.stock === 0"
        @click.stop="emit('add-to-cart', product)"
      >
        Add to Cart
      </v-btn>
    </v-card-actions>
  </v-card>
</template>
```

---

## Component Composition Patterns

### Using Composables with Vuetify

```vue
<script setup lang="ts">
import { useDisplay, useTheme } from "vuetify";

// Vuetify composables
const display = useDisplay();
const theme = useTheme();

// Custom composables
const { user, loading, fetchUser } = useUser();
const { showToast } = useToast();

// Responsive behavior
const isMobile = computed(() => display.mobile.value);
const isTablet = computed(() => display.smAndDown.value);

// Theme management
const isDark = computed(() => theme.global.current.value.dark);

function toggleTheme() {
  theme.global.name.value = isDark.value ? "light" : "dark";
}

// Data fetching
onMounted(async () => {
  try {
    await fetchUser();
  } catch (error) {
    showToast("Failed to load user", "error");
  }
});
</script>

<template>
  <v-container :fluid="isMobile">
    <v-row>
      <v-col :cols="isMobile ? 12 : 6">
        <UserCard v-if="user" :user="user" :loading="loading" />
      </v-col>
    </v-row>
  </v-container>
</template>
```

### Provide/Inject with Vuetify

```vue
<!-- Parent Component -->
<script setup lang="ts">
import { provide, ref } from "vue";
import type { InjectionKey, Ref } from "vue";

interface FormContext {
  disabled: Ref<boolean>;
  loading: Ref<boolean>;
  density: Ref<"default" | "comfortable" | "compact">;
}

export const FormContextKey: InjectionKey<FormContext> = Symbol("form-context");

const disabled = ref(false);
const loading = ref(false);
const density = ref<"default" | "comfortable" | "compact">("comfortable");

provide(FormContextKey, { disabled, loading, density });
</script>

<!-- Child Component -->
<script setup lang="ts">
import { inject } from "vue";
import { FormContextKey } from "./ParentComponent.vue";

const formContext = inject(FormContextKey);

if (!formContext) {
  throw new Error("FormContext not provided");
}

const { disabled, loading, density } = formContext;
</script>

<template>
  <v-text-field :disabled="disabled" :loading="loading" :density="density" />
</template>
```

---

## Testing Components

### Component Test Structure

```typescript
// tests/integration/components/user/UserCard.test.ts
import { describe, it, expect, vi } from "vitest";
import { mount } from "@vue/test-utils";
import { createVuetify } from "vuetify";
import * as components from "vuetify/components";
import * as directives from "vuetify/directives";
import UserCard from "~/components/user/UserCard.vue";
import { createMockUser } from "~/tests/fixtures/users";

const vuetify = createVuetify({ components, directives });

function mountWithVuetify(component: any, options = {}) {
  return mount(component, {
    global: {
      plugins: [vuetify],
    },
    ...options,
  });
}

describe("UserCard", () => {
  const mockUser = createMockUser();

  it("renders user name and email", () => {
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: mockUser },
    });

    expect(wrapper.text()).toContain(mockUser.name);
    expect(wrapper.text()).toContain(mockUser.email);
  });

  it("displays role badge with correct color", () => {
    const adminUser = createMockUser({ role: "admin" });
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: adminUser },
    });

    const badge = wrapper.find('[data-testid="role-badge"]');
    expect(badge.exists()).toBe(true);
    expect(badge.text()).toBe("admin");
  });

  it("emits click event when clickable", async () => {
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: mockUser, clickable: true },
    });

    await wrapper.trigger("click");
    expect(wrapper.emitted("click")).toBeTruthy();
    expect(wrapper.emitted("click")![0]).toEqual([mockUser]);
  });

  it("shows action buttons when showActions is true", () => {
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: mockUser, showActions: true },
    });

    expect(wrapper.find('[data-testid="edit-button"]').exists()).toBe(true);
    expect(wrapper.find('[data-testid="delete-button"]').exists()).toBe(true);
  });

  it("emits edit event when edit button clicked", async () => {
    const wrapper = mountWithVuetify(UserCard, {
      props: { user: mockUser, showActions: true },
    });

    await wrapper.find('[data-testid="edit-button"]').trigger("click");
    expect(wrapper.emitted("edit")).toBeTruthy();
    expect(wrapper.emitted("edit")![0]).toEqual([mockUser]);
  });
});
```

---

## Best Practices Summary

### Component Design

1. **Single Responsibility**: One component, one purpose
2. **Props Down, Events Up**: Clear data flow direction
3. **Type Everything**: Full TypeScript coverage
4. **Document Props**: Add JSDoc comments for complex props
5. **Default Values**: Always provide sensible defaults

### Vuetify Integration

1. **Use Vuetify Defaults**: Configure global defaults in plugin
2. **Extend, Don't Replace**: Create wrappers that extend Vuetify components
3. **Consistent Styling**: Use theme colors and variables
4. **Responsive Design**: Use Vuetify's display composable
5. **Accessibility**: Leverage Vuetify's built-in a11y features

### Organization

1. **Domain-Driven**: Group by feature, not type
2. **Consistent Naming**: Follow naming conventions strictly
3. **Colocate Tests**: Keep tests near components
4. **Extract Logic**: Move complex logic to composables

---

## Next Steps

Continue to [Composables Patterns](./05-composables-patterns.md) to learn about creating and organizing Vue 3 composables for logic reuse with Vuetify integration.
