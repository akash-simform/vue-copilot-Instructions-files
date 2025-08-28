---
applyTo: "**/*.{ts,js,vue}"
---

## Naming Conventions
1. **Variables/Functions**: camelCase (e.g., `fetchUserData()`)
2. **Constants**: SCREAMING_SNAKE_CASE (e.g., `MAX_RETRY_ATTEMPTS`)
3. **Private members**: Prefix with underscore (e.g., `_validateForm()`)
4. **Booleans**: Use positive phrasing (e.g., `isEnabled`, `hasError`)
5. **Components**: PascalCase (e.g., `UserProfile.vue`)
6. **Composables**: Prefix with "use" (e.g., `useUserData()`)

### File Naming Standards
1. **Vue Components**: PascalCase with `.vue` extension - e.g., `UserProfile.vue`
2. **TypeScript files**: camelCase with appropriate suffix:
   - Services: `userService.ts`
   - Types: `user.types.ts` or `userTypes.ts`
   - Composables: `useUserData.ts`
   - Utils: `formatters.ts`, `validators.ts`
3. **Store files**: camelCase with "Store" suffix - e.g., `userStore.ts`
4. **Test files**: Same as source file with `.test.ts` or `.spec.ts` suffix

### Component Naming Standards
1. **Multi-word components**: Always use multi-word names (e.g., `BaseButton`, `UserCard`)
2. **Base components**: Prefix with "Base" (e.g., `BaseButton`, `BaseInput`)
3. **Single-instance components**: Prefix with "The" (e.g., `TheHeader`, `TheSidebar`)
4. **Child components**: Include parent name (e.g., `UserCard`, `UserCardActions`)

## Modern Vue 3 & TypeScript Features
1. **Composition API**: `<script setup lang="ts">` syntax
2. **Reactivity**: `ref()`, `reactive()`, `computed()`, `watch()`
3. **TypeScript**: Proper type annotations and interfaces
4. **Composables**: Reusable stateful logic
5. **Template refs**: Type-safe template references

## Vue 3 Component Structure
```vue
<template>
  <!-- Use semantic HTML and proper accessibility -->
  <div class="user-profile">
    <h2 class="user-profile__title">{{ user.name }}</h2>
    <p class="user-profile__email">{{ user.email }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import type { User } from '@/types/user'

// Props with TypeScript
interface Props {
  userId: string
  showDetails?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  showDetails: true
})

// Emits
interface Emits {
  userUpdated: [user: User]
  error: [message: string]
}

const emit = defineEmits<Emits>()

// Reactive state
const user = ref<User | null>(null)
const loading = ref(false)

// Computed properties
const displayName = computed(() => 
  user.value ? `${user.value.firstName} ${user.value.lastName}` : ''
)

// Lifecycle hooks
onMounted(async () => {
  await fetchUser()
})

// Methods
const fetchUser = async () => {
  try {
    loading.value = true
    // API call logic
  } catch (error) {
    emit('error', 'Failed to fetch user')
  } finally {
    loading.value = false
  }
}
</script>

<style scoped lang="scss">
.user-profile {
  padding: 1rem;
  
  &__title {
    font-size: 1.5rem;
    font-weight: 600;
    color: var(--color-text-primary);
  }
  
  &__email {
    color: var(--color-text-secondary);
  }
}
</style>
```

## Development & Performance Guidelines
1. Always ask for more information if you are not sure about something rather than making assumptions
2. Use Single File Components (SFC) with `<template>`, `<script>`, and `<style>` sections
3. Prefer Composition API over Options API for new components
4. Use TypeScript for type safety and better developer experience
5. Keep components focused and single-purpose
6. Use props for parent-child communication and events for child-parent communication
7. Implement proper error boundaries and loading states
8. Use `v-memo` for expensive list rendering optimizations
9. Implement lazy loading for heavy components using `defineAsyncComponent`
10. Use `<Suspense>` for handling async components gracefully
11. Properly clean up event listeners and timers in `onUnmounted`
12. Use computed properties instead of methods for derived state
13. Implement efficient caching strategies with composables
14. Use Web Workers for heavy computations
15. Always run `npm run type-check` and `npm run lint` before committing
16. Avoid creating sample demo code and ask for which component/feature is to be modified
17. Follow software engineering best practices, principles and design patterns like SOLID, DRY, returning early in functions, etc.

## CSS/SCSS Guidelines
1. Use scoped styles (`<style scoped>`) to prevent style leaking
2. Follow BEM methodology for class naming when needed
3. Use CSS custom properties (variables) for theming
4. Prefer CSS Grid and Flexbox for layouts
5. Use semantic HTML elements for better accessibility
6. Implement responsive design with mobile-first approach

## Accessibility Guidelines
1. Use semantic HTML elements
2. Provide proper ARIA labels and roles
3. Ensure keyboard navigation support
4. Maintain proper color contrast ratios
5. Provide alt text for images
6. Use proper heading hierarchy

## Documentation
1. Use JSDoc comments for complex functions and components
2. Document component props, emits, and slots
3. Include usage examples in component documentation
4. Write clear comments explaining business logic
5. Document composables and their return values
6. Include TypeScript types for better self-documentation

## Code Quality and Linting
1. Run `npm run format` to format your code with Prettier
2. Run `npm run lint` to check for ESLint issues
3. Run `npm run type-check` to verify TypeScript types
4. Use `npm run lint:fix` to automatically fix linting issues
5. Follow rules mentioned in `.eslintrc.js` and `prettier.config.js`

## Testing Guidelines
1. Write unit tests for utilities and composables
2. Write component tests using Vue Testing Library
3. Test user interactions and component behavior
4. Mock external dependencies and API calls
5. Aim for good test coverage but focus on critical paths
6. Use descriptive test names that explain the expected behavior

## Resources
Refer to the following resources for further guidance and best practices:
- [Vue.js Style Guide](https://vuejs.org/style-guide/) for official Vue conventions
- [Vue 3 Documentation](https://vuejs.org/) for official guides and API references
- [TypeScript with Vue](https://vuejs.org/guide/typescript/overview.html) for type safety
- [ESLint Vue Plugin](https://eslint.vuejs.org/) for code linting rules
- [Vue Accessibility Guide](https://vue-a11y.github.io/) for accessibility best practices
