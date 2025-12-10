# RealWorld (Conduit) - 기술 스택

## 1. 개요

이 프로젝트는 프론트엔드와 백엔드 관리를 통합하는 모노레포 구조를 사용합니다.
Claude Code를 사용한 효율적인 개발을 최우선으로 하여 검증된 도구 체인을 채택합니다.

---

## 2. 프론트엔드 기술 스택

### 2.1 핵심 프레임워크

| 기술           | 버전 | 목적                   |
| -------------- | ---- | ---------------------- |
| **React**      | 19.x | UI 프레임워크          |
| **TypeScript** | 5.x  | 타입 안전성            |
| **Vite**       | 6.x  | 빌드 도구 및 개발 서버 |

### 2.2 상태 관리 및 데이터 페칭

| 기술                | 버전 | 목적                        |
| ------------------- | ---- | --------------------------- |
| **TanStack Query**  | 5.x  | 서버 상태 관리, 데이터 페칭 |
| **TanStack Router** | 1.x  | 파일 기반 라우팅            |

### 2.3 UI 라이브러리

| 기술                  | 버전 | 목적              |
| --------------------- | ---- | ----------------- |
| **MUI (Material UI)** | 7.x  | UI 컴포넌트       |
| **@emotion/react**    | 11.x | CSS-in-JS         |
| **@emotion/styled**   | 11.x | Styled Components |

### 2.4 유틸리티

| 기술               | 목적            |
| ------------------ | --------------- |
| **react-markdown** | Markdown 렌더링 |
| **date-fns**       | 날짜 포맷팅     |
| **zod**            | 스키마 검증     |

### 2.5 선택 이유

- **React 19**: 최신 Suspense, 동시성 렌더링 기능 활용
- **TanStack Query**: useSuspenseQuery를 사용한 선언적 데이터 페칭
- **TanStack Router**: 타입 안전 파일 기반 라우팅
- **MUI v7**: 최신 디자인 시스템, 우수한 접근성
- **Vite**: 빠른 HMR, 최적화된 빌드

---

## 3. 백엔드 기술 스택

### 3.1 런타임 및 프레임워크

| 기술           | 버전     | 목적              |
| -------------- | -------- | ----------------- |
| **Node.js**    | 20.x LTS | JavaScript 런타임 |
| **Express**    | 4.x      | 웹 프레임워크     |
| **TypeScript** | 5.x      | 타입 안전성       |

### 3.2 데이터베이스 및 ORM

| 기술       | 버전 | 목적         |
| ---------- | ---- | ------------ |
| **SQLite** | 3.x  | 데이터베이스 |
| **Prisma** | 5.x  | ORM          |

### 3.3 인증 및 보안

| 기술             | 목적                  |
| ---------------- | --------------------- |
| **jsonwebtoken** | JWT 토큰 생성 및 검증 |
| **bcryptjs**     | 비밀번호 해싱         |
| **cors**         | CORS 설정             |
| **helmet**       | 보안 헤더             |

### 3.4 검증 및 유틸리티

| 기술        | 목적             |
| ----------- | ---------------- |
| **zod**     | 요청 검증        |
| **slugify** | URL Slug 생성    |
| **uuid**    | 고유 식별자 생성 |

### 3.5 아키텍처 패턴

**레이어드 아키텍처** 채택:

```
Routes → Controllers → Services → Repositories → Database
```

| 레이어           | 책임                           |
| ---------------- | ------------------------------ |
| **Routes**       | 엔드포인트 정의, 미들웨어 적용 |
| **Controllers**  | 요청/응답 처리, 검증           |
| **Services**     | 비즈니스 로직                  |
| **Repositories** | 데이터 액세스 (Prisma 작업)    |

### 3.6 선택 이유

- **Express**: 간단하고 유연함, 풍부한 생태계
- **Prisma**: 타입 안전 ORM, 마이그레이션 관리
- **SQLite**: 개발 환경을 위한 간소화된 설정
- **레이어드 아키텍처**: 관심사 분리, 테스트 용이성 향상

---

## 4. 공통 도구

### 4.1 개발 도구

| 기술         | 목적                           |
| ------------ | ------------------------------ |
| **pnpm**     | 패키지 매니저 (Workspace 지원) |
| **ESLint**   | 코드 품질 검사                 |
| **Prettier** | 코드 포맷팅                    |
| **tsx**      | TypeScript 실행 (개발)         |

### 4.2 테스팅

| 기술                | 목적                       |
| ------------------- | -------------------------- |
| **Vitest**          | 유닛 테스트 및 통합 테스트 |
| **Testing Library** | React 컴포넌트 테스팅      |
| **Supertest**       | API 엔드포인트 테스팅      |

### 4.3 프로세스 관리 및 모니터링

| 기술       | 목적                         |
| ---------- | ---------------------------- |
| **PM2**    | 프로세스 관리, 로그 모니터링 |
| **Sentry** | 에러 추적, 성능 모니터링     |

---

## 5. 디렉토리 구조

```
claude-code-demo/
├── docs/                          # 프로젝트 문서
│   ├── PRD.md
│   ├── TechStack.md
│   ├── Architecture.md
│   └── API-Spec.md
│
├── frontend/                      # 프론트엔드 애플리케이션
│   ├── src/
│   │   ├── features/              # 기능 기반 모듈
│   │   │   ├── auth/              # 인증 기능
│   │   │   ├── articles/          # 게시글 기능
│   │   │   ├── comments/          # 댓글 기능
│   │   │   ├── profiles/          # 프로필 기능
│   │   │   └── tags/              # 태그 기능
│   │   ├── components/            # 공유 컴포넌트
│   │   │   ├── layout/            # 레이아웃
│   │   │   └── ui/                # UI 파츠
│   │   ├── hooks/                 # 커스텀 Hooks
│   │   ├── lib/                   # 유틸리티
│   │   │   ├── api/               # API 클라이언트
│   │   │   └── utils/             # 헬퍼 함수
│   │   ├── routes/                # 라우트 정의 (TanStack Router)
│   │   ├── types/                 # 타입 정의
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── public/
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── package.json
│
├── backend/                       # 백엔드 애플리케이션
│   ├── src/
│   │   ├── routes/                # 라우트 정의
│   │   ├── controllers/           # 컨트롤러
│   │   ├── services/              # 비즈니스 로직
│   │   ├── repositories/          # 데이터 액세스
│   │   ├── middleware/            # 미들웨어
│   │   │   ├── auth.ts            # 인증 미들웨어
│   │   │   ├── errorHandler.ts    # 에러 처리
│   │   │   └── validation.ts      # 검증
│   │   ├── lib/                   # 유틸리티
│   │   │   ├── prisma.ts          # Prisma 클라이언트
│   │   │   └── jwt.ts             # JWT 유틸리티
│   │   ├── types/                 # 타입 정의
│   │   └── index.ts               # 진입점
│   ├── prisma/
│   │   ├── schema.prisma          # 데이터베이스 스키마
│   │   └── migrations/            # 마이그레이션
│   ├── tsconfig.json
│   └── package.json
│
├── .claude/                       # Claude Code 설정 (기존)
├── dev/                           # 개발 문서 (기존)
├── pnpm-workspace.yaml            # Workspace 설정
├── package.json                   # 루트 package.json
├── .prettierrc                    # Prettier 설정
├── .eslintrc.js                   # ESLint 설정
└── CLAUDE.md                      # 프로젝트 설정 (기존)
```

---

## 6. 개발 환경 설정

### 6.1 필수 조건

- Node.js 20.x 이상
- pnpm 8.x 이상

### 6.2 초기 설정

```bash
# 의존성 설치
pnpm install

# 데이터베이스 마이그레이션
pnpm --filter backend prisma migrate dev

# 개발 서버 시작 (프론트엔드 + 백엔드)
pnpm dev
```

### 6.3 사용 가능한 스크립트

| 명령어                       | 설명                  |
| ---------------------------- | --------------------- |
| `pnpm dev`                   | 개발 서버 시작 (전체) |
| `pnpm --filter frontend dev` | 프론트엔드만 시작     |
| `pnpm --filter backend dev`  | 백엔드만 시작         |
| `pnpm build`                 | 프로덕션 빌드         |
| `pnpm test`                  | 테스트 실행           |
| `pnpm lint`                  | 린트 검사             |
| `pnpm format`                | 코드 포맷팅           |

---

## 7. 환경 변수

### 7.1 백엔드 (.env)

```env
# 데이터베이스
DATABASE_URL="file:./dev.db"

# JWT
JWT_SECRET="your-secret-key"
JWT_EXPIRES_IN="7d"

# 서버
PORT=3000
NODE_ENV="development"

# Sentry (선택)
SENTRY_DSN=""
```

### 7.2 프론트엔드 (.env)

```env
# API
VITE_API_URL="http://localhost:3000/api"
```

---

## 8. 의존성 목록

### 8.1 프론트엔드

```json
{
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "@tanstack/react-query": "^5.0.0",
    "@tanstack/react-router": "^1.0.0",
    "@mui/material": "^7.0.0",
    "@emotion/react": "^11.0.0",
    "@emotion/styled": "^11.0.0",
    "react-markdown": "^9.0.0",
    "date-fns": "^3.0.0",
    "zod": "^3.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vite": "^6.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "vitest": "^2.0.0",
    "@testing-library/react": "^16.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0"
  }
}
```

### 8.2 백엔드

```json
{
  "dependencies": {
    "express": "^4.21.0",
    "@prisma/client": "^5.0.0",
    "jsonwebtoken": "^9.0.0",
    "bcryptjs": "^2.4.0",
    "cors": "^2.8.0",
    "helmet": "^8.0.0",
    "zod": "^3.0.0",
    "slugify": "^1.6.0",
    "uuid": "^10.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "prisma": "^5.0.0",
    "tsx": "^4.0.0",
    "vitest": "^2.0.0",
    "supertest": "^7.0.0",
    "@types/express": "^5.0.0",
    "@types/node": "^22.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0"
  }
}
```

---

## 9. Claude Code 통합

### 9.1 기존 Skills 사용

이 프로젝트는 기존 Claude Code skills와 통합됩니다:

- **backend-dev-guidelines**: Express/TypeScript 패턴
- **frontend-dev-guidelines**: React/MUI 패턴
- **error-tracking**: Sentry 통합
- **route-tester**: API 테스팅

### 9.2 개발 플로우

1. 기능 요구사항 검토 (PRD 참조)
2. API 설계 (API-Spec 참조)
3. 백엔드 구현 (레이어드 아키텍처)
4. 프론트엔드 구현 (기능 기반 모듈)
5. 테스트 실행
6. 개발 문서 업데이트
