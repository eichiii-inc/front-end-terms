# CSS Coding Guide (2025年版)

## 基本原則

### パフォーマンス優先
- Critical CSSの分離
- 未使用CSSの削除
- 効率的なセレクタの使用

### アクセシビリティファースト
- `prefers-reduced-motion`への対応
- 十分なコントラスト比の確保
- フォーカス表示の明確化

### モダンCSS機能の活用
- Container Queries
- CSS Grid と Flexbox
- Logical Properties
- CSS Variables (カスタムプロパティ)

## セレクタ規約

### IDセレクターの禁止

```css
/* 悪い例 */
#header {
  background: blue;
}

/* 良い例 */
[id="header"] {
  background: blue;
}

/* 更に良い例：クラスベース */
.header {
  background: blue;
}
```

### Type Selectorの制限

```css
/* 悪い例 */
a {
  color: red;
}

/* 良い例：コンテキスト付き */
.navigation a {
  color: red;
}

/* 最良：クラスベース */
.link {
  color: red;
}
```

## Modern CSS Features (2025年)

### Container Queries

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

.card {
  padding: 1rem;
}

@container card (min-width: 300px) {
  .card {
    padding: 2rem;
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
```

### CSS Grid & Flexbox

```css
/* CSS Grid：レイアウト用 */
.layout {
  display: grid;
  grid-template-areas: 
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

/* Flexbox：コンポーネント内配置用 */
.button-group {
  display: flex;
  gap: 0.5rem;
  justify-content: flex-end;
}
```

### Logical Properties

```css
/* 悪い例：物理的プロパティ */
.card {
  margin-left: 1rem;
  margin-right: 1rem;
  padding-top: 2rem;
  border-left: 2px solid blue;
}

/* 良い例：論理的プロパティ */
.card {
  margin-inline: 1rem;
  padding-block-start: 2rem;
  border-inline-start: 2px solid blue;
}
```

## カラーシステム（2025年）

### Modern Color Functions

```css
:root {
  /* OKLCH：知覚的に均一な色空間 */
  --primary: oklch(60% 0.15 250);
  --secondary: oklch(80% 0.12 180);
  
  /* Color-mix：動的色生成 */
  --primary-light: color-mix(in oklch, var(--primary) 80%, white);
  --primary-dark: color-mix(in oklch, var(--primary) 80%, black);
}

.button {
  background: var(--primary);
  color: white;
}

.button:hover {
  background: var(--primary-light);
}
```

### アクセシブルなカラーコントラスト

```css
.text-primary {
  color: oklch(30% 0.1 250); /* AAA準拠のコントラスト */
}

.text-secondary {
  color: oklch(45% 0.05 250); /* AA準拠のコントラスト */
}
```

## CSS Variables 2.0

```css
:root {
  /* ネストした変数 */
  --spacing-base: 1rem;
  --spacing-small: calc(var(--spacing-base) * 0.5);
  --spacing-large: calc(var(--spacing-base) * 2);
  
  /* 条件付き変数 */
  --font-size: clamp(1rem, 4vw, 1.5rem);
  
  /* ダークテーマ対応 */
  --bg-color: light-dark(white, #1a1a1a);
  --text-color: light-dark(#333, #f0f0f0);
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg-color: #1a1a1a;
    --text-color: #f0f0f0;
  }
}
```

## Cascade Layers

```css
@layer reset, base, components, utilities;

@layer reset {
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }
}

@layer base {
  body {
    font-family: system-ui, sans-serif;
    line-height: 1.6;
  }
}

@layer components {
  .button {
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 0.25rem;
    cursor: pointer;
  }
}

@layer utilities {
  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
  }
}
```

## アクセシビリティ対応

### Motion Sensitivity

```css
@media (prefers-reduced-motion: no-preference) {
  .card {
    transition: transform 0.3s ease;
  }
  
  .card:hover {
    transform: translateY(-2px);
  }
}

@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Focus Management

```css
.button {
  border: 2px solid transparent;
}

.button:focus-visible {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
  border-color: var(--primary);
}

.button:focus:not(:focus-visible) {
  outline: none;
}
```

### High Contrast Mode

```css
@media (prefers-contrast: high) {
  .button {
    border: 2px solid;
  }
}

@media (forced-colors: active) {
  .button {
    forced-color-adjust: none;
    border: 1px solid ButtonText;
  }
}
```

## パフォーマンス最適化

### Critical CSS

```css
/* Critical CSS：Above the fold */
.hero {
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.navigation {
  display: flex;
  justify-content: space-between;
  padding: 1rem 2rem;
}
```

### Efficient Selectors

```css
/* 悪い例：複雑なセレクタ */
.sidebar .widget .title h3 span {
  color: red;
}

/* 良い例：シンプルで効率的 */
.widget-title {
  color: red;
}
```

### CSS Containment

```css
.card {
  contain: layout style paint;
}

.sidebar {
  contain: layout;
}
```

## レスポンシブデザイン

### Fluid Typography

```css
.heading {
  font-size: clamp(1.5rem, 4vw + 1rem, 3rem);
}
```

### Container-based Responsive

```css
.card-grid {
  container-type: inline-size;
  display: grid;
  gap: 1rem;
}

@container (min-width: 400px) {
  .card-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@container (min-width: 800px) {
  .card-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

## 命名規約（BEM拡張）

```css
/* Block */
.card { }

/* Element */
.card__title { }
.card__content { }

/* Modifier */
.card--featured { }
.card__title--large { }

/* State（2025年追加） */
.card.is-loading { }
.card.has-error { }

/* Responsive Modifier */
.card--mobile { }
.card--desktop { }
```

## CSS-in-JS代替：CSS Modules

```css
/* Button.module.css */
.button {
  composes: base-button from './shared.module.css';
  background: var(--primary);
}

.primary {
  composes: button;
  background: var(--primary);
}

.secondary {
  composes: button;
  background: var(--secondary);
}
```

## 禁止事項

- `!important`の使用（Cascade Layersを使用）
- 深いセレクタネスト（3階層まで）
- ベンダープレフィックスの手動記述（PostCSSを使用）
- 物理的プロパティ（logical propertiesを使用）
- 固定単位（相対単位を優先）

## 推奨ツール

- **PostCSS**: 自動プレフィックス、未来のCSS機能
- **Lightning CSS**: 高速CSS処理
- **Stylelint**: CSS linting
- **Critical**: Critical CSS抽出
- **PurgeCSS**: 未使用CSS削除

## パフォーマンス指標

- **First Contentful Paint (FCP)**: < 1.8秒
- **Largest Contentful Paint (LCP)**: < 2.5秒
- **Cumulative Layout Shift (CLS)**: < 0.1
