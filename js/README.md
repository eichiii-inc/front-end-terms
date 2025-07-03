# JavaScript コーディング規約 (2025年版)

## 基本原則

### TypeScriptファースト
- 新規プロジェクトは TypeScript を使用
- 型安全性を最優先に考慮
- `any` の使用を避け、適切な型定義を行う

### 関数型プログラミングの活用
- 純粋関数を優先
- 副作用の分離
- 不変性（Immutability）の維持

### パフォーマンス最適化
- 不必要な再計算の回避
- メモ化の活用
- 効率的なアルゴリズムの選択

## ES2025+ Modern Features

### Pattern Matching (実験的)

```javascript
// パターンマッチングによる条件分岐
const handleApiResponse = (response) => {
  return match (response) {
    when { status: 200, data } => processSuccess(data),
    when { status: 404 } => handleNotFound(),
    when { status: s } if (s >= 500) => handleServerError(s),
    else => handleUnknownError()
  };
};
```

### Iterator Helpers

```javascript
// 新しいイテレータヘルパー
const numbers = [1, 2, 3, 4, 5];

const result = numbers
  .values()
  .filter(n => n % 2 === 0)
  .map(n => n * 2)
  .take(2)
  .toArray(); // [4, 8]
```

### Promise.try

```javascript
// Promise.try: 同期・非同期を統一的に処理
const processData = async (data) => {
  return Promise.try(() => {
    if (typeof data === 'string') {
      return parseData(data); // 同期処理
    }
    return fetchData(data); // 非同期処理
  });
};
```

## 文字列処理

### テンプレートリテラル優先

```javascript
// 悪い例
const message = 'Hello' + ' ' + name + '!';

// 良い例
const message = `Hello ${name}!`;

// タグ付きテンプレートリテラル
const safeHtml = html`<div>${userInput}</div>`;
```

### 文字列操作の最適化

```javascript
// 大量の文字列結合時は配列を使用
const buildLargeString = (items) => {
  return items
    .map(item => `<li>${item}</li>`)
    .join('');
};
```

## オブジェクト操作

### Modern Object Methods

```javascript
// Object.hasOwn (IE非対応のため hasOwnProperty より安全)
const hasProperty = (obj, prop) => Object.hasOwn(obj, prop);

// Object.groupBy (ES2024)
const groupedData = Object.groupBy(users, user => user.department);

// Structured Clone
const deepCopy = structuredClone(complexObject);
```

### 不変性の維持

```javascript
// 悪い例：元オブジェクトを変更
const updateUser = (user, updates) => {
  Object.assign(user, updates);
  return user;
};

// 良い例：新しいオブジェクトを返す
const updateUser = (user, updates) => ({
  ...user,
  ...updates,
  updatedAt: new Date().toISOString()
});
```

## 配列操作

### 関数型メソッドの活用

```javascript
// チェイニングによる効率的な処理
const processUsers = (users) => 
  users
    .filter(user => user.isActive)
    .map(user => ({
      ...user,
      displayName: `${user.firstName} ${user.lastName}`
    }))
    .sort((a, b) => a.displayName.localeCompare(b.displayName));

// 早期終了のためのsome/every
const hasAdminUser = users.some(user => user.role === 'admin');
const allUsersVerified = users.every(user => user.isVerified);
```

### Modern Array Methods

```javascript
// Array.fromAsync (ES2024)
const results = await Array.fromAsync(
  asyncIterable.map(async item => await processItem(item))
);

// Array.prototype.with (非破壊的更新)
const updatedArray = originalArray.with(index, newValue);
```

## 関数定義

### Pure Functions優先

```javascript
// 悪い例：副作用あり
let counter = 0;
const increment = () => {
  counter += 1;
  return counter;
};

// 良い例：純粋関数
const increment = (value) => value + 1;
```

### Higher-Order Functions

```javascript
// 高階関数による再利用性向上
const withLogging = (fn) => (...args) => {
  console.log(`Calling ${fn.name} with`, args);
  const result = fn(...args);
  console.log(`Result:`, result);
  return result;
};

const loggedAdd = withLogging((a, b) => a + b);
```

### Curry and Partial Application

```javascript
// カリー化による関数の部分適用
const curry = (fn) => (...args) =>
  args.length >= fn.length
    ? fn(...args)
    : (...nextArgs) => curry(fn)(...args, ...nextArgs);

const add = curry((a, b, c) => a + b + c);
const addFive = add(5);
const addFiveAndTen = addFive(10);
```

## 非同期処理

### Async/Await with Error Handling

```javascript
// エラーハンドリングの標準化
const fetchUserData = async (userId) => {
  try {
    const response = await fetch(`/api/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('Failed to fetch user data:', error);
    throw error; // 再スロー
  }
};
```

### Concurrent Processing

```javascript
// 並行処理の最適化
const fetchMultipleUsers = async (userIds) => {
  const userPromises = userIds.map(id => fetchUserData(id));
  return Promise.allSettled(userPromises);
};

// 順次処理が必要な場合
const processUsersSequentially = async (users) => {
  const results = [];
  for (const user of users) {
    const result = await processUser(user);
    results.push(result);
  }
  return results;
};
```

### AbortController の活用

```javascript
const fetchWithTimeout = async (url, timeout = 5000) => {
  const controller = new AbortController();
  const id = setTimeout(() => controller.abort(), timeout);
  
  try {
    const response = await fetch(url, {
      signal: controller.signal
    });
    return await response.json();
  } finally {
    clearTimeout(id);
  }
};
```

## モジュールシステム

### ES Modules優先

```javascript
// 名前付きエクスポート推奨
export const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
};

export const createApiClient = (config) => {
  // implementation
};

// デフォルトエクスポートは避ける（リファクタリング時の問題を防ぐ）
// export default createApiClient; // 避ける
```

### Dynamic Imports

```javascript
// 動的インポートによるコード分割
const loadFeature = async (featureName) => {
  const module = await import(`./features/${featureName}.js`);
  return module.default || module;
};

// 条件付きインポート
const loadPolyfill = async () => {
  if (!window.IntersectionObserver) {
    await import('intersection-observer');
  }
};
```

## エラーハンドリング

### Custom Error Classes

```javascript
// カスタムエラークラス
class ApiError extends Error {
  constructor(message, status, code) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
    this.code = code;
  }
}

class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}
```

### Result Pattern

```javascript
// Result パターンによるエラーハンドリング
const Result = {
  ok: (value) => ({ success: true, value }),
  error: (error) => ({ success: false, error })
};

const divideNumbers = (a, b) => {
  if (b === 0) {
    return Result.error(new Error('Division by zero'));
  }
  return Result.ok(a / b);
};

// 使用例
const result = divideNumbers(10, 2);
if (result.success) {
  console.log('Result:', result.value);
} else {
  console.error('Error:', result.error.message);
}
```

## パフォーマンス最適化

### Memoization

```javascript
// メモ化による最適化
const memoize = (fn) => {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
};

// 使用例
const expensiveCalculation = memoize((n) => {
  // 重い計算処理
  return n * n * n;
});
```

### Debounce and Throttle

```javascript
// デバウンス
const debounce = (fn, delay) => {
  let timeoutId;
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
};

// スロットル
const throttle = (fn, limit) => {
  let inThrottle;
  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
};
```

## Web APIs活用

### Intersection Observer

```javascript
// パフォーマンス最適化のための遅延読み込み
const createLazyLoader = (callback) => {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        callback(entry.target);
        observer.unobserve(entry.target);
      }
    });
  });
  
  return {
    observe: (element) => observer.observe(element),
    disconnect: () => observer.disconnect()
  };
};
```

### Web Workers

```javascript
// Web Worker による重い処理の分離
const createWorker = (workerScript) => {
  return new Promise((resolve, reject) => {
    const worker = new Worker(workerScript);
    
    worker.onmessage = (event) => {
      resolve(event.data);
      worker.terminate();
    };
    
    worker.onerror = (error) => {
      reject(error);
      worker.terminate();
    };
    
    return worker;
  });
};
```

## セキュリティ

### Input Sanitization

```javascript
// XSS対策
const sanitizeHtml = (str) => {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
};

// Trusted Types (Modern browsers)
const policy = trustedTypes?.createPolicy('app-policy', {
  createHTML: (string) => sanitizeHtml(string),
  createScript: (string) => string, // スクリプト用
  createScriptURL: (string) => string // URL用
});
```

### CSP対応

```javascript
// Content Security Policy に対応したコード
const createSecureScript = (content) => {
  const script = document.createElement('script');
  script.textContent = content;
  // nonceやhashを使用してCSPを満たす
  script.nonce = getNonce();
  return script;
};
```

## テスト指向開発

### テスタブルなコード

```javascript
// 依存性注入によるテスタブルなコード
const createApiService = (httpClient = fetch) => ({
  getUser: async (id) => {
    const response = await httpClient(`/api/users/${id}`);
    if (!response.ok) throw new Error('User not found');
    return response.json();
  }
});

// テスト時はモックを注入
const mockHttpClient = jest.fn();
const apiService = createApiService(mockHttpClient);
```

## AI活用（2025年新規）

### LLM API Integration

```javascript
// AI APIとの統合例
class AIAssistant {
  constructor(apiKey, model = 'gpt-4') {
    this.apiKey = apiKey;
    this.model = model;
  }
  
  async generateCode(prompt, language = 'javascript') {
    const response = await fetch('/api/ai/generate', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.apiKey}`
      },
      body: JSON.stringify({
        prompt: `Generate ${language} code: ${prompt}`,
        model: this.model
      })
    });
    
    return response.json();
  }
}
```

## 禁止事項

- `var` の使用（`const`/`let` を使用）
- `==` の使用（`===` を使用）
- `eval()` の使用
- グローバル変数の乱用
- 深いネストの条件分岐（3層まで）
- 副作用のある関数の乱用
- `any` 型の使用（TypeScript）

## 推奨ツール

### 開発ツール
- **ESLint**: コード品質チェック
- **Prettier**: コード整形
- **TypeScript**: 型安全性
- **Vitest**: 高速テストランナー
- **Biome**: 高速 linter/formatter

### パフォーマンス
- **Lighthouse**: パフォーマンス測定
- **Web Vitals**: Core Web Vitals測定
- **Bundle Analyzer**: バンドルサイズ分析

### セキュリティ
- **Snyk**: 脆弱性検査
- **ESLint Security**: セキュリティルール
- **Trusted Types**: XSS対策

## パフォーマンス指標

- **Time to Interactive (TTI)**: < 3秒
- **First Input Delay (FID)**: < 100ms
- **Bundle Size**: Initial < 200KB, Total < 1MB
- **Memory Usage**: < 50MB (平均)
