---
name: route-tester
description: RealWorld 프로젝트에서 JWT 인증을 사용하는 API 라우트 테스트. API 엔드포인트 테스트, 라우트 기능 검증, 인증 문제 디버깅 시 사용합니다. curl과 Supertest를 사용한 테스트 패턴을 포함합니다.
---

# RealWorld Route Tester Skill

## 목적

이 skill은 RealWorld (Conduit) 프로젝트에서 JWT 인증을 사용하는 API 라우트를 테스트하기 위한 패턴을 제공합니다.

## 이 Skill 사용 시점

- 새 API 엔드포인트 테스트
- 변경 후 라우트 기능 검증
- 인증 문제 디버깅
- POST/PUT/DELETE 작업 테스트
- 요청/응답 데이터 검증

## RealWorld 인증 개요

RealWorld 프로젝트에서 사용하는 것:

- **JWT 토큰**: localStorage에 저장 (프론트엔드)
- **Authorization 헤더**: `Token jwt.token.here`
- **bcrypt**: 비밀번호 해싱

**중요**: RealWorld 사양은 `Bearer` 대신 `Token` 접두사를 사용합니다.

## 테스트 방법

### 방법 1: curl을 사용한 직접 테스트 (권장)

#### 토큰 획득 (로그인)

```bash
# 로그인하여 JWT 토큰 받기
curl -X POST http://localhost:3000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"test@example.com","password":"testpassword"}}'

# 응답에서 토큰 추출:
# {"user":{"email":"test@example.com","token":"eyJhbG...","username":"testuser",...}}
```

#### 인증된 요청

```bash
# 토큰 저장
TOKEN="eyJhbG..."

# 현재 사용자 조회
curl http://localhost:3000/api/user \
  -H "Authorization: Token $TOKEN"

# 게시글 작성
curl -X POST http://localhost:3000/api/articles \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"article":{"title":"Test Article","description":"About testing","body":"Test content","tagList":["test"]}}'

# 게시글 좋아요
curl -X POST http://localhost:3000/api/articles/test-article/favorite \
  -H "Authorization: Token $TOKEN"
```

### 방법 2: Supertest를 사용한 통합 테스트

```typescript
// backend/src/tests/articles.test.ts
import request from 'supertest';
import app from '../app';

describe('Articles API', () => {
  let authToken: string;

  beforeAll(async () => {
    // 테스트 사용자로 로그인
    const res = await request(app)
      .post('/api/users/login')
      .send({
        user: {
          email: 'test@example.com',
          password: 'testpassword',
        },
      });
    authToken = res.body.user.token;
  });

  it('should create article', async () => {
    const res = await request(app)
      .post('/api/articles')
      .set('Authorization', `Token ${authToken}`)
      .send({
        article: {
          title: 'Test Article',
          description: 'Test description',
          body: 'Test body',
          tagList: ['test'],
        },
      });

    expect(res.status).toBe(201);
    expect(res.body.article.title).toBe('Test Article');
  });
});
```

## RealWorld API 엔드포인트

### 인증 엔드포인트

| 메서드 | 엔드포인트         | 인증   | 설명             |
| ------ | ------------------ | ------ | ---------------- |
| POST   | `/api/users`       | 불필요 | 사용자 가입      |
| POST   | `/api/users/login` | 불필요 | 로그인           |
| GET    | `/api/user`        | 필수   | 현재 사용자 조회 |
| PUT    | `/api/user`        | 필수   | 사용자 정보 수정 |

### 프로필 엔드포인트

| 메서드 | 엔드포인트                       | 인증 | 설명        |
| ------ | -------------------------------- | ---- | ----------- |
| GET    | `/api/profiles/:username`        | 선택 | 프로필 조회 |
| POST   | `/api/profiles/:username/follow` | 필수 | 팔로우      |
| DELETE | `/api/profiles/:username/follow` | 필수 | 언팔로우    |

### 게시글 엔드포인트

| 메서드 | 엔드포인트            | 인증 | 설명                   |
| ------ | --------------------- | ---- | ---------------------- |
| GET    | `/api/articles`       | 선택 | 게시글 목록            |
| GET    | `/api/articles/feed`  | 필수 | 피드 (팔로우한 사용자) |
| GET    | `/api/articles/:slug` | 선택 | 게시글 상세            |
| POST   | `/api/articles`       | 필수 | 게시글 작성            |
| PUT    | `/api/articles/:slug` | 필수 | 게시글 수정 (작성자만) |
| DELETE | `/api/articles/:slug` | 필수 | 게시글 삭제 (작성자만) |

### 댓글 엔드포인트

| 메서드 | 엔드포인트                         | 인증 | 설명                 |
| ------ | ---------------------------------- | ---- | -------------------- |
| GET    | `/api/articles/:slug/comments`     | 선택 | 댓글 목록            |
| POST   | `/api/articles/:slug/comments`     | 필수 | 댓글 작성            |
| DELETE | `/api/articles/:slug/comments/:id` | 필수 | 댓글 삭제 (작성자만) |

### 좋아요 & 태그 엔드포인트

| 메서드 | 엔드포인트                     | 인증   | 설명        |
| ------ | ------------------------------ | ------ | ----------- |
| POST   | `/api/articles/:slug/favorite` | 필수   | 좋아요 추가 |
| DELETE | `/api/articles/:slug/favorite` | 필수   | 좋아요 취소 |
| GET    | `/api/tags`                    | 불필요 | 태그 목록   |

## 일반적인 테스트 시나리오

### 사용자 가입 & 로그인

```bash
# 1. 새 사용자 가입
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"user":{"username":"newuser","email":"new@example.com","password":"password123"}}'

# 2. 로그인
curl -X POST http://localhost:3000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"new@example.com","password":"password123"}}'
```

### 게시글 CRUD

```bash
TOKEN="your-jwt-token"

# 작성
curl -X POST http://localhost:3000/api/articles \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"article":{"title":"My Article","description":"About stuff","body":"Content here","tagList":["test"]}}'

# 조회
curl http://localhost:3000/api/articles/my-article

# 수정
curl -X PUT http://localhost:3000/api/articles/my-article \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"article":{"title":"Updated Title"}}'

# 삭제
curl -X DELETE http://localhost:3000/api/articles/my-article \
  -H "Authorization: Token $TOKEN"
```

### 필터가 있는 게시글 목록

```bash
# 태그로 필터
curl "http://localhost:3000/api/articles?tag=javascript"

# 작성자로 필터
curl "http://localhost:3000/api/articles?author=jake"

# 좋아요한 사용자로 필터
curl "http://localhost:3000/api/articles?favorited=jake"

# 페이지네이션
curl "http://localhost:3000/api/articles?limit=10&offset=0"
```

## 실패한 테스트 디버깅

### 401 Unauthorized

**가능한 원인**:

1. 토큰이 없거나 잘못됨
2. 토큰 형식 오류 (`Token` 대신 `Bearer` 사용)
3. 토큰 만료

**해결책**:

```bash
# 헤더 형식 확인 - Token 사용 필수!
-H "Authorization: Token eyJhbG..."  # ✅ 올바름
-H "Authorization: Bearer eyJhbG..." # ❌ 잘못됨 (RealWorld)

# 재로그인하여 새 토큰 받기
```

### 403 Forbidden

**가능한 원인**:

1. 다른 사용자의 리소스에 접근 시도
2. 작성자만 수정/삭제 가능

**해결책**:

```bash
# 리소스 소유자 확인
curl http://localhost:3000/api/articles/some-article
# author.username이 현재 사용자와 일치하는지 확인
```

### 404 Not Found

**가능한 원인**:

1. 잘못된 URL 또는 slug
2. 리소스가 존재하지 않음

**해결책**:

1. 엔드포인트 경로 확인
2. slug 철자 확인
3. 리소스가 실제로 존재하는지 확인

### 422 Unprocessable Entity

**가능한 원인**:

1. 검증 오류
2. 필수 필드 누락
3. 잘못된 데이터 형식

**해결책**:

```bash
# 응답의 errors 객체 확인
{"errors":{"title":["can't be blank"]}}
```

## 테스트 체크리스트

라우트 테스트 전:

- [ ] 서버가 실행 중인지 확인 (`pnpm --filter backend dev`)
- [ ] 올바른 엔드포인트 URL 사용
- [ ] 인증 필요 여부 확인
- [ ] 요청 본문 형식 확인 (POST/PUT인 경우)
- [ ] Content-Type 헤더 포함 (JSON인 경우)
- [ ] Authorization 헤더가 `Token` 접두사 사용하는지 확인
- [ ] 응답 상태와 데이터 검증
- [ ] 에러 응답의 경우 `errors` 객체 확인

## 데이터베이스 확인

테스트 후 데이터베이스 확인:

```bash
# Prisma Studio 실행
pnpm --filter backend prisma studio

# 또는 직접 쿼리 (SQLite CLI)
sqlite3 backend/prisma/dev.db
sqlite> SELECT * FROM User;
sqlite> SELECT * FROM Article WHERE slug = 'test-article';
```

## 관련 Skills

- **backend-dev-guidelines** - 백엔드 레이어드 아키텍처
- **error-tracking** - Sentry 에러 추적
