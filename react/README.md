# React コーディング規約 (2025年版)

## 基本原則

### React Server Components (RSC) ファースト
- サーバーサイドで実行可能なコンポーネントを優先
- クライアントサイドのJavaScript削減
- データフェッチの最適化

### TypeScript統合
- すべてのコンポーネントでTypeScript使用
- 厳密な型定義によるランタイムエラーの防止
- Props の型安全性確保

### パフォーマンス最優先
- 不要な再レンダリングの回避
- Code Splittingとlazy loading
- メモ化戦略の適切な活用

## React 19 New Features

### Server Components

```tsx
// Server Component (デフォルト)
async function UserProfile({ userId }: { userId: string }) {
  // サーバーサイドでデータフェッチ
  const user = await fetchUser(userId);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <UserActions userId={userId} />
    </div>
  );
}

// Client Component (明示的に指定)
'use client';
function UserActions({ userId }: { userId: string }) {
  const [loading, setLoading] = useState(false);
  
  const handleDelete = async () => {
    setLoading(true);
    await deleteUser(userId);
    setLoading(false);
  };
  
  return (
    <button onClick={handleDelete} disabled={loading}>
      {loading ? 'Deleting...' : 'Delete User'}
    </button>
  );
}
```

### New Hooks (React 19)

```tsx
// useOptimistic - 楽観的更新
function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, newTodo]
  );
  
  const createTodo = async (text: string) => {
    const tempTodo = { id: `temp-${Date.now()}`, text, completed: false };
    addOptimisticTodo(tempTodo);
    
    try {
      await saveTodo(text);
    } catch (error) {
      // 自動的にロールバック
    }
  };
  
  return (
    <ul>
      {optimisticTodos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}

// useFormStatus - フォーム状態管理
function SubmitButton() {
  const { pending, data } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

// use - Promise/Context のサスペンス対応
function UserData({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise); // Suspenseと連携
  
  return <div>{user.name}</div>;
}
```

## コンポーネント設計

### Props定義

```tsx
// 基本的なProps
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
}

// 良い例：明確な型定義
const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  loading = false,
  onClick
}) => {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled || loading}
      onClick={onClick}
      aria-busy={loading}
    >
      {loading ? 'Loading...' : children}
    </button>
  );
};

// 悪い例：型定義なし、属性値の省略形乱用
const BadButton = ({ open, className }) => (
  <button className={className} open={open}>
    Click me
  </button>
);
```

### Forward Ref パターン

```tsx
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
  helperText?: string;
}

const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, helperText, className, ...props }, ref) => {
    const id = useId();
    
    return (
      <div className="input-group">
        {label && (
          <label htmlFor={id} className="input-label">
            {label}
          </label>
        )}
        <input
          ref={ref}
          id={id}
          className={`input ${error ? 'input-error' : ''} ${className || ''}`}
          aria-invalid={!!error}
          aria-describedby={error ? `${id}-error` : helperText ? `${id}-help` : undefined}
          {...props}
        />
        {error && (
          <div id={`${id}-error`} className="input-error-text" role="alert">
            {error}
          </div>
        )}
        {helperText && !error && (
          <div id={`${id}-help`} className="input-help-text">
            {helperText}
          </div>
        )}
      </div>
    );
  }
);

Input.displayName = 'Input';
```

### Generic Components

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T, index: number) => string | number;
  emptyMessage?: string;
  loading?: boolean;
}

function List<T>({
  items,
  renderItem,
  keyExtractor,
  emptyMessage = 'No items found',
  loading = false
}: ListProps<T>) {
  if (loading) {
    return <div className="list-loading">Loading...</div>;
  }
  
  if (items.length === 0) {
    return <div className="list-empty">{emptyMessage}</div>;
  }
  
  return (
    <div className="list">
      {items.map((item, index) => (
        <div key={keyExtractor(item, index)} className="list-item">
          {renderItem(item, index)}
        </div>
      ))}
    </div>
  );
}

// 使用例
const UserList = () => {
  const [users, setUsers] = useState<User[]>([]);
  
  return (
    <List
      items={users}
      renderItem={(user) => <UserCard user={user} />}
      keyExtractor={(user) => user.id}
      emptyMessage="No users found"
    />
  );
};
```

## Hooks Best Practices

### State Management

```tsx
// useState - シンプルな状態管理
function Counter() {
  const [count, setCount] = useState(0);
  
  // 良い例：関数形式で更新
  const increment = () => setCount(prev => prev + 1);
  
  // 悪い例：直接値を設定
  // const increment = () => setCount(count + 1);
  
  return (
    <div>
      <span>{count}</span>
      <button onClick={increment}>+</button>
    </div>
  );
}

// useReducer - 複雑な状態管理
interface State {
  count: number;
  step: number;
  history: number[];
}

type Action = 
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'set_step'; step: number }
  | { type: 'reset' };

function counterReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return {
        ...state,
        count: state.count + state.step,
        history: [...state.history, state.count + state.step]
      };
    case 'decrement':
      return {
        ...state,
        count: state.count - state.step,
        history: [...state.history, state.count - state.step]
      };
    case 'set_step':
      return { ...state, step: action.step };
    case 'reset':
      return { count: 0, step: 1, history: [0] };
    default:
      return state;
  }
}

function AdvancedCounter() {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    step: 1,
    history: [0]
  });
  
  return (
    <div>
      <span>{state.count}</span>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <input 
        value={state.step}
        onChange={(e) => dispatch({ type: 'set_step', step: +e.target.value })}
      />
    </div>
  );
}
```

### Custom Hooks

```tsx
// API フェッチフック
function useApi<T>(url: string, options?: RequestInit) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(url, options);
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  }, [url, options]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { data, loading, error, refetch: fetchData };
}

// ローカルストレージフック
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });
  
  const setValue = useCallback((value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setValue] as const;
}
```

## パフォーマンス最適化

### Memoization

```tsx
// React.memo - コンポーネントのメモ化
interface UserCardProps {
  user: User;
  onEdit: (id: string) => void;
}

const UserCard = React.memo<UserCardProps>(({ user, onEdit }) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user.id)}>Edit</button>
    </div>
  );
});

// useMemo - 値のメモ化
function ExpensiveComponent({ items }: { items: Item[] }) {
  const expensiveValue = useMemo(() => {
    return items
      .filter(item => item.isActive)
      .reduce((sum, item) => sum + item.value, 0);
  }, [items]);
  
  return <div>Total: {expensiveValue}</div>;
}

// useCallback - 関数のメモ化
function TodoApp() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  // 良い例：useCallbackでメモ化
  const addTodo = useCallback((text: string) => {
    setTodos(prev => [...prev, { id: Date.now(), text, completed: false }]);
  }, []);
  
  const toggleTodo = useCallback((id: number) => {
    setTodos(prev => 
      prev.map(todo => 
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, []);
  
  return (
    <div>
      <TodoForm onSubmit={addTodo} />
      <TodoList todos={todos} onToggle={toggleTodo} />
    </div>
  );
}
```

### Code Splitting

```tsx
// Lazy loading
const DashboardPage = lazy(() => import('./pages/Dashboard'));
const ProfilePage = lazy(() => import('./pages/Profile'));
const SettingsPage = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Router>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/dashboard" element={<DashboardPage />} />
          <Route path="/profile" element={<ProfilePage />} />
          <Route path="/settings" element={<SettingsPage />} />
        </Routes>
      </Suspense>
    </Router>
  );
}

// 動的インポート
function AdminPanel() {
  const [showAdvanced, setShowAdvanced] = useState(false);
  const [AdvancedSettings, setAdvancedSettings] = useState<React.ComponentType | null>(null);
  
  useEffect(() => {
    if (showAdvanced && !AdvancedSettings) {
      import('./AdvancedSettings').then(module => {
        setAdvancedSettings(() => module.default);
      });
    }
  }, [showAdvanced, AdvancedSettings]);
  
  return (
    <div>
      <button onClick={() => setShowAdvanced(true)}>
        Show Advanced Settings
      </button>
      {showAdvanced && AdvancedSettings && <AdvancedSettings />}
    </div>
  );
}
```

## 状態管理

### Context API

```tsx
// Theme Context
interface ThemeContextValue {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);
  
  const value = useMemo(() => ({
    theme,
    toggleTheme
  }), [theme, toggleTheme]);
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

### External State Management

```tsx
// Zustand example
interface UserStore {
  users: User[];
  loading: boolean;
  error: string | null;
  fetchUsers: () => Promise<void>;
  addUser: (user: CreateUserRequest) => Promise<void>;
}

const useUserStore = create<UserStore>((set, get) => ({
  users: [],
  loading: false,
  error: null,
  
  fetchUsers: async () => {
    set({ loading: true, error: null });
    try {
      const users = await fetchUsers();
      set({ users, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },
  
  addUser: async (userData) => {
    try {
      const newUser = await createUser(userData);
      set(state => ({ users: [...state.users, newUser] }));
    } catch (error) {
      set({ error: error.message });
    }
  }
}));
```

## Suspense と Error Boundaries

### Suspense

```tsx
// データフェッチでのSuspense
function UserProfile({ userId }: { userId: string }) {
  return (
    <Suspense fallback={<UserProfileSkeleton />}>
      <UserData userId={userId} />
    </Suspense>
  );
}

// 複数レベルのSuspense
function App() {
  return (
    <Suspense fallback={<AppSkeleton />}>
      <Header />
      <Suspense fallback={<ContentSkeleton />}>
        <MainContent />
      </Suspense>
      <Footer />
    </Suspense>
  );
}
```

### Error Boundaries

```tsx
interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
  errorInfo?: ErrorInfo;
}

class ErrorBoundary extends Component<
  { children: React.ReactNode; fallback?: React.ComponentType<{ error: Error }> },
  ErrorBoundaryState
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // エラーレポーティングサービスに送信
    reportError(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      const FallbackComponent = this.props.fallback || DefaultErrorFallback;
      return <FallbackComponent error={this.state.error!} />;
    }
    
    return this.props.children;
  }
}

// 関数コンポーネント版 (react-error-boundary使用推奨)
function ErrorFallback({ error, resetErrorBoundary }: ErrorFallbackProps) {
  return (
    <div className="error-fallback">
      <h2>Something went wrong:</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}
```

## アクセシビリティ

### ARIA属性とセマンティクス

```tsx
function Modal({ isOpen, onClose, title, children }: ModalProps) {
  const titleId = useId();
  const descriptionId = useId();
  
  // フォーカストラップ
  useEffect(() => {
    if (isOpen) {
      const focusableElements = modalRef.current?.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      const firstElement = focusableElements?.[0] as HTMLElement;
      const lastElement = focusableElements?.[focusableElements.length - 1] as HTMLElement;
      
      const handleTabKey = (e: KeyboardEvent) => {
        if (e.key === 'Tab') {
          if (e.shiftKey) {
            if (document.activeElement === firstElement) {
              lastElement?.focus();
              e.preventDefault();
            }
          } else {
            if (document.activeElement === lastElement) {
              firstElement?.focus();
              e.preventDefault();
            }
          }
        }
      };
      
      document.addEventListener('keydown', handleTabKey);
      firstElement?.focus();
      
      return () => document.removeEventListener('keydown', handleTabKey);
    }
  }, [isOpen]);
  
  if (!isOpen) return null;
  
  return createPortal(
    <div
      className="modal-backdrop"
      onClick={onClose}
      role="dialog"
      aria-modal="true"
      aria-labelledby={titleId}
      aria-describedby={descriptionId}
    >
      <div
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
        ref={modalRef}
      >
        <header className="modal-header">
          <h2 id={titleId}>{title}</h2>
          <button
            onClick={onClose}
            aria-label="Close modal"
            className="modal-close"
          >
            ×
          </button>
        </header>
        <div id={descriptionId} className="modal-body">
          {children}
        </div>
      </div>
    </div>,
    document.body
  );
}
```

## テスト戦略

### Component Testing

```tsx
// React Testing Library
describe('Button Component', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument();
  });
  
  it('handles click events', async () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('shows loading state', () => {
    render(<Button loading>Submit</Button>);
    expect(screen.getByRole('button')).toHaveAttribute('aria-busy', 'true');
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
});

// Custom Hook Testing
function renderUseCounter(initialValue = 0) {
  return renderHook(() => useCounter(initialValue));
}

describe('useCounter', () => {
  it('initializes with correct value', () => {
    const { result } = renderUseCounter(5);
    expect(result.current.count).toBe(5);
  });
  
  it('increments count', () => {
    const { result } = renderUseCounter();
    act(() => {
      result.current.increment();
    });
    expect(result.current.count).toBe(1);
  });
});
```

## 禁止事項

- インラインオブジェクト/配列の多用（パフォーマンス影響）
- useEffect の依存配列省略
- DOM直接操作（`document.getElementById` 等）
- キーとしてのインデックス使用（動的リストで）
- 不要な Fragment（`<>...</>`）の使用
- Props drilling（5階層以上）
- 巨大なコンポーネント（200行超）

## 推奨ツール

### 開発ツール
- **React DevTools**: デバッグとプロファイリング
- **React Hook Form**: フォーム管理
- **React Query/TanStack Query**: サーバー状態管理
- **Zustand/Valtio**: クライアント状態管理

### テストツール
- **React Testing Library**: コンポーネントテスト
- **Jest**: テストフレームワーク
- **MSW**: APIモック
- **Storybook**: コンポーネントドキュメント

### 品質管理
- **ESLint**: コード品質
- **React Hook ESLint**: Hooksルール
- **axe-core**: アクセシビリティテスト

## パフォーマンス指標

- **Initial Bundle Size**: < 244KB (gzipped)
- **Lighthouse Performance**: > 90
- **First Contentful Paint**: < 1.8秒
- **Time to Interactive**: < 3.8秒
- **Component Re-renders**: 最小限に抑制
