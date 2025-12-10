# CLAUDE.md

이 파일은 이 저장소의 코드 작업 시 Claude Code (claude.ai/code)에게 가이드를 제공합니다.

## 프로젝트 개요

RealWorld (Conduit) - Medium.com 클론 소셜 블로깅 플랫폼. 프론트엔드와 백엔드를 통합한 모노레포 구조.

## 개발 명령어

```bash
# 의존성 설치
pnpm install

# 데이터베이스 마이그레이션
pnpm --filter backend prisma migrate dev

# 개발 서버 시작 (전체)
pnpm dev

# 개별 시작
pnpm --filter frontend dev    # 프론트엔드만
pnpm --filter backend dev     # 백엔드만

# 테스트
pnpm test                     # 전체 테스트
pnpm --filter backend test    # 백엔드 테스트만
pnpm --filter frontend test   # 프론트엔드 테스트만

# 린트 & 포맷
pnpm lint
pnpm format

# 빌드
pnpm build
```

## 아키텍처

### 모노레포 구조

```
├── frontend/          # React 19 + TypeScript + MUI v7 + TanStack Query/Router
├── backend/           # Express + TypeScript + Prisma + SQLite
└── docs/              # 프로젝트 문서 (PRD, TechStack, Architecture, API-Spec)
```

### 백엔드 - 레이어드 아키텍처

```
Routes → Controllers → Services → Repositories → Database (Prisma)
```

각 레이어의 책임:

- **Routes**: 엔드포인트 정의, 미들웨어 적용
- **Controllers**: 요청/응답 처리, 검증
- **Services**: 비즈니스 로직
- **Repositories**: Prisma를 통한 데이터 액세스

### 프론트엔드 - 기능 기반 모듈

```
src/features/     # auth, articles, comments, profiles, tags
src/components/   # 공유 컴포넌트 (layout, ui)
src/routes/       # TanStack Router 라우트 정의
src/lib/          # API 클라이언트, 유틸리티
```

## 기술 스택

| 영역       | 기술                                                                  |
| ---------- | --------------------------------------------------------------------- |
| 프론트엔드 | React 19, TypeScript, MUI v7, TanStack Query 5, TanStack Router, Vite |
| 백엔드     | Node.js 20, Express 4, TypeScript, Prisma 5, SQLite                   |
| 인증       | JWT (localStorage 저장)                                               |
| 테스팅     | Vitest, Testing Library, Supertest                                    |

## Claude Code Skills 통합

### 자동 활성화 Skills

- **backend-dev-guidelines**: `backend/**/*.ts` 파일 편집 시
- **frontend-dev-guidelines**: `frontend/src/**/*.tsx` 파일 편집 시 (block 설정)
- **error-tracking**: Sentry 통합 패턴
- **route-tester**: API 라우트 테스트

### 중요 제약사항

- MUI v7 필수: Grid는 `size={{}}` prop 사용 (xs, sm 금지), makeStyles 사용 금지
- 백엔드는 반드시 레이어드 아키텍처 준수
- 모든 Prisma 작업은 Repository 레이어에서 실행

## 프로젝트 문서

자세한 내용은 `docs/` 폴더 참조:

- `PRD.md` - 기능 요구사항, 화면 설계
- `TechStack.md` - 기술 선정 이유, 디렉토리 구조
- `Architecture.md` - 시스템 설계, ERD, 인증 플로우
- `API-Spec.md` - API 엔드포인트 명세

## 언어 설정

- 이 프로젝트의 모든 문서는 한국어로 작성합니다
- 한국어로 소통합니다
