# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

RealWorld (Conduit) - Medium.com クローンのソーシャルブログプラットフォーム。フロントエンドとバックエンドを統合したモノレポ構成。

## 開発コマンド

```bash
# 依存関係インストール
pnpm install

# データベースマイグレーション
pnpm --filter backend prisma migrate dev

# 開発サーバー起動（全体）
pnpm dev

# 個別起動
pnpm --filter frontend dev    # フロントエンドのみ
pnpm --filter backend dev     # バックエンドのみ

# テスト
pnpm test                     # 全テスト
pnpm --filter backend test    # バックエンドテストのみ
pnpm --filter frontend test   # フロントエンドテストのみ

# リント・フォーマット
pnpm lint
pnpm format

# ビルド
pnpm build
```

## アーキテクチャ

### モノレポ構成
```
├── frontend/          # React 19 + TypeScript + MUI v7 + TanStack Query/Router
├── backend/           # Express + TypeScript + Prisma + SQLite
└── docs/              # プロジェクト文書（PRD, TechStack, Architecture, API-Spec）
```

### バックエンド - レイヤードアーキテクチャ
```
Routes → Controllers → Services → Repositories → Database (Prisma)
```

各レイヤーの責務:
- **Routes**: エンドポイント定義、ミドルウェア適用
- **Controllers**: リクエスト/レスポンス処理、バリデーション
- **Services**: ビジネスロジック
- **Repositories**: Prisma によるデータアクセス

### フロントエンド - 機能ベースモジュール
```
src/features/     # auth, articles, comments, profiles, tags
src/components/   # 共通コンポーネント（layout, ui）
src/routes/       # TanStack Router ルート定義
src/lib/          # API クライアント、ユーティリティ
```

## 技術スタック

| 領域 | 技術 |
|------|------|
| Frontend | React 19, TypeScript, MUI v7, TanStack Query 5, TanStack Router, Vite |
| Backend | Node.js 20, Express 4, TypeScript, Prisma 5, SQLite |
| 認証 | JWT (localStorage 保存) |
| テスト | Vitest, Testing Library, Supertest |

## Claude Code スキル統合

### 自動活性化スキル
- **backend-dev-guidelines**: `backend/**/*.ts` ファイル編集時
- **frontend-dev-guidelines**: `frontend/src/**/*.tsx` ファイル編集時（ブロック設定）
- **error-tracking**: Sentry 統合パターン
- **route-tester**: API ルートテスト

### 重要な制約
- MUI v7 必須: Grid は `size={{}}` prop を使用（xs, sm 禁止）、makeStyles 使用禁止
- バックエンドは必ずレイヤードアーキテクチャを遵守
- すべての Prisma 操作は Repository レイヤーで実行

## プロジェクト文書

詳細は `docs/` フォルダ参照:
- `PRD.md` - 機能要件、画面設計
- `TechStack.md` - 技術選定理由、ディレクトリ構成
- `Architecture.md` - システム設計、ERD、認証フロー
- `API-Spec.md` - API エンドポイント仕様

## 言語設定

- このプロジェクトの文書はすべて日本語で作成すること
- 日本語でコミュニケーションする事。