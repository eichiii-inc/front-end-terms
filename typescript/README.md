# TypeScript コーディング規約 (2025年版)

## 基本原則

### 型安全性の最大化
- `any` を使用せず、適切な型定義を行う
- `strict` モードを有効にして厳密な型チェックを実施
- 型推論を最大限活用し、不要な型注釈は避ける

### 表現力豊かな型システム
- ユーティリティタイプを積極的に活用
- 条件型とテンプレートリテラル型による高度な型定義
- 型レベルでのロジック実装

### パフォーマンス意識
- コンパイル時間の最適化
- 型計算の効率化
- 循環参照の回避

## TypeScript 5.7+ 新機能活用

### Const Type Parameters

```typescript
// 型パラメータの const 修飾子
function createTuple<const T extends readonly unknown[]>(...args: T): T {
  return args;
}

// 型推論がより正確に
const tuple = createTuple(1, 'hello', true);
// type: readonly [1, "hello", true]
```

### Satisfies Operator

```typescript
// satisfies で型安全性を保ちつつ推論を最適化
const config = {
  endpoint: '/api/v1',
  timeout: 5000,
  retries: 3
} satisfies Record<string, string | number>;

// config.endpoint は string 型として推論される
```

### Import Types

```typescript
// 型のみのインポート（TypeScript 5.x）
import type { User } from './types/user';
import type * as API from './api/types';

// 値と型の混在インポート
import { createUser, type UserOptions } from './user-service';
```

## 型定義の基本ルール

### インターフェース vs タイプエイリアス

```typescript
// インターフェース：拡張可能なオブジェクト型
interface User {
  readonly id: string;
  name: string;
  email: string;
  createdAt: Date;
}

// 拡張
interface AdminUser extends User {
  permissions: string[];
}

// タイプエイリアス：計算型、ユニオン型、条件型
type Status = 'loading' | 'success' | 'error';
type ApiResponse<T> = 
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

// 条件型
type NonNullable<T> = T extends null | undefined ? never : T;
```

### 命名規約

```typescript
// パスカルケース：型、インターフェース、クラス
interface UserProfile {}
type ApiEndpoint = string;
class DataService {}

// キャメルケース：変数、関数、プロパティ
const userName: string = 'john';
function getUserById(id: string): Promise<User> {}

// アッパーケース：定数、列挙型の値
const API_VERSION = 'v1';
enum Status {
  PENDING = 'pending',
  APPROVED = 'approved',
  REJECTED = 'rejected'
}

// 型パラメータ：T, U, V または説明的な名前
type MapFunction<TInput, TOutput> = (input: TInput) => TOutput;
```

## 高度な型パターン

### ユーティリティタイプの活用

```typescript
// 標準ユーティリティタイプ
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
  role: 'admin' | 'user';
}

// 部分的な型
type UserUpdate = Partial<User>;

// 必須フィールドのみ
type UserCreation = Pick<User, 'name' | 'email' | 'password'>;

// 除外型
type PublicUser = Omit<User, 'password'>;

// 読み取り専用
type ImmutableUser = Readonly<User>;

// レコード型
type UserRoles = Record<string, User['role']>;
```

### カスタムユーティリティタイプ

```typescript
// Deep Partial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// Required Keys
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

// Optional Keys
type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

// Flatten
type Flatten<T> = T extends (infer U)[] ? U : T;

// Promise Value
type PromiseValue<T> = T extends Promise<infer U> ? U : T;
```

### 条件型とテンプレートリテラル型

```typescript
// 条件型による型計算
type IsArray<T> = T extends any[] ? true : false;
type ArrayElement<T> = T extends (infer U)[] ? U : never;

// テンプレートリテラル型
type EventName<T extends string> = `on${Capitalize<T>}`;
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type ApiEndpoint<T extends HttpMethod> = `${Lowercase<T>}:/api/v1`;

// 複合パターン
type EventHandler<T extends string> = {
  [K in EventName<T>]: (data: any) => void;
};

// 使用例
type UserEvents = EventHandler<'create' | 'update' | 'delete'>;
// { onCreate: (data: any) => void; onUpdate: (data: any) => void; onDelete: (data: any) => void; }
```

## 関数型定義

### 関数シグネチャの最適化

```typescript
// オーバーロード
function createResource(name: string): Resource;
function createResource(config: ResourceConfig): Resource;
function createResource(nameOrConfig: string | ResourceConfig): Resource {
  // 実装
}

// Generic Functions
function identity<T>(value: T): T {
  return value;
}

// Constraint付きジェネリクス
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// 関数型の定義
type EventListener<T> = (event: T) => void;
type AsyncFunction<T, R> = (input: T) => Promise<R>;
type Middleware<T> = (input: T, next: () => void) => void;
```

### 高階関数の型定義

```typescript
// カリー化関数
type Curry<T extends any[], R> = T extends [infer H, ...infer Rest]
  ? (arg: H) => Rest extends [] ? R : Curry<Rest, R>
  : () => R;

// パイプ関数
type Pipe<T extends any[]> = T extends [
  (...args: any[]) => infer R1,
  (arg: R1) => infer R2,
  ...infer Rest
] ? Rest extends [] ? R2 : Pipe<[(arg: R1) => R2, ...Rest]>
  : never;

// 使用例
const pipe = <T extends any[]>(...fns: T) => 
  (input: any): Pipe<T> => fns.reduce((acc, fn) => fn(acc), input);
```

## API型定義

### REST API の型安全性

```typescript
// API エンドポイント定義
interface ApiEndpoints {
  '/users': {
    GET: { response: User[] };
    POST: { body: CreateUserRequest; response: User };
  };
  '/users/:id': {
    GET: { params: { id: string }; response: User };
    PUT: { params: { id: string }; body: UpdateUserRequest; response: User };
    DELETE: { params: { id: string }; response: void };
  };
}

// 型安全なAPI クライアント
type ApiClient = {
  [K in keyof ApiEndpoints]: {
    [M in keyof ApiEndpoints[K]]: (
      ...args: ApiEndpoints[K][M] extends { params: infer P; body: infer B }
        ? [params: P, body: B]
        : ApiEndpoints[K][M] extends { params: infer P }
        ? [params: P]
        : ApiEndpoints[K][M] extends { body: infer B }
        ? [body: B]
        : []
    ) => Promise<ApiEndpoints[K][M] extends { response: infer R } ? R : void>;
  };
};
```

### GraphQL型定義

```typescript
// GraphQL Schema型定義
interface GraphQLSchema {
  Query: {
    user: { args: { id: string }; return: User };
    users: { args: { filter?: UserFilter }; return: User[] };
  };
  Mutation: {
    createUser: { args: { input: CreateUserInput }; return: User };
    updateUser: { args: { id: string; input: UpdateUserInput }; return: User };
  };
}

// 型安全なGraphQL クライアント
type GraphQLClient = {
  query<K extends keyof GraphQLSchema['Query']>(
    query: K,
    args: GraphQLSchema['Query'][K]['args']
  ): Promise<GraphQLSchema['Query'][K]['return']>;
  
  mutate<K extends keyof GraphQLSchema['Mutation']>(
    mutation: K,
    args: GraphQLSchema['Mutation'][K]['args']
  ): Promise<GraphQLSchema['Mutation'][K]['return']>;
};
```

## 状態管理の型定義

### Redux with TypeScript

```typescript
// Actions
interface LoadUsersAction {
  type: 'LOAD_USERS';
}

interface LoadUsersSuccessAction {
  type: 'LOAD_USERS_SUCCESS';
  payload: User[];
}

interface LoadUsersErrorAction {
  type: 'LOAD_USERS_ERROR';
  payload: string;
}

type UserAction = LoadUsersAction | LoadUsersSuccessAction | LoadUsersErrorAction;

// State
interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
}

// Reducer
type UserReducer = (state: UserState, action: UserAction) => UserState;

// Store
interface AppState {
  users: UserState;
  // other slices
}
```

### Zustand Type-Safe Store

```typescript
// Store Interface
interface UserStore {
  users: User[];
  loading: boolean;
  error: string | null;
  
  // Actions
  loadUsers: () => Promise<void>;
  addUser: (user: CreateUserRequest) => Promise<void>;
  updateUser: (id: string, updates: Partial<User>) => Promise<void>;
  deleteUser: (id: string) => Promise<void>;
}

// Store Implementation
const useUserStore = create<UserStore>((set, get) => ({
  users: [],
  loading: false,
  error: null,
  
  loadUsers: async () => {
    set({ loading: true, error: null });
    try {
      const users = await fetchUsers();
      set({ users, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },
  
  // other actions...
}));
```

## React with TypeScript

### Component Props

```typescript
// 基本的な Props
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
}

// Generic Component
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string;
  emptyMessage?: string;
}

function List<T>({ items, renderItem, keyExtractor, emptyMessage }: ListProps<T>) {
  // implementation
}

// Forward Ref
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
}

const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, className, ...props }, ref) => {
    // implementation
  }
);
```

### Custom Hooks

```typescript
// 型安全なカスタムフック
function useApi<T>(url: string): {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
} {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(response.statusText);
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }, [url]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { data, loading, error, refetch: fetchData };
}
```

## エラーハンドリングと型安全性

### Result Pattern in TypeScript

```typescript
// Result型の定義
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

// Result型のユーティリティ
class ResultUtils {
  static ok<T>(data: T): Result<T> {
    return { success: true, data };
  }
  
  static error<E>(error: E): Result<never, E> {
    return { success: false, error };
  }
  
  static map<T, U, E>(
    result: Result<T, E>,
    mapper: (data: T) => U
  ): Result<U, E> {
    return result.success 
      ? ResultUtils.ok(mapper(result.data))
      : result;
  }
  
  static flatMap<T, U, E>(
    result: Result<T, E>,
    mapper: (data: T) => Result<U, E>
  ): Result<U, E> {
    return result.success ? mapper(result.data) : result;
  }
}

// 使用例
async function fetchUser(id: string): Promise<Result<User, ApiError>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      return ResultUtils.error(new ApiError(response.status, response.statusText));
    }
    const user = await response.json();
    return ResultUtils.ok(user);
  } catch (error) {
    return ResultUtils.error(new ApiError(500, 'Network error'));
  }
}
```

## パフォーマンス最適化

### 型計算の最適化

```typescript
// 悪い例：再帰が深すぎる
type BadDeepMerge<T, U> = {
  [K in keyof T | keyof U]: K extends keyof T
    ? K extends keyof U
      ? T[K] extends object
        ? U[K] extends object
          ? BadDeepMerge<T[K], U[K]>
          : U[K]
        : U[K]
      : T[K]
    : K extends keyof U
    ? U[K]
    : never;
};

// 良い例：制限された再帰
type DeepMerge<T, U, Depth extends any[] = []> = 
  Depth['length'] extends 10
    ? T & U
    : {
        [K in keyof T | keyof U]: K extends keyof T
          ? K extends keyof U
            ? T[K] extends object
              ? U[K] extends object
                ? DeepMerge<T[K], U[K], [...Depth, any]>
                : U[K]
              : U[K]
            : T[K]
          : K extends keyof U
          ? U[K]
          : never;
      };
```

### 型の最適化技法

```typescript
// インデックスシグネチャの最適化
interface OptimizedConfig {
  [key: `api_${string}`]: string;
  [key: `db_${string}`]: number;
  // 特定のキーは別途定義
  timeout: number;
  retries: number;
}

// 計算型のキャッシュ
type CachedKeyof<T> = T extends any ? keyof T : never;

// 条件型の最適化
type IsExactly<T, U> = (<G>() => G extends T ? 1 : 2) extends (<G>() => G extends U ? 1 : 2)
  ? true
  : false;
```

## 設定とベストプラクティス

### tsconfig.json 推奨設定

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    
    // Strict Type Checking
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    
    // Additional Checks
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitOverride": true,
    
    // Path Mapping
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"],
      "@/types/*": ["src/types/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## 禁止事項

- `any` 型の使用（`unknown` を使用）
- 型アサーション（`as`）の乱用
- `Function` 型の使用（具体的な関数型を定義）
- 暗黙的な `any`（`noImplicitAny` で検出）
- 非null断言演算子（`!`）の乱用
- `Object` 型の使用（`object` または具体的な型を使用）

## 推奨ツール

- **TypeScript**: 最新安定版（5.7+）
- **ts-node**: 開発時実行
- **tsc-alias**: パスマッピング対応
- **type-fest**: 追加ユーティリティタイプ
- **ts-reset**: 標準ライブラリの型改善
- **typescript-eslint**: TypeScript用ESLintルール

## パフォーマンス指標

- **コンパイル時間**: < 30秒（中規模プロジェクト）
- **型チェック時間**: < 10秒（増分チェック）
- **メモリ使用量**: < 1GB（大規模プロジェクト）
- **型エラー検出率**: > 95%
