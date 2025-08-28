# Vue.js Development Copilot Instructions

## Overview
- {{cookiecutter._app_class_name}} is a modular Vue.js application using {{cookiecutter.state_management}} state management
- Project follows feature-first, layered architecture with Vue.js best practices
- Main directory: `{{cookiecutter.repo_name}}/`

## Project Architecture & Structure
```
src/
  ├── main.ts           # Entry point for the application
  ├── App.vue           # Root Vue component
  ├── router/           # Vue Router configuration
  │   └── index.ts      # Router setup and route definitions
  ├── store/            # State management (Vuex/Pinia)
  │   ├── index.ts      # Store configuration
  │   └── modules/      # Feature-specific store modules
  ├── views/            # Page-level components
  ├── components/       # Reusable UI components
  │   ├── common/       # Global shared components
  │   └── feature/      # Feature-specific components
  ├── composables/      # Vue 3 composition functions
  ├── services/         # API and external service integrations
  │   ├── api/          # API service layer
  │   └── http/         # HTTP client configuration
  ├── types/            # TypeScript type definitions
  │   ├── api.ts        # API response/request types
  │   └── global.ts     # Global type definitions
  ├── utils/            # Utility functions and helpers
  │   ├── constants.ts  # Application constants
  │   ├── formatters.ts # Data formatting utilities
  │   └── validators.ts # Validation functions
  ├── assets/           # Static assets
  │   ├── images/       # Image files
  │   ├── styles/       # Global styles and SCSS/CSS
  │   └── icons/        # Icon files
  └── locales/          # Internationalization files
      ├── en.json       # English translations
      └── index.ts      # i18n configuration
```

## Application Build Environments
| **Environment** | **Build Command**        | **Description**      |
|-----------------|--------------------------|----------------------|
| development     | `npm run dev`            | Development server   |
| staging         | `npm run build:staging`  | Staging build        |
| production      | `npm run build`          | Production build     |

## State Management Setup
{% if cookiecutter.state_management == "pinia" %}
1. This project uses `Pinia` for state management.
2. Define stores using the Composition API style in `src/store/modules/`.
3. Access stores in components using `const store = useFeatureStore()`.
4. Use stores for reactive state, getters, and actions.
{% else %}
1. This project uses `Vuex` for state management.
2. Define modules in `src/store/modules/` following the Vuex pattern.
3. Access store state using `$store.state.module.property` or computed properties.
4. Use actions for async operations and mutations for state changes.
{% endif %}

## Navigation Usage
1. This project uses Vue Router for navigation and routing.
2. Route definitions are managed in `src/router/index.ts`.
3. Use `router.push()` for programmatic navigation.
4. Use `<router-link>` for declarative navigation in templates.
5. Implement route guards for authentication and authorization.

## Component Architecture
1. Use Single File Components (SFC) with `<template>`, `<script>`, and `<style>` sections.
2. Prefer Composition API over Options API for new components.
3. Use TypeScript with `<script setup lang="ts">` syntax.
4. Keep components focused and single-purpose.
5. Use props for parent-child communication and events for child-parent communication.

## String Localization Usage
- Add user-facing strings to appropriate locale files in `src/locales/`
- Use `$t('key')` in templates or `t('key')` in composition functions
- While adding new strings, ensure translations exist for all supported locales
- Avoid static strings in components; always use localized strings
- Use nested objects for organizing translation keys by feature

## Asset Management
This project uses Vite's asset handling for managing static assets in a type-safe manner.

### Adding and Using Assets
**Structure**: Place assets in the `src/assets/` directory following the organized structure.

**Usage**:
```typescript
// In TypeScript/JavaScript
import logoUrl from '@/assets/images/logo.png'

// In templates
<img src="@/assets/images/logo.png" alt="Logo" />
```

## Environment Variables
The project uses Vite's environment variable system for configuration.

### Adding Environment Variables
1. Create `.env`, `.env.local`, `.env.staging`, or `.env.production` files in the project root.
2. Prefix variables with `VITE_` to expose them to the client-side code.
3. Access variables using `import.meta.env.VITE_VARIABLE_NAME`.

## Other Instruction Files
ALWAYS ensure to identify which of the following files is relevant to your task and refer to it for detailed instructions:
1. For anything related to code style, refer to [Code Style](instructions/code-style.instructions.md) file.
2. For anything related to state management, refer to [State Management](instructions/state-management.instructions.md) file.
3. For anything related to API integration, refer to [Api Integration](instructions/api-integration.instructions.md) file.
4. For anything related to package management, refer to [Package Update](instructions/package-update.instructions.md) file.
5. For anything related to pagination, refer to [Pagination](instructions/pagination.instructions.md) file.
6. For anything related to git workflow, refer to [Git Workflow](instructions/git-workflow.instructions.md) file.

## Resources
Refer to the following resources for more information and best practices:
- [Vue.js Documentation](https://vuejs.org/) for official guides and API references
- [Vue 3 Composition API](https://vuejs.org/guide/extras/composition-api-faq.html) for modern Vue development
- [TypeScript with Vue](https://vuejs.org/guide/typescript/overview.html) for type safety
- [Vue Router](https://router.vuejs.org/) for navigation and routing
{% if cookiecutter.state_management == "pinia" %}
- [Pinia Documentation](https://pinia.vuejs.org/) for state management
{% else %}
- [Vuex Documentation](https://vuex.vuejs.org/) for state management
{% endif %}
- [Vite Documentation](https://vitejs.dev/) for build tool and configuration
- [Vue I18n](https://vue-i18n.intlify.dev/) for internationalization
- [ESLint Vue Plugin](https://eslint.vuejs.org/) for code linting
- [Axios Documentation](https://axios-http.com/) for HTTP requests
- [Vue Testing Library](https://testing-library.com/docs/vue-testing-library/intro/) for component testing
