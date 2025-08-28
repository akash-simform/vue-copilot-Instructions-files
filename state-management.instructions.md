---
applyTo: "**/*.{ts,js,vue}"
---

This project uses `{{cookiecutter.state_management}}` for state management.

{% if cookiecutter.state_management == "pinia" %}
## Pinia Store Development
**Important**: Only create Pinia stores when application state needs to be managed across components.

### Store Responsibility
1. API calls and data transformation
2. Business logic and validation  
3. Cross-component state management
4. Error handling and user feedback
5. Keep stores focused and single-purpose

### Store Structure with TypeScript

```typescript
// src/stores/userStore.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { userService } from '@/services/user/userService'
import type { User, CreateUserRequest } from '@/types/user'

export const useUserStore = defineStore('user', () => {
  // State
  const users = ref<User[]>([])
  const currentUser = ref<User | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)
  
  // Getters (computed)
  const userCount = computed(() => users.value.length)
  const isAuthenticated = computed(() => currentUser.value !== null)
  const activeUsers = computed(() => 
    users.value.filter(user => user.isActive)
  )
  
  // Actions
  const fetchUsers = async () => {
    try {
      loading.value = true
      error.value = null
      users.value = await userService.getUsers()
    } catch (err) {
      error.value = 'Failed to fetch users'
      console.error('Error fetching users:', err)
    } finally {
      loading.value = false
    }
  }
  
  const createUser = async (userData: CreateUserRequest) => {
    try {
      loading.value = true
      const newUser = await userService.createUser(userData)
      users.value.push(newUser)
      return newUser
    } catch (err) {
      error.value = 'Failed to create user'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  const updateUser = async (id: string, userData: Partial<User>) => {
    try {
      const updatedUser = await userService.updateUser(id, userData)
      const index = users.value.findIndex(user => user.id === id)
      if (index !== -1) {
        users.value[index] = updatedUser
      }
      return updatedUser
    } catch (err) {
      error.value = 'Failed to update user'
      throw err
    }
  }
  
  const deleteUser = async (id: string) => {
    try {
      await userService.deleteUser(id)
      users.value = users.value.filter(user => user.id !== id)
    } catch (err) {
      error.value = 'Failed to delete user'
      throw err
    }
  }
  
  const setCurrentUser = (user: User | null) => {
    currentUser.value = user
  }
  
  const clearError = () => {
    error.value = null
  }
  
  const resetStore = () => {
    users.value = []
    currentUser.value = null
    loading.value = false
    error.value = null
  }
  
  return {
    // State
    users,
    currentUser,
    loading,
    error,
    // Getters
    userCount,
    isAuthenticated,
    activeUsers,
    // Actions
    fetchUsers,
    createUser,
    updateUser,
    deleteUser,
    setCurrentUser,
    clearError,
    resetStore,
  }
})
```

### Using Stores in Components

```vue
<script setup lang="ts">
import { onMounted } from 'vue'
import { useUserStore } from '@/stores/userStore'
import { storeToRefs } from 'pinia'

const userStore = useUserStore()

// Destructure reactive state and getters
const { users, loading, error, userCount } = storeToRefs(userStore)

// Destructure actions (no need for storeToRefs)
const { fetchUsers, createUser, clearError } = userStore

onMounted(() => {
  fetchUsers()
})

const handleCreateUser = async (userData: CreateUserRequest) => {
  try {
    await createUser(userData)
    // Success handling
  } catch (error) {
    // Error is already handled in store
    console.error('Failed to create user')
  }
}
</script>

<template>
  <div>
    <div v-if="loading">Loading users...</div>
    <div v-else-if="error" class="error">
      {{ error }}
      <button @click="clearError">Dismiss</button>
    </div>
    <div v-else>
      <p>Total users: {{ userCount }}</p>
      <UserList :users="users" @create="handleCreateUser" />
    </div>
  </div>
</template>
```

### Store Composition and Modularity

```typescript
// src/stores/authStore.ts
export const useAuthStore = defineStore('auth', () => {
  // Use other stores
  const userStore = useUserStore()
  
  const login = async (credentials: LoginCredentials) => {
    try {
      const user = await authService.login(credentials)
      userStore.setCurrentUser(user)
      localStorage.setItem('authToken', user.token)
    } catch (error) {
      throw new Error('Login failed')
    }
  }
  
  const logout = () => {
    userStore.setCurrentUser(null)
    localStorage.removeItem('authToken')
  }
  
  return { login, logout }
})
```

### Store Persistence

```typescript
// src/stores/settingsStore.ts
import { defineStore } from 'pinia'
import { ref, watch } from 'vue'

export const useSettingsStore = defineStore('settings', () => {
  const theme = ref<'light' | 'dark'>('light')
  const language = ref('en')
  
  // Load from localStorage on initialization
  const loadSettings = () => {
    const savedTheme = localStorage.getItem('theme')
    const savedLanguage = localStorage.getItem('language')
    
    if (savedTheme) theme.value = savedTheme as 'light' | 'dark'
    if (savedLanguage) language.value = savedLanguage
  }
  
  // Watch for changes and save to localStorage
  watch(theme, (newTheme) => {
    localStorage.setItem('theme', newTheme)
  })
  
  watch(language, (newLanguage) => {
    localStorage.setItem('language', newLanguage)
  })
  
  return {
    theme,
    language,
    loadSettings,
  }
})
```

## Resource
Refer to the Pinia documentation for detailed guidance on creating and using stores:
- [Pinia Documentation](https://pinia.vuejs.org/)
- [Pinia Best Practices](https://pinia.vuejs.org/cookbook/)

{% else %}
## Vuex Store Development
**Important**: Only create Vuex stores when application state needs to be managed across components.

### Store Responsibility
1. API calls and data transformation
2. Business logic and validation
3. Cross-component state management  
4. Error handling and user feedback
5. Keep stores modular and focused

### Store Module Structure with TypeScript

```typescript
// src/store/modules/user/types.ts
export interface User {
  id: string
  name: string
  email: string
  isActive: boolean
}

export interface UserState {
  users: User[]
  currentUser: User | null
  loading: boolean
  error: string | null
}

export interface CreateUserRequest {
  name: string
  email: string
  password: string
}
```

```typescript
// src/store/modules/user/mutations.ts
import type { MutationTree } from 'vuex'
import type { UserState, User } from './types'

export enum UserMutations {
  SET_USERS = 'SET_USERS',
  ADD_USER = 'ADD_USER',
  UPDATE_USER = 'UPDATE_USER',
  REMOVE_USER = 'REMOVE_USER',
  SET_CURRENT_USER = 'SET_CURRENT_USER',
  SET_LOADING = 'SET_LOADING',
  SET_ERROR = 'SET_ERROR',
  CLEAR_ERROR = 'CLEAR_ERROR',
}

export const mutations: MutationTree<UserState> = {
  [UserMutations.SET_USERS](state, users: User[]) {
    state.users = users
  },
  
  [UserMutations.ADD_USER](state, user: User) {
    state.users.push(user)
  },
  
  [UserMutations.UPDATE_USER](state, updatedUser: User) {
    const index = state.users.findIndex(user => user.id === updatedUser.id)
    if (index !== -1) {
      state.users.splice(index, 1, updatedUser)
    }
  },
  
  [UserMutations.REMOVE_USER](state, userId: string) {
    state.users = state.users.filter(user => user.id !== userId)
  },
  
  [UserMutations.SET_CURRENT_USER](state, user: User | null) {
    state.currentUser = user
  },
  
  [UserMutations.SET_LOADING](state, loading: boolean) {
    state.loading = loading
  },
  
  [UserMutations.SET_ERROR](state, error: string | null) {
    state.error = error
  },
  
  [UserMutations.CLEAR_ERROR](state) {
    state.error = null
  },
}
```

```typescript
// src/store/modules/user/actions.ts
import type { ActionTree } from 'vuex'
import type { UserState, CreateUserRequest } from './types'
import type { RootState } from '../../types'
import { UserMutations } from './mutations'
import { userService } from '@/services/user/userService'

export enum UserActions {
  FETCH_USERS = 'FETCH_USERS',
  CREATE_USER = 'CREATE_USER',
  UPDATE_USER = 'UPDATE_USER',
  DELETE_USER = 'DELETE_USER',
}

export const actions: ActionTree<UserState, RootState> = {
  async [UserActions.FETCH_USERS]({ commit }) {
    try {
      commit(UserMutations.SET_LOADING, true)
      commit(UserMutations.SET_ERROR, null)
      
      const users = await userService.getUsers()
      commit(UserMutations.SET_USERS, users)
    } catch (error) {
      commit(UserMutations.SET_ERROR, 'Failed to fetch users')
      console.error('Error fetching users:', error)
    } finally {
      commit(UserMutations.SET_LOADING, false)
    }
  },
  
  async [UserActions.CREATE_USER]({ commit }, userData: CreateUserRequest) {
    try {
      commit(UserMutations.SET_LOADING, true)
      
      const newUser = await userService.createUser(userData)
      commit(UserMutations.ADD_USER, newUser)
      
      return newUser
    } catch (error) {
      commit(UserMutations.SET_ERROR, 'Failed to create user')
      throw error
    } finally {
      commit(UserMutations.SET_LOADING, false)
    }
  },
  
  async [UserActions.UPDATE_USER]({ commit }, { id, userData }: { id: string; userData: Partial<User> }) {
    try {
      const updatedUser = await userService.updateUser(id, userData)
      commit(UserMutations.UPDATE_USER, updatedUser)
      return updatedUser
    } catch (error) {
      commit(UserMutations.SET_ERROR, 'Failed to update user')
      throw error
    }
  },
  
  async [UserActions.DELETE_USER]({ commit }, userId: string) {
    try {
      await userService.deleteUser(userId)
      commit(UserMutations.REMOVE_USER, userId)
    } catch (error) {
      commit(UserMutations.SET_ERROR, 'Failed to delete user')
      throw error
    }
  },
}
```

```typescript
// src/store/modules/user/getters.ts
import type { GetterTree } from 'vuex'
import type { UserState } from './types'
import type { RootState } from '../../types'

export const getters: GetterTree<UserState, RootState> = {
  userCount: (state) => state.users.length,
  
  isAuthenticated: (state) => state.currentUser !== null,
  
  activeUsers: (state) => state.users.filter(user => user.isActive),
  
  getUserById: (state) => (id: string) => 
    state.users.find(user => user.id === id),
}
```

```typescript
// src/store/modules/user/index.ts
import type { Module } from 'vuex'
import type { RootState } from '../../types'
import type { UserState } from './types'
import { mutations } from './mutations'
import { actions } from './actions'
import { getters } from './getters'

const state: UserState = {
  users: [],
  currentUser: null,
  loading: false,
  error: null,
}

export const userModule: Module<UserState, RootState> = {
  namespaced: true,
  state,
  mutations,
  actions,
  getters,
}
```

### Using Vuex in Components

```vue
<script setup lang="ts">
import { computed, onMounted } from 'vue'
import { useStore } from 'vuex'
import { UserActions } from '@/store/modules/user/actions'
import type { CreateUserRequest } from '@/store/modules/user/types'

const store = useStore()

// Computed properties for state and getters
const users = computed(() => store.state.user.users)
const loading = computed(() => store.state.user.loading)
const error = computed(() => store.state.user.error)
const userCount = computed(() => store.getters['user/userCount'])

onMounted(() => {
  store.dispatch(`user/${UserActions.FETCH_USERS}`)
})

const handleCreateUser = async (userData: CreateUserRequest) => {
  try {
    await store.dispatch(`user/${UserActions.CREATE_USER}`, userData)
    // Success handling
  } catch (error) {
    console.error('Failed to create user')
  }
}

const clearError = () => {
  store.commit('user/CLEAR_ERROR')
}
</script>

<template>
  <div>
    <div v-if="loading">Loading users...</div>
    <div v-else-if="error" class="error">
      {{ error }}
      <button @click="clearError">Dismiss</button>
    </div>
    <div v-else>
      <p>Total users: {{ userCount }}</p>
      <UserList :users="users" @create="handleCreateUser" />
    </div>
  </div>
</template>
```

### Store Root Configuration

```typescript
// src/store/index.ts
import { createStore } from 'vuex'
import { userModule } from './modules/user'
import type { RootState } from './types'

export default createStore<RootState>({
  modules: {
    user: userModule,
  },
  strict: process.env.NODE_ENV !== 'production',
})
```

## Resources
Refer to the Vuex documentation for detailed guidance on using Vuex in Vue 3:
- [Vuex Documentation](https://vuex.vuejs.org/)
- [Vuex with TypeScript](https://vuex.vuejs.org/guide/typescript-support.html)
{% endif %}
