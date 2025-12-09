# RealWorld (Conduit) - アーキテクチャ設計書

## 1. システム概要

### 1.1 アーキテクチャ図

```mermaid
flowchart TB
    subgraph Client["Client Browser"]
        Browser[Browser]
    end

    subgraph Frontend["Frontend (React + Vite)"]
        Routes[Routes<br/>TanStack Router]
        Components[Components<br/>MUI]
        Query[TanStack Query<br/>Server State]
        Routes --> Components --> Query
    end

    subgraph Backend["Backend (Express + TypeScript)"]
        BRoutes[Routes]
        Controllers[Controllers]
        Services[Services<br/>Business Logic]
        Middleware[Middleware<br/>Auth, CORS]
        Repositories[Repositories<br/>Data Access]
        BRoutes --> Controllers --> Services --> Repositories
        Middleware -.-> BRoutes
    end

    subgraph Database["Database"]
        SQLite[(SQLite)]
    end

    Browser -->|HTTP/HTTPS| Frontend
    Query -->|REST API| BRoutes
    Repositories -->|Prisma ORM| SQLite
```

### 1.2 コンポーネント間の通信

| 通信経路 | プロトコル | 形式 |
|---------|-----------|------|
| Browser ↔ Frontend | HTTP/HTTPS | HTML/JS/CSS |
| Frontend ↔ Backend | REST API | JSON |
| Backend ↔ Database | Prisma Client | SQL |

---

## 2. バックエンドアーキテクチャ

### 2.1 レイヤードアーキテクチャ

```mermaid
flowchart TB
    Request[HTTP Request]

    subgraph Routes["Routes Layer"]
        R1["エンドポイント定義"]
        R2["ミドルウェア適用（Auth, Validation）"]
        R3["ルーティング"]
    end

    subgraph Controllers["Controllers Layer"]
        C1["リクエストパラメータ抽出"]
        C2["バリデーション実行"]
        C3["レスポンス整形"]
        C4["エラーハンドリング"]
    end

    subgraph Services["Services Layer"]
        S1["ビジネスロジック実装"]
        S2["トランザクション管理"]
        S3["複数リポジトリの調整"]
    end

    subgraph Repositories["Repositories Layer"]
        Repo1["Prisma クエリ実行"]
        Repo2["データ変換"]
        Repo3["キャッシュ（将来）"]
    end

    DB[(Database)]

    Request --> Routes --> Controllers --> Services --> Repositories --> DB
```

### 2.2 各レイヤーの責務

| レイヤー | 責務 | 依存関係 |
|---------|------|---------|
| **Routes** | エンドポイント定義、ミドルウェア適用 | Controllers |
| **Controllers** | HTTP 処理、バリデーション、レスポンス | Services |
| **Services** | ビジネスロジック | Repositories |
| **Repositories** | データアクセス | Prisma |

### 2.3 ミドルウェアスタック

```mermaid
flowchart TB
    Request[Request]
    Helmet["Helmet<br/>セキュリティヘッダー"]
    CORS["CORS<br/>クロスオリジン設定"]
    JSONParser["JSON Parser<br/>リクエストボディ解析"]
    Auth["Auth<br/>JWT 検証（必要な場合）"]
    Validation["Validation<br/>リクエストバリデーション"]
    Controller["Controller<br/>ビジネスロジック実行"]
    ErrorHandler["Error Handler<br/>エラー処理・レスポンス"]
    Response[Response]

    Request --> Helmet --> CORS --> JSONParser --> Auth --> Validation --> Controller --> ErrorHandler --> Response
```

---

## 3. フロントエンドアーキテクチャ

### 3.1 機能ベースモジュール構成

```
src/
├── features/                    # 機能モジュール
│   ├── auth/                    # 認証機能
│   │   ├── components/          # 認証関連コンポーネント
│   │   ├── hooks/               # 認証フック
│   │   ├── api/                 # 認証 API 呼び出し
│   │   └── types.ts             # 型定義
│   │
│   ├── articles/                # 記事機能
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── api/
│   │   └── types.ts
│   │
│   ├── comments/                # コメント機能
│   ├── profiles/                # プロフィール機能
│   └── tags/                    # タグ機能
│
├── components/                  # 共通コンポーネント
│   ├── layout/
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   └── Layout.tsx
│   └── ui/
│       ├── Loading.tsx
│       ├── ErrorMessage.tsx
│       └── Pagination.tsx
│
├── hooks/                       # 共通フック
│   ├── useAuth.ts
│   └── useLocalStorage.ts
│
├── lib/                         # ユーティリティ
│   ├── api/
│   │   └── client.ts            # API クライアント
│   └── utils/
│       └── formatDate.ts
│
└── routes/                      # TanStack Router
    ├── __root.tsx
    ├── index.tsx
    ├── login.tsx
    ├── register.tsx
    └── ...
```

### 3.2 状態管理戦略

| 状態の種類 | 管理方法 | 例 |
|-----------|---------|-----|
| **サーバー状態** | TanStack Query | 記事一覧、ユーザー情報 |
| **認証状態** | Context + localStorage | ログイン状態、JWT |
| **UI 状態** | React State | モーダル、フォーム |
| **URL 状態** | TanStack Router | フィルター、ページ番号 |

### 3.3 データフェッチパターン

```typescript
// useSuspenseQuery パターン
function ArticleList() {
  const { data: articles } = useSuspenseQuery({
    queryKey: ['articles'],
    queryFn: () => api.articles.list()
  });

  return <ArticleListView articles={articles} />;
}

// Suspense 境界
function ArticlesPage() {
  return (
    <Suspense fallback={<Loading />}>
      <ArticleList />
    </Suspense>
  );
}
```

---

## 4. データモデル（ERD）

### 4.1 エンティティ関係図

```mermaid
erDiagram
    User {
        string id PK
        string email UK
        string username UK
        string password
        string bio
        string image
        datetime createdAt
        datetime updatedAt
    }

    Article {
        string id PK
        string slug UK
        string title
        string description
        string body
        string authorId FK
        datetime createdAt
        datetime updatedAt
    }

    Tag {
        string id PK
        string name UK
    }

    ArticleTag {
        string articleId FK
        string tagId FK
    }

    Comment {
        string id PK
        string body
        string articleId FK
        string authorId FK
        datetime createdAt
        datetime updatedAt
    }

    Follow {
        string followerId FK
        string followingId FK
    }

    Favorite {
        string userId FK
        string articleId FK
    }

    User ||--o{ Article : "writes"
    User ||--o{ Comment : "writes"
    User ||--o{ Favorite : "favorites"
    User ||--o{ Follow : "follower"
    User ||--o{ Follow : "following"
    Article ||--o{ Comment : "has"
    Article ||--o{ ArticleTag : "has"
    Article ||--o{ Favorite : "favorited by"
    Tag ||--o{ ArticleTag : "tagged"
```

### 4.2 Prisma スキーマ

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  username  String   @unique
  password  String
  bio       String?
  image     String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  articles   Article[]
  comments   Comment[]
  favorites  Favorite[]
  followers  Follow[]   @relation("Following")
  following  Follow[]   @relation("Follower")
}

model Article {
  id          String   @id @default(uuid())
  slug        String   @unique
  title       String
  description String
  body        String
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Relations
  author    User       @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId  String
  tags      ArticleTag[]
  comments  Comment[]
  favorites Favorite[]

  @@index([authorId])
  @@index([createdAt])
}

model Tag {
  id       String       @id @default(uuid())
  name     String       @unique
  articles ArticleTag[]
}

model ArticleTag {
  article   Article @relation(fields: [articleId], references: [id], onDelete: Cascade)
  articleId String
  tag       Tag     @relation(fields: [tagId], references: [id], onDelete: Cascade)
  tagId     String

  @@id([articleId, tagId])
}

model Comment {
  id        String   @id @default(uuid())
  body      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  article   Article @relation(fields: [articleId], references: [id], onDelete: Cascade)
  articleId String
  author    User    @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId  String

  @@index([articleId])
}

model Favorite {
  user      User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    String
  article   Article @relation(fields: [articleId], references: [id], onDelete: Cascade)
  articleId String

  @@id([userId, articleId])
}

model Follow {
  follower    User   @relation("Follower", fields: [followerId], references: [id], onDelete: Cascade)
  followerId  String
  following   User   @relation("Following", fields: [followingId], references: [id], onDelete: Cascade)
  followingId String

  @@id([followerId, followingId])
}
```

---

## 5. 認証フロー

### 5.1 JWT 認証シーケンス

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant DB

    Client->>Server: POST /api/users/login<br/>{email, password}
    Server->>DB: Find user by email
    DB-->>Server: User data
    Server->>Server: Verify password<br/>(bcrypt.compare)
    Server->>Server: Generate JWT
    Server-->>Client: {user: {..., token}}
    Client->>Client: Store token in localStorage
```

### 5.2 認証済みリクエスト

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant DB

    Client->>Server: GET /api/user<br/>Authorization: Token {jwt}
    Server->>Server: Verify JWT<br/>Extract userId
    Server->>DB: Find user by id
    DB-->>Server: User data
    Server-->>Client: {user: {...}}
```

### 5.3 トークン構造

```json
// JWT Payload
{
  "userId": "uuid-string",
  "iat": 1234567890,
  "exp": 1234567890
}
```

---

## 6. エラーハンドリング

### 6.1 エラーレスポンス形式

```json
{
  "errors": {
    "body": ["can't be empty"]
  }
}
```

### 6.2 HTTP ステータスコード

| コード | 意味 | 使用場面 |
|--------|------|---------|
| 200 | OK | 成功（GET, PUT） |
| 201 | Created | 成功（POST） |
| 204 | No Content | 成功（DELETE） |
| 400 | Bad Request | バリデーションエラー |
| 401 | Unauthorized | 認証必須 |
| 403 | Forbidden | 権限不足 |
| 404 | Not Found | リソースなし |
| 422 | Unprocessable Entity | バリデーションエラー |
| 500 | Internal Server Error | サーバーエラー |

### 6.3 バックエンドエラーハンドリング

```typescript
// middleware/errorHandler.ts
export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  if (err instanceof ValidationError) {
    return res.status(422).json({
      errors: err.errors
    });
  }

  if (err instanceof UnauthorizedError) {
    return res.status(401).json({
      errors: { message: ['Unauthorized'] }
    });
  }

  // Sentry にエラー送信
  Sentry.captureException(err);

  return res.status(500).json({
    errors: { message: ['Internal server error'] }
  });
}
```

---

## 7. パフォーマンス最適化

### 7.1 バックエンド

| 最適化 | 実装方法 |
|--------|---------|
| **データベースインデックス** | 頻繁にクエリされるフィールドにインデックス |
| **N+1 問題回避** | Prisma の include で関連データを一括取得 |
| **ページネーション** | offset/limit でデータ取得を制限 |

### 7.2 フロントエンド

| 最適化 | 実装方法 |
|--------|---------|
| **コード分割** | React.lazy + Suspense |
| **キャッシュ** | TanStack Query の staleTime 設定 |
| **メモ化** | React.memo, useMemo, useCallback |
| **バンドル最適化** | Vite のチャンク分割 |

---

## 8. セキュリティ対策

### 8.1 実装するセキュリティ対策

| 脅威 | 対策 |
|------|------|
| **XSS** | React のエスケープ、CSP ヘッダー |
| **CSRF** | SameSite Cookie（将来） |
| **SQL Injection** | Prisma ORM（パラメータ化クエリ） |
| **Brute Force** | Rate Limiting（将来） |
| **認証情報漏洩** | bcrypt ハッシュ、HTTPS |

### 8.2 セキュリティヘッダー（Helmet）

```typescript
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"]
    }
  }
}));
```
