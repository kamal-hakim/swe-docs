# Form Validation

## Overview

This document covers form validation patterns for Nuxt applications using **VeeValidate 4** integrated with Vuetify 3 components. VeeValidate provides a composition API-based approach to form validation that works seamlessly with Vue 3 and Vuetify's form components.

## Technology Stack

| Tool                    | Purpose                     |
| ----------------------- | --------------------------- |
| **VeeValidate 4**       | Form validation library     |
| **@vee-validate/rules** | Common validation rules     |
| **@vee-validate/yup**   | Yup schema integration      |
| **@vee-validate/zod**   | Zod schema integration      |
| **Yup / Zod**           | Schema validation libraries |

---

## Installation and Setup

### Install Dependencies

```bash
# Core VeeValidate
pnpm add vee-validate

# Validation rules
pnpm add @vee-validate/rules

# Schema validation (choose one)
pnpm add @vee-validate/yup yup
# or
pnpm add @vee-validate/zod zod
```

### Plugin Configuration

```typescript
// plugins/vee-validate.ts
import { defineRule, configure } from "vee-validate";
import {
  required,
  email,
  min,
  max,
  min_value,
  max_value,
  confirmed,
  regex,
  alpha,
  alpha_num,
  alpha_dash,
  numeric,
  integer,
  url,
} from "@vee-validate/rules";
import { localize, setLocale } from "@vee-validate/i18n";
import en from "@vee-validate/i18n/dist/locale/en.json";

export default defineNuxtPlugin(() => {
  // Define global rules
  defineRule("required", required);
  defineRule("email", email);
  defineRule("min", min);
  defineRule("max", max);
  defineRule("min_value", min_value);
  defineRule("max_value", max_value);
  defineRule("confirmed", confirmed);
  defineRule("regex", regex);
  defineRule("alpha", alpha);
  defineRule("alpha_num", alpha_num);
  defineRule("alpha_dash", alpha_dash);
  defineRule("numeric", numeric);
  defineRule("integer", integer);
  defineRule("url", url);

  // Custom rules
  defineRule("phone", (value: string) => {
    if (!value) return true;
    const phoneRegex = /^[\d\s\-+()]+$/;
    return phoneRegex.test(value) || "Invalid phone number format";
  });

  defineRule("password_strength", (value: string) => {
    if (!value) return true;
    const hasUppercase = /[A-Z]/.test(value);
    const hasLowercase = /[a-z]/.test(value);
    const hasNumber = /\d/.test(value);
    const hasSpecial = /[!@#$%^&*(),.?":{}|<>]/.test(value);

    if (!hasUppercase)
      return "Password must contain at least one uppercase letter";
    if (!hasLowercase)
      return "Password must contain at least one lowercase letter";
    if (!hasNumber) return "Password must contain at least one number";
    if (!hasSpecial)
      return "Password must contain at least one special character";
    return true;
  });

  // Configure VeeValidate
  configure({
    generateMessage: localize({
      en: {
        ...en,
        messages: {
          ...en.messages,
          required: "This field is required",
          email: "Please enter a valid email address",
          min: "This field must be at least {length} characters",
          max: "This field must not exceed {length} characters",
          confirmed: "Passwords do not match",
        },
      },
    }),
    validateOnInput: true,
    validateOnChange: true,
    validateOnBlur: true,
    validateOnModelUpdate: true,
  });

  setLocale("en");
});
```

---

## Basic Form Validation

### Using useForm and useField

```vue
<!-- components/forms/LoginForm.vue -->
<script setup lang="ts">
import { useForm, useField } from "vee-validate";
import * as yup from "yup";

// Define validation schema
const schema = yup.object({
  email: yup
    .string()
    .required("Email is required")
    .email("Please enter a valid email"),
  password: yup
    .string()
    .required("Password is required")
    .min(8, "Password must be at least 8 characters"),
  rememberMe: yup.boolean(),
});

// Initialize form
const { handleSubmit, errors, isSubmitting, resetForm } = useForm({
  validationSchema: schema,
  initialValues: {
    email: "",
    password: "",
    rememberMe: false,
  },
});

// Define fields
const { value: email, errorMessage: emailError } = useField<string>("email");
const { value: password, errorMessage: passwordError } =
  useField<string>("password");
const { value: rememberMe } = useField<boolean>("rememberMe");

// Emit events
const emit = defineEmits<{
  submit: [
    credentials: { email: string; password: string; rememberMe: boolean }
  ];
}>();

// Handle form submission
const onSubmit = handleSubmit(async (values) => {
  emit("submit", values);
});
</script>

<template>
  <v-form @submit.prevent="onSubmit">
    <v-text-field
      v-model="email"
      label="Email"
      type="email"
      :error-messages="emailError"
      prepend-inner-icon="mdi-email"
      variant="outlined"
      autocomplete="email"
    />

    <v-text-field
      v-model="password"
      label="Password"
      type="password"
      :error-messages="passwordError"
      prepend-inner-icon="mdi-lock"
      variant="outlined"
      autocomplete="current-password"
      class="mt-4"
    />

    <v-checkbox
      v-model="rememberMe"
      label="Remember me"
      color="primary"
      class="mt-2"
    />

    <v-btn
      type="submit"
      color="primary"
      block
      size="large"
      :loading="isSubmitting"
      class="mt-4"
    >
      Sign In
    </v-btn>
  </v-form>
</template>
```

### Using Zod Schema

```vue
<!-- components/forms/RegisterForm.vue -->
<script setup lang="ts">
import { useForm, useField } from "vee-validate";
import { toTypedSchema } from "@vee-validate/zod";
import { z } from "zod";

// Define Zod schema
const schema = toTypedSchema(
  z
    .object({
      firstName: z
        .string()
        .min(1, "First name is required")
        .max(50, "First name is too long"),
      lastName: z
        .string()
        .min(1, "Last name is required")
        .max(50, "Last name is too long"),
      email: z
        .string()
        .min(1, "Email is required")
        .email("Invalid email address"),
      password: z
        .string()
        .min(8, "Password must be at least 8 characters")
        .regex(/[A-Z]/, "Password must contain an uppercase letter")
        .regex(/[a-z]/, "Password must contain a lowercase letter")
        .regex(/\d/, "Password must contain a number"),
      confirmPassword: z.string().min(1, "Please confirm your password"),
      acceptTerms: z.literal(true, {
        errorMap: () => ({
          message: "You must accept the terms and conditions",
        }),
      }),
    })
    .refine((data) => data.password === data.confirmPassword, {
      message: "Passwords do not match",
      path: ["confirmPassword"],
    })
);

// Initialize form
const { handleSubmit, isSubmitting, meta } = useForm({
  validationSchema: schema,
});

// Define fields
const { value: firstName, errorMessage: firstNameError } =
  useField<string>("firstName");
const { value: lastName, errorMessage: lastNameError } =
  useField<string>("lastName");
const { value: email, errorMessage: emailError } = useField<string>("email");
const { value: password, errorMessage: passwordError } =
  useField<string>("password");
const { value: confirmPassword, errorMessage: confirmPasswordError } =
  useField<string>("confirmPassword");
const { value: acceptTerms, errorMessage: acceptTermsError } =
  useField<boolean>("acceptTerms");

const emit = defineEmits<{
  submit: [
    data: {
      firstName: string;
      lastName: string;
      email: string;
      password: string;
    }
  ];
}>();

const onSubmit = handleSubmit(async (values) => {
  const { confirmPassword: _, acceptTerms: __, ...userData } = values;
  emit("submit", userData);
});
</script>

<template>
  <v-form @submit.prevent="onSubmit">
    <v-row>
      <v-col cols="12" md="6">
        <v-text-field
          v-model="firstName"
          label="First Name"
          :error-messages="firstNameError"
          variant="outlined"
        />
      </v-col>
      <v-col cols="12" md="6">
        <v-text-field
          v-model="lastName"
          label="Last Name"
          :error-messages="lastNameError"
          variant="outlined"
        />
      </v-col>
    </v-row>

    <v-text-field
      v-model="email"
      label="Email"
      type="email"
      :error-messages="emailError"
      variant="outlined"
      class="mt-4"
    />

    <v-text-field
      v-model="password"
      label="Password"
      type="password"
      :error-messages="passwordError"
      variant="outlined"
      class="mt-4"
    />

    <v-text-field
      v-model="confirmPassword"
      label="Confirm Password"
      type="password"
      :error-messages="confirmPasswordError"
      variant="outlined"
      class="mt-4"
    />

    <v-checkbox
      v-model="acceptTerms"
      :error-messages="acceptTermsError"
      color="primary"
      class="mt-2"
    >
      <template #label>
        I accept the
        <NuxtLink to="/terms" class="text-primary ml-1">
          Terms and Conditions
        </NuxtLink>
      </template>
    </v-checkbox>

    <v-btn
      type="submit"
      color="primary"
      block
      size="large"
      :loading="isSubmitting"
      :disabled="!meta.valid"
      class="mt-4"
    >
      Create Account
    </v-btn>
  </v-form>
</template>
```

---

## Custom Validated Components

### AppTextField with Validation

```vue
<!-- components/common/AppTextField.vue -->
<script setup lang="ts">
import { useField } from "vee-validate";

interface Props {
  name: string;
  label?: string;
  type?: string;
  placeholder?: string;
  hint?: string;
  prependInnerIcon?: string;
  appendInnerIcon?: string;
  variant?:
    | "outlined"
    | "filled"
    | "underlined"
    | "solo"
    | "solo-filled"
    | "solo-inverted"
    | "plain";
  density?: "default" | "comfortable" | "compact";
  disabled?: boolean;
  readonly?: boolean;
  clearable?: boolean;
  rules?: string | Record<string, any>;
}

const props = withDefaults(defineProps<Props>(), {
  type: "text",
  variant: "outlined",
  density: "default",
  disabled: false,
  readonly: false,
  clearable: false,
});

// Use VeeValidate field
const { value, errorMessage, handleBlur, handleChange, meta } =
  useField<string>(() => props.name, props.rules, {
    validateOnValueUpdate: true,
  });

// Compute validation state for Vuetify
const validationState = computed(() => {
  if (!meta.touched) return undefined;
  return meta.valid ? "success" : "error";
});
</script>

<template>
  <v-text-field
    v-model="value"
    :label="label"
    :type="type"
    :placeholder="placeholder"
    :hint="hint"
    :prepend-inner-icon="prependInnerIcon"
    :append-inner-icon="appendInnerIcon"
    :variant="variant"
    :density="density"
    :disabled="disabled"
    :readonly="readonly"
    :clearable="clearable"
    :error-messages="errorMessage"
    :color="validationState === 'success' ? 'success' : undefined"
    @blur="handleBlur"
    @update:model-value="handleChange"
  >
    <template v-if="$slots.prepend" #prepend>
      <slot name="prepend" />
    </template>
    <template v-if="$slots.append" #append>
      <slot name="append" />
    </template>
    <template v-if="$slots['append-inner']" #append-inner>
      <slot name="append-inner" />
    </template>
  </v-text-field>
</template>
```

### AppSelect with Validation

```vue
<!-- components/common/AppSelect.vue -->
<script setup lang="ts">
import { useField } from "vee-validate";

interface SelectItem {
  title: string;
  value: string | number;
  disabled?: boolean;
}

interface Props {
  name: string;
  label?: string;
  items: SelectItem[] | string[];
  itemTitle?: string;
  itemValue?: string;
  placeholder?: string;
  hint?: string;
  variant?:
    | "outlined"
    | "filled"
    | "underlined"
    | "solo"
    | "solo-filled"
    | "solo-inverted"
    | "plain";
  density?: "default" | "comfortable" | "compact";
  multiple?: boolean;
  chips?: boolean;
  closableChips?: boolean;
  clearable?: boolean;
  disabled?: boolean;
  rules?: string | Record<string, any>;
}

const props = withDefaults(defineProps<Props>(), {
  itemTitle: "title",
  itemValue: "value",
  variant: "outlined",
  density: "default",
  multiple: false,
  chips: false,
  closableChips: false,
  clearable: false,
  disabled: false,
});

const { value, errorMessage, handleBlur, handleChange, meta } = useField(
  () => props.name,
  props.rules
);
</script>

<template>
  <v-select
    v-model="value"
    :label="label"
    :items="items"
    :item-title="itemTitle"
    :item-value="itemValue"
    :placeholder="placeholder"
    :hint="hint"
    :variant="variant"
    :density="density"
    :multiple="multiple"
    :chips="chips"
    :closable-chips="closableChips"
    :clearable="clearable"
    :disabled="disabled"
    :error-messages="errorMessage"
    @blur="handleBlur"
    @update:model-value="handleChange"
  />
</template>
```

### AppTextarea with Validation

```vue
<!-- components/common/AppTextarea.vue -->
<script setup lang="ts">
import { useField } from "vee-validate";

interface Props {
  name: string;
  label?: string;
  placeholder?: string;
  hint?: string;
  rows?: number;
  autoGrow?: boolean;
  counter?: boolean | number;
  maxLength?: number;
  variant?:
    | "outlined"
    | "filled"
    | "underlined"
    | "solo"
    | "solo-filled"
    | "solo-inverted"
    | "plain";
  disabled?: boolean;
  readonly?: boolean;
  rules?: string | Record<string, any>;
}

const props = withDefaults(defineProps<Props>(), {
  rows: 3,
  autoGrow: false,
  counter: false,
  variant: "outlined",
  disabled: false,
  readonly: false,
});

const { value, errorMessage, handleBlur, handleChange } = useField<string>(
  () => props.name,
  props.rules
);
</script>

<template>
  <v-textarea
    v-model="value"
    :label="label"
    :placeholder="placeholder"
    :hint="hint"
    :rows="rows"
    :auto-grow="autoGrow"
    :counter="counter"
    :maxlength="maxLength"
    :variant="variant"
    :disabled="disabled"
    :readonly="readonly"
    :error-messages="errorMessage"
    @blur="handleBlur"
    @update:model-value="handleChange"
  />
</template>
```

---

## Form Composable

### useFormValidation Composable

```typescript
// composables/useFormValidation.ts
import { useForm, useField, type FormContext } from "vee-validate";
import type { ObjectSchema } from "yup";
import type { ZodSchema } from "zod";
import { toTypedSchema } from "@vee-validate/yup";

interface UseFormValidationOptions<T extends Record<string, any>> {
  schema: ObjectSchema<T> | ZodSchema<T>;
  initialValues?: Partial<T>;
  onSubmit?: (values: T) => void | Promise<void>;
}

export function useFormValidation<T extends Record<string, any>>(
  options: UseFormValidationOptions<T>
) {
  const { schema, initialValues = {}, onSubmit } = options;

  // Convert schema to typed schema
  const validationSchema = toTypedSchema(schema as ObjectSchema<T>);

  // Initialize form
  const form = useForm<T>({
    validationSchema,
    initialValues: initialValues as T,
  });

  // Create field helper
  function createField<K extends keyof T>(name: K) {
    return useField<T[K]>(name as string);
  }

  // Handle submission
  const submitForm = form.handleSubmit(async (values) => {
    if (onSubmit) {
      await onSubmit(values);
    }
  });

  // Reset form to initial values
  function reset() {
    form.resetForm();
  }

  // Set field value programmatically
  function setFieldValue<K extends keyof T>(field: K, value: T[K]) {
    form.setFieldValue(field as string, value);
  }

  // Set multiple values
  function setValues(values: Partial<T>) {
    form.setValues(values);
  }

  // Validate single field
  async function validateField(field: keyof T) {
    return form.validateField(field as string);
  }

  // Validate entire form
  async function validate() {
    return form.validate();
  }

  return {
    // Form state
    values: form.values,
    errors: form.errors,
    meta: form.meta,
    isSubmitting: form.isSubmitting,

    // Field helper
    createField,

    // Actions
    submitForm,
    reset,
    setFieldValue,
    setValues,
    validateField,
    validate,

    // Original form context
    form,
  };
}
```

### Usage Example

```vue
<!-- components/forms/ProfileForm.vue -->
<script setup lang="ts">
import * as yup from "yup";

interface ProfileData {
  firstName: string;
  lastName: string;
  email: string;
  phone: string;
  bio: string;
}

const schema = yup.object({
  firstName: yup.string().required().max(50),
  lastName: yup.string().required().max(50),
  email: yup.string().required().email(),
  phone: yup.string().matches(/^[\d\s\-+()]*$/, "Invalid phone format"),
  bio: yup.string().max(500),
});

const { createField, submitForm, reset, isSubmitting, meta } =
  useFormValidation<ProfileData>({
    schema,
    initialValues: {
      firstName: "",
      lastName: "",
      email: "",
      phone: "",
      bio: "",
    },
    onSubmit: async (values) => {
      await updateProfile(values);
    },
  });

// Create fields
const firstName = createField("firstName");
const lastName = createField("lastName");
const email = createField("email");
const phone = createField("phone");
const bio = createField("bio");
</script>

<template>
  <v-form @submit.prevent="submitForm">
    <v-row>
      <v-col cols="12" md="6">
        <v-text-field
          v-model="firstName.value.value"
          label="First Name"
          :error-messages="firstName.errorMessage.value"
          variant="outlined"
        />
      </v-col>
      <v-col cols="12" md="6">
        <v-text-field
          v-model="lastName.value.value"
          label="Last Name"
          :error-messages="lastName.errorMessage.value"
          variant="outlined"
        />
      </v-col>
    </v-row>

    <v-text-field
      v-model="email.value.value"
      label="Email"
      type="email"
      :error-messages="email.errorMessage.value"
      variant="outlined"
      class="mt-4"
    />

    <v-text-field
      v-model="phone.value.value"
      label="Phone"
      :error-messages="phone.errorMessage.value"
      variant="outlined"
      class="mt-4"
    />

    <v-textarea
      v-model="bio.value.value"
      label="Bio"
      :error-messages="bio.errorMessage.value"
      variant="outlined"
      rows="4"
      counter="500"
      class="mt-4"
    />

    <div class="d-flex ga-4 mt-6">
      <v-btn
        type="submit"
        color="primary"
        :loading="isSubmitting"
        :disabled="!meta.value.valid"
      >
        Save Changes
      </v-btn>
      <v-btn variant="outlined" @click="reset"> Reset </v-btn>
    </div>
  </v-form>
</template>
```

---

## Advanced Validation Patterns

### Cross-Field Validation

```vue
<script setup lang="ts">
import { useForm, useField } from "vee-validate";
import * as yup from "yup";

const schema = yup.object({
  startDate: yup.date().required("Start date is required"),
  endDate: yup
    .date()
    .required("End date is required")
    .min(yup.ref("startDate"), "End date must be after start date"),
  minPrice: yup
    .number()
    .required("Minimum price is required")
    .min(0, "Price must be positive"),
  maxPrice: yup
    .number()
    .required("Maximum price is required")
    .min(yup.ref("minPrice"), "Maximum price must be greater than minimum"),
});

const { handleSubmit } = useForm({ validationSchema: schema });

const { value: startDate, errorMessage: startDateError } =
  useField<string>("startDate");
const { value: endDate, errorMessage: endDateError } =
  useField<string>("endDate");
const { value: minPrice, errorMessage: minPriceError } =
  useField<number>("minPrice");
const { value: maxPrice, errorMessage: maxPriceError } =
  useField<number>("maxPrice");
</script>

<template>
  <v-form @submit.prevent="handleSubmit">
    <v-row>
      <v-col cols="6">
        <v-text-field
          v-model="startDate"
          label="Start Date"
          type="date"
          :error-messages="startDateError"
          variant="outlined"
        />
      </v-col>
      <v-col cols="6">
        <v-text-field
          v-model="endDate"
          label="End Date"
          type="date"
          :error-messages="endDateError"
          variant="outlined"
        />
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="6">
        <v-text-field
          v-model.number="minPrice"
          label="Min Price"
          type="number"
          :error-messages="minPriceError"
          variant="outlined"
          prefix="$"
        />
      </v-col>
      <v-col cols="6">
        <v-text-field
          v-model.number="maxPrice"
          label="Max Price"
          type="number"
          :error-messages="maxPriceError"
          variant="outlined"
          prefix="$"
        />
      </v-col>
    </v-row>
  </v-form>
</template>
```

### Async Validation

```typescript
// Custom async validation rule
import { defineRule } from "vee-validate";

defineRule("unique_email", async (value: string) => {
  if (!value) return true;

  try {
    const response = await $fetch("/api/check-email", {
      method: "POST",
      body: { email: value },
    });

    return response.available || "This email is already registered";
  } catch {
    return "Unable to verify email availability";
  }
});

// Usage with debounce
defineRule("unique_username", async (value: string, _, ctx) => {
  if (!value || value.length < 3) return true;

  // Debounce validation
  await new Promise((resolve) => setTimeout(resolve, 500));

  // Check if value changed during debounce
  if (ctx.value !== value) return true;

  const response = await $fetch("/api/check-username", {
    method: "POST",
    body: { username: value },
  });

  return response.available || "This username is taken";
});
```

```vue
<script setup lang="ts">
const { value: email, errorMessage: emailError } = useField<string>(
  "email",
  "required|email|unique_email"
);

const { value: username, errorMessage: usernameError } = useField<string>(
  "username",
  "required|min:3|unique_username"
);
</script>
```

### Conditional Validation

```vue
<script setup lang="ts">
import { useForm, useField } from "vee-validate";
import * as yup from "yup";

const schema = yup.object({
  contactMethod: yup.string().required().oneOf(["email", "phone", "mail"]),
  email: yup.string().when("contactMethod", {
    is: "email",
    then: (schema) => schema.required("Email is required").email(),
    otherwise: (schema) => schema.notRequired(),
  }),
  phone: yup.string().when("contactMethod", {
    is: "phone",
    then: (schema) => schema.required("Phone is required"),
    otherwise: (schema) => schema.notRequired(),
  }),
  address: yup.string().when("contactMethod", {
    is: "mail",
    then: (schema) => schema.required("Address is required"),
    otherwise: (schema) => schema.notRequired(),
  }),
});

const { handleSubmit } = useForm({ validationSchema: schema });

const { value: contactMethod } = useField<string>("contactMethod");
const { value: email, errorMessage: emailError } = useField<string>("email");
const { value: phone, errorMessage: phoneError } = useField<string>("phone");
const { value: address, errorMessage: addressError } =
  useField<string>("address");

const contactMethods = [
  { title: "Email", value: "email" },
  { title: "Phone", value: "phone" },
  { title: "Mail", value: "mail" },
];
</script>

<template>
  <v-form @submit.prevent="handleSubmit">
    <v-select
      v-model="contactMethod"
      label="Preferred Contact Method"
      :items="contactMethods"
      variant="outlined"
    />

    <v-text-field
      v-if="contactMethod === 'email'"
      v-model="email"
      label="Email Address"
      type="email"
      :error-messages="emailError"
      variant="outlined"
      class="mt-4"
    />

    <v-text-field
      v-if="contactMethod === 'phone'"
      v-model="phone"
      label="Phone Number"
      :error-messages="phoneError"
      variant="outlined"
      class="mt-4"
    />

    <v-textarea
      v-if="contactMethod === 'mail'"
      v-model="address"
      label="Mailing Address"
      :error-messages="addressError"
      variant="outlined"
      class="mt-4"
    />
  </v-form>
</template>
```

### Array Field Validation

```vue
<script setup lang="ts">
import { useForm, useFieldArray, useField } from "vee-validate";
import * as yup from "yup";

const schema = yup.object({
  teamName: yup.string().required("Team name is required"),
  members: yup
    .array()
    .of(
      yup.object({
        name: yup.string().required("Member name is required"),
        email: yup
          .string()
          .required("Email is required")
          .email("Invalid email"),
        role: yup.string().required("Role is required"),
      })
    )
    .min(1, "At least one team member is required")
    .max(10, "Maximum 10 team members allowed"),
});

const { handleSubmit, errors } = useForm({
  validationSchema: schema,
  initialValues: {
    teamName: "",
    members: [{ name: "", email: "", role: "" }],
  },
});

const { value: teamName, errorMessage: teamNameError } =
  useField<string>("teamName");

const { fields, push, remove } = useFieldArray<{
  name: string;
  email: string;
  role: string;
}>("members");

const roles = ["Developer", "Designer", "Manager", "QA"];

function addMember() {
  push({ name: "", email: "", role: "" });
}

function removeMember(index: number) {
  if (fields.value.length > 1) {
    remove(index);
  }
}
</script>

<template>
  <v-form @submit.prevent="handleSubmit">
    <v-text-field
      v-model="teamName"
      label="Team Name"
      :error-messages="teamNameError"
      variant="outlined"
    />

    <v-divider class="my-6" />

    <div class="d-flex justify-space-between align-center mb-4">
      <h3 class="text-h6">Team Members</h3>
      <v-btn
        color="primary"
        variant="outlined"
        size="small"
        :disabled="fields.length >= 10"
        @click="addMember"
      >
        <v-icon start>mdi-plus</v-icon>
        Add Member
      </v-btn>
    </div>

    <v-card
      v-for="(field, index) in fields"
      :key="field.key"
      variant="outlined"
      class="mb-4 pa-4"
    >
      <div class="d-flex justify-space-between align-center mb-4">
        <span class="text-subtitle-2">Member {{ index + 1 }}</span>
        <v-btn
          icon
          size="small"
          variant="text"
          color="error"
          :disabled="fields.length <= 1"
          @click="removeMember(index)"
        >
          <v-icon>mdi-delete</v-icon>
        </v-btn>
      </div>

      <v-row>
        <v-col cols="12" md="4">
          <v-text-field
            v-model="field.value.name"
            label="Name"
            :error-messages="errors[`members[${index}].name`]"
            variant="outlined"
            density="compact"
          />
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field
            v-model="field.value.email"
            label="Email"
            type="email"
            :error-messages="errors[`members[${index}].email`]"
            variant="outlined"
            density="compact"
          />
        </v-col>
        <v-col cols="12" md="4">
          <v-select
            v-model="field.value.role"
            label="Role"
            :items="roles"
            :error-messages="errors[`members[${index}].role`]"
            variant="outlined"
            density="compact"
          />
        </v-col>
      </v-row>
    </v-card>

    <v-alert v-if="errors.members" type="error" variant="tonal" class="mb-4">
      {{ errors.members }}
    </v-alert>

    <v-btn type="submit" color="primary" size="large"> Create Team </v-btn>
  </v-form>
</template>
```

---

## Form Submission Handling

### With Loading and Error States

```vue
<script setup lang="ts">
import { useForm, useField } from "vee-validate";
import * as yup from "yup";

const schema = yup.object({
  email: yup.string().required().email(),
  message: yup.string().required().min(10),
});

const { handleSubmit, isSubmitting, resetForm } = useForm({
  validationSchema: schema,
});

const { value: email, errorMessage: emailError } = useField<string>("email");
const { value: message, errorMessage: messageError } =
  useField<string>("message");

const submitError = ref<string | null>(null);
const submitSuccess = ref(false);

const onSubmit = handleSubmit(async (values) => {
  submitError.value = null;
  submitSuccess.value = false;

  try {
    await $fetch("/api/contact", {
      method: "POST",
      body: values,
    });

    submitSuccess.value = true;
    resetForm();

    // Auto-hide success message
    setTimeout(() => {
      submitSuccess.value = false;
    }, 5000);
  } catch (error: any) {
    submitError.value =
      error.data?.message || "Failed to send message. Please try again.";
  }
});
</script>

<template>
  <v-form @submit.prevent="onSubmit">
    <v-alert
      v-if="submitSuccess"
      type="success"
      variant="tonal"
      closable
      class="mb-4"
      @click:close="submitSuccess = false"
    >
      Your message has been sent successfully!
    </v-alert>

    <v-alert
      v-if="submitError"
      type="error"
      variant="tonal"
      closable
      class="mb-4"
      @click:close="submitError = null"
    >
      {{ submitError }}
    </v-alert>

    <v-text-field
      v-model="email"
      label="Email"
      type="email"
      :error-messages="emailError"
      :disabled="isSubmitting"
      variant="outlined"
    />

    <v-textarea
      v-model="message"
      label="Message"
      :error-messages="messageError"
      :disabled="isSubmitting"
      variant="outlined"
      rows="5"
      class="mt-4"
    />

    <v-btn
      type="submit"
      color="primary"
      size="large"
      :loading="isSubmitting"
      class="mt-4"
    >
      Send Message
    </v-btn>
  </v-form>
</template>
```

---

## Testing Form Validation

### Unit Tests

```typescript
// tests/unit/forms/LoginForm.test.ts
import { describe, it, expect, vi } from "vitest";
import { mount, flushPromises } from "@vue/test-utils";
import { createVuetify } from "vuetify";
import LoginForm from "~/components/forms/LoginForm.vue";

describe("LoginForm", () => {
  const vuetify = createVuetify();

  function mountForm() {
    return mount(LoginForm, {
      global: {
        plugins: [vuetify],
      },
    });
  }

  it("shows validation errors for empty fields", async () => {
    // Arrange
    const wrapper = mountForm();

    // Act
    await wrapper.find("form").trigger("submit");
    await flushPromises();

    // Assert
    expect(wrapper.text()).toContain("Email is required");
    expect(wrapper.text()).toContain("Password is required");
  });

  it("shows error for invalid email format", async () => {
    // Arrange
    const wrapper = mountForm();
    const emailInput = wrapper.find('input[type="email"]');

    // Act
    await emailInput.setValue("invalid-email");
    await emailInput.trigger("blur");
    await flushPromises();

    // Assert
    expect(wrapper.text()).toContain("valid email");
  });

  it("shows error for short password", async () => {
    // Arrange
    const wrapper = mountForm();
    const passwordInput = wrapper.find('input[type="password"]');

    // Act
    await passwordInput.setValue("short");
    await passwordInput.trigger("blur");
    await flushPromises();

    // Assert
    expect(wrapper.text()).toContain("at least 8 characters");
  });

  it("emits submit event with valid data", async () => {
    // Arrange
    const wrapper = mountForm();
    const emailInput = wrapper.find('input[type="email"]');
    const passwordInput = wrapper.find('input[type="password"]');

    // Act
    await emailInput.setValue("test@example.com");
    await passwordInput.setValue("password123");
    await wrapper.find("form").trigger("submit");
    await flushPromises();

    // Assert
    expect(wrapper.emitted("submit")).toBeTruthy();
    expect(wrapper.emitted("submit")![0]).toEqual([
      {
        email: "test@example.com",
        password: "password123",
        rememberMe: false,
      },
    ]);
  });

  it("disables submit button while submitting", async () => {
    // Arrange
    const wrapper = mountForm();
    await wrapper.find('input[type="email"]').setValue("test@example.com");
    await wrapper.find('input[type="password"]').setValue("password123");

    // Act
    wrapper.find("form").trigger("submit");
    await wrapper.vm.$nextTick();

    // Assert
    const submitButton = wrapper.find('button[type="submit"]');
    expect(submitButton.attributes("disabled")).toBeDefined();
  });
});
```

### Integration Tests

```typescript
// tests/integration/forms/RegisterForm.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { mountSuspended } from "@nuxt/test-utils/runtime";
import RegisterForm from "~/components/forms/RegisterForm.vue";

describe("RegisterForm Integration", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("validates all fields before submission", async () => {
    // Arrange
    const wrapper = await mountSuspended(RegisterForm);

    // Act
    await wrapper.find("form").trigger("submit");

    // Assert
    const errors = wrapper.findAll(".v-messages__message");
    expect(errors.length).toBeGreaterThan(0);
  });

  it("validates password confirmation matches", async () => {
    // Arrange
    const wrapper = await mountSuspended(RegisterForm);

    // Act
    await wrapper.find('[name="password"]').setValue("Password123!");
    await wrapper
      .find('[name="confirmPassword"]')
      .setValue("DifferentPassword!");
    await wrapper.find('[name="confirmPassword"]').trigger("blur");

    // Assert
    expect(wrapper.text()).toContain("Passwords do not match");
  });

  it("requires terms acceptance", async () => {
    // Arrange
    const wrapper = await mountSuspended(RegisterForm);

    // Fill all fields except terms
    await wrapper.find('[name="firstName"]').setValue("John");
    await wrapper.find('[name="lastName"]').setValue("Doe");
    await wrapper.find('[name="email"]').setValue("john@example.com");
    await wrapper.find('[name="password"]').setValue("Password123!");
    await wrapper.find('[name="confirmPassword"]').setValue("Password123!");

    // Act
    await wrapper.find("form").trigger("submit");

    // Assert
    expect(wrapper.text()).toContain("accept the terms");
  });
});
```

---

## Best Practices

### 1. Use Schema Validation

```typescript
// ✅ Good: Define schema separately
const schema = yup.object({
  email: yup.string().required().email(),
  password: yup.string().required().min(8),
});

const { handleSubmit } = useForm({ validationSchema: schema });

// ❌ Bad: Inline validation rules
const { value: email } = useField("email", "required|email");
```

### 2. Provide Clear Error Messages

```typescript
// ✅ Good: User-friendly messages
const schema = yup.object({
  email: yup
    .string()
    .required("Please enter your email address")
    .email("Please enter a valid email address"),
});

// ❌ Bad: Technical messages
const schema = yup.object({
  email: yup.string().required().email(),
});
```

### 3. Validate on Appropriate Events

```typescript
// Configure validation timing
configure({
  validateOnInput: true, // Validate as user types
  validateOnChange: true, // Validate on value change
  validateOnBlur: true, // Validate when field loses focus
  validateOnModelUpdate: true, // Validate on v-model update
});
```

### 4. Handle Async Validation Properly

```typescript
// ✅ Good: Debounce async validation
defineRule("unique_email", async (value, _, ctx) => {
  await new Promise((resolve) => setTimeout(resolve, 500));
  if (ctx.value !== value) return true; // Value changed during debounce

  return await checkEmailAvailability(value);
});
```

### 5. Show Loading States

```vue
<template>
  <v-btn
    type="submit"
    :loading="isSubmitting"
    :disabled="!meta.valid || isSubmitting"
  >
    Submit
  </v-btn>
</template>
```

---

## Next Steps

Continue to [i18n Architecture](./10-i18n-architecture.md) to learn about implementing comprehensive internationalization with @nuxtjs/i18n and Vuetify locale integration.
