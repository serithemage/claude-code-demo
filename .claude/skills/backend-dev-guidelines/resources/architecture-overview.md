# 아키텍처 개요 - 백엔드 서비스

백엔드 마이크로서비스에서 사용되는 계층형 아키텍처 패턴에 대한 완전한 가이드입니다.

---

## RealWorld (Conduit) 프로젝트 개요

이 프로젝트는 **Medium.com 클론** 소셜 블로그 플랫폼입니다.

### 기술 스택

| 영역 | 기술 | 버전 |
|------|------|------|
| **런타임** | Node.js | 20.x LTS |
| **프레임워크** | Express | 4.x |
| **언어** | TypeScript | 5.x |
| **ORM** | Prisma | 5.x |
| **데이터베이스** | SQLite | 3.x |
| **인증** | JWT | jsonwebtoken |
| **비밀번호 해싱** | bcryptjs | 2.4.x |
| **검증** | Zod | 3.x |
| **보안** | Helmet, CORS | 최신 |

### 핵심 도메인

| 도메인 | 설명 | 주요 기능 |
|--------|------|----------|
| **Users** | 사용자 관리 | 가입, 로그인, 프로필 수정 |
| **Articles** | 게시글 관리 | CRUD, 피드, 필터링 |
| **Comments** | 댓글 관리 | 게시글별 댓글 CRUD |
| **Tags** | 태그 관리 | 게시글 태그, 인기 태그 |
| **Favorites** | 좋아요 | 게시글 좋아요/취소 |
| **Follows** | 팔로우 | 사용자 팔로우/언팔로우 |

### 인증 방식

RealWorld 사양에 따른 **JWT 토큰 인증**:

```
Authorization: Token jwt.token.here
```

**중요**: `Bearer` 대신 `Token` 접두사를 사용합니다.

## 목차

- [계층형 아키텍처 패턴](#계층형-아키텍처-패턴)
- [요청 생명주기](#요청-생명주기)
- [서비스 비교](#서비스-비교)
- [디렉토리 구조 근거](#디렉토리-구조-근거)
- [모듈 구성](#모듈-구성)
- [관심사 분리](#관심사-분리)

---

## 계층형 아키텍처 패턴

### 4개 계층

```
┌─────────────────────────────────────┐
│         HTTP 요청                    │
└───────────────┬─────────────────────┘
                ↓
┌─────────────────────────────────────┐
│  계층 1: ROUTES                      │
│  - 라우트 정의만                      │
│  - Middleware 등록                   │
│  - Controller로 위임                 │
│  - 비즈니스 로직 없음                 │
└───────────────┬─────────────────────┘
                ↓
┌─────────────────────────────────────┐
│  계층 2: CONTROLLERS                 │
│  - 요청/응답 처리                     │
│  - 입력 유효성 검사                   │
│  - Service 호출                      │
│  - 응답 포맷팅                        │
│  - 에러 처리                          │
└───────────────┬─────────────────────┘
                ↓
┌─────────────────────────────────────┐
│  계층 3: SERVICES                    │
│  - 비즈니스 로직                      │
│  - 오케스트레이션                     │
│  - Repository 호출                   │
│  - HTTP 지식 없음                     │
└───────────────┬─────────────────────┘
                ↓
┌─────────────────────────────────────┐
│  계층 4: REPOSITORIES                │
│  - 데이터 액세스 추상화               │
│  - Prisma 작업                       │
│  - 쿼리 최적화                        │
│  - 캐싱                               │
└───────────────┬─────────────────────┘
                ↓
┌─────────────────────────────────────┐
│         데이터베이스 (SQLite)          │
└─────────────────────────────────────┘
```

### RealWorld 백엔드 구조

```
backend/
├── src/
│   ├── routes/           # Route 정의
│   │   ├── userRoutes.ts
│   │   ├── articleRoutes.ts
│   │   ├── commentRoutes.ts
│   │   ├── profileRoutes.ts
│   │   └── tagRoutes.ts
│   ├── controllers/      # 요청 처리
│   │   ├── UserController.ts
│   │   ├── ArticleController.ts
│   │   └── ...
│   ├── services/         # 비즈니스 로직
│   │   ├── userService.ts
│   │   ├── articleService.ts
│   │   └── ...
│   ├── repositories/     # 데이터 액세스
│   │   ├── UserRepository.ts
│   │   ├── ArticleRepository.ts
│   │   └── ...
│   ├── middleware/       # 미들웨어
│   │   ├── auth.ts
│   │   ├── errorHandler.ts
│   │   └── validation.ts
│   ├── lib/              # 유틸리티
│   │   ├── prisma.ts
│   │   └── jwt.ts
│   └── types/            # 타입 정의
├── prisma/
│   ├── schema.prisma     # 데이터베이스 스키마
│   └── migrations/       # 마이그레이션
└── package.json
```

### 이 아키텍처를 사용하는 이유

**테스트 용이성:**
- 각 계층을 독립적으로 테스트 가능
- 의존성을 쉽게 모킹
- 명확한 테스트 경계

**유지보수성:**
- 변경 사항이 특정 계층에 격리됨
- 비즈니스 로직이 HTTP 관심사와 분리됨
- 버그 위치를 쉽게 파악

**재사용성:**
- Service를 routes, cron jobs, scripts에서 사용 가능
- Repository가 데이터베이스 구현을 숨김
- 비즈니스 로직이 HTTP에 종속되지 않음

**확장성:**
- 새 엔드포인트 추가가 쉬움
- 따라야 할 명확한 패턴
- 일관된 구조

---

## 요청 생명주기

### 전체 흐름 예시

```typescript
1. HTTP POST /api/users
   ↓
2. Express가 userRoutes.ts에서 라우트 매칭
   ↓
3. Middleware 체인 실행:
   - SSOMiddleware.verifyLoginStatus (인증)
   - auditMiddleware (컨텍스트 추적)
   ↓
4. Route handler가 controller로 위임:
   router.post('/users', (req, res) => userController.create(req, res))
   ↓
5. Controller가 유효성 검사 후 service 호출:
   - Zod로 입력 검증
   - userService.create(data) 호출
   - 성공/에러 처리
   ↓
6. Service가 비즈니스 로직 실행:
   - 비즈니스 규칙 확인
   - userRepository.create(data) 호출
   - 결과 반환
   ↓
7. Repository가 데이터베이스 작업 수행:
   - PrismaService.main.user.create({ data })
   - 데이터베이스 에러 처리
   - 생성된 사용자 반환
   ↓
8. 응답이 역순으로 흐름:
   Repository → Service → Controller → Express → 클라이언트
```

### Middleware 실행 순서

**중요:** Middleware는 등록 순서대로 실행됩니다

```typescript
app.use(Sentry.Handlers.requestHandler());  // 1. Sentry 추적 (첫 번째)
app.use(express.json());                     // 2. Body 파싱
app.use(express.urlencoded({ extended: true })); // 3. URL 인코딩
app.use(cookieParser());                     // 4. Cookie 파싱
app.use(SSOMiddleware.initialize());         // 5. 인증 초기화
// ... 여기서 routes 등록
app.use(auditMiddleware);                    // 6. 감사 (글로벌인 경우)
app.use(errorBoundary);                      // 7. 에러 핸들러 (마지막)
app.use(Sentry.Handlers.errorHandler());     // 8. Sentry 에러 (마지막)
```

**규칙:** 에러 핸들러는 반드시 routes 이후에 등록해야 합니다!

---

## 서비스 비교

### Email Service (성숙한 패턴 ✅)

**강점:**
- Sentry 통합이 포함된 포괄적인 BaseController
- 깔끔한 라우트 위임 (routes에 비즈니스 로직 없음)
- 일관된 의존성 주입 패턴
- 좋은 middleware 구성
- 전체적으로 타입 안전
- 훌륭한 에러 처리

**예시 구조:**
```
email/src/
├── controllers/
│   ├── BaseController.ts          ✅ 훌륭한 템플릿
│   ├── NotificationController.ts  ✅ BaseController 확장
│   └── EmailController.ts         ✅ 깔끔한 패턴
├── routes/
│   ├── notificationRoutes.ts      ✅ 깔끔한 위임
│   └── emailRoutes.ts             ✅ 비즈니스 로직 없음
├── services/
│   ├── NotificationService.ts     ✅ 의존성 주입
│   └── BatchingService.ts         ✅ 명확한 책임
└── middleware/
    ├── errorBoundary.ts           ✅ 포괄적
    └── DevImpersonationSSOMiddleware.ts
```

새 서비스의 **템플릿으로 사용**하세요!

### Form Service (전환 중 ⚠️)

**강점:**
- 우수한 workflow 아키텍처 (이벤트 소싱)
- 좋은 Sentry 통합
- 혁신적인 audit middleware (AsyncLocalStorage)
- 포괄적인 권한 시스템

**약점:**
- 일부 routes에 200줄 이상의 비즈니스 로직
- 일관성 없는 controller 네이밍
- 직접적인 process.env 사용 (60개 이상 발생)
- 최소한의 repository 패턴 사용

**예시:**
```
form/src/
├── routes/
│   ├── responseRoutes.ts          ❌ routes에 비즈니스 로직
│   └── proxyRoutes.ts             ✅ 좋은 유효성 검사 패턴
├── controllers/
│   ├── formController.ts          ⚠️ 소문자 네이밍
│   └── UserProfileController.ts   ✅ PascalCase 네이밍
├── workflow/                      ✅ 훌륭한 아키텍처!
│   ├── core/
│   │   ├── WorkflowEngineV3.ts   ✅ 이벤트 소싱
│   │   └── DryRunWrapper.ts      ✅ 혁신적
│   └── services/
└── middleware/
    └── auditMiddleware.ts         ✅ AsyncLocalStorage 패턴
```

**참고할 것:** workflow/, middleware/auditMiddleware.ts
**피할 것:** responseRoutes.ts, 직접적인 process.env

---

## 디렉토리 구조 근거

### Controllers 디렉토리

**목적:** HTTP 요청/응답 관심사 처리

**내용:**
- `BaseController.ts` - 공통 메서드가 있는 기본 클래스
- `{Feature}Controller.ts` - 기능별 controllers

**네이밍:** PascalCase + Controller

**책임:**
- 요청 파라미터 파싱
- 입력 유효성 검사 (Zod)
- 적절한 service 메서드 호출
- 응답 포맷팅
- 에러 처리 (BaseController 통해)
- HTTP 상태 코드 설정

### Services 디렉토리

**목적:** 비즈니스 로직과 오케스트레이션

**내용:**
- `{feature}Service.ts` - 기능 비즈니스 로직

**네이밍:** camelCase + Service (또는 PascalCase + Service)

**책임:**
- 비즈니스 규칙 구현
- 여러 repositories 오케스트레이션
- 트랜잭션 관리
- 비즈니스 유효성 검사
- HTTP 지식 없음 (Request/Response 타입)

### Repositories 디렉토리

**목적:** 데이터 액세스 추상화

**내용:**
- `{Entity}Repository.ts` - 엔티티의 데이터베이스 작업

**네이밍:** PascalCase + Repository

**책임:**
- Prisma 쿼리 작업
- 쿼리 최적화
- 데이터베이스 에러 처리
- 캐싱 계층
- Prisma 구현 세부사항 숨김

**현재 갭:** 1개의 repository만 존재 (WorkflowRepository)

### Routes 디렉토리

**목적:** 라우트 등록만

**내용:**
- `{feature}Routes.ts` - 기능을 위한 Express router

**네이밍:** camelCase + Routes

**책임:**
- Express에 routes 등록
- Middleware 적용
- Controllers로 위임
- **비즈니스 로직 없음!**

### Middleware 디렉토리

**목적:** 횡단 관심사

**내용:**
- 인증 middleware
- Audit middleware
- Error boundaries
- Validation middleware
- 커스텀 middleware

**네이밍:** camelCase

**유형:**
- 요청 처리 (핸들러 이전)
- 응답 처리 (핸들러 이후)
- 에러 처리 (error boundary)

### Config 디렉토리

**목적:** 설정 관리

**내용:**
- `unifiedConfig.ts` - 타입 안전 설정
- 환경별 설정

**패턴:** 단일 진실 공급원

### Types 디렉토리

**목적:** TypeScript 타입 정의

**내용:**
- `{feature}.types.ts` - 기능별 타입
- DTOs (Data Transfer Objects)
- Request/Response 타입
- 도메인 모델

---

## 모듈 구성

### 기능 기반 구성

대규모 기능의 경우 하위 디렉토리 사용:

```
src/workflow/
├── core/              # 핵심 엔진
├── services/          # Workflow 전용 services
├── actions/           # 시스템 액션
├── models/            # 도메인 모델
├── validators/        # Workflow 유효성 검사
└── utils/             # Workflow 유틸리티
```

**사용 시점:**
- 기능에 5개 이상의 파일이 있을 때
- 명확한 하위 도메인이 존재할 때
- 논리적 그룹화가 명확성을 높일 때

### 평면 구성

간단한 기능의 경우:

```
src/
├── controllers/UserController.ts
├── services/userService.ts
├── routes/userRoutes.ts
└── repositories/UserRepository.ts
```

**사용 시점:**
- 간단한 기능 (< 5개 파일)
- 명확한 하위 도메인 없음
- 평면 구조가 더 명확할 때

---

## 관심사 분리

### 무엇을 어디에 둘 것인가

**Routes 계층:**
- ✅ 라우트 정의
- ✅ Middleware 등록
- ✅ Controller 위임
- ❌ 비즈니스 로직
- ❌ 데이터베이스 작업
- ❌ 유효성 검사 로직 (validator 또는 controller에 있어야 함)

**Controllers 계층:**
- ✅ 요청 파싱 (params, body, query)
- ✅ 입력 유효성 검사 (Zod)
- ✅ Service 호출
- ✅ 응답 포맷팅
- ✅ 에러 처리
- ❌ 비즈니스 로직
- ❌ 데이터베이스 작업

**Services 계층:**
- ✅ 비즈니스 로직
- ✅ 비즈니스 규칙 적용
- ✅ 오케스트레이션 (여러 repos)
- ✅ 트랜잭션 관리
- ❌ HTTP 관심사 (Request/Response)
- ❌ 직접 Prisma 호출 (repositories 사용)

**Repositories 계층:**
- ✅ Prisma 작업
- ✅ 쿼리 구성
- ✅ 데이터베이스 에러 처리
- ✅ 캐싱
- ❌ 비즈니스 로직
- ❌ HTTP 관심사

### 예시: 사용자 생성

**Route:**
```typescript
router.post('/users',
    SSOMiddleware.verifyLoginStatus,
    auditMiddleware,
    (req, res) => userController.create(req, res)
);
```

**Controller:**
```typescript
async create(req: Request, res: Response): Promise<void> {
    try {
        const validated = createUserSchema.parse(req.body);
        const user = await this.userService.create(validated);
        this.handleSuccess(res, user, 'User created');
    } catch (error) {
        this.handleError(error, res, 'create');
    }
}
```

**Service:**
```typescript
async create(data: CreateUserDTO): Promise<User> {
    // 비즈니스 규칙: 이메일이 이미 존재하는지 확인
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) throw new ConflictError('Email already exists');

    // 사용자 생성
    return await this.userRepository.create(data);
}
```

**Repository:**
```typescript
async create(data: CreateUserDTO): Promise<User> {
    return PrismaService.main.user.create({ data });
}

async findByEmail(email: string): Promise<User | null> {
    return PrismaService.main.user.findUnique({ where: { email } });
}
```

**주목:** 각 계층이 명확하고 구별된 책임을 가집니다!

---

**관련 파일:**
- [SKILL.md](SKILL.md) - 메인 가이드
- [routing-and-controllers.md](routing-and-controllers.md) - Routes와 controllers 세부사항
- [services-and-repositories.md](services-and-repositories.md) - Service와 repository 패턴
