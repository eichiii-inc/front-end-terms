# Vue.js コーディング規約 2025

**Vue 3 Composition API時代の包括的ガイドライン**

## 🎯 基本方針

### Vue 3 Composition API First
- `<script setup>`構文を標準とする（必須）
- Options APIは既存コードのメンテナンスのみ
- 新規開発は全てComposition APIで実装
- Vapor Mode対応を意識した実装

### TypeScript標準
- 全てのVueファイルでTypeScriptを使用
- PropとEmitの型定義を必須とする
- `defineProps`と`defineEmits`でランタイム型チェック
- Generic Components対応

### パフォーマンス重視
- 仮想DOM最適化とVapor Mode準備
- Composition APIによる効率的なリアクティビティ
- Tree-shaking対応の実装
- Bundle Size最適化

## 📁 ファイル・ディレクトリ規約

### ファイル命名
```
✅ 良い例
components/
├─ UserProfile.vue          // PascalCase
├─ BaseButton.vue           // Base接頭辞
├─ AppHeader.vue           // App接頭辞
├─ TheNavigation.vue       // 単一インスタンス

❌ 悪い例
components/
├─ userProfile.vue         // camelCase
├─ user-profile.vue        // kebab-case
├─ button.vue             // 汎用的すぎる
```

### ディレクトリ構造（推奨）
```
src/
├─ components/
│  ├─ base/              // 基本コンポーネント
│  ├─ common/            // 共通コンポーネント
│  └─ pages/             // ページ固有コンポーネント
├─ composables/          // 再利用可能なロジック
├─ stores/               // Pinia stores
├─ types/                // TypeScript型定義
├─ utils/                // ユーティリティ関数
└─ assets/               // 静的アセット
```

## 🧩 コンポーネント設計

### `<script setup>`構文（必須）

```vue
<script setup lang="ts">
// ✅ 推奨パターン
import { ref, computed, onMounted } from 'vue'
import type { User } from '@/types/user'

// Props定義（TypeScript型付き）
interface Props {
  user: User
  isLoading?: boolean
  maxItems?: number
}

const props = withDefaults(defineProps<Props>(), {
  isLoading: false,
  maxItems: 10
})

// Emits定義
interface Emits {
  update: [user: User]
  delete: [userId: string]
  'selection-change': [selectedIds: string[]]
}

const emit = defineEmits<Emits>()

// Reactive state
const count = ref(0)
const selectedItems = ref<string[]>([])

// Computed properties
const displayName = computed(() => 
  props.user.firstName + ' ' + props.user.lastName
)

const hasSelection = computed(() => 
  selectedItems.value.length > 0
)

// Methods
const handleUpdate = () => {
  emit('update', { ...props.user, lastModified: Date.now() })
}

const handleSelection = (ids: string[]) => {
  selectedItems.value = ids
  emit('selection-change', ids)
}

// Lifecycle
onMounted(() => {
  console.log('Component mounted')
})

// Watchers
watch(() => props.user, (newUser, oldUser) => {
  console.log('User changed:', { newUser, oldUser })
}, { deep: true })

// Expose（親コンポーネントからアクセス可能にする）
defineExpose({
  resetSelection: () => {
    selectedItems.value = []
  }
})
</script>

<template>
  <div class="user-card">
    <h2>{{ displayName }}</h2>
    <div v-if="isLoading" class="loading">
      読み込み中...
    </div>
    <div v-else class="content">
      <!-- コンテンツ -->
    </div>
  </div>
</template>

<style scoped>
.user-card {
  @apply p-4 border rounded-lg;
}

.loading {
  @apply text-gray-500 animate-pulse;
}
</style>
```

### Composables（カスタムフック）

```typescript
// composables/useCounter.ts
import { ref, computed, readonly } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  
  const increment = () => {
    count.value++
  }
  
  const decrement = () => {
    count.value--
  }
  
  const reset = () => {
    count.value = initialValue
  }
  
  const isEven = computed(() => count.value % 2 === 0)
  const isPositive = computed(() => count.value > 0)
  
  // 外部から直接変更できないように readonly で返す
  return {
    count: readonly(count),
    increment,
    decrement,
    reset,
    isEven,
    isPositive
  }
}

// 使用例
<script setup lang="ts">
import { useCounter } from '@/composables/useCounter'

const { count, increment, decrement, reset, isEven } = useCounter(0)
</script>
```

### 非同期データ取得

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import type { User } from '@/types/user'

interface Props {
  userId: string
}

const props = defineProps<Props>()

// 状態管理
const user = ref<User | null>(null)
const loading = ref(false)
const error = ref<string | null>(null)

// データ取得関数
const fetchUser = async () => {
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch(`/api/users/${props.userId}`)
    if (!response.ok) {
      throw new Error('ユーザーの取得に失敗しました')
    }
    user.value = await response.json()
  } catch (err) {
    error.value = err instanceof Error ? err.message : '未知のエラー'
  } finally {
    loading.value = false
  }
}

// 初期化
onMounted(fetchUser)

// プロップスの変更を監視
watch(() => props.userId, fetchUser)
</script>

<template>
  <div class="user-loader">
    <div v-if="loading" class="loading">
      読み込み中...
    </div>
    <div v-else-if="error" class="error">
      エラー: {{ error }}
      <button @click="fetchUser">再試行</button>
    </div>
    <div v-else-if="user" class="user-content">
      <h1>{{ user.name }}</h1>
      <p>{{ user.email }}</p>
    </div>
  </div>
</template>
```

## 🏪 状態管理（Pinia）

### Store定義

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { User } from '@/types/user'

export const useUserStore = defineStore('user', () => {
  // State
  const currentUser = ref<User | null>(null)
  const users = ref<User[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)
  
  // Getters
  const isAuthenticated = computed(() => currentUser.value !== null)
  const userCount = computed(() => users.value.length)
  const activeUsers = computed(() => 
    users.value.filter(user => user.isActive)
  )
  
  // Actions
  const login = async (credentials: LoginCredentials) => {
    loading.value = true
    error.value = null
    
    try {
      const response = await authAPI.login(credentials)
      currentUser.value = response.user
      // トークンをローカルストレージに保存
      localStorage.setItem('authToken', response.token)
    } catch (err) {
      error.value = 'ログインに失敗しました'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  const logout = () => {
    currentUser.value = null
    localStorage.removeItem('authToken')
  }
  
  const fetchUsers = async () => {
    loading.value = true
    try {
      users.value = await userAPI.getUsers()
    } catch (err) {
      error.value = 'ユーザー一覧の取得に失敗しました'
    } finally {
      loading.value = false
    }
  }
  
  const updateUser = async (id: string, updates: Partial<User>) => {
    try {
      const updatedUser = await userAPI.updateUser(id, updates)
      const index = users.value.findIndex(u => u.id === id)
      if (index !== -1) {
        users.value[index] = updatedUser
      }
      if (currentUser.value?.id === id) {
        currentUser.value = updatedUser
      }
    } catch (err) {
      error.value = 'ユーザーの更新に失敗しました'
      throw err
    }
  }
  
  return {
    // State
    currentUser,
    users,
    loading,
    error,
    // Getters
    isAuthenticated,
    userCount,
    activeUsers,
    // Actions
    login,
    logout,
    fetchUsers,
    updateUser
  }
})

// 使用例
<script setup lang="ts">
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

const handleLogin = async (credentials: LoginCredentials) => {
  try {
    await userStore.login(credentials)
    // ログイン成功時の処理
  } catch (error) {
    // エラーハンドリング
  }
}
</script>
```

## 🎨 スタイリング規約

### Tailwind CSS推奨

```vue
<template>
  <div class="user-card">
    <!-- ✅ Tailwindクラスを使用 -->
    <div class="p-6 bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow">
      <h2 class="text-xl font-semibold text-gray-900 mb-2">
        {{ user.name }}
      </h2>
      <p class="text-gray-600 text-sm">
        {{ user.email }}
      </p>
      <button 
        class="mt-4 px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 
               disabled:opacity-50 disabled:cursor-not-allowed"
        :disabled="loading"
        @click="handleAction"
      >
        {{ loading ? '処理中...' : 'アクション' }}
      </button>
    </div>
  </div>
</template>

<style scoped>
/* カスタムスタイルは必要最小限に */
.user-card {
  /* Tailwindで表現できない複雑なスタイルのみ */
  container-type: inline-size;
}

/* ダークモード対応 */
@media (prefers-color-scheme: dark) {
  .user-card {
    @apply bg-gray-800 text-white;
  }
}
</style>
```

### CSS変数とカスタムプロパティ

```vue
<style scoped>
.component {
  /* CSS変数を活用 */
  --primary-color: theme('colors.blue.500');
  --border-radius: theme('borderRadius.lg');
  --spacing: theme('spacing.4');
  
  background-color: var(--primary-color);
  border-radius: var(--border-radius);
  padding: var(--spacing);
}

/* レスポンシブデザイン */
@container (min-width: 768px) {
  .component {
    --spacing: theme('spacing.6');
  }
}
</style>
```

## 🎯 ディレクティブ規約

### v-forとkey

```vue
<template>
  <!-- ✅ 必ずkeyを指定 -->
  <div v-for="user in users" :key="user.id" class="user-item">
    {{ user.name }}
  </div>
  
  <!-- ✅ インデックスと値の両方が必要な場合 -->
  <div 
    v-for="(user, index) in users" 
    :key="user.id"
    :class="{ 'bg-gray-100': index % 2 === 0 }"
  >
    {{ index + 1 }}. {{ user.name }}
  </div>
  
  <!-- ❌ インデックスをkeyに使用は避ける -->
  <div v-for="(user, index) in users" :key="index">
    {{ user.name }}
  </div>
</template>
```

### v-ifとv-show

```vue
<template>
  <!-- ✅ 条件が変わりにくい場合はv-if -->
  <div v-if="user.isAdmin" class="admin-panel">
    管理者パネル
  </div>
  
  <!-- ✅ 頻繁に切り替わる場合はv-show -->
  <div v-show="isModalOpen" class="modal">
    モーダルコンテンツ
  </div>
  
  <!-- ✅ v-ifとv-forは同じ要素で使用しない -->
  <template v-for="user in users" :key="user.id">
    <div v-if="user.isActive" class="user-item">
      {{ user.name }}
    </div>
  </template>
  
  <!-- ❌ 悪い例 -->
  <div v-for="user in users" v-if="user.isActive" :key="user.id">
    {{ user.name }}
  </div>
</template>
```

### ディレクティブの短縮記法

```vue
<template>
  <!-- ✅ 短縮記法を使用 -->
  <input 
    :value="searchQuery"
    @input="handleInput"
    @keyup.enter="handleSearch"
  />
  
  <!-- ✅ スロットの短縮記法 -->
  <BaseModal>
    <template #header>
      <h2>タイトル</h2>
    </template>
    <template #default>
      <p>コンテンツ</p>
    </template>
  </BaseModal>
  
  <!-- ❌ 短縮記法を使わない -->
  <input 
    v-bind:value="searchQuery"
    v-on:input="handleInput"
  />
</template>
```

## 🧪 テスト

### コンポーネントテスト（Vitest + Vue Test Utils）

```typescript
// UserCard.test.ts
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import UserCard from '@/components/UserCard.vue'
import type { User } from '@/types/user'

const mockUser: User = {
  id: '1',
  name: 'テストユーザー',
  email: 'test@example.com',
  isActive: true
}

describe('UserCard', () => {
  it('ユーザー情報を正しく表示する', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser }
    })
    
    expect(wrapper.text()).toContain(mockUser.name)
    expect(wrapper.text()).toContain(mockUser.email)
  })
  
  it('更新ボタンクリック時にイベントを発行する', async () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser }
    })
    
    await wrapper.find('[data-testid="update-button"]').trigger('click')
    
    expect(wrapper.emitted()).toHaveProperty('update')
    expect(wrapper.emitted('update')?.[0]).toEqual([mockUser])
  })
  
  it('ローディング中は適切な表示をする', () => {
    const wrapper = mount(UserCard, {
      props: { 
        user: mockUser,
        isLoading: true 
      }
    })
    
    expect(wrapper.find('[data-testid="loading"]').exists()).toBe(true)
    expect(wrapper.find('[data-testid="update-button"]').attributes('disabled')).toBeDefined()
  })
})
```

### Composablesテスト

```typescript
// useCounter.test.ts
import { describe, it, expect } from 'vitest'
import { useCounter } from '@/composables/useCounter'

describe('useCounter', () => {
  it('初期値を正しく設定する', () => {
    const { count } = useCounter(5)
    expect(count.value).toBe(5)
  })
  
  it('increment関数が正しく動作する', () => {
    const { count, increment } = useCounter(0)
    increment()
    expect(count.value).toBe(1)
  })
  
  it('isEvenが正しく計算される', () => {
    const { count, increment, isEven } = useCounter(0)
    expect(isEven.value).toBe(true)
    
    increment()
    expect(isEven.value).toBe(false)
  })
})
```

## ♿ アクセシビリティ

### セマンティックHTML

```vue
<template>
  <article class="user-profile">
    <!-- ✅ 適切な見出し階層 -->
    <header>
      <h1>{{ user.name }}</h1>
      <p class="subtitle">{{ user.title }}</p>
    </header>
    
    <main>
      <section aria-labelledby="contact-heading">
        <h2 id="contact-heading">連絡先情報</h2>
        <address>
          <a href="mailto:{{ user.email }}">{{ user.email }}</a>
        </address>
      </section>
      
      <section aria-labelledby="bio-heading">
        <h2 id="bio-heading">プロフィール</h2>
        <p>{{ user.bio }}</p>
      </section>
    </main>
    
    <footer>
      <button 
        type="button"
        aria-describedby="edit-description"
        @click="editProfile"
      >
        プロフィールを編集
      </button>
      <div id="edit-description" class="sr-only">
        プロフィール編集画面を開きます
      </div>
    </footer>
  </article>
</template>
```

### ARIA属性

```vue
<template>
  <div class="search-component">
    <!-- フォームラベル -->
    <label for="search-input">ユーザー検索</label>
    <input
      id="search-input"
      v-model="searchQuery"
      type="search"
      aria-describedby="search-help"
      aria-expanded="false"
      aria-owns="search-results"
      autocomplete="off"
      @input="handleSearch"
      @keydown="handleKeydown"
    />
    <div id="search-help" class="help-text">
      名前またはメールアドレスで検索できます
    </div>
    
    <!-- 検索結果 -->
    <ul 
      id="search-results"
      role="listbox"
      aria-label="検索結果"
      :aria-hidden="results.length === 0"
    >
      <li 
        v-for="(user, index) in results"
        :key="user.id"
        role="option"
        :aria-selected="selectedIndex === index"
        :class="{ 'selected': selectedIndex === index }"
        @click="selectUser(user)"
      >
        {{ user.name }}
      </li>
    </ul>
  </div>
</template>
```

## 📱 パフォーマンス最適化

### 遅延読み込み

```vue
<script setup lang="ts">
import { defineAsyncComponent } from 'vue'

// ✅ 重いコンポーネントの遅延読み込み
const HeavyChart = defineAsyncComponent({
  loader: () => import('@/components/HeavyChart.vue'),
  loadingComponent: () => h('div', { class: 'loading' }, '読み込み中...'),
  errorComponent: () => h('div', { class: 'error' }, '読み込みエラー'),
  delay: 200,
  timeout: 3000
})

// ✅ 条件付き遅延読み込み
const AdminPanel = defineAsyncComponent(() => 
  user.value?.isAdmin 
    ? import('@/components/AdminPanel.vue')
    : Promise.resolve(null)
)
</script>
```

### メモ化とキャッシュ

```vue
<script setup lang="ts">
import { computed, shallowRef } from 'vue'

// ✅ 計算量の多い処理をメモ化
const expensiveComputation = computed(() => {
  // 重い計算処理
  return heavyCalculation(props.data)
})

// ✅ 大きなオブジェクトはshallowRefを使用
const largeDataset = shallowRef<LargeData[]>([])

// ✅ watchEffectで最適化
watchEffect(() => {
  if (props.shouldUpdate) {
    updateData()
  }
})
</script>
```

## 🔧 開発ツール設定

### ESLint設定（.eslintrc.js）

```javascript
module.exports = {
  extends: [
    '@vue/eslint-config-typescript',
    '@vue/eslint-config-prettier',
    'plugin:vue/vue3-recommended'
  ],
  rules: {
    // Vue固有ルール
    'vue/component-name-in-template-casing': ['error', 'PascalCase'],
    'vue/define-macros-order': ['error', {
      order: ['defineProps', 'defineEmits']
    }],
    'vue/no-unused-vars': 'error',
    'vue/script-setup-uses-vars': 'error',
    
    // TypeScript
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'warn'
  }
}
```

### Viteプロジェクト設定

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  },
  css: {
    devSourcemap: true
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['vue', 'vue-router', 'pinia'],
          ui: ['@headlessui/vue', '@heroicons/vue']
        }
      }
    }
  }
})
```

---

**このガイドラインは2025年のVue.js開発のベストプラクティスを反映しており、定期的に更新されます。**
