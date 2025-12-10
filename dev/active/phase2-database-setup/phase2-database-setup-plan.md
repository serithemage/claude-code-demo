# Phase 2: 데이터베이스 설정 - 구현 계획

## 요약

RealWorld (Conduit) 프로젝트를 위한 Prisma ORM 및 SQLite 데이터베이스 설정.
Architecture.md의 ERD를 기반으로 모든 데이터 모델을 정의합니다.

---

## 현재 상태

- Phase 1 완료: 모노레포 구조 설정됨
- `backend/package.json` 존재 (placeholder)
- Prisma 미설치
- 데이터베이스 스키마 미정의

---

## 목표 상태

```
backend/
├── package.json           # Prisma 의존성 추가
├── tsconfig.json          # TypeScript 설정
├── prisma/
│   ├── schema.prisma      # 데이터베이스 스키마
│   └── migrations/        # 마이그레이션 파일
├── src/
│   └── lib/
│       └── prisma.ts      # Prisma 클라이언트 싱글톤
└── .env                   # DATABASE_URL
```

---

## 구현 단계

### Step 1: backend/package.json 업데이트

**추가할 의존성:**

```json
{
  "dependencies": {
    "@prisma/client": "^5.0.0"
  },
  "devDependencies": {
    "prisma": "^5.0.0",
    "typescript": "^5.0.0",
    "@types/node": "^22.0.0",
    "tsx": "^4.0.0"
  }
}
```

**추가할 스크립트:**

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate dev",
    "db:push": "prisma db push",
    "db:studio": "prisma studio"
  }
}
```

**수락 기준:**
- `pnpm --filter backend install` 성공
- Prisma CLI 사용 가능

---

### Step 2: backend/tsconfig.json 생성

**설정 내용:**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**수락 기준:**
- TypeScript 컴파일 가능

---

### Step 3: Prisma 초기화

**명령어:**

```bash
cd backend
pnpm prisma init --datasource-provider sqlite
```

**생성되는 파일:**
- `prisma/schema.prisma`
- `.env` (DATABASE_URL)

**수락 기준:**
- `prisma/schema.prisma` 파일 존재
- `.env`에 DATABASE_URL 설정됨

---

### Step 4: Prisma 스키마 정의

**파일:** `backend/prisma/schema.prisma`

**모델 정의 (Architecture.md ERD 기반):**

1. **User** - 사용자
   - id, email, username, password, bio, image, createdAt, updatedAt

2. **Article** - 게시글
   - id, slug, title, description, body, authorId, createdAt, updatedAt

3. **Tag** - 태그
   - id, name

4. **ArticleTag** - 게시글-태그 관계
   - articleId, tagId (복합 키)

5. **Comment** - 댓글
   - id, body, articleId, authorId, createdAt, updatedAt

6. **Favorite** - 좋아요
   - userId, articleId (복합 키)

7. **Follow** - 팔로우
   - followerId, followingId (복합 키)

**수락 기준:**
- 모든 모델 정의됨
- 관계 설정 완료
- 인덱스 설정됨

---

### Step 5: 초기 마이그레이션 생성

**명령어:**

```bash
cd backend
pnpm prisma migrate dev --name init
```

**수락 기준:**
- `prisma/migrations/` 디렉토리에 마이그레이션 파일 생성
- SQLite 데이터베이스 파일 생성 (`prisma/dev.db`)

---

### Step 6: Prisma 클라이언트 유틸리티 생성

**파일:** `backend/src/lib/prisma.ts`

**내용:**

```typescript
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

**수락 기준:**
- 싱글톤 패턴으로 Prisma 클라이언트 내보내기
- Hot reload 시 연결 누수 방지

---

### Step 7: .env.example 생성

**파일:** `backend/.env.example`

**내용:**

```env
# Database
DATABASE_URL="file:./dev.db"

# JWT
JWT_SECRET="your-secret-key-change-in-production"
JWT_EXPIRES_IN="7d"

# Server
PORT=3000
NODE_ENV="development"
```

**수락 기준:**
- 모든 필요한 환경 변수 문서화
- 민감한 정보 없음

---

## 리스크 평가

| 리스크 | 영향도 | 대응책 |
|--------|--------|--------|
| Prisma 버전 호환성 | 중간 | 안정 버전 사용 (5.x) |
| SQLite 동시성 제한 | 낮음 | 개발 환경 전용, 프로덕션은 PostgreSQL 권장 |
| 마이그레이션 실패 | 중간 | 스키마 검증 후 마이그레이션 |

---

## 성공 지표

- [ ] `pnpm --filter backend prisma generate` 성공
- [ ] `pnpm --filter backend prisma migrate dev` 성공
- [ ] `pnpm --filter backend prisma studio` 에서 모든 테이블 확인
- [ ] Prisma 클라이언트 import 가능

---

## 다음 단계

Phase 2 완료 후:
- Phase 3: 백엔드 인증 시스템 (Users API)
  - JWT 유틸리티
  - 인증 미들웨어
  - 사용자 가입/로그인 API
