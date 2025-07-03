# React.js コーディング規約 2025

**React 19時代の包括的開発ガイドライン**

## 🎯 基本方針

### React 19 First
- React 19の最新機能を積極的に活用
- **Server Components（RSC）**: サーバーサイドレンダリング最適化
- **Client Components**: インタラクティビティが必要な場合のみ
- **新しいHooks**: `use`、`useFormStatus`、`useActionState`、`useOptimistic`
- **React Compiler**: 自動最適化機能活用

### TypeScript標準
- 全てのコンポーネントでTypeScriptを使用
- Props、State、Eventsの厳密な型定義
- `React.FC`よりも関数宣言を推奨
- Generic Components対応

### 関数コンポーネント優先
- 全ての新規コンポーネントは関数コンポーネント
- Hooksによる状態管理とライフサイクル
- クラスコンポーネントは遺産コードのみ

## 📁 ファイル・ディレクトリ規約

### ファイル命名
```
✅ 良い例
components/
├─ UserProfile.tsx         // PascalCase（.tsx必須）
├─ BaseButton.tsx          // Base接頭辞
├─ UserProfile.test.tsx    // テストファイル
├─ UserProfile.stories.tsx // Storybookストーリー
└─ index.ts               // エクスポートファイル

❌ 悪い例
components/
├─ userProfile.jsx        // camelCase
├─ user-profile.js        // kebab-case、JavaScript
├─ button.tsx            // 汎用的すぎる
```

### ディレクトリ構造（推奨）
```
src/
├─ app/                   // App Router（Next.js）
│  ├─ page.tsx
│  ├─ layout.tsx
│  └─ loading.tsx
├─ components/
│  ├─ ui/                // 基本UIコンポーネント
│  ├─ features/          // 機能別コンポーネント
│  └─ layout/            // レイアウトコンポーネント
├─ hooks/                // カスタムHooks
├─ lib/                  // ユーティリティ・設定
├─ types/                // TypeScript型定義
├─ actions/              // Server Actions
└─ __tests__/            // テストファイル
```

## 🧩 コンポーネント設計

### Server Components（RSC）

```tsx
// app/posts/page.tsx - Server Component
import { Suspense } from 'react'
import { PostList } from '@/components/features/posts/PostList'
import { SearchBox } from '@/components/features/posts/SearchBox'

interface Props {
  searchParams: { q?: string }
}

// サーバーコンポーネント（async関数可能）
export default async function PostsPage({ searchParams }: Props) {
  // サーバーでデータ取得
  const posts = await fetchPosts(searchParams.q)
  
  return (
    <div className="posts-page">
      <h1>投稿一覧</h1>
      
      {/* Client Component */}
      <SearchBox defaultValue={searchParams.q} />
      
      {/* Server Componentでデータ渡し */}
      <Suspense fallback={<PostListSkeleton />}>
        <PostList posts={posts} />
      </Suspense>
    </div>
  )
}

// データ取得（サーバーでのみ実行）
async function fetchPosts(query?: string) {
  const response = await fetch(`${process.env.API_URL}/posts?q=${query}`, {
    headers: {
      'Authorization': `Bearer ${process.env.API_SECRET}`
    }
  })
  
  if (!response.ok) {
    throw new Error('Failed to fetch posts')
  }
  
  return response.json()
}
```

### Client Components

```tsx
// components/features/posts/SearchBox.tsx
'use client' // Client Component宣言

import { useState, useTransition } from 'react'
import { useRouter, useSearchParams } from 'next/navigation'

interface Props {
  defaultValue?: string
}

export function SearchBox({ defaultValue = '' }: Props) {
  const router = useRouter()
  const searchParams = useSearchParams()
  const [query, setQuery] = useState(defaultValue)
  const [isPending, startTransition] = useTransition()
  
  const handleSearch = (e: React.FormEvent) => {
    e.preventDefault()
    
    startTransition(() => {
      const params = new URLSearchParams(searchParams)
      if (query) {
        params.set('q', query)
      } else {
        params.delete('q')
      }
      router.push(`/posts?${params.toString()}`)
    })
  }
  
  return (
    <form onSubmit={handleSearch} className="search-form">
      <input
        type="search"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="投稿を検索..."
        className="search-input"
        aria-label="投稿を検索"
      />
      <button 
        type="submit" 
        disabled={isPending}
        className="search-button"
      >
        {isPending ? '検索中...' : '検索'}
      </button>
    </form>
  )
}
```

### TypeScript Props定義

```tsx
// types/user.ts
export interface User {
  id: string
  name: string
  email: string
  avatar?: string
  isActive: boolean
  createdAt: Date
}

// components/features/users/UserCard.tsx
import type { User } from '@/types/user'

interface UserCardProps {
  user: User
  isLoading?: boolean
  onUpdate?: (user: User) => void
  onDelete?: (userId: string) => void
  className?: string
}

export function UserCard({ 
  user, 
  isLoading = false, 
  onUpdate,
  onDelete,
  className = ''
}: UserCardProps) {
  const handleUpdate = () => {
    onUpdate?.(user)
  }
  
  const handleDelete = () => {
    if (window.confirm('本当に削除しますか？')) {
      onDelete?.(user.id)
    }
  }
  
  return (
    <div className={`user-card ${className}`}>
      <div className="user-avatar">
        {user.avatar ? (
          <img 
            src={user.avatar} 
            alt={`${user.name}のアバター`}
            className="w-12 h-12 rounded-full"
          />
        ) : (
          <div className="w-12 h-12 bg-gray-300 rounded-full flex items-center justify-center">
            {user.name.charAt(0).toUpperCase()}
          </div>
        )}
      </div>
      
      <div className="user-info">
        <h3 className="text-lg font-semibold">{user.name}</h3>
        <p className="text-gray-600">{user.email}</p>
        <p className="text-sm text-gray-500">
          登録日: {user.createdAt.toLocaleDateString('ja-JP')}
        </p>
      </div>
      
      <div className="user-actions">
        <button 
          onClick={handleUpdate}
          disabled={isLoading}
          className="btn btn-primary"
        >
          編集
        </button>
        <button 
          onClick={handleDelete}
          disabled={isLoading}
          className="btn btn-danger"
        >
          削除
        </button>
      </div>
    </div>
  )
}
```

## 🪝 Hooks規約

### React 19の新しいHooks

```tsx
// React 19: use関数
import { use } from 'react'

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  // Promiseを直接処理
  const user = use(userPromise)
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  )
}

// React 19: useFormStatus
'use client'

import { useFormStatus } from 'react-dom'

function SubmitButton() {
  const { pending, data, method } = useFormStatus()
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  )
}

// React 19: useActionState
import { useActionState } from 'react'

async function updateUser(prevState: any, formData: FormData) {
  'use server'
  
  const name = formData.get('name') as string
  
  try {
    await saveUserToDatabase({ name })
    return { success: true, message: 'User updated!' }
  } catch (error) {
    return { success: false, message: 'Failed to update user' }
  }
}

function UserForm() {
  const [state, action] = useActionState(updateUser, { success: false, message: '' })
  
  return (
    <form action={action}>
      {state.message && (
        <div className={state.success ? 'success' : 'error'}>
          {state.message}
        </div>
      )}
      <input name="name" required />
      <SubmitButton />
    </form>
  )
}

// React 19: useOptimistic
import { useOptimistic, useTransition } from 'react'

function Comments({ comments }: { comments: Comment[] }) {
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (state, newComment: Comment) => [...state, { ...newComment, pending: true }]
  )
  
  const [isPending, startTransition] = useTransition()
  
  const addComment = async (text: string) => {
    const newComment = { id: Date.now(), text, author: 'You' }
    
    // 楽観的更新
    addOptimisticComment(newComment)
    
    // 実際の更新
    startTransition(async () => {
      await saveComment(newComment)
    })
  }
  
  return (
    <div>
      {optimisticComments.map(comment => (
        <div 
          key={comment.id} 
          className={comment.pending ? 'opacity-50' : ''}
        >
          {comment.text}
        </div>
      ))}
    </div>
  )
}
```

### カスタムHooks

```tsx
// hooks/useUser.ts
import { useState, useEffect } from 'react'
import type { User } from '@/types/user'

interface UseUserReturn {
  user: User | null
  loading: boolean
  error: string | null
  refetch: () => void
}

export function useUser(userId: string): UseUserReturn {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)
  
  const fetchUser = async () => {
    try {
      setLoading(true)
      setError(null)
      
      const response = await fetch(`/api/users/${userId}`)
      if (!response.ok) {
        throw new Error('Failed to fetch user')
      }
      
      const userData = await response.json()
      setUser(userData)
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error')
    } finally {
      setLoading(false)
    }
  }
  
  useEffect(() => {
    fetchUser()
  }, [userId])
  
  return { user, loading, error, refetch: fetchUser }
}

// hooks/useLocalStorage.ts
import { useState, useEffect } from 'react'

export function useLocalStorage<T>(
  key: string, 
  initialValue: T
): [T, (value: T | ((val: T) => T)) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') {
      return initialValue
    }
    
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error)
      return initialValue
    }
  })
  
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value
      setStoredValue(valueToStore)
      
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore))
      }
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error)
    }
  }
  
  return [storedValue, setValue]
}

// 使用例
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light')
  
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Current theme: {theme}
    </button>
  )
}
```

## 🎨 スタイリング規約

### Tailwind CSS（推奨）

```tsx
// components/ui/Button.tsx
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'underline-offset-4 hover:underline text-primary'
      },
      size: {
        default: 'h-10 py-2 px-4',
        sm: 'h-9 px-3 rounded-md',
        lg: 'h-11 px-8 rounded-md',
        icon: 'h-10 w-10'
      }
    },
    defaultVariants: {
      variant: 'default',
      size: 'default'
    }
  }
)

interface ButtonProps 
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

export function Button({ 
  className, 
  variant, 
  size, 
  asChild = false, 
  ...props 
}: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}
```

### CSS Modules（代替案）

```tsx
// components/Card.module.css
.card {
  @apply bg-white rounded-lg shadow-md p-6;
  border: 1px solid var(--border-color);
}

.card:hover {
  @apply shadow-lg;
  transform: translateY(-2px);
  transition: all 0.2s ease-in-out;
}

.cardHeader {
  @apply mb-4 pb-2 border-b border-gray-200;
}

.cardTitle {
  @apply text-xl font-semibold text-gray-900;
}

// components/Card.tsx
import styles from './Card.module.css'
import { cn } from '@/lib/utils'

interface CardProps {
  title?: string
  children: React.ReactNode
  className?: string
}

export function Card({ title, children, className }: CardProps) {
  return (
    <div className={cn(styles.card, className)}>
      {title && (
        <header className={styles.cardHeader}>
          <h2 className={styles.cardTitle}>{title}</h2>
        </header>
      )}
      {children}
    </div>
  )
}
```

## 🎯 イベントハンドリング

### 型安全なイベントハンドラ

```tsx
interface FormProps {
  onSubmit: (data: FormData) => void
}

function ContactForm({ onSubmit }: FormProps) {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    
    const formData = new FormData(e.currentTarget)
    const data = {
      name: formData.get('name') as string,
      email: formData.get('email') as string,
      message: formData.get('message') as string
    }
    
    onSubmit(data)
  }
  
  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target
    console.log(`${name} changed to:`, value)
  }
  
  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter' && e.metaKey) {
      // Cmd+Enter で送信
      e.currentTarget.form?.requestSubmit()
    }
  }
  
  return (
    <form onSubmit={handleSubmit} className="contact-form">
      <div className="field">
        <label htmlFor="name">名前</label>
        <input
          id="name"
          name="name"
          type="text"
          required
          onChange={handleInputChange}
          onKeyDown={handleKeyDown}
        />
      </div>
      
      <div className="field">
        <label htmlFor="email">メール</label>
        <input
          id="email"
          name="email"
          type="email"
          required
          onChange={handleInputChange}
        />
      </div>
      
      <div className="field">
        <label htmlFor="message">メッセージ</label>
        <textarea
          id="message"
          name="message"
          required
          rows={4}
        />
      </div>
      
      <button type="submit">送信</button>
    </form>
  )
}
```

## 🧪 テスト

### コンポーネントテスト（Vitest + Testing Library）

```tsx
// __tests__/UserCard.test.tsx
import { describe, it, expect, vi } from 'vitest'
import { render, screen, fireEvent } from '@testing-library/react'
import '@testing-library/jest-dom'
import { UserCard } from '@/components/features/users/UserCard'
import type { User } from '@/types/user'

const mockUser: User = {
  id: '1',
  name: 'テストユーザー',
  email: 'test@example.com',
  isActive: true,
  createdAt: new Date('2024-01-01')
}

describe('UserCard', () => {
  it('ユーザー情報を正しく表示する', () => {
    render(<UserCard user={mockUser} />)
    
    expect(screen.getByText(mockUser.name)).toBeInTheDocument()
    expect(screen.getByText(mockUser.email)).toBeInTheDocument()
    expect(screen.getByText(/登録日:/)).toBeInTheDocument()
  })
  
  it('編集ボタンクリック時にonUpdateが呼ばれる', () => {
    const mockOnUpdate = vi.fn()
    render(<UserCard user={mockUser} onUpdate={mockOnUpdate} />)
    
    const editButton = screen.getByText('編集')
    fireEvent.click(editButton)
    
    expect(mockOnUpdate).toHaveBeenCalledWith(mockUser)
  })
  
  it('削除ボタンクリック時に確認ダイアログが表示される', () => {
    const mockOnDelete = vi.fn()
    window.confirm = vi.fn().mockReturnValue(true)
    
    render(<UserCard user={mockUser} onDelete={mockOnDelete} />)
    
    const deleteButton = screen.getByText('削除')
    fireEvent.click(deleteButton)
    
    expect(window.confirm).toHaveBeenCalledWith('本当に削除しますか？')
    expect(mockOnDelete).toHaveBeenCalledWith(mockUser.id)
  })
  
  it('ローディング中はボタンが無効化される', () => {
    render(<UserCard user={mockUser} isLoading={true} />)
    
    const editButton = screen.getByText('編集')
    const deleteButton = screen.getByText('削除')
    
    expect(editButton).toBeDisabled()
    expect(deleteButton).toBeDisabled()
  })
})
```

### Hooksテスト

```tsx
// __tests__/useUser.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { renderHook, waitFor } from '@testing-library/react'
import { useUser } from '@/hooks/useUser'

// fetch のモック
global.fetch = vi.fn()

describe('useUser', () => {
  beforeEach(() => {
    vi.resetAllMocks()
  })
  
  it('初期状態では loading が true', () => {
    const { result } = renderHook(() => useUser('1'))
    
    expect(result.current.loading).toBe(true)
    expect(result.current.user).toBe(null)
    expect(result.current.error).toBe(null)
  })
  
  it('正常にユーザーデータを取得する', async () => {
    const mockUser = { id: '1', name: 'Test User', email: 'test@example.com' }
    
    vi.mocked(fetch).mockResolvedValueOnce({
      ok: true,
      json: async () => mockUser
    } as Response)
    
    const { result } = renderHook(() => useUser('1'))
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false)
    })
    
    expect(result.current.user).toEqual(mockUser)
    expect(result.current.error).toBe(null)
  })
  
  it('APIエラー時にエラー状態を設定する', async () => {
    vi.mocked(fetch).mockResolvedValueOnce({
      ok: false,
      status: 404
    } as Response)
    
    const { result } = renderHook(() => useUser('1'))
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false)
    })
    
    expect(result.current.user).toBe(null)
    expect(result.current.error).toBe('Failed to fetch user')
  })
})
```

## ♿ アクセシビリティ

### セマンティックHTML

```tsx
function ArticlePage({ article }: { article: Article }) {
  return (
    <article className="article-page">
      <header className="article-header">
        <h1>{article.title}</h1>
        <div className="article-meta">
          <address className="author">
            By <a href={`/authors/${article.author.id}`}>{article.author.name}</a>
          </address>
          <time dateTime={article.publishedAt.toISOString()}>
            {article.publishedAt.toLocaleDateString('ja-JP')}
          </time>
        </div>
      </header>
      
      <main className="article-content">
        <section aria-labelledby="summary-heading">
          <h2 id="summary-heading" className="sr-only">要約</h2>
          <p className="article-summary">{article.summary}</p>
        </section>
        
        <section aria-labelledby="content-heading">
          <h2 id="content-heading" className="sr-only">本文</h2>
          <div dangerouslySetInnerHTML={{ __html: article.content }} />
        </section>
      </main>
      
      <footer className="article-footer">
        <nav aria-label="記事アクション">
          <ShareButtons article={article} />
          <LikeButton articleId={article.id} />
        </nav>
      </footer>
    </article>
  )
}
```

### ARIA属性とキーボードナビゲーション

```tsx
interface DropdownProps {
  trigger: React.ReactNode
  children: React.ReactNode
}

function Dropdown({ trigger, children }: DropdownProps) {
  const [isOpen, setIsOpen] = useState(false)
  const [focusedIndex, setFocusedIndex] = useState(-1)
  const dropdownRef = useRef<HTMLDivElement>(null)
  const itemsRef = useRef<(HTMLElement | null)[]>([])
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'Escape':
        setIsOpen(false)
        break
      case 'ArrowDown':
        e.preventDefault()
        setFocusedIndex(prev => 
          prev < itemsRef.current.length - 1 ? prev + 1 : 0
        )
        break
      case 'ArrowUp':
        e.preventDefault()
        setFocusedIndex(prev => 
          prev > 0 ? prev - 1 : itemsRef.current.length - 1
        )
        break
      case 'Enter':
      case ' ':
        e.preventDefault()
        itemsRef.current[focusedIndex]?.click()
        break
    }
  }
  
  useEffect(() => {
    if (isOpen && focusedIndex >= 0) {
      itemsRef.current[focusedIndex]?.focus()
    }
  }, [focusedIndex, isOpen])
  
  return (
    <div className="dropdown" ref={dropdownRef}>
      <button
        type="button"
        aria-expanded={isOpen}
        aria-haspopup="true"
        onClick={() => setIsOpen(!isOpen)}
        onKeyDown={handleKeyDown}
      >
        {trigger}
      </button>
      
      {isOpen && (
        <div
          role="menu"
          aria-orientation="vertical"
          className="dropdown-menu"
          onKeyDown={handleKeyDown}
        >
          {React.Children.map(children, (child, index) => (
            <div
              role="menuitem"
              tabIndex={-1}
              ref={el => itemsRef.current[index] = el}
              className={`dropdown-item ${focusedIndex === index ? 'focused' : ''}`}
            >
              {child}
            </div>
          ))}
        </div>
      )}
    </div>
  )
}
```

## 📱 パフォーマンス最適化

### React.lazy と Suspense

```tsx
// 遅延読み込み
const HeavyComponent = React.lazy(() => import('@/components/HeavyComponent'))
const AdminPanel = React.lazy(() => import('@/components/AdminPanel'))

function App() {
  const [showAdmin, setShowAdmin] = useState(false)
  
  return (
    <div>
      <h1>アプリケーション</h1>
      
      <Suspense fallback={<div>読み込み中...</div>}>
        <HeavyComponent />
      </Suspense>
      
      {showAdmin && (
        <Suspense fallback={<div>管理パネル読み込み中...</div>}>
          <AdminPanel />
        </Suspense>
      )}
    </div>
  )
}
```

### React.memo と useMemo

```tsx
// 高コストな計算のメモ化
function ExpensiveComponent({ data, filter }: { data: Item[], filter: string }) {
  const filteredData = useMemo(() => {
    return data.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    )
  }, [data, filter])
  
  return (
    <div>
      {filteredData.map(item => (
        <ItemCard key={item.id} item={item} />
      ))}
    </div>
  )
}

// コンポーネントのメモ化
const ItemCard = React.memo(function ItemCard({ item }: { item: Item }) {
  return (
    <div className="item-card">
      <h3>{item.name}</h3>
      <p>{item.description}</p>
    </div>
  )
})

// useCallback でコールバック関数をメモ化
function Parent() {
  const [count, setCount] = useState(0)
  const [items, setItems] = useState<Item[]>([])
  
  const handleItemClick = useCallback((itemId: string) => {
    console.log('Item clicked:', itemId)
  }, [])
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ItemList items={items} onItemClick={handleItemClick} />
    </div>
  )
}
```

## 🔧 開発ツール設定

### ESLint設定（.eslintrc.js）

```javascript
module.exports = {
  extends: [
    'next/core-web-vitals',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended'
  ],
  rules: {
    // React
    'react/react-in-jsx-scope': 'off', // React 17+では不要
    'react/prop-types': 'off', // TypeScriptを使用時は不要
    'react/display-name': 'error',
    'react/no-array-index-key': 'warn',
    'react/no-unescaped-entities': 'error',
    
    // Hooks
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    
    // TypeScript
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
    
    // アクセシビリティ
    'jsx-a11y/alt-text': 'error',
    'jsx-a11y/anchor-has-content': 'error',
    'jsx-a11y/aria-role': 'error'
  }
}
```

### TypeScript設定（tsconfig.json）

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

---

**このガイドラインは2025年のReact開発のベストプラクティスを反映し、React 19の最新機能を活用した効率的な開発を支援します。**
