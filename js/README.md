# JavaScript コーディング規約 2025

**ES2023+ モダンJavaScript開発ガイドライン**

## 🎯 基本方針

### TypeScript移行推奨
- **新規プロジェクトはTypeScriptを強く推奨**
- 既存JavaScriptプロジェクトの段階的TypeScript移行
- JSDocによる型注釈の活用（移行期間中）
- AI支援ツールによる自動TypeScript変換

### モダンJavaScript活用
- ES2023+の最新機能を積極的に活用
- ブラウザ対応（ES2022+）とトランスパイル戦略
- Web標準APIの優先使用
- WebAssembly統合対応

### パフォーマンス重視
- Bundle sizeの最適化
- Tree-shakingを意識した実装
- Core Web Vitals（LCP、INP、CLS）の改善
- エッジコンピューティング対応

## 📚 ES2023+ モダン機能

### モジュールシステム

```javascript
// ✅ ES Modules使用（CommonJSは避ける）
import { fetchUserData, createUser } from './api/users.js'
import type { User, CreateUserRequest } from './types/user.js'

// ✅ Dynamic Import（コード分割）
const loadFeature = async () => {
  const { heavyFeature } = await import('./heavy-feature.js')
  return heavyFeature()
}

// ✅ Import Assertions/Attributes
import config from './config.json' with { type: 'json' }
import styles from './styles.css' with { type: 'css' }

// ✅ Re-exports
export { default as Button } from './Button.js'
export type { ButtonProps } from './Button.js'

// ❌ 古い方法（CommonJS）
// const { fetchUserData } = require('./api/users')
```

### 分割代入の高度な活用

```javascript
// ✅ ネストした分割代入
const { 
  user: { profile: { name, avatar } },
  settings: { theme = 'light', language = 'ja' }
} = userData

// ✅ 配列分割代入と残り要素
const [first, second, ...rest] = items
const [, , third] = items // 不要な要素をスキップ

// ✅ 関数引数の分割代入
function createUser({
  name,
  email,
  role = 'user',
  permissions = [],
  ...metadata
}) {
  return { name, email, role, permissions, metadata, id: generateId() }
}

// ✅ 分割代入での名前変更
const { 
  data: userData,
  loading: isUserLoading,
  error: userError 
} = useUser(userId)
```

### テンプレートリテラル

```javascript
// ✅ 基本的なテンプレートリテラル
const message = `Hello, ${user.name}! You have ${count} notifications.`

// ✅ 複数行文字列
const htmlTemplate = `
  <div class="user-card">
    <h2>${user.name}</h2>
    <p>${user.bio}</p>
  </div>
`

// ✅ タグ付きテンプレートリテラル
function html(strings, ...values) {
  return strings.reduce((result, string, i) => {
    const value = values[i] ? escapeHtml(values[i]) : ''
    return result + string + value
  }, '')
}

const safeHtml = html`<p>Hello, ${userInput}!</p>`

// ✅ CSS-in-JS での活用
const styled = (styles) => styles.join('')
const buttonStyles = styled`
  background: ${props => props.primary ? '#007bff' : '#6c757d'};
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
`
```

### スプレッド演算子と残り要素

```javascript
// ✅ オブジェクトのマージと更新
const defaultConfig = { theme: 'light', language: 'en' }
const userConfig = { language: 'ja', notifications: true }
const finalConfig = { ...defaultConfig, ...userConfig }

// ✅ 条件付きプロパティ
const createRequest = (data, options = {}) => ({
  ...data,
  timestamp: Date.now(),
  ...(options.includeMetadata && { metadata: getMetadata() }),
  ...(options.priority && { priority: options.priority })
})

// ✅ 配列の操作
const addItem = (items, newItem) => [...items, newItem]
const removeItem = (items, index) => [
  ...items.slice(0, index),
  ...items.slice(index + 1)
]
const updateItem = (items, index, updates) => [
  ...items.slice(0, index),
  { ...items[index], ...updates },
  ...items.slice(index + 1)
]

// ✅ 関数引数での残り要素
function logMessages(level, ...messages) {
  const timestamp = new Date().toISOString()
  console[level](`[${timestamp}]`, ...messages)
}

logMessages('info', 'User logged in:', user.name, 'at', new Date())
```

### Optional Chaining & Nullish Coalescing

```javascript
// ✅ Optional Chaining
const userName = user?.profile?.name
const firstHobby = user?.hobbies?.[0]
const saveUser = user?.profile?.save?.()

// ✅ Nullish Coalescing
const displayName = user?.displayName ?? user?.name ?? 'Anonymous'
const maxItems = settings?.maxItems ?? 10

// ✅ 条件付き代入
let userSettings = null
userSettings ??= getDefaultSettings()

// ✅ 組み合わせ使用
const notificationCount = user?.notifications?.unread?.length ?? 0
const isAdmin = user?.permissions?.includes?.('admin') ?? false

// ✅ エラーハンドリング
function safeJsonParse(jsonString, fallback = null) {
  try {
    return JSON.parse(jsonString)
  } catch {
    return fallback
  }
}

const config = safeJsonParse(configString) ?? defaultConfig
```

### 非同期処理（async/await）

```javascript
// ✅ 基本的なasync/await
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`)
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    
    return await response.json()
  } catch (error) {
    console.error('Failed to fetch user:', error)
    throw error
  }
}

// ✅ 並列実行
async function loadDashboardData(userId) {
  const [user, posts, notifications] = await Promise.all([
    fetchUser(userId),
    fetchUserPosts(userId),
    fetchNotifications(userId)
  ])
  
  return { user, posts, notifications }
}

// ✅ 逐次処理が必要な場合
async function processItems(items) {
  const results = []
  
  for (const item of items) {
    const processed = await processItem(item)
    results.push(processed)
  }
  
  return results
}

// ✅ Promise.allSettled（一部失敗でも続行）
async function fetchMultipleResources(urls) {
  const results = await Promise.allSettled(
    urls.map(url => fetch(url).then(r => r.json()))
  )
  
  return results
    .filter(result => result.status === 'fulfilled')
    .map(result => result.value)
}

// ✅ タイムアウト付きfetch
async function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController()
  const timeoutId = setTimeout(() => controller.abort(), timeout)
  
  try {
    const response = await fetch(url, { 
      signal: controller.signal 
    })
    clearTimeout(timeoutId)
    return response
  } catch (error) {
    clearTimeout(timeoutId)
    throw error
  }
}
```

### 配列メソッドの高度な活用

```javascript
// ✅ 関数型プログラミングスタイル
const processUsers = (users) =>
  users
    .filter(user => user.isActive)
    .map(user => ({
      ...user,
      displayName: user.displayName || user.name,
      avatar: user.avatar || generateAvatarUrl(user.name)
    }))
    .sort((a, b) => a.displayName.localeCompare(b.displayName))

// ✅ reduce の活用
const groupByRole = (users) =>
  users.reduce((groups, user) => {
    const role = user.role || 'user'
    groups[role] = groups[role] || []
    groups[role].push(user)
    return groups
  }, {})

const sumByCategory = (items) =>
  items.reduce((sums, item) => {
    sums[item.category] = (sums[item.category] || 0) + item.amount
    return sums
  }, {})

// ✅ flatMap の活用
const extractAllTags = (posts) =>
  posts.flatMap(post => post.tags || [])

// ✅ find と findIndex
const findUserByEmail = (users, email) =>
  users.find(user => user.email === email)

const updateUserById = (users, userId, updates) => {
  const index = users.findIndex(user => user.id === userId)
  if (index === -1) return users
  
  return [
    ...users.slice(0, index),
    { ...users[index], ...updates },
    ...users.slice(index + 1)
  ]
}

// ✅ every と some
const allUsersActive = (users) => users.every(user => user.isActive)
const hasAdminUser = (users) => users.some(user => user.role === 'admin')
```

### モダンなクラス構文

```javascript
// ✅ クラスフィールド宣言
class UserManager {
  // プライベートフィールド
  #users = []
  #cache = new Map()
  
  // パブリックフィールド
  maxUsers = 1000
  
  constructor(options = {}) {
    this.maxUsers = options.maxUsers ?? this.maxUsers
  }
  
  // プライベートメソッド
  #validateUser(user) {
    if (!user.email || !user.name) {
      throw new Error('Invalid user data')
    }
  }
  
  // パブリックメソッド
  async addUser(userData) {
    this.#validateUser(userData)
    
    if (this.#users.length >= this.maxUsers) {
      throw new Error('Maximum users reached')
    }
    
    const user = { ...userData, id: generateId(), createdAt: new Date() }
    this.#users.push(user)
    this.#cache.set(user.id, user)
    
    return user
  }
  
  getUser(id) {
    return this.#cache.get(id) ?? null
  }
  
  // Static メソッド
  static createWithDefaults() {
    return new UserManager({ maxUsers: 500 })
  }
}

// ✅ 継承
class AdminUserManager extends UserManager {
  #adminUsers = []
  
  async addAdminUser(userData) {
    const user = await this.addUser({ ...userData, role: 'admin' })
    this.#adminUsers.push(user)
    return user
  }
  
  getAdminUsers() {
    return [...this.#adminUsers]
  }
}
```

## 🚀 パフォーマンス最適化

### Tree-shaking対応

```javascript
// ✅ 名前付きインポート（Tree-shaking有効）
import { debounce, throttle } from 'lodash-es'

// ❌ デフォルトインポート（Tree-shaking無効）
// import _ from 'lodash'

// ✅ 小さなユーティリティ関数は自前実装
const debounce = (func, wait) => {
  let timeoutId
  return (...args) => {
    clearTimeout(timeoutId)
    timeoutId = setTimeout(() => func.apply(this, args), wait)
  }
}

// ✅ 条件付きインポート
const isMobileDevice = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent)

if (isMobileDevice) {
  const { setupMobileFeatures } = await import('./mobile-features.js')
  setupMobileFeatures()
}
```

### メモ化とキャッシュ

```javascript
// ✅ 簡単なメモ化
function memoize(fn) {
  const cache = new Map()
  
  return (...args) => {
    const key = JSON.stringify(args)
    
    if (cache.has(key)) {
      return cache.get(key)
    }
    
    const result = fn(...args)
    cache.set(key, result)
    return result
  }
}

const expensiveCalculation = memoize((n) => {
  // 重い計算処理
  return n * n * n
})

// ✅ WeakMap を使ったオブジェクトキャッシュ
const processedDataCache = new WeakMap()

function processUserData(user) {
  if (processedDataCache.has(user)) {
    return processedDataCache.get(user)
  }
  
  const processed = {
    ...user,
    displayName: user.displayName || user.name,
    initials: getInitials(user.name)
  }
  
  processedDataCache.set(user, processed)
  return processed
}
```

### Web Workers活用

```javascript
// ✅ Web Worker での重い処理
// worker.js
self.onmessage = function(e) {
  const { data, operation } = e.data
  
  let result
  switch (operation) {
    case 'heavyCalculation':
      result = performHeavyCalculation(data)
      break
    case 'dataProcessing':
      result = processLargeDataset(data)
      break
    default:
      result = { error: 'Unknown operation' }
  }
  
  self.postMessage(result)
}

// main.js
async function runHeavyTask(data, operation) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js')
    
    worker.postMessage({ data, operation })
    
    worker.onmessage = (e) => {
      worker.terminate()
      resolve(e.data)
    }
    
    worker.onerror = (error) => {
      worker.terminate()
      reject(error)
    }
  })
}

// 使用例
const result = await runHeavyTask(largeDataset, 'dataProcessing')
```

## 🛡️ エラーハンドリング

### 堅牢なエラー処理

```javascript
// ✅ カスタムエラークラス
class APIError extends Error {
  constructor(message, status, code) {
    super(message)
    this.name = 'APIError'
    this.status = status
    this.code = code
  }
}

class ValidationError extends Error {
  constructor(message, field) {
    super(message)
    this.name = 'ValidationError'
    this.field = field
  }
}

// ✅ エラーハンドリングユーティリティ
async function safeAsync(asyncFn, fallback = null) {
  try {
    return await asyncFn()
  } catch (error) {
    console.error('Async operation failed:', error)
    return fallback
  }
}

// ✅ Result型パターン（関数型アプローチ）
class Result {
  constructor(success, data, error) {
    this.success = success
    this.data = data
    this.error = error
  }
  
  static ok(data) {
    return new Result(true, data, null)
  }
  
  static error(error) {
    return new Result(false, null, error)
  }
  
  map(fn) {
    if (this.success) {
      try {
        return Result.ok(fn(this.data))
      } catch (error) {
        return Result.error(error)
      }
    }
    return this
  }
  
  flatMap(fn) {
    if (this.success) {
      try {
        return fn(this.data)
      } catch (error) {
        return Result.error(error)
      }
    }
    return this
  }
}

async function fetchUserSafe(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`)
    
    if (!response.ok) {
      return Result.error(new APIError(
        `Failed to fetch user: ${response.statusText}`,
        response.status
      ))
    }
    
    const user = await response.json()
    return Result.ok(user)
  } catch (error) {
    return Result.error(error)
  }
}

// 使用例
const userResult = await fetchUserSafe(userId)
const processedResult = userResult
  .map(user => ({ ...user, processed: true }))
  .map(user => validateUser(user))

if (processedResult.success) {
  console.log('User:', processedResult.data)
} else {
  console.error('Error:', processedResult.error)
}
```

### バリデーション

```javascript
// ✅ バリデーション関数
const validators = {
  required: (value, field) => {
    if (value === null || value === undefined || value === '') {
      throw new ValidationError(`${field} is required`, field)
    }
    return value
  },
  
  email: (value, field) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (!emailRegex.test(value)) {
      throw new ValidationError(`${field} must be a valid email`, field)
    }
    return value
  },
  
  minLength: (min) => (value, field) => {
    if (value.length < min) {
      throw new ValidationError(
        `${field} must be at least ${min} characters`,
        field
      )
    }
    return value
  },
  
  maxLength: (max) => (value, field) => {
    if (value.length > max) {
      throw new ValidationError(
        `${field} must be no more than ${max} characters`,
        field
      )
    }
    return value
  }
}

// ✅ バリデーションスキーマ
const userSchema = {
  name: [validators.required, validators.minLength(2), validators.maxLength(50)],
  email: [validators.required, validators.email],
  age: [validators.required, (value, field) => {
    const age = Number(value)
    if (isNaN(age) || age < 0 || age > 120) {
      throw new ValidationError(`${field} must be a valid age`, field)
    }
    return age
  }]
}

function validateUser(userData) {
  const validated = {}
  const errors = []
  
  for (const [field, fieldValidators] of Object.entries(userSchema)) {
    try {
      let value = userData[field]
      
      for (const validator of fieldValidators) {
        value = validator(value, field)
      }
      
      validated[field] = value
    } catch (error) {
      if (error instanceof ValidationError) {
        errors.push(error)
      } else {
        throw error
      }
    }
  }
  
  if (errors.length > 0) {
    const error = new Error('Validation failed')
    error.validationErrors = errors
    throw error
  }
  
  return validated
}
```

## 🔧 開発ツール設定

### ESLint設定

```javascript
// .eslintrc.js
module.exports = {
  env: {
    browser: true,
    es2023: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended'
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2023,
    sourceType: 'module'
  },
  rules: {
    // モダンJavaScript
    'prefer-const': 'error',
    'prefer-arrow-callback': 'error',
    'prefer-template': 'error',
    'prefer-destructuring': 'error',
    'prefer-spread': 'error',
    'prefer-rest-params': 'error',
    'object-shorthand': 'error',
    'arrow-spacing': 'error',
    'template-curly-spacing': 'error',
    
    // 非同期処理
    'prefer-promise-reject-errors': 'error',
    'no-async-promise-executor': 'error',
    'require-atomic-updates': 'error',
    
    // パフォーマンス
    'no-loop-func': 'error',
    'no-inner-declarations': 'error',
    
    // コード品質
    'no-var': 'error',
    'no-console': 'warn',
    'no-debugger': 'error',
    'no-unused-vars': 'error',
    'no-unreachable': 'error',
    'consistent-return': 'error',
    'default-case': 'error',
    'eqeqeq': 'error',
    'no-eval': 'error',
    'no-implied-eval': 'error',
    'no-new-func': 'error',
    'no-param-reassign': 'error',
    'no-return-assign': 'error',
    'no-self-compare': 'error',
    'no-throw-literal': 'error',
    'no-unmodified-loop-condition': 'error',
    'no-useless-call': 'error',
    'no-useless-concat': 'error',
    'prefer-object-spread': 'error'
  }
}
```

### Prettier設定

```javascript
// .prettierrc.js
module.exports = {
  semi: false,
  singleQuote: true,
  trailingComma: 'es5',
  tabWidth: 2,
  printWidth: 100,
  arrowParens: 'avoid',
  endOfLine: 'lf',
  quoteProps: 'as-needed',
  bracketSpacing: true,
  bracketSameLine: false,
  jsxSingleQuote: true,
  proseWrap: 'preserve'
}
```

---

**このガイドラインは2025年のJavaScript開発における最新のベストプラクティスを反映し、TypeScript移行とモダンWeb開発を支援します。**
