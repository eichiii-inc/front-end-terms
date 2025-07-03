# Eichiii inc Frontend コード規約 (2025年版)

2025年のフロントエンドトレンドに対応したコーディング規約

## 概要

この規約は2025年の最新フロントエンド開発トレンドに基づいて策定されています：

- **TypeScript ファースト**: 型安全性を最優先とした開発
- **パフォーマンス重視**: Core Web Vitals と UX の最適化
- **アクセシビリティ標準**: WCAG準拠とインクルーシブデザイン
- **モダンCSS活用**: Container Queries、CSS Grid、Logical Properties
- **React Server Components**: サーバーサイドレンダリングの活用
- **AI統合開発**: 開発効率向上のためのAIツール活用

## 対象技術

| 技術 | バージョン | 主な特徴 |
|------|------------|----------|
| HTML | HTML5+ | セマンティック重視、PWA対応 |
| CSS | CSS3+ | Container Queries、Cascade Layers |
| JavaScript | ES2025+ | TypeScript優先、関数型プログラミング |
| TypeScript | 5.7+ | 厳密な型安全性、最新機能活用 |
| React | 19+ | Server Components、新Hooks |
| Vue | 3.5+ | 古い規約のまま維持 |

## 基本原則

### 1. 型安全性の最大化
- すべての新規プロジェクトでTypeScriptを使用
- `any`型の使用を禁止
- 厳密な型定義による品質向上

### 2. パフォーマンス最優先
- **目標指標**:
  - First Contentful Paint: < 1.8秒
  - Largest Contentful Paint: < 2.5秒
  - Time to Interactive: < 3秒
  - Cumulative Layout Shift: < 0.1

### 3. アクセシビリティファースト
- WCAG 2.1 AA準拠
- セマンティックHTML の徹底
- ARIA属性の適切な使用

### 4. 開発者体験の向上
- AI開発ツールの積極活用
- 自動化されたテスト・リント
- 効率的なワークフロー

## 目次

### [HTML](html/) - セマンティック & アクセシビリティ
- セマンティックHTMLとアクセシビリティ対応
- PWA対応とパフォーマンス最適化
- Web Components準拠
- Core Web Vitals最適化

### [CSS](css/) - モダンCSS & パフォーマンス
- Container Queries、CSS Grid活用
- Logical Properties、Cascade Layers
- 新しいcolor関数（OKLCH、color-mix）
- アクセシビリティ対応（motion sensitivity等）

### [JavaScript](js/) - ES2025+ & 関数型
- ES2025新機能（Pattern Matching、Iterator Helpers）
- TypeScriptファースト、関数型プログラミング
- 非同期処理最適化、パフォーマンス重視
- AI活用とWeb APIs統合

### [TypeScript](typescript/) - 厳密な型安全性
- TypeScript 5.7+新機能活用
- 高度な型パターン、ユーティリティタイプ
- API型定義、状態管理型安全性
- パフォーマンス最適化とベストプラクティス

### [React](react/) - Server Components & React 19
- React Server Components (RSC)
- React 19新機能（useOptimistic、useFormStatus）
- TypeScript統合とパフォーマンス最適化
- テスト戦略とアクセシビリティ

### [Vue](vue/) - 従来規約維持
- Vue 3.5対応（既存規約維持）
- ※ユーザー要求により更新対象外

## 推奨開発環境

### 必須ツール
- **Node.js**: 20.x以上
- **TypeScript**: 5.7以上
- **ESLint**: 最新版
- **Prettier**: 最新版

### 推奨ツール
- **Vite**: 高速ビルドツール
- **Vitest**: 高速テストランナー
- **Biome**: 高速linter/formatter
- **Lighthouse**: パフォーマンス測定

### AI開発支援
- **GitHub Copilot**: コード生成支援
- **Cursor**: AI統合エディタ
- **Claude/ChatGPT**: 技術相談・レビュー

## プロジェクト設定例

```json
{
  "name": "frontend-project-2025",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "test": "vitest",
    "lint": "biome check src/",
    "format": "biome format --write src/"
  },
  "dependencies": {
    "react": "^19.0.0",
    "typescript": "^5.7.0"
  }
}
```

## 品質基準

### コード品質
- **TypeScript**: strict mode有効
- **テストカバレッジ**: 80%以上
- **ESLint**: エラー0、警告5以下
- **Bundle Size**: Initial < 200KB

### パフォーマンス
- **Lighthouse Score**: 90以上
- **Core Web Vitals**: すべて緑
- **メモリ使用量**: < 50MB

### アクセシビリティ
- **axe-core**: 違反0件
- **コントラスト比**: AAA準拠
- **キーボードナビゲーション**: 完全対応

## 更新履歴

### 2025年版の主な変更点
- TypeScriptファーストアプローチの採用
- React Server Componentsの導入
- モダンCSS機能の全面採用
- AI開発ツールの統合
- アクセシビリティ標準の強化
- パフォーマンス指標の明確化

---

**最終更新**: 2025年1月  
**対象プロジェクト**: すべての新規フロントエンドプロジェクト  
**適用開始**: 即座（既存プロジェクトは段階的移行）
