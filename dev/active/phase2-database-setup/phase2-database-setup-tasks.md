# Phase 2: 데이터베이스 설정 - Task 체크리스트

## 개요

Prisma ORM 및 SQLite 데이터베이스 설정.
Architecture.md의 ERD 기반 7개 모델 정의.

---

## Step 1: backend/package.json 업데이트 ⏳ 미시작

- [ ] dependencies 추가
  - [ ] @prisma/client ^5.0.0

- [ ] devDependencies 추가
  - [ ] prisma ^5.0.0
  - [ ] typescript ^5.0.0
  - [ ] @types/node ^22.0.0
  - [ ] tsx ^4.0.0

- [ ] scripts 추가
  - [ ] dev: tsx watch src/index.ts
  - [ ] build: tsc
  - [ ] db:generate: prisma generate
  - [ ] db:migrate: prisma migrate dev
  - [ ] db:push: prisma db push
  - [ ] db:studio: prisma studio

**수락 기준:**

- `pnpm --filter backend install` 성공

---

## Step 2: backend/tsconfig.json 생성 ⏳ 미시작

- [ ] `backend/tsconfig.json` 파일 생성
- [ ] ES2022 타겟 설정
- [ ] NodeNext 모듈 설정
- [ ] strict 모드 활성화

**수락 기준:**

- TypeScript 컴파일 설정 완료

---

## Step 3: Prisma 초기화 ⏳ 미시작

- [ ] `pnpm --filter backend prisma init --datasource-provider sqlite` 실행
- [ ] `backend/prisma/schema.prisma` 생성 확인
- [ ] `backend/.env` 생성 확인
- [ ] DATABASE_URL 설정 확인

**수락 기준:**

- Prisma 초기화 완료
- schema.prisma 파일 존재

---

## Step 4: Prisma 스키마 정의 ⏳ 미시작

- [ ] User 모델 정의
  - [ ] id, email (unique), username (unique), password
  - [ ] bio, image, createdAt, updatedAt
  - [ ] 관계: articles, comments, favorites, followers, following

- [ ] Article 모델 정의
  - [ ] id, slug (unique), title, description, body
  - [ ] authorId, createdAt, updatedAt
  - [ ] 관계: author, tags, comments, favorites
  - [ ] 인덱스: authorId, createdAt

- [ ] Tag 모델 정의
  - [ ] id, name (unique)
  - [ ] 관계: articles

- [ ] ArticleTag 모델 정의
  - [ ] articleId, tagId (복합 키)
  - [ ] 관계: article, tag

- [ ] Comment 모델 정의
  - [ ] id, body, createdAt, updatedAt
  - [ ] articleId, authorId
  - [ ] 관계: article, author
  - [ ] 인덱스: articleId

- [ ] Favorite 모델 정의
  - [ ] userId, articleId (복합 키)
  - [ ] 관계: user, article

- [ ] Follow 모델 정의
  - [ ] followerId, followingId (복합 키)
  - [ ] 관계: follower, following

**수락 기준:**

- 7개 모델 모두 정의됨
- 모든 관계 설정됨
- Cascade delete 적용됨

---

## Step 5: 초기 마이그레이션 생성 ⏳ 미시작

- [ ] `pnpm --filter backend prisma migrate dev --name init` 실행
- [ ] `backend/prisma/migrations/` 디렉토리 생성 확인
- [ ] `backend/prisma/dev.db` 파일 생성 확인
- [ ] Prisma Client 자동 생성 확인

**수락 기준:**

- 마이그레이션 성공
- 데이터베이스 파일 생성됨

---

## Step 6: Prisma 클라이언트 유틸리티 생성 ⏳ 미시작

- [ ] `backend/src/lib/` 디렉토리 생성
- [ ] `backend/src/lib/prisma.ts` 파일 생성
- [ ] 싱글톤 패턴 구현
- [ ] globalThis 캐싱 (hot reload 대응)

**수락 기준:**

- Prisma 클라이언트 import 가능
- 개발 환경에서 연결 누수 방지

---

## Step 7: 환경 변수 템플릿 생성 ⏳ 미시작

- [ ] `backend/.env.example` 파일 생성
- [ ] DATABASE_URL 포함
- [ ] JWT_SECRET, JWT_EXPIRES_IN 포함
- [ ] PORT, NODE_ENV 포함
- [ ] `.gitignore`에 `.env` 추가 확인

**수락 기준:**

- 모든 환경 변수 문서화됨
- 실제 .env는 git에서 제외됨

---

## 최종 검증 ⏳ 미시작

- [ ] `pnpm --filter backend prisma generate` 성공
- [ ] `pnpm --filter backend prisma migrate dev` 성공
- [ ] `pnpm --filter backend prisma studio` 실행 가능
- [ ] 모든 7개 테이블 존재 확인

---

## 빠른 재개

**다음 태스크 찾기:**

1. 위 체크리스트에서 첫 번째 미완료 항목 (`[ ]`) 찾기
2. 해당 Step의 수락 기준 확인
3. `docs/Architecture.md`의 ERD 참조
4. 완료 후 체크표시 (`[x]`)로 업데이트

**현재 진행 상황:** Step 1부터 시작 예정
