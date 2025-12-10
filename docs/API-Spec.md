# RealWorld (Conduit) - API 명세

## 1. 개요

### 1.1 Base URL
```
http://localhost:3000/api
```

### 1.2 인증 헤더
```
Authorization: Token jwt.token.here
```

### 1.3 공통 응답 포맷

**성공**
```json
{
  "user": { ... },      // 또는 "article", "articles" 등
}
```

**에러**
```json
{
  "errors": {
    "body": ["can't be empty"],
    "email": ["has already been taken"]
  }
}
```

---

## 2. 인증 API

### 2.1 사용자 가입

**엔드포인트**
```
POST /api/users
```

**인증**: 불필요

**요청**
```json
{
  "user": {
    "username": "Jacob",
    "email": "jake@jake.jake",
    "password": "jakejake"
  }
}
```

**응답** (201 Created)
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

**에러** (422 Unprocessable Entity)
```json
{
  "errors": {
    "email": ["has already been taken"],
    "username": ["has already been taken"]
  }
}
```

---

### 2.2 로그인

**엔드포인트**
```
POST /api/users/login
```

**인증**: 불필요

**요청**
```json
{
  "user": {
    "email": "jake@jake.jake",
    "password": "jakejake"
  }
}
```

**응답** (200 OK)
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

**에러** (401 Unauthorized)
```json
{
  "errors": {
    "email or password": ["is invalid"]
  }
}
```

---

### 2.3 현재 사용자 조회

**엔드포인트**
```
GET /api/user
```

**인증**: 필수

**응답** (200 OK)
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

### 2.4 사용자 수정

**엔드포인트**
```
PUT /api/user
```

**인증**: 필수

**요청**
```json
{
  "user": {
    "email": "jake@jake.jake",
    "bio": "I like to skateboard",
    "image": "https://i.stack.imgur.com/xHWG8.jpg"
  }
}
```

※ 지정된 필드만 업데이트됩니다. password 필드로 새 비밀번호를 설정할 수 있습니다.

**응답** (200 OK)
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

## 3. 프로필 API

### 3.1 프로필 조회

**엔드포인트**
```
GET /api/profiles/:username
```

**인증**: 선택 (로그인 시 following 플래그 포함)

**응답** (200 OK)
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

### 3.2 사용자 팔로우

**엔드포인트**
```
POST /api/profiles/:username/follow
```

**인증**: 필수

**요청**: 본문 없음

**응답** (200 OK)
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

### 3.3 사용자 언팔로우

**엔드포인트**
```
DELETE /api/profiles/:username/follow
```

**인증**: 필수

**응답** (200 OK)
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

## 4. 게시글 API

### 4.1 게시글 목록 조회

**엔드포인트**
```
GET /api/articles
```

**인증**: 선택

**쿼리 파라미터**

| 파라미터 | 설명 | 예시 |
|----------|------|------|
| `tag` | 태그로 필터링 | `?tag=AngularJS` |
| `author` | 작성자로 필터링 | `?author=jake` |
| `favorited` | 좋아요한 사용자로 필터링 | `?favorited=jake` |
| `limit` | 항목 수 (기본: 20) | `?limit=20` |
| `offset` | 오프셋 (기본: 0) | `?offset=0` |

**응답** (200 OK)
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

### 4.2 피드 조회

**엔드포인트**
```
GET /api/articles/feed
```

**인증**: 필수

**쿼리 파라미터**

| 파라미터 | 설명 | 예시 |
|----------|------|------|
| `limit` | 항목 수 (기본: 20) | `?limit=20` |
| `offset` | 오프셋 (기본: 0) | `?offset=0` |

**응답** (200 OK)
```json
{
  "articles": [ ... ],
  "articlesCount": 10
}
```

---

### 4.3 게시글 조회

**엔드포인트**
```
GET /api/articles/:slug
```

**인증**: 선택

**응답** (200 OK)
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

### 4.4 게시글 작성

**엔드포인트**
```
POST /api/articles
```

**인증**: 필수

**요청**
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

**응답** (201 Created)
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

### 4.5 게시글 수정

**엔드포인트**
```
PUT /api/articles/:slug
```

**인증**: 필수 (작성자만)

**요청**
```json
{
  "article": {
    "title": "Did you train your dragon?"
  }
}
```

※ 지정된 필드만 업데이트됩니다. title 업데이트 시 slug가 재생성됩니다.

**응답** (200 OK)
```json
{
  "article": { ... }
}
```

---

### 4.6 게시글 삭제

**엔드포인트**
```
DELETE /api/articles/:slug
```

**인증**: 필수 (작성자만)

**응답** (204 No Content)

---

## 5. 댓글 API

### 5.1 댓글 목록 조회

**엔드포인트**
```
GET /api/articles/:slug/comments
```

**인증**: 선택

**응답** (200 OK)
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

### 5.2 댓글 작성

**엔드포인트**
```
POST /api/articles/:slug/comments
```

**인증**: 필수

**요청**
```json
{
  "comment": {
    "body": "His name was my name too."
  }
}
```

**응답** (201 Created)
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

### 5.3 댓글 삭제

**엔드포인트**
```
DELETE /api/articles/:slug/comments/:id
```

**인증**: 필수 (댓글 작성자만)

**응답** (204 No Content)

---

## 6. 좋아요 API

### 6.1 좋아요 추가

**엔드포인트**
```
POST /api/articles/:slug/favorite
```

**인증**: 필수

**요청**: 본문 없음

**응답** (200 OK)
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

### 6.2 좋아요 취소

**엔드포인트**
```
DELETE /api/articles/:slug/favorite
```

**인증**: 필수

**응답** (200 OK)
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

## 7. 태그 API

### 7.1 태그 목록 조회

**엔드포인트**
```
GET /api/tags
```

**인증**: 불필요

**응답** (200 OK)
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

## 8. 데이터 타입 정의

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

## 9. 에러 코드

| HTTP 상태 | 에러 유형 | 설명 |
|-----------|----------|------|
| 400 | Bad Request | 잘못된 요청 포맷 |
| 401 | Unauthorized | 인증 필요 |
| 403 | Forbidden | 권한 없음 (예: 다른 사람의 게시글 수정) |
| 404 | Not Found | 리소스가 존재하지 않음 |
| 422 | Unprocessable Entity | 검증 오류 |
| 500 | Internal Server Error | 서버 내부 오류 |

---

## 10. 검증 규칙

### 10.1 사용자 가입

| 필드 | 규칙 |
|------|------|
| `username` | 필수, 고유, 1-20자 |
| `email` | 필수, 고유, 유효한 이메일 형식 |
| `password` | 필수, 8자 이상 |

### 10.2 게시글 작성

| 필드 | 규칙 |
|------|------|
| `title` | 필수, 1-100자 |
| `description` | 필수, 1-200자 |
| `body` | 필수 |
| `tagList` | 선택, 배열 |

### 10.3 댓글 작성

| 필드 | 규칙 |
|------|------|
| `body` | 필수, 1-1000자 |
