# TypeScript コーディング規約 2025

**TypeScript 5.7時代の包括的開発ガイドライン**

## 🎯 基本方針

### TypeScript 5.7+ First
- 全ての新規プロジェクトはTypeScript 5.7+で開発
- JavaScriptからの段階的移行を支援
- `any`型の使用を最小限に抑制
- 最新言語機能の積極活用

### 厳密な型安全性
- `strict: true`設定を必須とする
- Null安全性を確保（`strictNullChecks`）
- 型推論とType Narrowingを最大限活用
- Branded Typesによる実行時安全性

### パフォーマンス重視
- コンパイル時間の最適化（Project References活用）
- Bundle sizeを考慮した型設計
- Tree-shakingを活用
- 型レベル計算の最適化

## 📁 プロジェクト構成

### ディレクトリ構造
```
src/
├─ types/
│  ├─ api.ts              // API関連の型定義
│  ├─ common.ts           // 共通型定義
│  ├─ components.ts       // コンポーネント型定義
│  └─ utils.ts            // ユーティリティ型定義
├─ utils/
│  ├─ type-guards.ts      // 型ガード関数
│  └─ validators.ts       // バリデーション関数
├─ lib/
│  └─ type-helpers.ts     // 型ヘルパー関数
└─ __tests__/
   └─ type-tests.ts       // 型テスト
```

### tsconfig.json（推奨設定）
```json
{
  "compilerOptions": {
    // 基本設定
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "allowJs": false,
    "checkJs": false,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "removeComments": false,
    "importHelpers": true,
    "downlevelIteration": true,
    "isolatedModules": true,
    
    // 厳密性設定
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    
    // 追加チェック
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noPropertyAccessFromIndexSignature": true,
    
    // モジュール解決
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    
    // パス設定
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/types/*": ["./src/types/*"],
      "@/utils/*": ["./src/utils/*"],
      "@/components/*": ["./src/components/*"]
    },
    
    // スキップ設定
    "skipLibCheck": true
  },
  "include": [
    "src/**/*",
    "types/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts",
    "**/*.spec.ts"
  ]
}
```

## 🏗️ 型定義規約

### 基本型定義

```typescript
// ✅ 良い例：明確で説明的な型名
interface UserProfile {
  readonly id: string
  readonly email: string
  readonly displayName: string
  readonly avatar?: URL
  readonly isActive: boolean
  readonly createdAt: Date
  readonly updatedAt: Date
}

// ✅ Union型の活用
type UserRole = 'admin' | 'moderator' | 'user' | 'guest'
type Theme = 'light' | 'dark' | 'system'
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH'

// ✅ Discriminated Union（判別可能ユニオン）
type ApiResponse<T> = 
  | { success: true; data: T }
  | { success: false; error: string }

type LoadingState = 
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: unknown }
  | { status: 'error'; error: string }

// ❌ 悪い例：曖昧な型名
interface Data {
  info: any // anyは避ける
  stuff: object // 具体的でない
}
```

### 高度な型パターン

```typescript
// Utility Types の活用
type CreateUserRequest = Omit<UserProfile, 'id' | 'createdAt' | 'updatedAt'>
type UpdateUserRequest = Partial<Pick<UserProfile, 'displayName' | 'avatar'>>
type UserSummary = Pick<UserProfile, 'id' | 'displayName' | 'avatar'>

// Generic Types
interface Repository<T, K extends keyof T> {
  findById(id: T[K]): Promise<T | null>
  findAll(): Promise<T[]>
  create(entity: Omit<T, K>): Promise<T>
  update(id: T[K], updates: Partial<T>): Promise<T>
  delete(id: T[K]): Promise<void>
}

// Conditional Types
type NonNullable<T> = T extends null | undefined ? never : T
type ApiResult<T> = T extends string 
  ? { message: T }
  : T extends object 
    ? { data: T }
    : never

// Template Literal Types
type EventName<T extends string> = `on${Capitalize<T>}`
type CSSProperty = `--${string}`
type HttpStatusCode = `${1 | 2 | 3 | 4 | 5}${number}${number}`

// Mapped Types
type ReadonlyDeep<T> = {
  readonly [P in keyof T]: T[P] extends object ? ReadonlyDeep<T[P]> : T[P]
}

type OptionalFields<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>

// Higher-order types
type EventHandler<T = unknown> = (event: T) => void
type AsyncFunction<T extends unknown[], R> = (...args: T) => Promise<R>
type Predicate<T> = (value: T) => boolean
```

### React向け型定義

```typescript
// コンポーネントProps
interface ButtonProps {
  readonly variant?: 'primary' | 'secondary' | 'danger'
  readonly size?: 'sm' | 'md' | 'lg'
  readonly disabled?: boolean
  readonly loading?: boolean
  readonly children: React.ReactNode
  readonly onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void
  readonly className?: string
}

// Event handlers
type FormSubmitHandler = (event: React.FormEvent<HTMLFormElement>) => void
type InputChangeHandler = (event: React.ChangeEvent<HTMLInputElement>) => void
type KeyboardEventHandler = (event: React.KeyboardEvent) => void

// Ref types
type InputRef = React.RefObject<HTMLInputElement>
type ButtonRef = React.RefObject<HTMLButtonElement>

// Hook return types
interface UseUserReturn {
  readonly user: UserProfile | null
  readonly loading: boolean
  readonly error: string | null
  readonly refetch: () => Promise<void>
}

// Generic Component Props
interface GenericListProps<T> {
  readonly items: readonly T[]
  readonly renderItem: (item: T, index: number) => React.ReactNode
  readonly keyExtractor: (item: T) => string | number
  readonly emptyMessage?: string
  readonly loading?: boolean
}

// Forward Ref Components（React 19では不要だが互換性のため）
type ForwardedComponent<T, P = {}> = React.ForwardRefExoticComponent<
  React.PropsWithoutRef<P> & React.RefAttributes<T>
>
```

### Vue向け型定義

```typescript
// Vue 3 Composition API
interface UseCounterReturn {
  readonly count: Readonly<Ref<number>>
  readonly increment: () => void
  readonly decrement: () => void
  readonly reset: () => void
  readonly isEven: ComputedRef<boolean>
}

// Component Props（Vue 3）
interface UserCardProps {
  readonly user: UserProfile
  readonly showActions?: boolean
  readonly variant?: 'card' | 'list' | 'compact'
}

// Emit Events
interface UserCardEmits {
  update: [user: UserProfile]
  delete: [userId: string]
  'selection-change': [selected: boolean]
}

// Pinia Store
interface UserStoreState {
  users: UserProfile[]
  currentUser: UserProfile | null
  loading: boolean
  error: string | null
}

interface UserStoreGetters {
  userCount: number
  activeUsers: UserProfile[]
  isAuthenticated: boolean
}

interface UserStoreActions {
  fetchUsers(): Promise<void>
  createUser(userData: CreateUserRequest): Promise<UserProfile>
  updateUser(id: string, updates: UpdateUserRequest): Promise<UserProfile>
  deleteUser(id: string): Promise<void>
  login(credentials: LoginCredentials): Promise<void>
  logout(): void
}
```

## 🛡️ 型安全性の強化

### 型ガード関数

```typescript
// 基本的な型ガード
function isString(value: unknown): value is string {
  return typeof value === 'string'
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number' && !Number.isNaN(value)
}

function isNonNull<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined
}

// オブジェクト型ガード
function isUserProfile(value: unknown): value is UserProfile {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'email' in value &&
    'displayName' in value &&
    typeof (value as UserProfile).id === 'string' &&
    typeof (value as UserProfile).email === 'string' &&
    typeof (value as UserProfile).displayName === 'string'
  )
}

// 配列型ガード
function isArrayOf<T>(
  value: unknown,
  guard: (item: unknown) => item is T
): value is T[] {
  return Array.isArray(value) && value.every(guard)
}

// 使用例
const userProfiles = isArrayOf(data, isUserProfile) ? data : []

// APIレスポンス型ガード
function isApiSuccess<T>(
  response: ApiResponse<T>
): response is { success: true; data: T } {
  return response.success === true
}

// 使用例
if (isApiSuccess(response)) {
  // response.data の型が T として推論される
  console.log(response.data)
} else {
  // response.error の型が string として推論される
  console.error(response.error)
}
```

### アサーション関数

```typescript
// アサーション関数
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== 'string') {
    throw new Error(`Expected string, got ${typeof value}`)
  }
}

function assertIsUserProfile(value: unknown): asserts value is UserProfile {
  if (!isUserProfile(value)) {
    throw new Error('Invalid user profile object')
  }
}

// 使用例
function processUserData(data: unknown) {
  assertIsUserProfile(data)
  // この時点で data は UserProfile 型として扱われる
  console.log(data.displayName)
}

// Non-null assertion helper
function assertNonNull<T>(value: T | null | undefined): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error('Value must not be null or undefined')
  }
}
```

### Branded Types（ブランド型）

```typescript
// ブランド型の定義
type Brand<T, B> = T & { readonly __brand: B }

type UserId = Brand<string, 'UserId'>
type Email = Brand<string, 'Email'>
type Timestamp = Brand<number, 'Timestamp'>
type PositiveNumber = Brand<number, 'PositiveNumber'>

// ファクトリー関数
function createUserId(id: string): UserId {
  if (!id || id.length === 0) {
    throw new Error('User ID cannot be empty')
  }
  return id as UserId
}

function createEmail(email: string): Email {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!emailRegex.test(email)) {
    throw new Error('Invalid email format')
  }
  return email as Email
}

function createPositiveNumber(num: number): PositiveNumber {
  if (num <= 0) {
    throw new Error('Number must be positive')
  }
  return num as PositiveNumber
}

// 使用例
interface User {
  readonly id: UserId
  readonly email: Email
  readonly createdAt: Timestamp
  readonly score: PositiveNumber
}

// 型安全な関数
function getUserById(id: UserId): Promise<User> {
  // id は確実に有効な UserId 型
  return fetch(`/api/users/${id}`).then(res => res.json())
}

// ❌ コンパイルエラー
// getUserById('invalid-id') // string は UserId に代入できない

// ✅ 正しい使用方法
const userId = createUserId('user-123')
getUserById(userId)
```

## 🔧 高度な型テクニック

### 型レベルプログラミング

```typescript
// 文字列操作型
type CamelCase<S extends string> = S extends `${infer Head}_${infer Tail}`
  ? `${Head}${Capitalize<CamelCase<Tail>>}`
  : S

type SnakeCase<S extends string> = S extends `${infer Head}${infer Tail}`
  ? Head extends Uppercase<Head>
    ? `_${Lowercase<Head>}${SnakeCase<Tail>}`
    : `${Head}${SnakeCase<Tail>}`
  : S

// オブジェクト変換型
type CamelCaseKeys<T> = {
  [K in keyof T as CamelCase<string & K>]: T[K]
}

type SnakeCaseKeys<T> = {
  [K in keyof T as SnakeCase<string & K>]: T[K]
}

// 数値計算型
type Length<T extends readonly unknown[]> = T['length']
type IsEmpty<T extends readonly unknown[]> = Length<T> extends 0 ? true : false
type Head<T extends readonly unknown[]> = T extends readonly [infer H, ...unknown[]] ? H : never
type Tail<T extends readonly unknown[]> = T extends readonly [unknown, ...infer Rest] ? Rest : never

// パス型生成
type PathsToStringProps<T> = T extends string
  ? []
  : {
      [K in Extract<keyof T, string>]: [K, ...PathsToStringProps<T[K]>]
    }[Extract<keyof T, string>]

type Join<T extends string[], D extends string> = T extends []
  ? never
  : T extends [infer F]
  ? F
  : T extends [infer F, ...infer R]
  ? F extends string
    ? `${F}${D}${Join<Extract<R, string[]>, D>}`
    : never
  : string

type PathsToStrings<T> = Join<PathsToStringProps<T>, '.'>

// 使用例
type UserPaths = PathsToStrings<UserProfile>
// 'id' | 'email' | 'displayName' | 'avatar' | 'isActive' | 'createdAt' | 'updatedAt'
```

### 関数オーバーロード

```typescript
// 基本的なオーバーロード
interface EventEmitter {
  on(event: 'user:created', listener: (user: UserProfile) => void): this
  on(event: 'user:updated', listener: (user: UserProfile) => void): this
  on(event: 'user:deleted', listener: (userId: string) => void): this
  on(event: 'error', listener: (error: Error) => void): this
  on(event: string, listener: (...args: any[]) => void): this
}

// 条件付きオーバーロード
function createElement<K extends keyof HTMLElementTagNameMap>(
  tagName: K
): HTMLElementTagNameMap[K]
function createElement<K extends keyof SVGElementTagNameMap>(
  tagName: K
): SVGElementTagNameMap[K]
function createElement(tagName: string): Element

function createElement(tagName: string): Element {
  return document.createElement(tagName)
}

// 使用例
const button = createElement('button') // HTMLButtonElement
const circle = createElement('circle') // SVGCircleElement
const custom = createElement('custom-element') // Element
```

### Template Literal Types

```typescript
// CSS プロパティ型
type CSSUnit = 'px' | 'em' | 'rem' | '%' | 'vh' | 'vw'
type CSSValue<T extends CSSUnit> = `${number}${T}`

type Margin = CSSValue<'px' | 'em' | 'rem' | '%'>
type FontSize = CSSValue<'px' | 'em' | 'rem'>

// API エンドポイント型
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE'
type ApiEndpoint<Method extends HttpMethod, Path extends string> = `${Method} ${Path}`

type UserEndpoints = 
  | ApiEndpoint<'GET', '/users'>
  | ApiEndpoint<'GET', '/users/${string}'>
  | ApiEndpoint<'POST', '/users'>
  | ApiEndpoint<'PUT', '/users/${string}'>
  | ApiEndpoint<'DELETE', '/users/${string}'>

// SQL クエリビルダー型
type SQLOperator = '=' | '!=' | '>' | '<' | '>=' | '<=' | 'LIKE' | 'IN'
type WhereClause<T, K extends keyof T> = `${string & K} ${SQLOperator} ${string}`

type UserWhereClause = WhereClause<UserProfile, 'id' | 'email' | 'displayName'>
```

## 🧪 型テスト

### 型アサーションテスト

```typescript
// 型テストヘルパー
type Expect<T extends true> = T
type Equal<X, Y> = (<T>() => T extends X ? 1 : 2) extends <T>() => T extends Y ? 1 : 2
  ? true
  : false

// 型テストケース
type TestCases = [
  // UserProfile 型のテスト
  Expect<Equal<UserProfile['id'], string>>,
  Expect<Equal<UserProfile['email'], string>>,
  Expect<Equal<UserProfile['avatar'], URL | undefined>>,
  
  // ユーティリティ型のテスト
  Expect<Equal<CreateUserRequest, Omit<UserProfile, 'id' | 'createdAt' | 'updatedAt'>>>,
  Expect<Equal<UpdateUserRequest, Partial<Pick<UserProfile, 'displayName' | 'avatar'>>>>,
  
  // 条件型のテスト
  Expect<Equal<NonNullable<string | null>, string>>,
  Expect<Equal<NonNullable<number | undefined>, number>>,
  
  // Template Literal型のテスト
  Expect<Equal<CamelCase<'user_profile'>, 'userProfile'>>,
  Expect<Equal<SnakeCase<'UserProfile'>, 'user_profile'>>,
]

// ランタイム型テスト
import { describe, it, expect } from 'vitest'

describe('Type Guards', () => {
  it('should correctly identify strings', () => {
    expect(isString('hello')).toBe(true)
    expect(isString(123)).toBe(false)
    expect(isString(null)).toBe(false)
  })
  
  it('should correctly identify UserProfile objects', () => {
    const validUser: UserProfile = {
      id: 'user-1',
      email: 'test@example.com',
      displayName: 'Test User',
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date()
    }
    
    expect(isUserProfile(validUser)).toBe(true)
    expect(isUserProfile({})).toBe(false)
    expect(isUserProfile(null)).toBe(false)
  })
})
```

### パフォーマンステスト

```typescript
// 型コンパイル時間の測定
// ※ TypeScript Compiler API使用

import * as ts from 'typescript'
import { performance } from 'perf_hooks'

function measureTypeCheckingPerformance(sourceCode: string): number {
  const start = performance.now()
  
  const compilerOptions: ts.CompilerOptions = {
    target: ts.ScriptTarget.ES2022,
    module: ts.ModuleKind.ESNext,
    strict: true,
    noEmit: true
  }
  
  const sourceFile = ts.createSourceFile(
    'test.ts',
    sourceCode,
    ts.ScriptTarget.ES2022
  )
  
  const program = ts.createProgram(['test.ts'], compilerOptions, {
    getSourceFile: (fileName) => fileName === 'test.ts' ? sourceFile : undefined,
    writeFile: () => {},
    getCurrentDirectory: () => '',
    getDirectories: () => [],
    fileExists: () => true,
    readFile: () => '',
    getCanonicalFileName: (fileName) => fileName,
    useCaseSensitiveFileNames: () => true,
    getNewLine: () => '\n'
  })
  
  const diagnostics = ts.getPreEmitDiagnostics(program)
  const end = performance.now()
  
  return end - start
}
```

## 📚 ライブラリ統合

### Zod によるランタイムバリデーション

```typescript
import { z } from 'zod'

// Zodスキーマ定義
const UserProfileSchema = z.object({
  id: z.string().min(1),
  email: z.string().email(),
  displayName: z.string().min(1).max(100),
  avatar: z.string().url().optional(),
  isActive: z.boolean(),
  createdAt: z.date(),
  updatedAt: z.date()
})

// TypeScriptの型をZodから推論
type UserProfile = z.infer<typeof UserProfileSchema>

// バリデーション関数
function validateUserProfile(data: unknown): UserProfile {
  return UserProfileSchema.parse(data)
}

function isValidUserProfile(data: unknown): data is UserProfile {
  return UserProfileSchema.safeParse(data).success
}

// APIレスポンスバリデーション
const ApiResponseSchema = <T extends z.ZodType>(dataSchema: T) =>
  z.discriminatedUnion('success', [
    z.object({
      success: z.literal(true),
      data: dataSchema
    }),
    z.object({
      success: z.literal(false),
      error: z.string()
    })
  ])

type ApiResponse<T> = z.infer<ReturnType<typeof ApiResponseSchema>>
```

### tRPC 統合

```typescript
import { z } from 'zod'
import { initTRPC } from '@trpc/server'

const t = initTRPC.create()

// 入力バリデーション
const CreateUserInput = z.object({
  email: z.string().email(),
  displayName: z.string().min(1).max(100),
  avatar: z.string().url().optional()
})

const UpdateUserInput = z.object({
  id: z.string(),
  displayName: z.string().min(1).max(100).optional(),
  avatar: z.string().url().optional()
})

// tRPCルーター
export const userRouter = t.router({
  getUser: t.procedure
    .input(z.object({ id: z.string() }))
    .output(UserProfileSchema)
    .query(async ({ input }) => {
      // 型安全なデータベースアクセス
      return await userRepository.findById(input.id)
    }),
    
  createUser: t.procedure
    .input(CreateUserInput)
    .output(UserProfileSchema)
    .mutation(async ({ input }) => {
      // 型安全なユーザー作成
      return await userRepository.create(input)
    }),
    
  updateUser: t.procedure
    .input(UpdateUserInput)
    .output(UserProfileSchema)
    .mutation(async ({ input }) => {
      const { id, ...updates } = input
      return await userRepository.update(id, updates)
    })
})

// クライアント側の型推論
type UserRouter = typeof userRouter
```

### Prisma 統合

```typescript
import { PrismaClient, User, Prisma } from '@prisma/client'

const prisma = new PrismaClient()

// Prismaの型を拡張
type UserWithProfile = Prisma.UserGetPayload<{
  include: { profile: true }
}>

type UserCreateInput = Prisma.UserCreateInput
type UserUpdateInput = Prisma.UserUpdateInput
type UserWhereInput = Prisma.UserWhereInput

// 型安全なリポジトリ
class UserRepository {
  async findById(id: string): Promise<User | null> {
    return prisma.user.findUnique({
      where: { id }
    })
  }
  
  async findWithProfile(id: string): Promise<UserWithProfile | null> {
    return prisma.user.findUnique({
      where: { id },
      include: { profile: true }
    })
  }
  
  async create(data: UserCreateInput): Promise<User> {
    return prisma.user.create({ data })
  }
  
  async update(id: string, data: UserUpdateInput): Promise<User> {
    return prisma.user.update({
      where: { id },
      data
    })
  }
  
  async findMany(where?: UserWhereInput): Promise<User[]> {
    return prisma.user.findMany({ where })
  }
}
```

## 🚀 パフォーマンス最適化

### 型レベル最適化

```typescript
// 深いネストを避ける
type DeepNested = {
  level1: {
    level2: {
      level3: {
        level4: {
          value: string
        }
      }
    }
  }
}

// ✅ フラットな構造に変更
type FlatStructure = {
  level1_level2_level3_level4_value: string
}

// 大きなUnion型を避ける
type LargeUnion = 
  | 'option1' | 'option2' | 'option3' | /* ... 100+ options */

// ✅ 段階的な型に分割
type CategoryA = 'option1' | 'option2' | 'option3'
type CategoryB = 'option4' | 'option5' | 'option6'
type Categories = CategoryA | CategoryB

// Conditional typesの最適化
type SlowConditional<T> = T extends string 
  ? T extends `${infer Start}${infer End}` 
    ? Start extends 'prefix' 
      ? End
      : never
    : never
  : never

// ✅ より効率的な実装
type FastConditional<T> = T extends `prefix${infer Rest}` ? Rest : never
```

### コンパイル時間の短縮

```typescript
// 型キャッシュの活用
type MemoizedType<T> = T extends infer U ? U : never

// インクリメンタルコンパイルの設定
// tsconfig.json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo"
  }
}

// プロジェクト参照の活用
// tsconfig.json
{
  "references": [
    { "path": "./packages/types" },
    { "path": "./packages/utils" },
    { "path": "./packages/components" }
  ]
}
```

## 🛠️ 開発ツール

### ESLint TypeScript設定

```javascript
// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    '@typescript-eslint/recommended-requiring-type-checking'
  ],
  rules: {
    // TypeScript 固有ルール
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/prefer-nullish-coalescing': 'error',
    '@typescript-eslint/prefer-optional-chain': 'error',
    '@typescript-eslint/no-non-null-assertion': 'warn',
    '@typescript-eslint/no-unnecessary-type-assertion': 'error',
    '@typescript-eslint/prefer-as-const': 'error',
    '@typescript-eslint/array-type': ['error', { default: 'array-simple' }],
    '@typescript-eslint/consistent-type-definitions': ['error', 'interface'],
    '@typescript-eslint/consistent-type-imports': ['error', { prefer: 'type-imports' }],
    
    // 名前規則
    '@typescript-eslint/naming-convention': [
      'error',
      {
        selector: 'interface',
        format: ['PascalCase']
      },
      {
        selector: 'typeAlias',
        format: ['PascalCase']
      },
      {
        selector: 'enum',
        format: ['PascalCase']
      },
      {
        selector: 'variable',
        format: ['camelCase', 'UPPER_CASE', 'PascalCase']
      },
      {
        selector: 'function',
        format: ['camelCase', 'PascalCase']
      }
    ]
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
  bracketSameLine: false
}
```

### VS Code設定

```json
// .vscode/settings.json
{
  "typescript.preferences.inlayHints.enumMemberValues.enabled": true,
  "typescript.preferences.inlayHints.functionLikeReturnTypes.enabled": true,
  "typescript.preferences.inlayHints.parameterNames.enabled": "all",
  "typescript.preferences.inlayHints.parameterTypes.enabled": true,
  "typescript.preferences.inlayHints.propertyDeclarationTypes.enabled": true,
  "typescript.preferences.inlayHints.variableTypes.enabled": true,
  "typescript.suggest.autoImports": true,
  "typescript.suggest.completeFunctionCalls": true,
  "typescript.preferences.importModuleSpecifier": "relative",
  "editor.codeActionsOnSave": {
    "source.organizeImports": true,
    "source.fixAll.eslint": true
  }
}
```

---

**このガイドラインは2025年のTypeScript開発における最新のベストプラクティスを反映し、型安全性と開発効率の両立を目指します。**
