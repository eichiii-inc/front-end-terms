# HTML マークアップガイド

## 基本原則（2025年版）

### セマンティックHTMLの優先
- 意味のあるHTML要素を使用し、アクセシビリティとSEOを向上させる
- `div`や`span`よりもセマンティック要素を優先する

### アクセシビリティファースト
- ARIA属性とセマンティックHTMLを組み合わせる
- キーボードナビゲーションとスクリーンリーダーの対応を必須とする

### パフォーマンス最適化
- Core Web Vitalsを意識したマークアップ
- リソースの効率的な読み込み

## スタイルルール

### インデント
<!-- markdownlint-disable -->
<a name="html-indent">インデント</a>
<!-- markdownlint-enable -->

2スペース（従来通り）

```html
<!-- 良い例 -->
<section>
  <article>記事</article>
  <article>記事</article>
</section>
```

### セマンティックHTML（2025年重要）

```html
<!-- 悪い例 -->
<div class="header">
  <div class="nav">
    <div class="nav-item">ホーム</div>
  </div>
</div>

<!-- 良い例 -->
<header>
  <nav>
    <a href="/" aria-current="page">ホーム</a>
  </nav>
</header>
```

### Web Components対応

```html
<!-- カスタム要素の使用を推奨 -->
<user-profile user-id="123"></user-profile>
<data-table sortable filterable></data-table>
```

## アクセシビリティ規約

### ARIA属性の適切な使用

```html
<!-- ボタンの例 -->
<button aria-label="メニューを開く" aria-expanded="false">
  <span class="icon-menu"></span>
</button>

<!-- ランドマークの例 -->
<main role="main" aria-labelledby="main-heading">
  <h1 id="main-heading">メインコンテンツ</h1>
</main>
```

### フォームのアクセシビリティ

```html
<form>
  <div class="form-group">
    <label for="email">メールアドレス</label>
    <input 
      type="email" 
      id="email" 
      name="email" 
      required 
      aria-describedby="email-help"
    >
    <div id="email-help">有効なメールアドレスを入力してください</div>
  </div>
</form>
```

## PWA対応

### 必須メタタグ

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="ページの説明">
  
  <!-- PWA対応 -->
  <link rel="manifest" href="/manifest.json">
  <meta name="theme-color" content="#000000">
  
  <!-- Apple PWA対応 -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="default">
  <link rel="apple-touch-icon" href="/icon-192x192.png">
</head>
```

## パフォーマンス最適化

### 画像の最適化

```html
<!-- レスポンシブ画像 -->
<picture>
  <source media="(min-width: 768px)" srcset="image-large.webp" type="image/webp">
  <source media="(min-width: 768px)" srcset="image-large.jpg">
  <source srcset="image-small.webp" type="image/webp">
  <img src="image-small.jpg" alt="説明" loading="lazy">
</picture>

<!-- 単純な画像 -->
<img 
  src="image.jpg" 
  alt="説明" 
  loading="lazy" 
  width="800" 
  height="600"
>
```

### リソースのプリロード

```html
<head>
  <!-- 重要なリソースのプリロード -->
  <link rel="preload" href="critical.css" as="style">
  <link rel="preload" href="main.js" as="script">
  
  <!-- 次のページのプリフェッチ -->
  <link rel="prefetch" href="/next-page.html">
</head>
```

## 新しいHTML機能（2025年）

### コンテナクエリ対応

```html
<div class="card-container">
  <div class="card" style="container-type: inline-size;">
    <h2>カードタイトル</h2>
    <p>コンテンツ</p>
  </div>
</div>
```

### View Transitions対応

```html
<head>
  <meta name="view-transition" content="same-origin">
</head>

<main style="view-transition-name: main-content;">
  <!-- メインコンテンツ -->
</main>
```

## セキュリティ

### Content Security Policy

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' 'unsafe-inline';">
```

### 安全なリンク

```html
<!-- 外部リンクのセキュリティ -->
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  外部サイト
</a>
```

## 禁止事項

- `<div>`や`<span>`の過度な使用
- インラインスタイルの多用
- アクセシビリティ属性の省略
- 適切でないheading構造（h1→h3など）
- `target="_blank"`でのrel属性省略

## 推奨ツール

- HTML5 Validator
- axe-core（アクセシビリティチェック）
- Lighthouse（パフォーマンス測定）
- WAVE（アクセシビリティ評価）
