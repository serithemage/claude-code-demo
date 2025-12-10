# Phase 2: 데이터베이스 설정 - Context

## SESSION PROGRESS (2025-12-10)

### ✅ 완료

- Phase 1 모노레포 설정 완료
- Phase 2 dev docs 작성 시작

### 🟡 진행 중

- Phase 2 dev docs 작성

### ⏳ 대기 중

- Step 1: backend/package.json 업데이트
- Step 2: backend/tsconfig.json 생성
- Step 3: Prisma 초기화
- Step 4: Prisma 스키마 정의
- Step 5: 초기 마이그레이션 생성
- Step 6: Prisma 클라이언트 유틸리티 생성
- Step 7: .env.example 생성

### ⚠️ 블로커

- 없음

---

## 중요 파일

### 참조 문서

**docs/Architecture.md**

- ERD 다이어그램 (4.1절)
- Prisma 스키마 예시 (4.2절)
- 7개 모델 정의: User, Article, Tag, ArticleTag, Comment, Favorite, Follow

**docs/TechStack.md**

- Prisma 5.x 버전 사용
- SQLite 개발 환경용

**docs/implementation-plan.md**

- Phase 2: Task 2.1, 2.2
- 의존성: Phase 1 완료 필요 ✅

### 생성 예정 파일

| 파일 | 목적 | 상태 |
|------|------|------|
| `backend/package.json` | 의존성 업데이트 | 수정 필요 |
| `backend/tsconfig.json` | TypeScript 설정 | ⏳ |
| `backend/prisma/schema.prisma` | 데이터베이스 스키마 | ⏳ |
| `backend/src/lib/prisma.ts` | Prisma 클라이언트 | ⏳ |
| `backend/.env.example` | 환경 변수 템플릿 | ⏳ |
| `backend/.env` | 실제 환경 변수 | ⏳ |

---

## 중요 결정 사항

### 1. 데이터베이스 선택

- **결정**: SQLite (개발), PostgreSQL 권장 (프로덕션)
- **이유**: 설정 간소화, 파일 기반 DB로 빠른 시작

### 2. ORM 선택

- **결정**: Prisma 5.x
- **이유**: 타입 안전성, 마이그레이션 관리, 우수한 DX

### 3. ID 전략

- **결정**: UUID (String @id @default(uuid()))
- **이유**: 분산 시스템 호환, 예측 불가능성

### 4. 관계 삭제 전략

- **결정**: Cascade delete
- **이유**: 참조 무결성 유지, 고아 레코드 방지

---

## 데이터 모델 요약

```
User (1) ──< Article (N)
User (1) ──< Comment (N)
User (1) ──< Favorite (N)
User (1) ──< Follow (N) >── User (1)

Article (1) ──< Comment (N)
Article (1) ──< ArticleTag (N) >── Tag (1)
Article (1) ──< Favorite (N)
```

---

## 기술적 제약

1. **SQLite 동시성**: 쓰기 잠금으로 동시 쓰기 제한
2. **Prisma Client 생성**: 스키마 변경 시 `prisma generate` 필요
3. **마이그레이션**: 프로덕션에서는 `prisma migrate deploy` 사용

---

## 빠른 재개 지침

**이 작업을 계속하려면:**

1. 이 파일과 `phase2-database-setup-plan.md` 읽기
2. `phase2-database-setup-tasks.md`에서 다음 미완료 태스크 확인
3. `docs/Architecture.md`의 ERD와 Prisma 스키마 참조
4. 각 태스크의 수락 기준 충족 확인
5. 완료 후 이 context 파일 업데이트

**현재 위치:** dev docs 작성 완료 후 Step 1부터 구현 시작

---

## 관련 링크

- [전체 구현 계획](../../../docs/implementation-plan.md)
- [아키텍처 - ERD](../../../docs/Architecture.md#4-데이터-모델-erd)
- [기술 스택](../../../docs/TechStack.md)
- [Phase 1 완료](../../complete/phase1-monorepo-setup/)
