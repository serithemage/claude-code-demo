# Routing과 Controllers - 모범 사례

깔끔한 라우트 정의와 controller 패턴에 대한 완전한 가이드입니다.

---

## RealWorld API 엔드포인트

### 인증 API

| 메서드 | 엔드포인트         | 인증   | 설명             |
| ------ | ------------------ | ------ | ---------------- |
| POST   | `/api/users`       | 불필요 | 사용자 가입      |
| POST   | `/api/users/login` | 불필요 | 로그인           |
| GET    | `/api/user`        | 필수   | 현재 사용자 조회 |
| PUT    | `/api/user`        | 필수   | 사용자 정보 수정 |

### 프로필 API

| 메서드 | 엔드포인트                       | 인증 | 설명        |
| ------ | -------------------------------- | ---- | ----------- |
| GET    | `/api/profiles/:username`        | 선택 | 프로필 조회 |
| POST   | `/api/profiles/:username/follow` | 필수 | 팔로우      |
| DELETE | `/api/profiles/:username/follow` | 필수 | 언팔로우    |

### 게시글 API

| 메서드 | 엔드포인트            | 인증 | 설명                      |
| ------ | --------------------- | ---- | ------------------------- |
| GET    | `/api/articles`       | 선택 | 게시글 목록 (필터링 지원) |
| GET    | `/api/articles/feed`  | 필수 | 피드 (팔로우한 사용자)    |
| GET    | `/api/articles/:slug` | 선택 | 게시글 상세               |
| POST   | `/api/articles`       | 필수 | 게시글 작성               |
| PUT    | `/api/articles/:slug` | 필수 | 게시글 수정 (작성자만)    |
| DELETE | `/api/articles/:slug` | 필수 | 게시글 삭제 (작성자만)    |

### 댓글 API

| 메서드 | 엔드포인트                         | 인증 | 설명                 |
| ------ | ---------------------------------- | ---- | -------------------- |
| GET    | `/api/articles/:slug/comments`     | 선택 | 댓글 목록            |
| POST   | `/api/articles/:slug/comments`     | 필수 | 댓글 작성            |
| DELETE | `/api/articles/:slug/comments/:id` | 필수 | 댓글 삭제 (작성자만) |

### 좋아요 & 태그 API

| 메서드 | 엔드포인트                     | 인증   | 설명        |
| ------ | ------------------------------ | ------ | ----------- |
| POST   | `/api/articles/:slug/favorite` | 필수   | 좋아요 추가 |
| DELETE | `/api/articles/:slug/favorite` | 필수   | 좋아요 취소 |
| GET    | `/api/tags`                    | 불필요 | 태그 목록   |

---

## 목차

- [Routes: 라우팅만](#routes-라우팅만)
- [BaseController 패턴](#basecontroller-패턴)
- [좋은 예시](#좋은-예시)
- [안티패턴](#안티패턴)
- [리팩토링 가이드](#리팩토링-가이드)
- [에러 처리](#에러-처리)
- [HTTP 상태 코드](#http-상태-코드)

---

## Routes: 라우팅만

### 황금 규칙

**Routes는 다음만 해야 합니다:**

- ✅ 라우트 경로 정의
- ✅ Middleware 등록
- ✅ Controllers로 위임

**Routes는 절대 해서는 안 됩니다:**

- ❌ 비즈니스 로직 포함
- ❌ 데이터베이스 직접 액세스
- ❌ 유효성 검사 로직 구현 (Zod + controller 사용)
- ❌ 복잡한 응답 포맷팅
- ❌ 복잡한 에러 시나리오 처리

### 깔끔한 Route 패턴

```typescript
// routes/userRoutes.ts
import { Router } from 'express';
import { UserController } from '../controllers/UserController';
import { SSOMiddlewareClient } from '../middleware/SSOMiddleware';
import { auditMiddleware } from '../middleware/auditMiddleware';

const router = Router();
const controller = new UserController();

// ✅ 깔끔함: 라우트 정의만
router.get('/:id', SSOMiddlewareClient.verifyLoginStatus, auditMiddleware, async (req, res) =>
  controller.getUser(req, res)
);

router.post('/', SSOMiddlewareClient.verifyLoginStatus, auditMiddleware, async (req, res) =>
  controller.createUser(req, res)
);

router.put('/:id', SSOMiddlewareClient.verifyLoginStatus, auditMiddleware, async (req, res) =>
  controller.updateUser(req, res)
);

export default router;
```

**핵심 포인트:**

- 각 route: method, path, middleware chain, controller 위임
- try-catch 불필요 (controller가 에러 처리)
- 깔끔하고, 읽기 쉽고, 유지보수 용이
- 한눈에 모든 엔드포인트 파악 가능

---

## BaseController 패턴

### BaseController를 사용하는 이유

**장점:**

- 모든 controllers에서 일관된 에러 처리
- 자동 Sentry 통합
- 표준화된 응답 포맷
- 재사용 가능한 헬퍼 메서드
- 성능 추적 유틸리티
- 로깅과 breadcrumb 헬퍼

### BaseController 패턴 (템플릿)

**파일:** `/email/src/controllers/BaseController.ts`

```typescript
import * as Sentry from '@sentry/node';
import { Response } from 'express';

export abstract class BaseController {
  /**
   * Sentry 통합으로 에러 처리
   */
  protected handleError(error: unknown, res: Response, context: string, statusCode = 500): void {
    Sentry.withScope((scope) => {
      scope.setTag('controller', this.constructor.name);
      scope.setTag('operation', context);
      scope.setUser({ id: res.locals?.claims?.userId });

      if (error instanceof Error) {
        scope.setContext('error_details', {
          message: error.message,
          stack: error.stack,
        });
      }

      Sentry.captureException(error);
    });

    res.status(statusCode).json({
      success: false,
      error: {
        message: error instanceof Error ? error.message : 'An error occurred',
        code: statusCode,
      },
    });
  }

  /**
   * 성공 응답 처리
   */
  protected handleSuccess<T>(res: Response, data: T, message?: string, statusCode = 200): void {
    res.status(statusCode).json({
      success: true,
      message,
      data,
    });
  }

  /**
   * 성능 추적 wrapper
   */
  protected async withTransaction<T>(
    name: string,
    operation: string,
    callback: () => Promise<T>
  ): Promise<T> {
    return await Sentry.startSpan({ name, op: operation }, callback);
  }

  /**
   * 필수 필드 유효성 검사
   */
  protected validateRequest(
    required: string[],
    actual: Record<string, any>,
    res: Response
  ): boolean {
    const missing = required.filter((field) => !actual[field]);

    if (missing.length > 0) {
      Sentry.captureMessage(`Missing required fields: ${missing.join(', ')}`, 'warning');

      res.status(400).json({
        success: false,
        error: {
          message: 'Missing required fields',
          code: 'VALIDATION_ERROR',
          details: { missing },
        },
      });
      return false;
    }
    return true;
  }

  /**
   * 로깅 헬퍼
   */
  protected logInfo(message: string, context?: Record<string, any>): void {
    Sentry.addBreadcrumb({
      category: this.constructor.name,
      message,
      level: 'info',
      data: context,
    });
  }

  protected logWarning(message: string, context?: Record<string, any>): void {
    Sentry.captureMessage(message, {
      level: 'warning',
      tags: { controller: this.constructor.name },
      extra: context,
    });
  }

  /**
   * Sentry breadcrumb 추가
   */
  protected addBreadcrumb(message: string, category: string, data?: Record<string, any>): void {
    Sentry.addBreadcrumb({ message, category, level: 'info', data });
  }

  /**
   * 커스텀 메트릭 캡처
   */
  protected captureMetric(name: string, value: number, unit: string): void {
    Sentry.metrics.gauge(name, value, { unit });
  }
}
```

### BaseController 사용하기

```typescript
// controllers/UserController.ts
import { Request, Response } from 'express';
import { BaseController } from './BaseController';
import { UserService } from '../services/userService';
import { createUserSchema } from '../validators/userSchemas';

export class UserController extends BaseController {
  private userService: UserService;

  constructor() {
    super();
    this.userService = new UserService();
  }

  async getUser(req: Request, res: Response): Promise<void> {
    try {
      this.addBreadcrumb('Fetching user', 'user_controller', { userId: req.params.id });

      const user = await this.userService.findById(req.params.id);

      if (!user) {
        return this.handleError(new Error('User not found'), res, 'getUser', 404);
      }

      this.handleSuccess(res, user);
    } catch (error) {
      this.handleError(error, res, 'getUser');
    }
  }

  async createUser(req: Request, res: Response): Promise<void> {
    try {
      // 입력 유효성 검사
      const validated = createUserSchema.parse(req.body);

      // 성능 추적
      const user = await this.withTransaction('user.create', 'db.query', () =>
        this.userService.create(validated)
      );

      this.handleSuccess(res, user, 'User created successfully', 201);
    } catch (error) {
      this.handleError(error, res, 'createUser');
    }
  }

  async updateUser(req: Request, res: Response): Promise<void> {
    try {
      const validated = updateUserSchema.parse(req.body);
      const user = await this.userService.update(req.params.id, validated);
      this.handleSuccess(res, user, 'User updated');
    } catch (error) {
      this.handleError(error, res, 'updateUser');
    }
  }
}
```

**장점:**

- 일관된 에러 처리
- 자동 Sentry 통합
- 성능 추적
- 깔끔하고 읽기 쉬운 코드
- 테스트하기 쉬움

---

## 좋은 예시

### 예시 1: Email Notification Routes (훌륭함 ✅)

**파일:** `/email/src/routes/notificationRoutes.ts`

```typescript
import { Router } from 'express';
import { NotificationController } from '../controllers/NotificationController';
import { SSOMiddlewareClient } from '../middleware/SSOMiddleware';

const router = Router();
const controller = new NotificationController();

// ✅ 훌륭함: 깔끔한 위임
router.get('/', SSOMiddlewareClient.verifyLoginStatus, async (req, res) =>
  controller.getNotifications(req, res)
);

router.post('/', SSOMiddlewareClient.verifyLoginStatus, async (req, res) =>
  controller.createNotification(req, res)
);

router.put('/:id/read', SSOMiddlewareClient.verifyLoginStatus, async (req, res) =>
  controller.markAsRead(req, res)
);

export default router;
```

**훌륭한 이유:**

- Routes에 비즈니스 로직 없음
- 명확한 middleware 체인
- 일관된 패턴
- 이해하기 쉬움

### 예시 2: 유효성 검사가 있는 Proxy Routes (좋음 ✅)

**파일:** `/form/src/routes/proxyRoutes.ts`

```typescript
import { z } from 'zod';

const createProxySchema = z.object({
  originalUserID: z.string().min(1),
  proxyUserID: z.string().min(1),
  startsAt: z.string().datetime(),
  expiresAt: z.string().datetime(),
});

router.post('/', SSOMiddlewareClient.verifyLoginStatus, async (req, res) => {
  try {
    const validated = createProxySchema.parse(req.body);
    const proxy = await proxyService.createProxyRelationship(validated);
    res.status(201).json({ success: true, data: proxy });
  } catch (error) {
    handler.handleException(res, error);
  }
});
```

**좋은 이유:**

- Zod 유효성 검사
- Service로 위임
- 적절한 HTTP 상태 코드
- 에러 처리

**개선할 수 있는 점:**

- 유효성 검사를 controller로 이동
- BaseController 사용

---

## 안티패턴

### 안티패턴 1: Routes에 비즈니스 로직 (나쁨 ❌)

**파일:** `/form/src/routes/responseRoutes.ts` (실제 프로덕션 코드)

```typescript
// ❌ 안티패턴: route에 200줄 이상의 비즈니스 로직
router.post('/:formID/submit', async (req: Request, res: Response) => {
  try {
    const username = res.locals.claims.preferred_username;
    const responses = req.body.responses;
    const stepInstanceId = req.body.stepInstanceId;

    // ❌ Route에서 권한 확인
    const userId = await userProfileService.getProfileByEmail(username).then((p) => p.id);
    const canComplete = await permissionService.canCompleteStep(userId, stepInstanceId);
    if (!canComplete) {
      return res.status(403).json({ error: 'No permission' });
    }

    // ❌ Route에서 Workflow 로직
    const {
      createWorkflowEngine,
      CompleteStepCommand,
    } = require('../workflow/core/WorkflowEngineV3');
    const engine = await createWorkflowEngine();
    const command = new CompleteStepCommand(stepInstanceId, userId, responses, additionalContext);
    const events = await engine.executeCommand(command);

    // ❌ Route에서 Impersonation 처리
    if (res.locals.isImpersonating) {
      impersonationContextStore.storeContext(stepInstanceId, {
        originalUserId: res.locals.originalUserId,
        effectiveUserId: userId,
      });
    }

    // ❌ Route에서 응답 처리
    const post = await PrismaService.main.post.findUnique({
      where: { id: postData.id },
      include: { comments: true },
    });

    // ❌ Route에서 권한 확인
    await checkPostPermissions(post, userId);

    // ... 100줄 이상의 비즈니스 로직

    res.json({ success: true, data: result });
  } catch (e) {
    handler.handleException(res, e);
  }
});
```

**왜 끔찍한가:**

- 200줄 이상의 비즈니스 로직
- 테스트하기 어려움 (HTTP 모킹 필요)
- 재사용하기 어려움 (route에 종속)
- 혼합된 책임
- 디버깅하기 어려움
- 성능 추적하기 어려움

### 리팩토링 방법 (단계별)

**단계 1: Controller 생성**

```typescript
// controllers/PostController.ts
export class PostController extends BaseController {
  private postService: PostService;

  constructor() {
    super();
    this.postService = new PostService();
  }

  async createPost(req: Request, res: Response): Promise<void> {
    try {
      const validated = createPostSchema.parse({
        ...req.body,
      });

      const result = await this.postService.createPost(validated, res.locals.userId);

      this.handleSuccess(res, result, 'Post created successfully');
    } catch (error) {
      this.handleError(error, res, 'createPost');
    }
  }
}
```

**단계 2: Service 생성**

```typescript
// services/postService.ts
export class PostService {
  async createPost(data: CreatePostDTO, userId: string): Promise<PostResult> {
    // 권한 확인
    const canCreate = await permissionService.canCreatePost(userId);
    if (!canCreate) {
      throw new ForbiddenError('No permission to create post');
    }

    // Workflow 실행
    const engine = await createWorkflowEngine();
    const command = new CompleteStepCommand(/* ... */);
    const events = await engine.executeCommand(command);

    // 필요한 경우 impersonation 처리
    if (context.isImpersonating) {
      await this.handleImpersonation(data.stepInstanceId, context);
    }

    // 역할 동기화
    await this.synchronizeRoles(events, userId);

    return { events, success: true };
  }

  private async handleImpersonation(stepInstanceId: number, context: any) {
    impersonationContextStore.storeContext(stepInstanceId, {
      originalUserId: context.originalUserId,
      effectiveUserId: context.effectiveUserId,
    });
  }

  private async synchronizeRoles(events: WorkflowEvent[], userId: string) {
    // 역할 동기화 로직
  }
}
```

**단계 3: Route 업데이트**

```typescript
// routes/postRoutes.ts
import { PostController } from '../controllers/PostController';

const router = Router();
const controller = new PostController();

// ✅ 깔끔함: 라우팅만
router.post('/', SSOMiddlewareClient.verifyLoginStatus, auditMiddleware, async (req, res) =>
  controller.createPost(req, res)
);
```

**결과:**

- Route: 8줄 (200줄 이상이었음)
- Controller: 25줄 (요청 처리)
- Service: 50줄 (비즈니스 로직)
- 테스트 가능, 재사용 가능, 유지보수 가능!

---

## 에러 처리

### Controller 에러 처리

```typescript
async createUser(req: Request, res: Response): Promise<void> {
    try {
        const result = await this.userService.create(req.body);
        this.handleSuccess(res, result, 'User created', 201);
    } catch (error) {
        // BaseController.handleError가 자동으로:
        // - 컨텍스트와 함께 Sentry에 캡처
        // - 적절한 상태 코드 설정
        // - 포맷된 에러 응답 반환
        this.handleError(error, res, 'createUser');
    }
}
```

### 커스텀 에러 상태 코드

```typescript
async getUser(req: Request, res: Response): Promise<void> {
    try {
        const user = await this.userService.findById(req.params.id);

        if (!user) {
            // 커스텀 404 상태
            return this.handleError(
                new Error('User not found'),
                res,
                'getUser',
                404  // 커스텀 상태 코드
            );
        }

        this.handleSuccess(res, user);
    } catch (error) {
        this.handleError(error, res, 'getUser');
    }
}
```

### 유효성 검사 에러

```typescript
async createUser(req: Request, res: Response): Promise<void> {
    try {
        const validated = createUserSchema.parse(req.body);
        const user = await this.userService.create(validated);
        this.handleSuccess(res, user, 'User created', 201);
    } catch (error) {
        // Zod 에러는 400 상태
        if (error instanceof z.ZodError) {
            return this.handleError(error, res, 'createUser', 400);
        }
        this.handleError(error, res, 'createUser');
    }
}
```

---

## HTTP 상태 코드

### 표준 코드

| 코드 | 사용 케이스           | 예시                      |
| ---- | --------------------- | ------------------------- |
| 200  | 성공 (GET, PUT)       | 사용자 조회, 업데이트     |
| 201  | 생성됨 (POST)         | 사용자 생성               |
| 204  | 내용 없음 (DELETE)    | 사용자 삭제               |
| 400  | 잘못된 요청           | 유효하지 않은 입력 데이터 |
| 401  | 인증 안 됨            | 인증되지 않음             |
| 403  | 금지됨                | 권한 없음                 |
| 404  | 찾을 수 없음          | 리소스가 존재하지 않음    |
| 409  | 충돌                  | 중복 리소스               |
| 422  | 처리할 수 없는 엔티티 | 유효성 검사 실패          |
| 500  | 내부 서버 에러        | 예상치 못한 에러          |

### 사용 예시

```typescript
// 200 - 성공 (기본값)
this.handleSuccess(res, user);

// 201 - 생성됨
this.handleSuccess(res, user, 'Created', 201);

// 400 - 잘못된 요청
this.handleError(error, res, 'operation', 400);

// 404 - 찾을 수 없음
this.handleError(new Error('Not found'), res, 'operation', 404);

// 403 - 금지됨
this.handleError(new ForbiddenError('No permission'), res, 'operation', 403);
```

---

## 리팩토링 가이드

### 리팩토링이 필요한 Routes 식별

**위험 신호:**

- Route 파일 > 100줄
- 하나의 route에 여러 try-catch 블록
- 직접 데이터베이스 액세스 (Prisma 호출)
- 복잡한 비즈니스 로직 (if 문, 루프)
- Routes에서 권한 확인

**routes 확인:**

```bash
# 큰 route 파일 찾기
wc -l form/src/routes/*.ts | sort -n

# Prisma 사용이 있는 routes 찾기
grep -r "PrismaService" form/src/routes/
```

### 리팩토링 프로세스

**1. Controller로 추출:**

```typescript
// 이전: 로직이 있는 Route
router.post('/action', async (req, res) => {
    try {
        // 50줄의 로직
    } catch (e) {
        handler.handleException(res, e);
    }
});

// 이후: 깔끔한 route
router.post('/action', (req, res) => controller.performAction(req, res));

// 새 controller 메서드
async performAction(req: Request, res: Response): Promise<void> {
    try {
        const result = await this.service.performAction(req.body);
        this.handleSuccess(res, result);
    } catch (error) {
        this.handleError(error, res, 'performAction');
    }
}
```

**2. Service로 추출:**

```typescript
// Controller는 얇게 유지
async performAction(req: Request, res: Response): Promise<void> {
    try {
        const validated = actionSchema.parse(req.body);
        const result = await this.actionService.execute(validated);
        this.handleSuccess(res, result);
    } catch (error) {
        this.handleError(error, res, 'performAction');
    }
}

// Service가 비즈니스 로직 포함
export class ActionService {
    async execute(data: ActionDTO): Promise<Result> {
        // 모든 비즈니스 로직은 여기
        // 권한 확인
        // 데이터베이스 작업
        // 복잡한 변환
        return result;
    }
}
```

**3. Repository 추가 (필요한 경우):**

```typescript
// Service가 repository 호출
export class ActionService {
  constructor(private actionRepository: ActionRepository) {}

  async execute(data: ActionDTO): Promise<Result> {
    // 비즈니스 로직
    const entity = await this.actionRepository.findById(data.id);
    // 더 많은 로직
    return await this.actionRepository.update(data.id, changes);
  }
}

// Repository가 데이터 액세스 처리
export class ActionRepository {
  async findById(id: number): Promise<Entity | null> {
    return PrismaService.main.entity.findUnique({ where: { id } });
  }

  async update(id: number, data: Partial<Entity>): Promise<Entity> {
    return PrismaService.main.entity.update({ where: { id }, data });
  }
}
```

---

**관련 파일:**

- [SKILL.md](SKILL.md) - 메인 가이드
- [services-and-repositories.md](services-and-repositories.md) - Service 계층 세부사항
- [complete-examples.md](complete-examples.md) - 전체 리팩토링 예제
