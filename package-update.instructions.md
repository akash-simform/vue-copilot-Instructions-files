---
applyTo: "**/package.json"
---

When new package(s) are added to the `package.json` file, ensure that the following steps are followed:

1. Always use the latest stable version of the package available on npm, unless a specific version is required for compatibility reasons.
2. Use exact version numbers (without `^` or `~`) for production dependencies to ensure consistency across environments.
3. Add proper brief description of the package above the line where package is added as a comment.
4. Categorize all packages into subcategories in package.json using comments: Frontend Framework, State Management, UI Components, HTTP Client, Development Tools, Testing, etc.
5. Ensure that packages are added in alphabetical order within their category.
6. Ensure that the package is added in the correct section:
   - `dependencies`: Runtime dependencies needed for production
   - `devDependencies`: Development-only dependencies (build tools, testing, linting)
   - `peerDependencies`: Dependencies that should be provided by the consuming application
7. Update README.md with the new package information if necessary.
8. Run `npm install` once all packages are added to install them and update package-lock.json.
9. Do the necessary setup for the package if required, such as:
   - Adding configurations in `vite.config.ts`
   - Setting up in `main.ts` file
   - Creating configuration files (e.g., `.eslintrc.js`, `tailwind.config.js`)
   - Refer to the package's documentation and examples on npm for guidance.
10. When resolving dependency conflicts:
    - Use `npm ls` to identify conflicting dependencies
    - Use `overrides` field in package.json for npm 8.3+ to resolve conflicts
    - Consider using `resolutions` field for Yarn compatibility
11. Verify the package works correctly:
    - Run `npm run dev` to test in development
    - Run `npm run build` to ensure production build works
    - Run `npm run type-check` for TypeScript projects
    - Run tests if applicable

## Example Package.json Structure

```json
{
  "name": "vue-project",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc && vite build",
    "preview": "vite preview",
    "type-check": "vue-tsc --noEmit",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix --ignore-path .gitignore",
    "format": "prettier --write src/",
    "test": "vitest",
    "test:coverage": "vitest --coverage"
  },
  "dependencies": {
    // Frontend Framework
    "vue": "3.4.0",
    "vue-router": "4.2.0",
    
    // State Management  
    "pinia": "2.1.0",
    
    // HTTP Client
    "axios": "1.6.0",
    
    // UI Components
    "@headlessui/vue": "1.7.0",
    
    // Utilities
    "lodash-es": "4.17.0"
  },
  "devDependencies": {
    // Build Tools
    "@vitejs/plugin-vue": "4.5.0",
    "vite": "5.0.0",
    
    // TypeScript
    "@vue/tsconfig": "0.5.0",
    "typescript": "5.3.0",
    "vue-tsc": "1.8.0",
    
    // Linting & Formatting
    "@typescript-eslint/eslint-plugin": "6.14.0",
    "@vue/eslint-config-prettier": "8.0.0",
    "@vue/eslint-config-typescript": "12.0.0",
    "eslint": "8.56.0",
    "eslint-plugin-vue": "9.19.0",
    "prettier": "3.1.0",
    
    // Testing
    "@vue/test-utils": "2.4.0",
    "jsdom": "23.0.0",
    "vitest": "1.0.0"
  },
  "overrides": {
    // Use overrides to resolve dependency conflicts
    "package-with-conflict": "specific-version"
  }
}
```

## Package Categories for Vue.js Projects

### Runtime Dependencies
- **Frontend Framework**: vue, vue-router
- **State Management**: pinia, vuex
- **HTTP Client**: axios, fetch-based libraries
- **UI Components**: headlessui, primevue, quasar
- **Styling**: tailwindcss, bootstrap-vue-next
- **Icons**: @heroicons/vue, lucide-vue-next
- **Utilities**: lodash-es, date-fns, validator
- **Internationalization**: vue-i18n
- **Animation**: @vueuse/motion, vue-transition

### Development Dependencies
- **Build Tools**: vite, webpack, rollup
- **TypeScript**: typescript, vue-tsc, @vue/tsconfig
- **Linting**: eslint, @vue/eslint-config-*
- **Formatting**: prettier
- **Testing**: vitest, @vue/test-utils, cypress
- **Type Definitions**: @types/*

## Vite Configuration for New Packages

When adding packages that require build configuration:

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [
    vue(),
    // Add new plugins here
  ],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      // Add new aliases here
    },
  },
  css: {
    preprocessorOptions: {
      scss: {
        // Add SCSS configurations
      },
    },
  },
  optimizeDeps: {
    include: [
      // Add packages that need pre-bundling
    ],
  },
})
```

## Common Package Setups

### TailwindCSS Setup
```bash
npm install tailwindcss @tailwindcss/forms @tailwindcss/typography autoprefixer postcss
npx tailwindcss init -p
```

### ESLint + Prettier Setup
```bash
npm install --save-dev eslint prettier @vue/eslint-config-prettier @vue/eslint-config-typescript
```

### Testing Setup
```bash
npm install --save-dev vitest @vue/test-utils jsdom @vitest/coverage-v8
```

## Troubleshooting Common Issues

1. **Module Resolution Issues**: Update `vite.config.ts` with proper aliases
2. **TypeScript Errors**: Add type definitions or configure `tsconfig.json`
3. **Build Failures**: Check for incompatible packages or missing peer dependencies
4. **Hot Reload Issues**: Restart development server after configuration changes
