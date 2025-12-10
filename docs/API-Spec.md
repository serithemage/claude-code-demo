# RealWorld (Conduit) - API Specification

## 1. Overview

### 1.1 Base URL
```
http://localhost:3000/api
```

### 1.2 Authentication Header
```
Authorization: Token jwt.token.here
```

### 1.3 Common Response Format

**Success**
```json
{
  "user": { ... },      // or "article", "articles", etc.
}
```

**Error**
```json
{
  "errors": {
    "body": ["can't be empty"],
    "email": ["has already been taken"]
  }
}
```

---

## 2. Authentication API

### 2.1 User Registration

**Endpoint**
```
POST /api/users
```

**Authentication**: Not required

**Request**
```json
{
  "user": {
    "username": "Jacob",
    "email": "jake@jake.jake",
    "password": "jakejake"
  }
}
```

**Response** (201 Created)
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

**Error** (422 Unprocessable Entity)
```json
{
  "errors": {
    "email": ["has already been taken"],
    "username": ["has already been taken"]
  }
}
```

---

### 2.2 Login

**Endpoint**
```
POST /api/users/login
```

**Authentication**: Not required

**Request**
```json
{
  "user": {
    "email": "jake@jake.jake",
    "password": "jakejake"
  }
}
```

**Response** (200 OK)
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

**Error** (401 Unauthorized)
```json
{
  "errors": {
    "email or password": ["is invalid"]
  }
}
```

---

### 2.3 Get Current User

**Endpoint**
```
GET /api/user
```

**Authentication**: Required

**Response** (200 OK)
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

### 2.4 Update User

**Endpoint**
```
PUT /api/user
```

**Authentication**: Required

**Request**
```json
{
  "user": {
    "email": "jake@jake.jake",
    "bio": "I like to skateboard",
    "image": "https://i.stack.imgur.com/xHWG8.jpg"
  }
}
```

※ Only specified fields are updated. New password can be set via password field.

**Response** (200 OK)
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

## 3. Profile API

### 3.1 Get Profile

**Endpoint**
```
GET /api/profiles/:username
```

**Authentication**: Optional (includes following flag when logged in)

**Response** (200 OK)
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

### 3.2 Follow User

**Endpoint**
```
POST /api/profiles/:username/follow
```

**Authentication**: Required

**Request**: No body

**Response** (200 OK)
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

### 3.3 Unfollow User

**Endpoint**
```
DELETE /api/profiles/:username/follow
```

**Authentication**: Required

**Response** (200 OK)
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

## 4. Articles API

### 4.1 List Articles

**Endpoint**
```
GET /api/articles
```

**Authentication**: Optional

**Query Parameters**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `tag` | Filter by tag | `?tag=AngularJS` |
| `author` | Filter by author | `?author=jake` |
| `favorited` | Filter by favorited user | `?favorited=jake` |
| `limit` | Number of items (default: 20) | `?limit=20` |
| `offset` | Offset (default: 0) | `?offset=0` |

**Response** (200 OK)
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

### 4.2 Get Feed

**Endpoint**
```
GET /api/articles/feed
```

**Authentication**: Required

**Query Parameters**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `limit` | Number of items (default: 20) | `?limit=20` |
| `offset` | Offset (default: 0) | `?offset=0` |

**Response** (200 OK)
```json
{
  "articles": [ ... ],
  "articlesCount": 10
}
```

---

### 4.3 Get Article

**Endpoint**
```
GET /api/articles/:slug
```

**Authentication**: Optional

**Response** (200 OK)
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

### 4.4 Create Article

**Endpoint**
```
POST /api/articles
```

**Authentication**: Required

**Request**
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

**Response** (201 Created)
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

### 4.5 Update Article

**Endpoint**
```
PUT /api/articles/:slug
```

**Authentication**: Required (author only)

**Request**
```json
{
  "article": {
    "title": "Did you train your dragon?"
  }
}
```

※ Only specified fields are updated. Updating title regenerates the slug.

**Response** (200 OK)
```json
{
  "article": { ... }
}
```

---

### 4.6 Delete Article

**Endpoint**
```
DELETE /api/articles/:slug
```

**Authentication**: Required (author only)

**Response** (204 No Content)

---

## 5. Comments API

### 5.1 List Comments

**Endpoint**
```
GET /api/articles/:slug/comments
```

**Authentication**: Optional

**Response** (200 OK)
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

### 5.2 Create Comment

**Endpoint**
```
POST /api/articles/:slug/comments
```

**Authentication**: Required

**Request**
```json
{
  "comment": {
    "body": "His name was my name too."
  }
}
```

**Response** (201 Created)
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

### 5.3 Delete Comment

**Endpoint**
```
DELETE /api/articles/:slug/comments/:id
```

**Authentication**: Required (comment author only)

**Response** (204 No Content)

---

## 6. Favorites API

### 6.1 Add Favorite

**Endpoint**
```
POST /api/articles/:slug/favorite
```

**Authentication**: Required

**Request**: No body

**Response** (200 OK)
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

### 6.2 Remove Favorite

**Endpoint**
```
DELETE /api/articles/:slug/favorite
```

**Authentication**: Required

**Response** (200 OK)
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

## 7. Tags API

### 7.1 List Tags

**Endpoint**
```
GET /api/tags
```

**Authentication**: Not required

**Response** (200 OK)
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

## 8. Data Type Definitions

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

## 9. Error Codes

| HTTP Status | Error Type | Description |
|-------------|------------|-------------|
| 400 | Bad Request | Invalid request format |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | No permission (e.g., editing another's article) |
| 404 | Not Found | Resource does not exist |
| 422 | Unprocessable Entity | Validation error |
| 500 | Internal Server Error | Server internal error |

---

## 10. Validation Rules

### 10.1 User Registration

| Field | Rules |
|-------|-------|
| `username` | Required, unique, 1-20 characters |
| `email` | Required, unique, valid email format |
| `password` | Required, 8+ characters |

### 10.2 Article Creation

| Field | Rules |
|-------|-------|
| `title` | Required, 1-100 characters |
| `description` | Required, 1-200 characters |
| `body` | Required |
| `tagList` | Optional, array |

### 10.3 Comment Creation

| Field | Rules |
|-------|-------|
| `body` | Required, 1-1000 characters |
