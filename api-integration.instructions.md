---
applyTo: "**/*.{ts,js,vue}"
---

This project uses `Axios` for API integration and TypeScript interfaces for request and response models.

## API Integration Workflow

1. Use the central [api service](../../src/services/api/index.ts) file to define API endpoints and methods.
2. Each feature has its own service under [services folder](../../src/services) for managing API calls.
   - For example, `services/user/userService.ts` for user-related API calls.
   - Each service should implement methods to fetch data from the API and handle responses.
   - Use the repository pattern for data access when needed.
3. Define TypeScript interfaces for request and response models in [types folder](../../src/types).

## API Service Structure

### Base API Configuration
```typescript
// src/services/http/index.ts
import axios from 'axios'

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
})

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    // Add auth token if available
    const token = localStorage.getItem('authToken')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => Promise.reject(error)
)

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    // Handle common errors
    if (error.response?.status === 401) {
      // Handle unauthorized access
      localStorage.removeItem('authToken')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default apiClient
```

### Feature-Specific Service
```typescript
// src/services/user/userService.ts
import apiClient from '../http'
import type { User, CreateUserRequest, UpdateUserRequest } from '@/types/user'

export class UserService {
  async getUsers(): Promise<User[]> {
    const response = await apiClient.get<User[]>('/users')
    return response.data
  }

  async getUserById(id: string): Promise<User> {
    const response = await apiClient.get<User>(`/users/${id}`)
    return response.data
  }

  async createUser(userData: CreateUserRequest): Promise<User> {
    const response = await apiClient.post<User>('/users', userData)
    return response.data
  }

  async updateUser(id: string, userData: UpdateUserRequest): Promise<User> {
    const response = await apiClient.put<User>(`/users/${id}`, userData)
    return response.data
  }

  async deleteUser(id: string): Promise<void> {
    await apiClient.delete(`/users/${id}`)
  }
}

export const userService = new UserService()
```

### Type Definitions
```typescript
// src/types/user.ts
export interface User {
  id: string
  name: string
  email: string
  createdAt: string
  updatedAt: string
}

export interface CreateUserRequest {
  name: string
  email: string
  password: string
}

export interface UpdateUserRequest {
  name?: string
  email?: string
}

// Generic API response wrapper
export interface ApiResponse<T> {
  data: T
  message: string
  success: boolean
}

// Paginated response
export interface PaginatedResponse<T> {
  data: T[]
  meta: {
    currentPage: number
    totalPages: number
    totalCount: number
    hasNextPage: boolean
    hasPreviousPage: boolean
  }
}
```

### Using Services in Components
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { userService } from '@/services/user/userService'
import type { User } from '@/types/user'

const users = ref<User[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

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

onMounted(() => {
  fetchUsers()
})
</script>
```

## Error Handling Best Practices

1. Create a centralized error handling utility:
```typescript
// src/utils/errorHandler.ts
export class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public code?: string
  ) {
    super(message)
    this.name = 'ApiError'
  }
}

export const handleApiError = (error: any): string => {
  if (error.response) {
    // Server responded with error status
    return error.response.data?.message || 'An error occurred'
  } else if (error.request) {
    // Request was made but no response received
    return 'Network error. Please check your connection.'
  } else {
    // Something else happened
    return error.message || 'An unexpected error occurred'
  }
}
```

2. Use try-catch blocks in all async operations
3. Implement loading states for better UX
4. Show appropriate error messages to users
5. Log errors for debugging purposes

## Resources
Refer to the following documentation for more details on the libraries used:
- [Axios Documentation](https://axios-http.com/)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [Vue 3 Composition API](https://vuejs.org/guide/essentials/reactivity-fundamentals.html)
