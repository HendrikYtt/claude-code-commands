---
name: frontend-design
description: Create frontend pages, components, and UI following project patterns. Use this skill when asked to build web components, create pages, design forms, implement UI features, or work with React/Vue/Svelte frontends.
---

# Frontend Design Skill

Create production-ready frontend pages and components that follow existing project patterns.

## When to Use This Skill

This skill should be used when:
- Creating new pages or routes
- Building form components
- Implementing UI features
- Styling components
- Setting up routing
- Adding translations/i18n

## Step 1: Detect Project Stack

Before creating any frontend code, detect the project's frontend setup:

### Framework Detection

```bash
# Check package.json for framework
cat package.json | grep -E "react|vue|svelte|angular|next|nuxt"
```

**Identify:**
- React / React + Vite / Next.js
- Vue / Vue + Vite / Nuxt
- Svelte / SvelteKit
- Angular

### Styling Detection

```bash
# Check for styling setup
ls -la tailwind.config.* postcss.config.* 2>/dev/null
cat package.json | grep -E "tailwindcss|styled-components|emotion|sass|less"
```

**Identify:**
- Tailwind CSS
- CSS Modules
- Styled Components / Emotion
- Sass/Less
- Plain CSS

### Component Library Detection

```bash
cat package.json | grep -E "shadcn|radix|headless|daisyui|material|antd|chakra"
```

**Identify:**
- shadcn/ui
- Radix UI
- Headless UI
- DaisyUI
- Material UI
- Ant Design
- Chakra UI
- Custom components

### State Management Detection

```bash
cat package.json | grep -E "redux|zustand|jotai|recoil|pinia|vuex|mobx"
```

### Form Library Detection

```bash
cat package.json | grep -E "formik|react-hook-form|vee-validate|zod|yup"
```

### i18n Detection

```bash
cat package.json | grep -E "i18next|react-intl|vue-i18n"
ls -la src/locales/ locales/ public/locales/ 2>/dev/null
```

## Step 2: Explore Existing Patterns

**Read existing components** to understand patterns:

```bash
# Find existing pages
find src -name "*.tsx" -path "*/pages/*" | head -5
find src -name "*.vue" -path "*/views/*" | head -5

# Find existing components
find src -name "*.tsx" -path "*/components/*" | head -10

# Find routing setup
find src -name "routes*" -o -name "router*" | head -5
```

**Read and analyze:**
1. Page structure and layout patterns
2. Component organization
3. Styling conventions
4. Form handling patterns
5. API call patterns
6. Error/loading state handling

## Step 3: Design Approach

Based on detected stack and existing patterns:

### Page Structure

**React/Next.js pattern:**
```tsx
export const MyPage = () => {
  const [isLoading, setIsLoading] = useState(false);
  const { t } = useTranslation(); // if i18n

  return (
    <PageLayout>
      <PageHeader title={t('pages.myPage.title')} />
      <PageContent>
        {/* Content */}
      </PageContent>
    </PageLayout>
  );
};
```

**Vue pattern:**
```vue
<template>
  <PageLayout>
    <PageHeader :title="$t('pages.myPage.title')" />
    <PageContent>
      <!-- Content -->
    </PageContent>
  </PageLayout>
</template>

<script setup lang="ts">
import { ref } from 'vue';
const isLoading = ref(false);
</script>
```

### Component Guidelines

1. **Reuse existing components** - Always check `src/components/ui/` first
2. **Read component props** - Understand every prop before using
3. **Follow naming conventions** - Match existing file/component naming
4. **Export style** - Match existing export pattern (named vs default)

### Styling Guidelines

**Tailwind (if detected):**
- Use existing utility patterns from other components
- Check tailwind.config for custom colors/spacing
- Use responsive prefixes: `md:`, `lg:`
- Use state prefixes: `hover:`, `focus:`, `disabled:`

**CSS Modules (if detected):**
- Create `.module.css` alongside component
- Use existing class naming conventions
- Import as `styles` and use `styles.className`

### Form Guidelines

**Formik + Yup (if detected):**
```tsx
const formik = useFormik({
  initialValues: { field: '' },
  validationSchema: Yup.object({
    field: Yup.string().required(t('validation.required'))
  }),
  onSubmit: async (values) => {
    // Handle submit
  }
});
```

**React Hook Form + Zod (if detected):**
```tsx
const form = useForm<FormData>({
  resolver: zodResolver(schema),
  defaultValues: { field: '' }
});
```

### Routing Guidelines

**React Router (if detected):**
- Add path constant to paths file
- Add route to routes config
- Use lazy loading for pages

**Next.js (if detected):**
- Create file in `app/` or `pages/` directory
- Follow file-based routing conventions

### API Call Guidelines

1. Check existing API patterns in `src/api/` or similar
2. Use existing HTTP client/wrapper
3. Handle loading states
4. Handle errors with toast/notification
5. Type request/response

### i18n Guidelines

If i18n detected:
1. Add keys to all locale files
2. Use translation function: `t('key')`
3. Follow existing key structure

## Step 4: Implementation

### Create Files in Order

1. **Types/interfaces** (if shared)
2. **API functions** (if new endpoints)
3. **Components** (smallest to largest)
4. **Pages** (compose components)
5. **Routes** (register new pages)
6. **Translations** (add all keys)

### Checklist

Before completing:

- [ ] Reused existing components where possible
- [ ] Followed existing code patterns
- [ ] Added proper TypeScript types
- [ ] Handled loading states
- [ ] Handled error states
- [ ] Added translations (if i18n used)
- [ ] Added route (if new page)
- [ ] Responsive design considered
- [ ] Accessibility basics (labels, alt text, focus states)

## Common Patterns

### Loading State
```tsx
{isLoading ? (
  <Skeleton className="h-10 w-full" />
) : (
  <Content />
)}
```

### Error Handling
```tsx
try {
  await api.call();
  toast.success(t('success'));
} catch (error) {
  toast.error(t('error'));
}
```

### Form with Validation
```tsx
<Input
  label={t('form.fieldLabel')}
  name="field"
  value={formik.values.field}
  onChange={formik.handleChange}
  onBlur={formik.handleBlur}
  error={formik.errors.field}
  touched={formik.touched.field}
/>
```

### Conditional Rendering
```tsx
{user.role === 'admin' && (
  <AdminOnlyComponent />
)}
```

### List Rendering
```tsx
{items.map((item) => (
  <ItemCard key={item.id} item={item} />
))}
```

## Anti-Patterns to Avoid

1. **Don't create new components** when existing ones suffice
2. **Don't inline styles** when Tailwind/CSS modules exist
3. **Don't skip loading states** for async operations
4. **Don't hardcode strings** when i18n is used
5. **Don't ignore TypeScript** - add proper types
6. **Don't forget mobile** - test responsive design
