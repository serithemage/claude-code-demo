# RealWorld (Conduit) - API 仕様書

## 1. 概要

### 1.1 ベース URL
```
http://localhost:3000/api
```

### 1.2 認証ヘッダー
```
Authorization: Token jwt.token.here
```

### 1.3 共通レスポンス形式

**成功時**
```json
{
  "user": { ... },      // または "article", "articles", etc.
}
```

**エラー時**
```json
{
  "errors": {
    "body": ["can't be empty"],
    "email": ["has already been taken"]
  }
}
```

---

## 2. 認証 API

### 2.1 ユーザー登録

**エンドポイント**
```
POST /api/users
```

**認証**: 不要

**リクエスト**
```json
{
  "user": {
    "username": "Jacob",
    "email": "jake@jake.jake",
    "password": "jakejake"
  }
}
```

**レスポンス** (201 Created)
```json
{
  "user": {
    "email": "jake@jake.jake",
    "token": "jwt.token.here",
    "username": "Jacob",
    "bio": null,
    "image": null
  }
}
```

**エラー** (422 Unprocessable Entity)
```json
{
  "errors": {
    "email": ["has already been taken"],
    "username": ["has already been taken"]
  }
}
```

---

### 2.2 ログイン

**エンドポイント**
```
POST /api/users/login
```

**認証**: 不要

**リクエスト**
```json
{
  "user": {
    "email": "jake@jake.jake",
    "password": "jakejake"
  }
}
```

**レスポンス** (200 OK)
```json
{
  "user": {
    "email": "jake@jake.jake",
    "token": "jwt.token.here",
    "username": "Jacob",
    "bio": "I work at statefarm",
    "image": null
  }
}
```

**エラー** (401 Unauthorized)
```json
{
  "errors": {
    "email or password": ["is invalid"]
  }
}
```

---

### 2.3 現在のユーザー取得

**エンドポイント**
```
GET /api/user
```

**認証**: 必須

**レスポンス** (200 OK)
```json
{
  "user": {
    "email": "jake@jake.jake",
    "token": "jwt.token.here",
    "username": "Jacob",
    "bio": "I work at statefarm",
    "image": null
  }
}
```

---

### 2.4 ユーザー情報更新

**エンドポイント**
```
PUT /api/user
```

**認証**: 必須

**リクエスト**
```json
{
  "user": {
    "email": "jake@jake.jake",
    "bio": "I like to skateboard",
    "image": "https://i.stack.imgur.com/xHWG8.jpg"
  }
}
```

※ 指定したフィールドのみ更新。password フィールドで新しいパスワードを設定可能。

**レスポンス** (200 OK)
```json
{
  "user": {
    "email": "jake@jake.jake",
    "token": "jwt.token.here",
    "username": "Jacob",
    "bio": "I like to skateboard",
    "image": "https://i.stack.imgur.com/xHWG8.jpg"
  }
}
```

---

## 3. プロフィール API

### 3.1 プロフィール取得

**エンドポイント**
```
GET /api/profiles/:username
```

**認証**: 任意（ログイン時は following フラグを含む）

**レスポンス** (200 OK)
```json
{
  "profile": {
    "username": "jake",
    "bio": "I work at statefarm",
    "image": "https://api.realworld.io/images/smiley-cyrus.jpg",
    "following": false
  }
}
```

---

### 3.2 ユーザーをフォロー

**エンドポイント**
```
POST /api/profiles/:username/follow
```

**認証**: 必須

**リクエスト**: ボディなし

**レスポンス** (200 OK)
```json
{
  "profile": {
    "username": "jake",
    "bio": "I work at statefarm",
    "image": "https://api.realworld.io/images/smiley-cyrus.jpg",
    "following": true
  }
}
```

---

### 3.3 フォロー解除

**エンドポイント**
```
DELETE /api/profiles/:username/follow
```

**認証**: 必須

**レスポンス** (200 OK)
```json
{
  "profile": {
    "username": "jake",
    "bio": "I work at statefarm",
    "image": "https://api.realworld.io/images/smiley-cyrus.jpg",
    "following": false
  }
}
```

---

## 4. 記事 API

### 4.1 記事一覧取得

**エンドポイント**
```
GET /api/articles
```

**認証**: 任意

**クエリパラメータ**

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `tag` | タグでフィルタ | `?tag=AngularJS` |
| `author` | 著者でフィルタ | `?author=jake` |
| `favorited` | お気に入りユーザーでフィルタ | `?favorited=jake` |
| `limit` | 取得件数（デフォルト: 20） | `?limit=20` |
| `offset` | オフセット（デフォルト: 0） | `?offset=0` |

**レスポンス** (200 OK)
```json
{
  "articles": [
    {
      "slug": "how-to-train-your-dragon",
      "title": "How to train your dragon",
      "description": "Ever wonder how?",
      "body": "It takes a Jacobian",
      "tagList": ["dragons", "training"],
      "createdAt": "2016-02-18T03:22:56.637Z",
      "updatedAt": "2016-02-18T03:48:35.824Z",
      "favorited": false,
      "favoritesCount": 0,
      "author": {
        "username": "jake",
        "bio": "I work at statefarm",
        "image": "https://i.stack.imgur.com/xHWG8.jpg",
        "following": false
      }
    }
  ],
  "articlesCount": 1
}
```

---

### 4.2 フィード取得

**エンドポイント**
```
GET /api/articles/feed
```

**認証**: 必須

**クエリパラメータ**

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `limit` | 取得件数（デフォルト: 20） | `?limit=20` |
| `offset` | オフセット（デフォルト: 0） | `?offset=0` |

**レスポンス** (200 OK)
```json
{
  "articles": [ ... ],
  "articlesCount": 10
}
```

---

### 4.3 記事詳細取得

**エンドポイント**
```
GET /api/articles/:slug
```

**認証**: 任意

**レスポンス** (200 OK)
```json
{
  "article": {
    "slug": "how-to-train-your-dragon",
    "title": "How to train your dragon",
    "description": "Ever wonder how?",
    "body": "It takes a Jacobian",
    "tagList": ["dragons", "training"],
    "createdAt": "2016-02-18T03:22:56.637Z",
    "updatedAt": "2016-02-18T03:48:35.824Z",
    "favorited": false,
    "favoritesCount": 0,
    "author": {
      "username": "jake",
      "bio": "I work at statefarm",
      "image": "https://i.stack.imgur.com/xHWG8.jpg",
      "following": false
    }
  }
}
```

---

### 4.4 記事作成

**エンドポイント**
```
POST /api/articles
```

**認証**: 必須

**リクエスト**
```json
{
  "article": {
    "title": "How to train your dragon",
    "description": "Ever wonder how?",
    "body": "You have to believe",
    "tagList": ["reactjs", "angularjs", "dragons"]
  }
}
```

**レスポンス** (201 Created)
```json
{
  "article": {
    "slug": "how-to-train-your-dragon",
    "title": "How to train your dragon",
    "description": "Ever wonder how?",
    "body": "You have to believe",
    "tagList": ["reactjs", "angularjs", "dragons"],
    "createdAt": "2016-02-18T03:22:56.637Z",
    "updatedAt": "2016-02-18T03:22:56.637Z",
    "favorited": false,
    "favoritesCount": 0,
    "author": {
      "username": "jake",
      "bio": "I work at statefarm",
      "image": "https://i.stack.imgur.com/xHWG8.jpg",
      "following": false
    }
  }
}
```

---

### 4.5 記事更新

**エンドポイント**
```
PUT /api/articles/:slug
```

**認証**: 必須（著者のみ）

**リクエスト**
```json
{
  "article": {
    "title": "Did you train your dragon?"
  }
}
```

※ 指定したフィールドのみ更新。title を更新すると slug も再生成。

**レスポンス** (200 OK)
```json
{
  "article": { ... }
}
```

---

### 4.6 記事削除

**エンドポイント**
```
DELETE /api/articles/:slug
```

**認証**: 必須（著者のみ）

**レスポンス** (204 No Content)

---

## 5. コメント API

### 5.1 コメント一覧取得

**エンドポイント**
```
GET /api/articles/:slug/comments
```

**認証**: 任意

**レスポンス** (200 OK)
```json
{
  "comments": [
    {
      "id": 1,
      "createdAt": "2016-02-18T03:22:56.637Z",
      "updatedAt": "2016-02-18T03:22:56.637Z",
      "body": "It takes a Jacobian",
      "author": {
        "username": "jake",
        "bio": "I work at statefarm",
        "image": "https://i.stack.imgur.com/xHWG8.jpg",
        "following": false
      }
    }
  ]
}
```

---

### 5.2 コメント作成

**エンドポイント**
```
POST /api/articles/:slug/comments
```

**認証**: 必須

**リクエスト**
```json
{
  "comment": {
    "body": "His name was my name too."
  }
}
```

**レスポンス** (201 Created)
```json
{
  "comment": {
    "id": 1,
    "createdAt": "2016-02-18T03:22:56.637Z",
    "updatedAt": "2016-02-18T03:22:56.637Z",
    "body": "His name was my name too.",
    "author": {
      "username": "jake",
      "bio": "I work at statefarm",
      "image": "https://i.stack.imgur.com/xHWG8.jpg",
      "following": false
    }
  }
}
```

---

### 5.3 コメント削除

**エンドポイント**
```
DELETE /api/articles/:slug/comments/:id
```

**認証**: 必須（コメント著者のみ）

**レスポンス** (204 No Content)

---

## 6. お気に入り API

### 6.1 お気に入り追加

**エンドポイント**
```
POST /api/articles/:slug/favorite
```

**認証**: 必須

**リクエスト**: ボディなし

**レスポンス** (200 OK)
```json
{
  "article": {
    "slug": "how-to-train-your-dragon",
    "title": "How to train your dragon",
    "description": "Ever wonder how?",
    "body": "It takes a Jacobian",
    "tagList": ["dragons", "training"],
    "createdAt": "2016-02-18T03:22:56.637Z",
    "updatedAt": "2016-02-18T03:48:35.824Z",
    "favorited": true,
    "favoritesCount": 1,
    "author": {
      "username": "jake",
      "bio": "I work at statefarm",
      "image": "https://i.stack.imgur.com/xHWG8.jpg",
      "following": false
    }
  }
}
```

---

### 6.2 お気に入り解除

**エンドポイント**
```
DELETE /api/articles/:slug/favorite
```

**認証**: 必須

**レスポンス** (200 OK)
```json
{
  "article": {
    ...
    "favorited": false,
    "favoritesCount": 0,
    ...
  }
}
```

---

## 7. タグ API

### 7.1 タグ一覧取得

**エンドポイント**
```
GET /api/tags
```

**認証**: 不要

**レスポンス** (200 OK)
```json
{
  "tags": [
    "reactjs",
    "angularjs",
    "dragons",
    "training"
  ]
}
```

---

## 8. データ型定義

### 8.1 User

```typescript
interface User {
  email: string;
  token: string;
  username: string;
  bio: string | null;
  image: string | null;
}
```

### 8.2 Profile

```typescript
interface Profile {
  username: string;
  bio: string | null;
  image: string | null;
  following: boolean;
}
```

### 8.3 Article

```typescript
interface Article {
  slug: string;
  title: string;
  description: string;
  body: string;
  tagList: string[];
  createdAt: string;      // ISO 8601
  updatedAt: string;      // ISO 8601
  favorited: boolean;
  favoritesCount: number;
  author: Profile;
}
```

### 8.4 Comment

```typescript
interface Comment {
  id: number;
  createdAt: string;      // ISO 8601
  updatedAt: string;      // ISO 8601
  body: string;
  author: Profile;
}
```

---

## 9. エラーコード一覧

| HTTP Status | エラー種別 | 説明 |
|-------------|-----------|------|
| 400 | Bad Request | リクエスト形式エラー |
| 401 | Unauthorized | 認証が必要 |
| 403 | Forbidden | 権限なし（他人の記事編集など） |
| 404 | Not Found | リソースが存在しない |
| 422 | Unprocessable Entity | バリデーションエラー |
| 500 | Internal Server Error | サーバー内部エラー |

---

## 10. バリデーションルール

### 10.1 ユーザー登録

| フィールド | ルール |
|-----------|--------|
| `username` | 必須、一意、1-20文字 |
| `email` | 必須、一意、有効なメール形式 |
| `password` | 必須、8文字以上 |

### 10.2 記事作成

| フィールド | ルール |
|-----------|--------|
| `title` | 必須、1-100文字 |
| `description` | 必須、1-200文字 |
| `body` | 必須 |
| `tagList` | 任意、配列 |

### 10.3 コメント作成

| フィールド | ルール |
|-----------|--------|
| `body` | 必須、1-1000文字 |
