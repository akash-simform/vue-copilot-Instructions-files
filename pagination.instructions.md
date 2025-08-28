---
applyTo: "**/*.{ts,js,vue}"
---

## Pagination Support in Vue.js
Always implement efficient pagination for large datasets to improve performance and user experience.

### Pagination Implementation Approaches

1. **Client-side Pagination**: For smaller datasets that can be loaded entirely
2. **Server-side Pagination**: For large datasets with API pagination support
3. **Infinite Scroll**: For social media-like continuous loading experience

### 1. Server-side Pagination with Composable

Create a reusable pagination composable:

```typescript
// src/composables/usePagination.ts
import { ref, computed, reactive } from 'vue'

export interface PaginationOptions {
  page: number
  limit: number
  total: number
}

export interface PaginationMeta {
  currentPage: number
  totalPages: number
  totalItems: number
  hasNextPage: boolean
  hasPreviousPage: boolean
}

export function usePagination<T>(
  fetchFunction: (page: number, limit: number) => Promise<{ data: T[]; meta: PaginationMeta }>
) {
  const items = ref<T[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)
  
  const pagination = reactive({
    currentPage: 1,
    limit: 10,
    totalItems: 0,
    totalPages: 0,
  })
  
  const meta = computed((): PaginationMeta => ({
    currentPage: pagination.currentPage,
    totalPages: pagination.totalPages,
    totalItems: pagination.totalItems,
    hasNextPage: pagination.currentPage < pagination.totalPages,
    hasPreviousPage: pagination.currentPage > 1,
  }))
  
  const fetchData = async (page = 1) => {
    try {
      loading.value = true
      error.value = null
      
      const response = await fetchFunction(page, pagination.limit)
      
      items.value = response.data
      pagination.currentPage = page
      pagination.totalItems = response.meta.totalItems
      pagination.totalPages = response.meta.totalPages
    } catch (err) {
      error.value = 'Failed to fetch data'
      console.error('Pagination fetch error:', err)
    } finally {
      loading.value = false
    }
  }
  
  const goToPage = (page: number) => {
    if (page >= 1 && page <= pagination.totalPages) {
      fetchData(page)
    }
  }
  
  const nextPage = () => {
    if (meta.value.hasNextPage) {
      fetchData(pagination.currentPage + 1)
    }
  }
  
  const previousPage = () => {
    if (meta.value.hasPreviousPage) {
      fetchData(pagination.currentPage - 1)
    }
  }
  
  const refresh = () => {
    fetchData(pagination.currentPage)
  }
  
  return {
    items,
    loading,
    error,
    pagination: readonly(pagination),
    meta,
    fetchData,
    goToPage,
    nextPage,
    previousPage,
    refresh,
  }
}
```

### 2. Pagination Component

Create a reusable pagination component:

```vue
<!-- src/components/common/BasePagination.vue -->
<template>
  <nav class="pagination" aria-label="Pagination Navigation">
    <button
      class="pagination__btn pagination__btn--prev"
      :disabled="!meta.hasPreviousPage"
      @click="$emit('previous')"
    >
      Previous
    </button>
    
    <div class="pagination__pages">
      <button
        v-for="page in visiblePages"
        :key="page"
        class="pagination__page"
        :class="{ 'pagination__page--active': page === meta.currentPage }"
        :aria-current="page === meta.currentPage ? 'page' : undefined"
        @click="$emit('goToPage', page)"
      >
        {{ page }}
      </button>
    </div>
    
    <button
      class="pagination__btn pagination__btn--next"
      :disabled="!meta.hasNextPage"
      @click="$emit('next')"
    >
      Next
    </button>
    
    <div class="pagination__info">
      Showing {{ startItem }} to {{ endItem }} of {{ meta.totalItems }} results
    </div>
  </nav>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { PaginationMeta } from '@/composables/usePagination'

interface Props {
  meta: PaginationMeta
  maxVisiblePages?: number
}

const props = withDefaults(defineProps<Props>(), {
  maxVisiblePages: 5,
})

interface Emits {
  goToPage: [page: number]
  next: []
  previous: []
}

defineEmits<Emits>()

const visiblePages = computed(() => {
  const { currentPage, totalPages } = props.meta
  const maxVisible = props.maxVisiblePages
  
  if (totalPages <= maxVisible) {
    return Array.from({ length: totalPages }, (_, i) => i + 1)
  }
  
  const halfVisible = Math.floor(maxVisible / 2)
  let start = Math.max(1, currentPage - halfVisible)
  let end = Math.min(totalPages, start + maxVisible - 1)
  
  if (end - start + 1 < maxVisible) {
    start = Math.max(1, end - maxVisible + 1)
  }
  
  return Array.from({ length: end - start + 1 }, (_, i) => start + i)
})

const startItem = computed(() => {
  const { currentPage, totalItems } = props.meta
  const itemsPerPage = Math.ceil(totalItems / props.meta.totalPages)
  return Math.min((currentPage - 1) * itemsPerPage + 1, totalItems)
})

const endItem = computed(() => {
  const { currentPage, totalItems } = props.meta
  const itemsPerPage = Math.ceil(totalItems / props.meta.totalPages)
  return Math.min(currentPage * itemsPerPage, totalItems)
})
</script>

<style scoped lang="scss">
.pagination {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin: 2rem 0;
  
  &__btn {
    padding: 0.5rem 1rem;
    border: 1px solid var(--color-border);
    background: var(--color-background);
    color: var(--color-text);
    border-radius: 0.25rem;
    cursor: pointer;
    transition: all 0.2s ease;
    
    &:hover:not(:disabled) {
      background: var(--color-primary);
      color: white;
    }
    
    &:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
  }
  
  &__pages {
    display: flex;
    gap: 0.25rem;
  }
  
  &__page {
    width: 2.5rem;
    height: 2.5rem;
    border: 1px solid var(--color-border);
    background: var(--color-background);
    color: var(--color-text);
    border-radius: 0.25rem;
    cursor: pointer;
    transition: all 0.2s ease;
    
    &:hover {
      background: var(--color-primary-light);
    }
    
    &--active {
      background: var(--color-primary);
      color: white;
      border-color: var(--color-primary);
    }
  }
  
  &__info {
    margin-left: auto;
    font-size: 0.875rem;
    color: var(--color-text-secondary);
  }
}
</style>
```

### 3. Using Pagination in Components

```vue
<!-- src/views/UserList.vue -->
<template>
  <div class="user-list">
    <h1>Users</h1>
    
    <!-- Loading State -->
    <div v-if="loading" class="loading">
      Loading users...
    </div>
    
    <!-- Error State -->
    <div v-else-if="error" class="error">
      {{ error }}
      <button @click="refresh">Retry</button>
    </div>
    
    <!-- User List -->
    <div v-else class="users">
      <UserCard
        v-for="user in items"
        :key="user.id"
        :user="user"
      />
    </div>
    
    <!-- Pagination -->
    <BasePagination
      v-if="!loading && !error && meta.totalPages > 1"
      :meta="meta"
      @go-to-page="goToPage"
      @next="nextPage"
      @previous="previousPage"
    />
  </div>
</template>

<script setup lang="ts">
import { onMounted } from 'vue'
import { usePagination } from '@/composables/usePagination'
import { userService } from '@/services/user/userService'
import UserCard from '@/components/user/UserCard.vue'
import BasePagination from '@/components/common/BasePagination.vue'
import type { User } from '@/types/user'

const {
  items,
  loading,
  error,
  meta,
  fetchData,
  goToPage,
  nextPage,
  previousPage,
  refresh,
} = usePagination<User>(async (page, limit) => {
  return await userService.getUsers({ page, limit })
})

onMounted(() => {
  fetchData()
})
</script>
```

### 4. Infinite Scroll Implementation

For infinite scroll pagination:

```typescript
// src/composables/useInfiniteScroll.ts
import { ref, onMounted, onUnmounted } from 'vue'

export function useInfiniteScroll<T>(
  fetchFunction: (page: number) => Promise<{ data: T[]; hasNextPage: boolean }>
) {
  const items = ref<T[]>([])
  const loading = ref(false)
  const hasNextPage = ref(true)
  const currentPage = ref(0)
  
  const loadMore = async () => {
    if (loading.value || !hasNextPage.value) return
    
    try {
      loading.value = true
      currentPage.value++
      
      const response = await fetchFunction(currentPage.value)
      items.value.push(...response.data)
      hasNextPage.value = response.hasNextPage
    } catch (error) {
      currentPage.value--
      console.error('Failed to load more items:', error)
    } finally {
      loading.value = false
    }
  }
  
  const handleScroll = () => {
    const { scrollTop, scrollHeight, clientHeight } = document.documentElement
    const threshold = 100 // Load more when 100px from bottom
    
    if (scrollTop + clientHeight >= scrollHeight - threshold) {
      loadMore()
    }
  }
  
  onMounted(() => {
    window.addEventListener('scroll', handleScroll)
    loadMore() // Load first page
  })
  
  onUnmounted(() => {
    window.removeEventListener('scroll', handleScroll)
  })
  
  return {
    items,
    loading,
    hasNextPage,
    loadMore,
  }
}
```

### API Service for Pagination

```typescript
// src/services/user/userService.ts
export class UserService {
  async getUsers(params: {
    page: number
    limit: number
    search?: string
    sortBy?: string
    sortOrder?: 'asc' | 'desc'
  }): Promise<{
    data: User[]
    meta: {
      currentPage: number
      totalPages: number
      totalItems: number
      hasNextPage: boolean
      hasPreviousPage: boolean
    }
  }> {
    const response = await apiClient.get<{
      users: User[]
      pagination: {
        page: number
        totalPages: number
        total: number
        hasNext: boolean
        hasPrev: boolean
      }
    }>('/users', { params })
    
    return {
      data: response.data.users,
      meta: {
        currentPage: response.data.pagination.page,
        totalPages: response.data.pagination.totalPages,
        totalItems: response.data.pagination.total,
        hasNextPage: response.data.pagination.hasNext,
        hasPreviousPage: response.data.pagination.hasPrev,
      },
    }
  }
}
```

### Performance Considerations

1. **Virtual Scrolling**: For very large lists, implement virtual scrolling
2. **Debounced Search**: Debounce search inputs to avoid excessive API calls
3. **Caching**: Cache paginated results when appropriate
4. **Loading States**: Always show loading indicators for better UX
5. **Error Handling**: Implement retry mechanisms for failed requests

### Resources
Refer to the following resources for more information on pagination in Vue.js:
- [Vue 3 Composition API](https://vuejs.org/guide/essentials/reactivity-fundamentals.html)
- [Vue Router](https://router.vuejs.org/) for URL-based pagination
- [VueUse](https://vueuse.org/) for additional composables
