# Eichiii inc Frontend コード規約 2025

**2025年版：モダンフロントエンド開発のための包括的ガイドライン**

## 🎯 基本方針

### TypeScript First
- **すべての新規プロジェクトはTypeScriptを標準とする**
- JavaScriptからの段階的移行を推奨
- 型安全性を最優先に考慮した設計

### パフォーマンス重視
- Core Web Vitalsを常に意識した開発
- バンドルサイズの最適化
- レンダリング性能の最適化

### アクセシビリティファースト
- WCAG 2.1 AA準拠を標準とする
- セマンティックHTMLの徹底
- キーボードナビゲーション対応

## 📚 技術スタック

### [TypeScript](typescript/) - 必須
- 型安全性とコード品質の向上
- IDE支援による開発効率向上
- TypeScript 5.7+対応

### [React](react/) - サーバーコンポーネント対応
- React 19の最新機能活用
- Server Componentsパターン
- 新しいHooks（use、useFormStatus、useActionState、useOptimistic）

### [Vue.js](vue/) - Composition API標準
- Vue 3 Composition API
- `<script setup>`構文必須
- Piniaによる状態管理

### [CSS/Styling](css/) - Tailwind CSS中心
- Tailwind CSS utilityファースト
- CSS-in-JSライブラリ併用
- レスポンシブデザイン
- Container Queries対応

### [JavaScript](js/) - ES2023+
- モダンJavaScript機能
- Web標準API活用
- TypeScript移行推奨

### [HTML](html/) - セマンティック重視
- HTML5セマンティック要素
- アクセシビリティ属性

## 🛠️ 開発ツール

### ビルドツール
- **Vite 6**: 高速開発サーバー（推奨）
- **Next.js 15**: Reactフルスタック開発
- **Nuxt.js 4**: Vue.jsフルスタック開発
- **Bun**: 高速JavaScript実行環境

### コード品質
- **ESLint**: Flat Config対応
- **Prettier**: コードフォーマット
- **Biome**: 高速linter/formatter（積極採用）
- **TypeScript**: 静的型チェック

### テスト
- **Vitest**: 単体テスト（推奨）
- **Playwright**: E2Eテスト
- **Testing Library**: コンポーネントテスト
- **Chromatic**: ビジュアルリグレッションテスト

### AI駆動開発支援
- **GitHub Copilot**: AIペアプログラミング
- **Cursor Composer**: コンテキスト対応コード生成
- **v0.dev**: AIコンポーネント生成
- **Claude/GPT Integration**: コードレビュー・リファクタリング支援

## 🏗️ アーキテクチャパターン

### コンポーネント駆動開発
- 再利用可能なコンポーネント設計
- Atomic Designの考え方
- Storybookによるコンポーネントドキュメント
- Design System構築

### マイクロフロントエンド（大規模アプリ）
- **Module Federation**: Webpackベース
- **Single-SPA**: フレームワーク非依存
- **Qiankun**: 企業級マイクロフロントエンド
- 独立したデプロイとスケーリング
- チーム独立性の確保

### PWA 2025標準
- **Service Worker API**: オフライン対応
- **Web App Manifest**: アプリライク体験
- **Background Sync**: オフライン同期
- **Web Push Notifications**: リアルタイム通知
- **Web Share API**: ネイティブ共有
- **Install Prompts**: アプリインストール促進

### エッジコンピューティング
- **Cloudflare Workers**: エッジでのJavaScript実行
- **Vercel Edge Functions**: グローバル分散実行
- **Netlify Edge Functions**: 低レイテンシ処理
- **AWS Lambda@Edge**: CDNエッジ処理

## ⚡ パフォーマンス指針

### Core Web Vitals 2025
- **LCP (Largest Contentful Paint)**: 2.5秒以下
- **INP (Interaction to Next Paint)**: 200ms以下（FIDの後継）
- **CLS (Cumulative Layout Shift)**: 0.1以下

### 最適化手法
- **画像最適化**: WebP/AVIF/JPEG XL対応
- **コード分割**: Dynamic Import活用
- **リソース遅延読み込み**: Intersection Observer
- **CDN活用**: エッジキャッシュ最適化
- **WebAssembly**: 計算集約的処理の高速化
- **HTTP/3**: 最新プロトコル対応

### WebAssembly活用
```javascript
// 高性能計算処理の例
const wasmModule = await WebAssembly.instantiateStreaming(
  fetch('/path/to/module.wasm')
)

// 画像処理、数値計算、暗号化などに活用
const result = wasmModule.instance.exports.processData(inputData)
```

## ♿ アクセシビリティ要件

### 必須対応
- セマンティックHTML使用
- 適切なARIA属性
- キーボードナビゲーション
- カラーコントラスト比4.5:1以上
- フォーカス管理
- スクリーンリーダー対応

### 自動テスト
- **axe-core**: 自動アクセシビリティチェック
- **Lighthouse Accessibility**: スコア90以上
- **Pa11y**: CI/CD統合テスト
- **WAVE**: ブラウザ拡張テスト

## 🧪 テスト戦略

### テストピラミッド
1. **単体テスト**: コンポーネント・関数レベル（70%）
2. **統合テスト**: コンポーネント間の連携（20%）
3. **E2Eテスト**: ユーザージャーニー全体（10%）

### 品質指標
- テストカバレッジ80%以上
- クリティカルパス100%カバー
- Visual Regression Test実装

### AIテスト支援
- **GitHub Copilot**: テストコード生成
- **Testim**: AI駆動E2Eテスト
- **Applitools**: AIビジュアルテスト

## 📱 レスポンシブ対応

### ブレークポイント（Tailwind CSS準拠）
- **xs**: 475px～
- **sm**: 640px～
- **md**: 768px～  
- **lg**: 1024px～
- **xl**: 1280px～
- **2xl**: 1536px～

### モバイルファースト
- モバイル体験を最優先に設計
- プログレッシブエンハンスメント
- Container Queries活用

## 🔒 セキュリティ

### 基本対策
- XSS対策（適切なエスケープ）
- CSRF対策
- セキュアな依存関係管理
- 定期的なセキュリティ監査

### Content Security Policy
- 適切なCSP設定
- インライン実行の最小化
- Trusted Types対応

### セキュリティツール
- **Snyk**: 脆弱性スキャン
- **npm audit**: 依存関係監査
- **OWASP ZAP**: セキュリティテスト

## 🌐 国際化（i18n）

### 必須対応
- 文字列の外部化
- 日付・数値フォーマット
- RTL言語対応考慮
- 動的言語切り替え

### 推奨ライブラリ
- **react-i18next**: React国際化
- **vue-i18n**: Vue.js国際化
- **FormatJS**: JavaScript国際化API

## 📈 モニタリング

### パフォーマンス監視
- **Real User Monitoring (RUM)**: 実ユーザー体験計測
- **Core Web Vitals**: 継続的計測
- **Error Tracking**: Sentry/Bugsnag
- **Analytics**: Google Analytics 4

### ユーザビリティ
- **ユーザー行動分析**: Hotjar/FullStory
- **A/Bテスト**: Optimizely/VWO
- **フィードバック収集**: UserVoice/Intercom

## 🤖 AI活用ガイドライン

### 開発フェーズでのAI活用
- **コード生成**: GitHub Copilot、Claude、ChatGPT
- **コードレビュー**: AI駆動レビュー支援
- **バグ検出**: 自動バグ発見ツール
- **テスト生成**: AIテストケース生成

### 運用フェーズでのAI活用
- **パフォーマンス最適化**: AI予測分析
- **ユーザー体験改善**: 行動予測
- **A/Bテスト最適化**: AI駆動実験設計

## 🚀 デプロイ・運用

### CI/CD
- **GitHub Actions**: 自動化ワークフロー
- **Vercel**: フロントエンド特化デプロイ
- **Netlify**: JAMstack最適化
- **Docker**: コンテナ化対応

### 環境構築
- 開発環境の標準化
- **Dev Containers**: 統一開発環境
- Environment Variables管理
- **dotenv**: 環境変数管理

### モニタリング・ログ
- **Datadog**: 統合監視
- **New Relic**: APM監視
- **LogRocket**: セッション記録
- **Honeycomb**: 分散トレーシング

## 📋 チェックリスト

新機能開発時の確認項目：

- [ ] TypeScript型定義完備
- [ ] 単体テスト作成（80%以上カバレッジ）
- [ ] アクセシビリティチェック（axe-core）
- [ ] パフォーマンステスト（Lighthouse 90+）
- [ ] クロスブラウザ確認
- [ ] レスポンシブ対応
- [ ] セキュリティチェック
- [ ] 国際化対応
- [ ] エラーハンドリング
- [ ] コードレビュー完了

### リリース前確認
- [ ] Core Web Vitals基準達成
- [ ] PWA機能テスト
- [ ] エラートラッキング設定
- [ ] モニタリング設定
- [ ] SEO対策完了
- [ ] ログ設定確認

## 🔄 継続的改善

### 定期見直し
- 四半期ごとの規約見直し
- 新技術動向の調査
- チーム振り返り
- メトリクス分析

### 学習・キャッチアップ
- **技術勉強会**: 月次開催
- **外部カンファレンス**: React Conf、Vue Conf、JSConf
- **OSS貢献**: 積極的参加推奨
- **AI学習**: 最新AI開発ツール習得

### 実験・検証
- **新技術PoC**: 四半期1件以上
- **パフォーマンス改善**: 継続的最適化
- **ユーザビリティテスト**: 月次実施

---

**最終更新**: 2025年1月
**次回見直し予定**: 2025年4月

## 🔗 関連リンク

- [TypeScript規約](typescript/)
- [React.js規約](react/)
- [Vue.js規約](vue/)
- [CSS/スタイリング規約](css/)
- [JavaScript規約](js/)
- [HTML規約](html/)

## 📞 サポート

技術的な質問や規約に関する相談は以下まで：
- **Slack**: #frontend-guidelines
- **GitHub**: Issues/Discussions
- **定期MTG**: 毎週火曜 10:00-11:00
