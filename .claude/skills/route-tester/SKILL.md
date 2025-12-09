---
name: route-tester
description: JWT 認証を使用して RealWorld プロジェクトで API routes をテスト。この skill は API エンドポイントテスト、route 機能検証、認証問題デバッグ時に使用します。
---

# RealWorld Route Tester Skill

## 目的
この skill は JWT 認証を使用して RealWorld (Conduit) プロジェクトで API routes をテストするためのパターンを提供します。

## この Skill 使用時点
- 新 API エンドポイントテスト
- 変更後の route 機能検証
- 認証問題デバッグ
- POST/PUT/DELETE 操作テスト
- リクエスト/レスポンスデータ検証

## プロジェクト認証概要

RealWorld プロジェクトが使用するもの:
- **JWT** トークンベース認証
- **Token ヘッダー**: `Authorization: Token <jwt>`（Bearer ではない）
- **トークン保存**: localStorage
- **パスワードハッシュ**: bcrypt

## API エンドポイント一覧

### 認証 API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| POST | `/api/users/login` | ログイン | 不要 |
| POST | `/api/users` | ユーザー登録 | 不要 |
| GET | `/api/user` | 現在のユーザー取得 | 必須 |
| PUT | `/api/user` | ユーザー情報更新 | 必須 |

### プロフィール API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/profiles/:username` | プロフィール取得 | 任意 |
| POST | `/api/profiles/:username/follow` | フォロー | 必須 |
| DELETE | `/api/profiles/:username/follow` | アンフォロー | 必須 |

### 記事 API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/articles` | 記事一覧取得 | 任意 |
| GET | `/api/articles/feed` | フィード取得 | 必須 |
| GET | `/api/articles/:slug` | 記事詳細取得 | 任意 |
| POST | `/api/articles` | 記事作成 | 必須 |
| PUT | `/api/articles/:slug` | 記事更新 | 必須 |
| DELETE | `/api/articles/:slug` | 記事削除 | 必須 |

### コメント API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/articles/:slug/comments` | コメント一覧取得 | 任意 |
| POST | `/api/articles/:slug/comments` | コメント作成 | 必須 |
| DELETE | `/api/articles/:slug/comments/:id` | コメント削除 | 必須 |

### お気に入り API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| POST | `/api/articles/:slug/favorite` | お気に入り追加 | 必須 |
| DELETE | `/api/articles/:slug/favorite` | お気に入り解除 | 必須 |

### タグ API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/tags` | タグ一覧取得 | 不要 |

## テスト方法

### 方法 1: curl コマンド（推奨）

#### ユーザー登録

```bash
curl -X POST http://localhost:3001/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "username": "testuser",
      "email": "test@example.com",
      "password": "password123"
    }
  }'
```

#### ログイン

```bash
curl -X POST http://localhost:3001/api/users/login \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "test@example.com",
      "password": "password123"
    }
  }'
```

レスポンスから `token` を取得して以降のリクエストで使用。

#### 認証が必要なリクエスト

```bash
# 現在のユーザー取得
curl http://localhost:3001/api/user \
  -H "Authorization: Token <YOUR_JWT_TOKEN>"

# 記事作成
curl -X POST http://localhost:3001/api/articles \
  -H "Content-Type: application/json" \
  -H "Authorization: Token <YOUR_JWT_TOKEN>" \
  -d '{
    "article": {
      "title": "Test Article",
      "description": "This is a test",
      "body": "Article body content",
      "tagList": ["test", "example"]
    }
  }'
```

### 方法 2: HTTPie

```bash
# ログイン
http POST localhost:3001/api/users/login \
  user:='{"email":"test@example.com","password":"password123"}'

# 認証付きリクエスト
http localhost:3001/api/user \
  "Authorization:Token <YOUR_JWT_TOKEN>"
```

### 方法 3: VS Code REST Client

`.http` ファイルを作成:

```http
### ユーザー登録
POST http://localhost:3001/api/users
Content-Type: application/json

{
  "user": {
    "username": "testuser",
    "email": "test@example.com",
    "password": "password123"
  }
}

### ログイン
POST http://localhost:3001/api/users/login
Content-Type: application/json

{
  "user": {
    "email": "test@example.com",
    "password": "password123"
  }
}

### 現在のユーザー取得
GET http://localhost:3001/api/user
Authorization: Token {{token}}

### 記事一覧取得
GET http://localhost:3001/api/articles?limit=10&offset=0

### 記事作成
POST http://localhost:3001/api/articles
Content-Type: application/json
Authorization: Token {{token}}

{
  "article": {
    "title": "Test Article",
    "description": "This is a test",
    "body": "Article body content",
    "tagList": ["test"]
  }
}
```

## サービスポート

| サービス | ポート | Base URL |
|---------|------|----------|
| Frontend | 5173 | http://localhost:5173 |
| Backend | 3001 | http://localhost:3001 |

## 一般的なテストパターン

### 認証フロー完全テスト

```bash
# 1. ユーザー登録
curl -X POST http://localhost:3001/api/users \
  -H "Content-Type: application/json" \
  -d '{"user":{"username":"newuser","email":"new@example.com","password":"password123"}}'

# 2. ログイン
TOKEN=$(curl -s -X POST http://localhost:3001/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"new@example.com","password":"password123"}}' \
  | jq -r '.user.token')

# 3. 認証確認
curl http://localhost:3001/api/user \
  -H "Authorization: Token $TOKEN"
```

### 記事 CRUD テスト

```bash
# 記事作成
SLUG=$(curl -s -X POST http://localhost:3001/api/articles \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"article":{"title":"Test","description":"Test desc","body":"Test body","tagList":["test"]}}' \
  | jq -r '.article.slug')

# 記事取得
curl http://localhost:3001/api/articles/$SLUG

# 記事更新
curl -X PUT http://localhost:3001/api/articles/$SLUG \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"article":{"title":"Updated Title"}}'

# 記事削除
curl -X DELETE http://localhost:3001/api/articles/$SLUG \
  -H "Authorization: Token $TOKEN"
```

### コメントテスト

```bash
# コメント作成
curl -X POST http://localhost:3001/api/articles/$SLUG/comments \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"comment":{"body":"Great article!"}}'

# コメント一覧取得
curl http://localhost:3001/api/articles/$SLUG/comments

# コメント削除
curl -X DELETE http://localhost:3001/api/articles/$SLUG/comments/1 \
  -H "Authorization: Token $TOKEN"
```

### フォロー/お気に入りテスト

```bash
# フォロー
curl -X POST http://localhost:3001/api/profiles/username/follow \
  -H "Authorization: Token $TOKEN"

# アンフォロー
curl -X DELETE http://localhost:3001/api/profiles/username/follow \
  -H "Authorization: Token $TOKEN"

# お気に入り追加
curl -X POST http://localhost:3001/api/articles/$SLUG/favorite \
  -H "Authorization: Token $TOKEN"

# お気に入り解除
curl -X DELETE http://localhost:3001/api/articles/$SLUG/favorite \
  -H "Authorization: Token $TOKEN"
```

## リクエスト/レスポンス形式

### リクエストラッパー

RealWorld API はリクエストボディをラッパーオブジェクトで包む:

```json
// ユーザー関連
{ "user": { ... } }

// 記事関連
{ "article": { ... } }

// コメント関連
{ "comment": { ... } }
```

### レスポンス形式

```json
// ユーザー
{
  "user": {
    "email": "test@example.com",
    "token": "jwt.token.here",
    "username": "testuser",
    "bio": null,
    "image": null
  }
}

// 記事
{
  "article": {
    "slug": "test-article-abc123",
    "title": "Test Article",
    "description": "Description",
    "body": "Body content",
    "tagList": ["test"],
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z",
    "favorited": false,
    "favoritesCount": 0,
    "author": {
      "username": "testuser",
      "bio": null,
      "image": null,
      "following": false
    }
  }
}

// エラー
{
  "errors": {
    "body": ["can't be blank"]
  }
}
```

## 失敗したテストのデバッグ

### 401 Unauthorized

**考えられる原因**:
1. トークンが無効または期限切れ
2. Authorization ヘッダー形式が間違い（`Bearer` ではなく `Token` を使用）
3. トークンが設定されていない

**解決策**:
```bash
# 再ログインしてトークン取得
curl -X POST http://localhost:3001/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"test@example.com","password":"password123"}}'

# ヘッダー形式確認: "Authorization: Token <jwt>"
```

### 403 Forbidden

**考えられる原因**:
1. リソースの所有者ではない（記事編集/削除）
2. 権限がない操作

**解決策**:
- 正しいユーザーでログインしているか確認
- リソースの所有者情報を確認

### 404 Not Found

**考えられる原因**:
1. URL パスが間違い
2. リソースが存在しない（記事 slug、ユーザー名）

**解決策**:
1. API パス確認（`/api/articles` など）
2. slug や username のスペル確認

### 422 Unprocessable Entity

**考えられる原因**:
1. バリデーションエラー
2. 必須フィールド欠落
3. 重複データ（email、username）

**解決策**:
```bash
# エラーレスポンスを確認
curl -v -X POST http://localhost:3001/api/users \
  -H "Content-Type: application/json" \
  -d '{"user":{"username":"","email":"invalid","password":"123"}}'

# レスポンス例:
# {"errors":{"username":["can't be blank"],"email":["is invalid"],"password":["is too short"]}}
```

## データベース検証

テスト後にデータベース状態を確認:

```bash
# SQLite データベースに接続
cd backend
npx prisma studio

# または CLI で確認
sqlite3 prisma/dev.db

# テーブル確認
sqlite> .tables
sqlite> SELECT * FROM User;
sqlite> SELECT * FROM Article;
sqlite> SELECT * FROM Comment;
```

## テストチェックリスト

Route テスト前:
- [ ] バックエンドサーバーが起動中か確認 (`pnpm --filter backend dev`)
- [ ] 正しいポート確認 (3001)
- [ ] API パス確認 (`/api/...`)
- [ ] リクエストボディのラッパー形式確認
- [ ] 認証ヘッダー形式確認 (`Token` not `Bearer`)
- [ ] Content-Type ヘッダー設定

Route テスト後:
- [ ] レスポンスステータスコード確認
- [ ] レスポンスボディ形式確認
- [ ] データベース変更確認（該当する場合）
- [ ] エラーメッセージ確認（失敗時）

## 関連ファイル

- `backend/src/routes/` - API ルート定義
- `backend/src/controllers/` - コントローラー
- `backend/src/middleware/auth.ts` - 認証ミドルウェア
- `backend/prisma/schema.prisma` - データベーススキーマ
- `docs/API-Spec.md` - 完全な API 仕様

## 関連 Skills

- データベースパターンに **backend-dev-guidelines** 使用
- エラー追跡に **error-tracking** 使用
