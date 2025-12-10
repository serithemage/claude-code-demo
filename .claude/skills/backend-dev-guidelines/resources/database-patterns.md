# 데이터베이스 패턴 - Prisma 모범 사례

백엔드 마이크로서비스에서 Prisma를 사용한 데이터베이스 액세스 패턴에 대한 완전한 가이드입니다.

---

## RealWorld Prisma 스키마

### ERD 개요

```
User ──┬── Article ──┬── Comment
       │             └── ArticleTag ── Tag
       │             └── Favorite
       └── Follow (self-relation)
```

### 스키마 정의

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

  // 관계
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

  // 관계
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

  // 관계
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

### 주요 관계 설명

| 관계                 | 타입 | 설명                                   |
| -------------------- | ---- | -------------------------------------- |
| User → Article       | 1:N  | 사용자가 여러 게시글 작성              |
| User → Comment       | 1:N  | 사용자가 여러 댓글 작성                |
| User → Favorite      | 1:N  | 사용자가 여러 게시글 좋아요            |
| User ↔ User (Follow) | N:N  | 팔로우/팔로워 관계                     |
| Article → Comment    | 1:N  | 게시글에 여러 댓글                     |
| Article ↔ Tag        | N:N  | 게시글에 여러 태그, 태그에 여러 게시글 |

---

## 목차

- [PrismaService 사용법](#prismaservice-사용법)
- [Repository 패턴](#repository-패턴)
- [트랜잭션 패턴](#트랜잭션-패턴)
- [쿼리 최적화](#쿼리-최적화)
- [N+1 쿼리 방지](#n1-쿼리-방지)
- [에러 처리](#에러-처리)

---

## PrismaService 사용법

### 기본 패턴

```typescript
import { PrismaService } from '@project-lifecycle-portal/database';

// 항상 PrismaService.main 사용
const users = await PrismaService.main.user.findMany();
```

### 가용성 확인

```typescript
if (!PrismaService.isAvailable) {
  throw new Error('Prisma client not initialized');
}

const user = await PrismaService.main.user.findUnique({ where: { id } });
```

---

## Repository 패턴

### Repository를 사용해야 할 때

✅ **다음 경우 repositories 사용:**

- 조인/include가 있는 복잡한 쿼리
- 여러 곳에서 사용되는 쿼리
- 캐싱 계층이 필요할 때
- 테스트를 위해 모킹하고 싶을 때

❌ **다음 경우 repositories 생략:**

- 간단한 일회성 쿼리
- 프로토타이핑 (나중에 리팩토링 가능)

### Repository 템플릿

```typescript
export class UserRepository {
  async findById(id: string): Promise<User | null> {
    return PrismaService.main.user.findUnique({
      where: { id },
      include: { profile: true },
    });
  }

  async findActive(): Promise<User[]> {
    return PrismaService.main.user.findMany({
      where: { isActive: true },
      orderBy: { createdAt: 'desc' },
    });
  }

  async create(data: Prisma.UserCreateInput): Promise<User> {
    return PrismaService.main.user.create({ data });
  }
}
```

---

## 트랜잭션 패턴

### 단순 트랜잭션

```typescript
const result = await PrismaService.main.$transaction(async (tx) => {
  const user = await tx.user.create({ data: userData });
  const profile = await tx.userProfile.create({ data: { userId: user.id } });
  return { user, profile };
});
```

### 인터랙티브 트랜잭션

```typescript
const result = await PrismaService.main.$transaction(
  async (tx) => {
    const user = await tx.user.findUnique({ where: { id } });
    if (!user) throw new Error('User not found');

    return await tx.user.update({
      where: { id },
      data: { lastLogin: new Date() },
    });
  },
  {
    maxWait: 5000,
    timeout: 10000,
  }
);
```

---

## 쿼리 최적화

### select로 필드 제한

```typescript
// ❌ 모든 필드 가져오기
const users = await PrismaService.main.user.findMany();

// ✅ 필요한 필드만 가져오기
const users = await PrismaService.main.user.findMany({
  select: {
    id: true,
    email: true,
    profile: { select: { firstName: true, lastName: true } },
  },
});
```

### include 신중히 사용

```typescript
// ❌ 과도한 includes
const user = await PrismaService.main.user.findUnique({
  where: { id },
  include: {
    profile: true,
    posts: { include: { comments: true } },
    workflows: { include: { steps: { include: { actions: true } } } },
  },
});

// ✅ 필요한 것만 include
const user = await PrismaService.main.user.findUnique({
  where: { id },
  include: { profile: true },
});
```

---

## N+1 쿼리 방지

### 문제: N+1 쿼리

```typescript
// ❌ N+1 쿼리 문제
const users = await PrismaService.main.user.findMany(); // 1개 쿼리

for (const user of users) {
  // N개 쿼리 (사용자당 하나)
  const profile = await PrismaService.main.userProfile.findUnique({
    where: { userId: user.id },
  });
}
```

### 해결책: include 또는 배칭 사용

```typescript
// ✅ include로 단일 쿼리
const users = await PrismaService.main.user.findMany({
  include: { profile: true },
});

// ✅ 또는 배치 쿼리
const userIds = users.map((u) => u.id);
const profiles = await PrismaService.main.userProfile.findMany({
  where: { userId: { in: userIds } },
});
```

---

## 에러 처리

### Prisma 에러 타입

```typescript
import { Prisma } from '@prisma/client';

try {
  await PrismaService.main.user.create({ data });
} catch (error) {
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    // 고유 제약 조건 위반
    if (error.code === 'P2002') {
      throw new ConflictError('Email already exists');
    }

    // 외래 키 제약 조건
    if (error.code === 'P2003') {
      throw new ValidationError('Invalid reference');
    }

    // 레코드 찾을 수 없음
    if (error.code === 'P2025') {
      throw new NotFoundError('Record not found');
    }
  }

  // 알 수 없는 에러
  Sentry.captureException(error);
  throw error;
}
```

---

**관련 파일:**

- [SKILL.md](SKILL.md)
- [services-and-repositories.md](services-and-repositories.md)
- [async-and-errors.md](async-and-errors.md)
