# RealWorld (Conduit) - 技術スタック

## 1. 概要

本プロジェクトは、モノレポ構成で Frontend と Backend を統合管理する。
Claude Code との効率的な開発を最優先とし、検証済みのツールチェーンを採用する。

---

## 2. フロントエンド技術スタック

### 2.1 コアフレームワーク

| 技術 | バージョン | 用途 |
|------|-----------|------|
| **React** | 19.x | UI フレームワーク |
| **TypeScript** | 5.x | 型安全性 |
| **Vite** | 6.x | ビルドツール・開発サーバー |

### 2.2 状態管理・データフェッチ

| 技術 | バージョン | 用途 |
|------|-----------|------|
| **TanStack Query** | 5.x | サーバー状態管理、データフェッチ |
| **TanStack Router** | 1.x | ファイルベースルーティング |

### 2.3 UI ライブラリ

| 技術 | バージョン | 用途 |
|------|-----------|------|
| **MUI (Material UI)** | 7.x | UIコンポーネント |
| **@emotion/react** | 11.x | CSS-in-JS |
| **@emotion/styled** | 11.x | スタイルドコンポーネント |

### 2.4 ユーティリティ

| 技術 | 用途 |
|------|------|
| **react-markdown** | Markdown レンダリング |
| **date-fns** | 日付フォーマット |
| **zod** | スキーマバリデーション |

### 2.5 選定理由

- **React 19**: 最新の Suspense、並行レンダリング機能を活用
- **TanStack Query**: useSuspenseQuery による宣言的データフェッチ
- **TanStack Router**: 型安全なファイルベースルーティング
- **MUI v7**: 最新のデザインシステム、良好なアクセシビリティ
- **Vite**: 高速な HMR、最適化されたビルド

---

## 3. バックエンド技術スタック

### 3.1 ランタイム・フレームワーク

| 技術 | バージョン | 用途 |
|------|-----------|------|
| **Node.js** | 20.x LTS | JavaScript ランタイム |
| **Express** | 4.x | Web フレームワーク |
| **TypeScript** | 5.x | 型安全性 |

### 3.2 データベース・ORM

| 技術 | バージョン | 用途 |
|------|-----------|------|
| **SQLite** | 3.x | データベース |
| **Prisma** | 5.x | ORM |

### 3.3 認証・セキュリティ

| 技術 | 用途 |
|------|------|
| **jsonwebtoken** | JWT トークン生成・検証 |
| **bcryptjs** | パスワードハッシュ化 |
| **cors** | CORS 設定 |
| **helmet** | セキュリティヘッダー |

### 3.4 バリデーション・ユーティリティ

| 技術 | 用途 |
|------|------|
| **zod** | リクエストバリデーション |
| **slugify** | URL スラッグ生成 |
| **uuid** | 一意識別子生成 |

### 3.5 アーキテクチャパターン

**レイヤードアーキテクチャ**を採用:

```
Routes → Controllers → Services → Repositories → Database
```

| レイヤー | 責務 |
|---------|------|
| **Routes** | エンドポイント定義、ミドルウェア適用 |
| **Controllers** | リクエスト/レスポンス処理、バリデーション |
| **Services** | ビジネスロジック |
| **Repositories** | データアクセス（Prisma 操作） |

### 3.6 選定理由

- **Express**: シンプルで柔軟、豊富なエコシステム
- **Prisma**: 型安全な ORM、マイグレーション管理
- **SQLite**: 開発環境でのセットアップ簡略化
- **レイヤードアーキテクチャ**: 関心の分離、テスタビリティ向上

---

## 4. 共通ツール

### 4.1 開発ツール

| 技術 | 用途 |
|------|------|
| **pnpm** | パッケージマネージャー（ワークスペース対応） |
| **ESLint** | コード品質チェック |
| **Prettier** | コードフォーマット |
| **tsx** | TypeScript 実行（開発時） |

### 4.2 テスト

| 技術 | 用途 |
|------|------|
| **Vitest** | ユニットテスト・統合テスト |
| **Testing Library** | React コンポーネントテスト |
| **Supertest** | API エンドポイントテスト |

### 4.3 プロセス管理・監視

| 技術 | 用途 |
|------|------|
| **PM2** | プロセス管理、ログ監視 |
| **Sentry** | エラートラッキング、パフォーマンス監視 |

---

## 5. ディレクトリ構成

```
claude-code-demo/
├── docs/                          # プロジェクトドキュメント
│   ├── PRD.md
│   ├── TechStack.md
│   ├── Architecture.md
│   └── API-Spec.md
│
├── frontend/                      # フロントエンドアプリケーション
│   ├── src/
│   │   ├── features/              # 機能別モジュール
│   │   │   ├── auth/              # 認証機能
│   │   │   ├── articles/          # 記事機能
│   │   │   ├── comments/          # コメント機能
│   │   │   ├── profiles/          # プロフィール機能
│   │   │   └── tags/              # タグ機能
│   │   ├── components/            # 共通コンポーネント
│   │   │   ├── layout/            # レイアウト
│   │   │   └── ui/                # UI パーツ
│   │   ├── hooks/                 # カスタムフック
│   │   ├── lib/                   # ユーティリティ
│   │   │   ├── api/               # API クライアント
│   │   │   └── utils/             # ヘルパー関数
│   │   ├── routes/                # ルート定義（TanStack Router）
│   │   ├── types/                 # 型定義
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── public/
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── package.json
│
├── backend/                       # バックエンドアプリケーション
│   ├── src/
│   │   ├── routes/                # ルート定義
│   │   ├── controllers/           # コントローラー
│   │   ├── services/              # ビジネスロジック
│   │   ├── repositories/          # データアクセス
│   │   ├── middleware/            # ミドルウェア
│   │   │   ├── auth.ts            # 認証ミドルウェア
│   │   │   ├── errorHandler.ts    # エラーハンドリング
│   │   │   └── validation.ts      # バリデーション
│   │   ├── lib/                   # ユーティリティ
│   │   │   ├── prisma.ts          # Prisma クライアント
│   │   │   └── jwt.ts             # JWT ユーティリティ
│   │   ├── types/                 # 型定義
│   │   └── index.ts               # エントリーポイント
│   ├── prisma/
│   │   ├── schema.prisma          # データベーススキーマ
│   │   └── migrations/            # マイグレーション
│   ├── tsconfig.json
│   └── package.json
│
├── .claude/                       # Claude Code 設定（既存）
├── dev/                           # Dev Docs（既存）
├── pnpm-workspace.yaml            # ワークスペース設定
├── package.json                   # ルート package.json
├── .prettierrc                    # Prettier 設定
├── .eslintrc.js                   # ESLint 設定
└── CLAUDE.md                      # プロジェクト設定（既存）
```

---

## 6. 開発環境セットアップ

### 6.1 前提条件

- Node.js 20.x 以上
- pnpm 8.x 以上

### 6.2 初期セットアップ

```bash
# 依存関係インストール
pnpm install

# データベースマイグレーション
pnpm --filter backend prisma migrate dev

# 開発サーバー起動（フロントエンド + バックエンド）
pnpm dev
```

### 6.3 利用可能なスクリプト

| コマンド | 説明 |
|---------|------|
| `pnpm dev` | 開発サーバー起動（全体） |
| `pnpm --filter frontend dev` | フロントエンドのみ起動 |
| `pnpm --filter backend dev` | バックエンドのみ起動 |
| `pnpm build` | プロダクションビルド |
| `pnpm test` | テスト実行 |
| `pnpm lint` | Lint チェック |
| `pnpm format` | コードフォーマット |

---

## 7. 環境変数

### 7.1 バックエンド (.env)

```env
# Database
DATABASE_URL="file:./dev.db"

# JWT
JWT_SECRET="your-secret-key"
JWT_EXPIRES_IN="7d"

# Server
PORT=3000
NODE_ENV="development"

# Sentry (Optional)
SENTRY_DSN=""
```

### 7.2 フロントエンド (.env)

```env
# API
VITE_API_URL="http://localhost:3000/api"
```

---

## 8. 依存関係一覧

### 8.1 フロントエンド

```json
{
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "@tanstack/react-query": "^5.0.0",
    "@tanstack/react-router": "^1.0.0",
    "@mui/material": "^7.0.0",
    "@emotion/react": "^11.0.0",
    "@emotion/styled": "^11.0.0",
    "react-markdown": "^9.0.0",
    "date-fns": "^3.0.0",
    "zod": "^3.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vite": "^6.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "vitest": "^2.0.0",
    "@testing-library/react": "^16.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0"
  }
}
```

### 8.2 バックエンド

```json
{
  "dependencies": {
    "express": "^4.21.0",
    "@prisma/client": "^5.0.0",
    "jsonwebtoken": "^9.0.0",
    "bcryptjs": "^2.4.0",
    "cors": "^2.8.0",
    "helmet": "^8.0.0",
    "zod": "^3.0.0",
    "slugify": "^1.6.0",
    "uuid": "^10.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "prisma": "^5.0.0",
    "tsx": "^4.0.0",
    "vitest": "^2.0.0",
    "supertest": "^7.0.0",
    "@types/express": "^5.0.0",
    "@types/node": "^22.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0"
  }
}
```

---

## 9. Claude Code 統合

### 9.1 既存スキル活用

本プロジェクトは既存の Claude Code スキルと統合:

- **backend-dev-guidelines**: Express/TypeScript パターン
- **frontend-dev-guidelines**: React/MUI パターン
- **error-tracking**: Sentry 統合
- **route-tester**: API テスト

### 9.2 開発フロー

1. 機能要件の確認（PRD 参照）
2. API 設計（API-Spec 参照）
3. バックエンド実装（レイヤードアーキテクチャ）
4. フロントエンド実装（機能ベースモジュール）
5. テスト実行
6. Dev Docs 更新
