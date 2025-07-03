# CSS/スタイリング規約 2025

**モダンCSS・Tailwind CSS時代の包括的ガイドライン**

## 🎯 基本方針

### Tailwind CSS First
- **Tailwind CSS を主要なスタイリング手法とする**
- Utility-firstアプローチによる効率的な開発
- デザインシステムとの一貫性を保持
- CSS-in-JSとの適切な使い分け

### パフォーマンス重視
- CSS バンドルサイズの最適化
- Critical CSS の実装
- 不要なスタイルの削除（PurgeCSS/JIT）
- Container Queries活用による効率的レスポンシブ

### アクセシビリティファースト
- セマンティックな色彩設計
- 十分なコントラスト比の確保（WCAG 2.1 AA）
- キーボードナビゲーション対応
- prefers-reduced-motion対応

## 📚 技術スタック優先順位

### 1. Tailwind CSS（最優先）
```html
<!-- ✅ 推奨：Tailwind utilityクラス -->
<button class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded-lg transition-colors duration-200 disabled:opacity-50">
  送信
</button>
```

### 2. CSS-in-JS（コンポーネント固有）
```tsx
// ✅ styled-components / emotion（動的スタイル）
const DynamicButton = styled.button<{ variant: string }>`
  background: ${props => props.variant === 'primary' ? '#3b82f6' : '#6b7280'};
  transition: all 0.2s ease;
  
  &:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
`
```

### 3. CSS Modules（スコープ化）
```css
/* Button.module.css */
.button {
  @apply inline-flex items-center justify-center;
  @apply px-4 py-2 text-sm font-medium rounded-md;
  @apply transition-all duration-200;
}

.button:focus {
  @apply outline-none ring-2 ring-offset-2 ring-blue-500;
}
```

### 4. Vanilla CSS（最小限）
```css
/* ✅ グローバルスタイルやTailwindで表現困難な場合のみ */
@layer utilities {
  .scroll-smooth {
    scroll-behavior: smooth;
  }
  
  .text-balance {
    text-wrap: balance;
  }
}
```

## 🎨 Tailwind CSS ベストプラクティス

### 設定ファイル（tailwind.config.js）

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      // カスタムカラーパレット
      colors: {
        brand: {
          50: '#eff6ff',
          100: '#dbeafe',
          500: '#3b82f6',
          600: '#2563eb',
          900: '#1e3a8a',
        },
        gray: {
          50: '#f9fafb',
          100: '#f3f4f6',
          500: '#6b7280',
          900: '#111827',
        }
      },
      
      // タイポグラフィシステム
      fontSize: {
        'xs': ['0.75rem', { lineHeight: '1rem' }],
        'sm': ['0.875rem', { lineHeight: '1.25rem' }],
        'base': ['1rem', { lineHeight: '1.5rem' }],
        'lg': ['1.125rem', { lineHeight: '1.75rem' }],
        'xl': ['1.25rem', { lineHeight: '1.75rem' }],
        '2xl': ['1.5rem', { lineHeight: '2rem' }],
        '3xl': ['1.875rem', { lineHeight: '2.25rem' }],
        '4xl': ['2.25rem', { lineHeight: '2.5rem' }],
      },
      
      // スペーシングシステム
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      },
      
      // アニメーション
      animation: {
        'fade-in': 'fadeIn 0.3s ease-in-out',
        'slide-up': 'slideUp 0.3s ease-out',
        'pulse-soft': 'pulse 2s infinite',
      },
      
      keyframes: {
        fadeIn: {
          '0%': { opacity: '0' },
          '100%': { opacity: '1' },
        },
        slideUp: {
          '0%': { transform: 'translateY(10px)', opacity: '0' },
          '100%': { transform: 'translateY(0)', opacity: '1' },
        },
      },
      
      // ブレークポイント
      screens: {
        'xs': '475px',
        'sm': '640px',
        'md': '768px',
        'lg': '1024px',
        'xl': '1280px',
        '2xl': '1536px',
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
    require('@tailwindcss/aspect-ratio'),
    require('@tailwindcss/container-queries'),
  ],
}
```

### コンポーネント設計パターン

```tsx
// ✅ Variant-based Design System
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const buttonVariants = cva(
  // ベースクラス
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'underline-offset-4 hover:underline text-primary',
      },
      size: {
        default: 'h-10 py-2 px-4',
        sm: 'h-9 px-3 rounded-md',
        lg: 'h-11 px-8 rounded-md',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
)

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}
```

### レスポンシブデザイン

```html
<!-- ✅ モバイルファーストアプローチ -->
<div class="
  w-full p-4
  sm:w-1/2 sm:p-6
  md:w-1/3 md:p-8
  lg:w-1/4 lg:p-10
  xl:w-1/5 xl:p-12
">
  <!-- コンテンツ -->
</div>

<!-- ✅ Container Queries対応 -->
<div class="@container">
  <div class="@md:flex @md:space-x-4">
    <div class="@md:w-1/2">左側</div>
    <div class="@md:w-1/2">右側</div>
  </div>
</div>
```

### ダークモード対応

```html
<!-- ✅ システム連動ダークモード -->
<div class="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  <h1 class="text-xl font-bold text-gray-900 dark:text-white">
    タイトル
  </h1>
  <p class="text-gray-600 dark:text-gray-300">
    説明文
  </p>
</div>

<!-- ✅ カスタムダークモード変数 -->
<style>
  :root {
    --color-bg: #ffffff;
    --color-text: #111827;
  }

  @media (prefers-color-scheme: dark) {
    :root {
      --color-bg: #111827;
      --color-text: #f9fafb;
    }
  }

  [data-theme="dark"] {
    --color-bg: #111827;
    --color-text: #f9fafb;
  }
</style>
```

## 🚀 モダンCSS機能

### CSS Grid & Flexbox

```css
/* ✅ モダンレイアウト */
.grid-auto-fit {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

.flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

/* ✅ CSS Subgrid */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1rem;
}

.card {
  display: grid;
  grid-template-rows: auto 1fr auto;
  gap: 0.5rem;
}

.card-content {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;
}
```

### Container Queries

```css
/* ✅ Container Queries */
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card-title {
    font-size: 1.5rem;
  }
  
  .card-layout {
    display: flex;
    gap: 1rem;
  }
}

@container card (min-width: 600px) {
  .card-image {
    width: 200px;
    height: 150px;
  }
}
```

### CSS カスタムプロパティ（CSS Variables）

```css
/* ✅ デザイントークン */
:root {
  /* カラーシステム */
  --color-primary-50: hsl(214, 100%, 97%);
  --color-primary-500: hsl(214, 100%, 59%);
  --color-primary-900: hsl(214, 100%, 21%);
  
  /* タイポグラフィシステム */
  --font-family-sans: system-ui, -apple-system, sans-serif;
  --font-family-mono: 'Fira Code', monospace;
  
  /* スペーシングシステム */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --space-8: 2rem;
  
  /* レイヤーシステム */
  --z-dropdown: 1000;
  --z-sticky: 1020;
  --z-fixed: 1030;
  --z-modal: 1040;
  --z-popover: 1050;
  --z-tooltip: 1060;
}

/* ✅ 動的テーマ切り替え */
[data-theme="light"] {
  --color-bg: var(--color-gray-50);
  --color-text: var(--color-gray-900);
}

[data-theme="dark"] {
  --color-bg: var(--color-gray-900);
  --color-text: var(--color-gray-50);
}

.themed-component {
  background-color: var(--color-bg);
  color: var(--color-text);
  transition: background-color 0.2s ease, color 0.2s ease;
}
```

### 現代的なCSS セレクター

```css
/* ✅ :has() セレクター */
.card:has(.card-image) {
  display: grid;
  grid-template-columns: 200px 1fr;
  gap: 1rem;
}

.form-field:has(input:invalid) .field-label {
  color: red;
}

/* ✅ :where() セレクター（詳細度0）*/
:where(.btn, .button) {
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

/* ✅ :is() セレクター */
:is(h1, h2, h3, h4, h5, h6) {
  font-weight: bold;
  line-height: 1.2;
}

/* ✅ CSS Nesting */
.card {
  padding: 1rem;
  border-radius: 0.5rem;
  
  & .card-title {
    font-size: 1.25rem;
    font-weight: 600;
    
    & + .card-subtitle {
      margin-top: 0.25rem;
      color: #6b7280;
    }
  }
  
  & .card-actions {
    margin-top: 1rem;
    display: flex;
    gap: 0.5rem;
    
    & button {
      flex: 1;
    }
  }
}
```

## ⚡ パフォーマンス最適化

### Critical CSS

```css
/* ✅ Above-the-fold CSS */
@layer critical {
  /* ページ表示に必要な最小限のスタイル */
  .header {
    position: sticky;
    top: 0;
    z-index: var(--z-sticky);
  }
  
  .hero {
    min-height: 50vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

/* ✅ Non-critical CSS（遅延読み込み） */
@layer deferred {
  .modal,
  .dropdown,
  .tooltip {
    /* 後から読み込まれるスタイル */
  }
}
```

### CSS最適化

```css
/* ✅ Containment による最適化 */
.isolated-component {
  contain: layout style paint;
}

.list-item {
  contain: layout paint;
}

/* ✅ will-change による最適化 */
.animated-element {
  will-change: transform;
}

.animated-element.animating {
  transform: translateX(100px);
  transition: transform 0.3s ease;
}

.animated-element:not(.animating) {
  will-change: auto; /* アニメーション後はリセット */
}

/* ✅ content-visibility による仮想化 */
.long-content-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}
```

### 画像最適化

```css
/* ✅ アスペクト比の維持 */
.aspect-video {
  aspect-ratio: 16 / 9;
}

.aspect-square {
  aspect-ratio: 1 / 1;
}

/* ✅ レスポンシブ画像 */
.responsive-image {
  width: 100%;
  height: auto;
  object-fit: cover;
  object-position: center;
}

/* ✅ 遅延読み込み最適化 */
img[loading="lazy"] {
  content-visibility: auto;
}
```

## 🎭 アニメーション & トランジション

### パフォーマンス重視のアニメーション

```css
/* ✅ GPU アクセラレーション活用 */
@keyframes slide-in {
  from {
    transform: translate3d(-100%, 0, 0);
    opacity: 0;
  }
  to {
    transform: translate3d(0, 0, 0);
    opacity: 1;
  }
}

/* ✅ ease 関数の活用 */
.smooth-transition {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.bounce-in {
  animation: bounce-in 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55);
}

/* ✅ prefers-reduced-motion 対応 */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

@media (prefers-reduced-motion: no-preference) {
  .fade-in {
    animation: fade-in 0.3s ease-in-out;
  }
}
```

### インタラクション アニメーション

```css
/* ✅ ホバー & フォーカス エフェクト */
.interactive-button {
  position: relative;
  overflow: hidden;
  transition: all 0.2s ease;
  
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
    transition: left 0.5s ease;
  }
  
  &:hover::before {
    left: 100%;
  }
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 10px 25px rgba(0,0,0,0.15);
  }
  
  &:active {
    transform: translateY(0);
  }
}

/* ✅ マイクロインタラクション */
.like-button {
  transition: all 0.2s ease;
  
  &:active {
    transform: scale(0.95);
  }
  
  &.liked {
    animation: heart-beat 0.6s ease;
    color: #ef4444;
  }
}

@keyframes heart-beat {
  0% { transform: scale(1); }
  25% { transform: scale(1.2); }
  50% { transform: scale(1); }
  75% { transform: scale(1.1); }
  100% { transform: scale(1); }
}
```

## ♿ アクセシビリティ

### カラーコントラスト

```css
/* ✅ WCAG 2.1 AA準拠のカラーコントラスト */
:root {
  /* 4.5:1 以上のコントラスト比 */
  --text-on-light: #1f2937; /* gray-800 */
  --text-on-dark: #f9fafb;  /* gray-50 */
  
  /* リンクカラー（下線なしでも識別可能） */
  --link-primary: #1d4ed8;  /* blue-700 */
  --link-visited: #7c3aed;  /* violet-600 */
}

/* ✅ フォーカス表示 */
.focusable {
  &:focus-visible {
    outline: 2px solid var(--color-primary-500);
    outline-offset: 2px;
    border-radius: 2px;
  }
}

/* ✅ ハイコントラストモード対応 */
@media (prefers-contrast: high) {
  .button {
    border: 2px solid currentColor;
    background: ButtonFace;
    color: ButtonText;
  }
}
```

### キーボードナビゲーション

```css
/* ✅ スキップリンク */
.skip-link {
  position: absolute;
  top: -40px;
  left: 6px;
  background: var(--color-primary-600);
  color: white;
  padding: 8px;
  border-radius: 4px;
  text-decoration: none;
  z-index: var(--z-modal);
  
  &:focus {
    top: 6px;
  }
}

/* ✅ フォーカストラップ */
.modal-overlay {
  &:focus-within {
    /* モーダル内でフォーカスを制限 */
  }
}

/* ✅ 視覚的フィードバック */
.interactive-element {
  &:hover,
  &:focus-visible {
    background-color: var(--color-primary-50);
    transform: scale(1.02);
  }
  
  &:active {
    transform: scale(0.98);
  }
}
```

## 🧪 CSS アーキテクチャ

### レイヤード CSS（CSS Cascade Layers）

```css
/* ✅ CSS Cascade Layers */
@layer reset, base, components, utilities, overrides;

@layer reset {
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }
}

@layer base {
  html {
    font-family: var(--font-family-sans);
    line-height: 1.5;
  }
  
  body {
    background-color: var(--color-bg);
    color: var(--color-text);
  }
}

@layer components {
  .button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0.5rem 1rem;
    border-radius: 0.375rem;
    font-weight: 500;
    transition: all 0.2s ease;
  }
}

@layer utilities {
  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
  }
}

@layer overrides {
  /* 必要に応じてオーバーライド */
}
```

### 組織化パターン

```scss
// ✅ ITCSS (Inverted Triangle CSS) 構造
@forward 'settings';     // 変数、設定
@forward 'tools';        // ミックスイン、関数
@forward 'generic';      // リセット、ノーマライズ
@forward 'elements';     // HTML要素
@forward 'objects';      // レイアウトパターン
@forward 'components';   // UIコンポーネント
@forward 'utilities';    // ユーティリティクラス

// settings/_colors.scss
$colors: (
  'primary': (
    50: #eff6ff,
    500: #3b82f6,
    900: #1e3a8a,
  ),
  'gray': (
    50: #f9fafb,
    500: #6b7280,
    900: #111827,
  )
);

// tools/_mixins.scss
@mixin button-base {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

@mixin focus-ring($color: blue) {
  &:focus-visible {
    outline: 2px solid #{$color};
    outline-offset: 2px;
  }
}
```

## 🔧 開発ツール設定

### PostCSS設定

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    'tailwindcss',
    'autoprefixer',
    'postcss-preset-env',
    'cssnano',
    [
      '@fullhuman/postcss-purgecss',
      {
        content: ['./src/**/*.{js,jsx,ts,tsx,html}'],
        defaultExtractor: content => content.match(/[\w-/:]+(?<!:)/g) || [],
        safelist: ['html', 'body']
      }
    ]
  ]
}
```

### Stylelint設定

```javascript
// .stylelintrc.js
module.exports = {
  extends: [
    'stylelint-config-standard',
    'stylelint-config-tailwindcss'
  ],
  plugins: [
    'stylelint-order'
  ],
  rules: {
    'order/properties-alphabetical-order': true,
    'selector-class-pattern': '^[a-z][a-zA-Z0-9]*$',
    'custom-property-pattern': '^[a-z][a-zA-Z0-9]*(-[a-zA-Z0-9]+)*$',
    'declaration-no-important': true,
    'selector-max-id': 0,
    'color-hex-length': 'short',
    'color-hex-case': 'lower'
  }
}
```

### VS Code設定

```json
// .vscode/settings.json
{
  "css.validate": false,
  "scss.validate": false,
  "tailwindCSS.includeLanguages": {
    "typescript": "html",
    "typescriptreact": "html"
  },
  "tailwindCSS.experimental.classRegex": [
    "cva\\(([^)]*)\\)",
    "[\"'`]([^\"'`]*).*?[\"'`]"
  ],
  "editor.quickSuggestions": {
    "strings": true
  }
}
```

---

**このガイドラインは2025年のCSS/スタイリング開発における最新のベストプラクティスを反映し、モダンなWebアプリケーション開発を支援します。**
