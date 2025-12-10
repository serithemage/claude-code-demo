# 완전한 예제 - 실제 작동하는 코드

모든 구현 패턴을 보여주는 실제 예제입니다.

## 목차

- [완전한 Controller 예제](#완전한-controller-예제)
- [DI가 포함된 완전한 Service](#di가-포함된-완전한-service)
- [완전한 Route 파일](#완전한-route-파일)
- [완전한 Repository](#완전한-repository)
- [리팩토링 예제: 나쁜 코드에서 좋은 코드로](#리팩토링-예제-나쁜-코드에서-좋은-코드로)
- [End-to-End 기능 예제](#end-to-end-기능-예제)

---

## 완전한 Controller 예제

### UserController (모든 모범 사례 적용)

```typescript
// controllers/UserController.ts
import { Request, Response } from 'express';
import { BaseController } from './BaseController';
import { UserService } from '../services/userService';
import { createUserSchema, updateUserSchema } from '../validators/userSchemas';
import { z } from 'zod';

export class UserController extends BaseController {
    private userService: UserService;

    constructor() {
        super();
        this.userService = new UserService();
    }

    async getUser(req: Request, res: Response): Promise<void> {
        try {
            this.addBreadcrumb('Fetching user', 'user_controller', {
                userId: req.params.id,
            });

            const user = await this.withTransaction(
                'user.get',
                'db.query',
                () => this.userService.findById(req.params.id)
            );

            if (!user) {
                return this.handleError(
                    new Error('User not found'),
                    res,
                    'getUser',
                    404
                );
            }

            this.handleSuccess(res, user);
        } catch (error) {
            this.handleError(error, res, 'getUser');
        }
    }

    async listUsers(req: Request, res: Response): Promise<void> {
        try {
            const users = await this.userService.getAll();
            this.handleSuccess(res, users);
        } catch (error) {
            this.handleError(error, res, 'listUsers');
        }
    }

    async createUser(req: Request, res: Response): Promise<void> {
        try {
            // Zod로 입력 유효성 검사
            const validated = createUserSchema.parse(req.body);

            // 성능 추적
            const user = await this.withTransaction(
                'user.create',
                'db.mutation',
                () => this.userService.create(validated)
            );

            this.handleSuccess(res, user, 'User created successfully', 201);
        } catch (error) {
            if (error instanceof z.ZodError) {
                return this.handleError(error, res, 'createUser', 400);
            }
            this.handleError(error, res, 'createUser');
        }
    }

    async updateUser(req: Request, res: Response): Promise<void> {
        try {
            const validated = updateUserSchema.parse(req.body);

            const user = await this.userService.update(
                req.params.id,
                validated
            );

            this.handleSuccess(res, user, 'User updated');
        } catch (error) {
            if (error instanceof z.ZodError) {
                return this.handleError(error, res, 'updateUser', 400);
            }
            this.handleError(error, res, 'updateUser');
        }
    }

    async deleteUser(req: Request, res: Response): Promise<void> {
        try {
            await this.userService.delete(req.params.id);
            this.handleSuccess(res, null, 'User deleted', 204);
        } catch (error) {
            this.handleError(error, res, 'deleteUser');
        }
    }
}
```

---

## DI가 포함된 완전한 Service

### UserService

```typescript
// services/userService.ts
import { UserRepository } from '../repositories/UserRepository';
import { ConflictError, NotFoundError, ValidationError } from '../types/errors';
import type { CreateUserDTO, UpdateUserDTO, User } from '../types/user.types';

export class UserService {
    private userRepository: UserRepository;

    constructor(userRepository?: UserRepository) {
        this.userRepository = userRepository || new UserRepository();
    }

    async findById(id: string): Promise<User | null> {
        return await this.userRepository.findById(id);
    }

    async getAll(): Promise<User[]> {
        return await this.userRepository.findActive();
    }

    async create(data: CreateUserDTO): Promise<User> {
        // 비즈니스 규칙: 나이 검증
        if (data.age < 18) {
            throw new ValidationError('User must be 18 or older');
        }

        // 비즈니스 규칙: 이메일 고유성 확인
        const existing = await this.userRepository.findByEmail(data.email);
        if (existing) {
            throw new ConflictError('Email already in use');
        }

        // 프로필과 함께 사용자 생성
        return await this.userRepository.create({
            email: data.email,
            profile: {
                create: {
                    firstName: data.firstName,
                    lastName: data.lastName,
                    age: data.age,
                },
            },
        });
    }

    async update(id: string, data: UpdateUserDTO): Promise<User> {
        // 존재 여부 확인
        const existing = await this.userRepository.findById(id);
        if (!existing) {
            throw new NotFoundError('User not found');
        }

        // 비즈니스 규칙: 변경 시 이메일 고유성
        if (data.email && data.email !== existing.email) {
            const emailTaken = await this.userRepository.findByEmail(data.email);
            if (emailTaken) {
                throw new ConflictError('Email already in use');
            }
        }

        return await this.userRepository.update(id, data);
    }

    async delete(id: string): Promise<void> {
        const existing = await this.userRepository.findById(id);
        if (!existing) {
            throw new NotFoundError('User not found');
        }

        await this.userRepository.delete(id);
    }
}
```

---

## 완전한 Route 파일

### userRoutes.ts

```typescript
// routes/userRoutes.ts
import { Router } from 'express';
import { UserController } from '../controllers/UserController';
import { SSOMiddlewareClient } from '../middleware/SSOMiddleware';
import { auditMiddleware } from '../middleware/auditMiddleware';

const router = Router();
const controller = new UserController();

// GET /users - 모든 사용자 목록
router.get('/',
    SSOMiddlewareClient.verifyLoginStatus,
    auditMiddleware,
    async (req, res) => controller.listUsers(req, res)
);

// GET /users/:id - 단일 사용자 조회
router.get('/:id',
    SSOMiddlewareClient.verifyLoginStatus,
    auditMiddleware,
    async (req, res) => controller.getUser(req, res)
);

// POST /users - 사용자 생성
router.post('/',
    SSOMiddlewareClient.verifyLoginStatus,
    auditMiddleware,
    async (req, res) => controller.createUser(req, res)
);

// PUT /users/:id - 사용자 업데이트
router.put('/:id',
    SSOMiddlewareClient.verifyLoginStatus,
    auditMiddleware,
    async (req, res) => controller.updateUser(req, res)
);

// DELETE /users/:id - 사용자 삭제
router.delete('/:id',
    SSOMiddlewareClient.verifyLoginStatus,
    auditMiddleware,
    async (req, res) => controller.deleteUser(req, res)
);

export default router;
```

---

## 완전한 Repository

### UserRepository

```typescript
// repositories/UserRepository.ts
import { PrismaService } from '@project-lifecycle-portal/database';
import type { User, Prisma } from '@prisma/client';

export class UserRepository {
    async findById(id: string): Promise<User | null> {
        return PrismaService.main.user.findUnique({
            where: { id },
            include: { profile: true },
        });
    }

    async findByEmail(email: string): Promise<User | null> {
        return PrismaService.main.user.findUnique({
            where: { email },
            include: { profile: true },
        });
    }

    async findActive(): Promise<User[]> {
        return PrismaService.main.user.findMany({
            where: { isActive: true },
            include: { profile: true },
            orderBy: { createdAt: 'desc' },
        });
    }

    async create(data: Prisma.UserCreateInput): Promise<User> {
        return PrismaService.main.user.create({
            data,
            include: { profile: true },
        });
    }

    async update(id: string, data: Prisma.UserUpdateInput): Promise<User> {
        return PrismaService.main.user.update({
            where: { id },
            data,
            include: { profile: true },
        });
    }

    async delete(id: string): Promise<User> {
        // Soft delete
        return PrismaService.main.user.update({
            where: { id },
            data: {
                isActive: false,
                deletedAt: new Date(),
            },
        });
    }
}
```

---

## 리팩토링 예제: 나쁜 코드에서 좋은 코드로

### 이전: Routes에 비즈니스 로직 ❌

```typescript
// routes/postRoutes.ts (나쁜 예 - 200줄 이상)
router.post('/posts', async (req, res) => {
    try {
        const username = res.locals.claims.preferred_username;
        const responses = req.body.responses;
        const stepInstanceId = req.body.stepInstanceId;

        // ❌ Route에서 권한 확인
        const userId = await userProfileService.getProfileByEmail(username).then(p => p.id);
        const canComplete = await permissionService.canCompleteStep(userId, stepInstanceId);
        if (!canComplete) {
            return res.status(403).json({ error: 'No permission' });
        }

        // ❌ Route에서 비즈니스 로직
        const post = await postRepository.create({
            title: req.body.title,
            content: req.body.content,
            authorId: userId
        });

        // ❌ 더 많은 비즈니스 로직...
        if (res.locals.isImpersonating) {
            impersonationContextStore.storeContext(...);
        }

        // ... 100줄 이상

        res.json({ success: true, data: result });
    } catch (e) {
        handler.handleException(res, e);
    }
});
```

### 이후: 깔끔한 분리 ✅

**1. 깔끔한 Route:**
```typescript
// routes/postRoutes.ts
import { PostController } from '../controllers/PostController';

const router = Router();
const controller = new PostController();

// ✅ 깔끔함: 총 8줄!
router.post('/',
    SSOMiddlewareClient.verifyLoginStatus,
    auditMiddleware,
    async (req, res) => controller.createPost(req, res)
);

export default router;
```

**2. Controller:**
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

            const result = await this.postService.createPost(
                validated,
                res.locals.userId
            );

            this.handleSuccess(res, result, 'Post created successfully');
        } catch (error) {
            this.handleError(error, res, 'createPost');
        }
    }
}
```

**3. Service:**
```typescript
// services/postService.ts
export class PostService {
    async createPost(
        data: CreatePostDTO,
        userId: string
    ): Promise<SubmissionResult> {
        // 권한 확인
        const canComplete = await permissionService.canCompleteStep(
            userId,
            data.stepInstanceId
        );

        if (!canComplete) {
            throw new ForbiddenError('No permission to complete step');
        }

        // Workflow 실행
        const engine = await createWorkflowEngine();
        const command = new CompleteStepCommand(
            data.stepInstanceId,
            userId,
            data.responses
        );
        const events = await engine.executeCommand(command);

        // Impersonation 처리
        if (context.isImpersonating) {
            await this.handleImpersonation(data.stepInstanceId, context);
        }

        return { events, success: true };
    }

    private async handleImpersonation(stepInstanceId: number, context: any) {
        impersonationContextStore.storeContext(stepInstanceId, {
            originalUserId: context.originalUserId,
            effectiveUserId: context.effectiveUserId,
        });
    }
}
```

**결과:**
- Route: 8줄 (200줄 이상이었음)
- Controller: 25줄
- Service: 40줄
- **테스트 가능, 유지보수 가능, 재사용 가능!**

---

## End-to-End 기능 예제

### 완전한 사용자 관리 기능

**1. Types:**
```typescript
// types/user.types.ts
export interface User {
    id: string;
    email: string;
    isActive: boolean;
    profile?: UserProfile;
}

export interface CreateUserDTO {
    email: string;
    firstName: string;
    lastName: string;
    age: number;
}

export interface UpdateUserDTO {
    email?: string;
    firstName?: string;
    lastName?: string;
}
```

**2. Validators:**
```typescript
// validators/userSchemas.ts
import { z } from 'zod';

export const createUserSchema = z.object({
    email: z.string().email(),
    firstName: z.string().min(1).max(100),
    lastName: z.string().min(1).max(100),
    age: z.number().int().min(18).max(120),
});

export const updateUserSchema = z.object({
    email: z.string().email().optional(),
    firstName: z.string().min(1).max(100).optional(),
    lastName: z.string().min(1).max(100).optional(),
});
```

**3. Repository:**
```typescript
// repositories/UserRepository.ts
export class UserRepository {
    async findById(id: string): Promise<User | null> {
        return PrismaService.main.user.findUnique({
            where: { id },
            include: { profile: true },
        });
    }

    async create(data: Prisma.UserCreateInput): Promise<User> {
        return PrismaService.main.user.create({
            data,
            include: { profile: true },
        });
    }
}
```

**4. Service:**
```typescript
// services/userService.ts
export class UserService {
    private userRepository: UserRepository;

    constructor() {
        this.userRepository = new UserRepository();
    }

    async create(data: CreateUserDTO): Promise<User> {
        const existing = await this.userRepository.findByEmail(data.email);
        if (existing) {
            throw new ConflictError('Email already exists');
        }

        return await this.userRepository.create({
            email: data.email,
            profile: {
                create: {
                    firstName: data.firstName,
                    lastName: data.lastName,
                    age: data.age,
                },
            },
        });
    }
}
```

**5. Controller:**
```typescript
// controllers/UserController.ts
export class UserController extends BaseController {
    private userService: UserService;

    constructor() {
        super();
        this.userService = new UserService();
    }

    async createUser(req: Request, res: Response): Promise<void> {
        try {
            const validated = createUserSchema.parse(req.body);
            const user = await this.userService.create(validated);
            this.handleSuccess(res, user, 'User created', 201);
        } catch (error) {
            this.handleError(error, res, 'createUser');
        }
    }
}
```

**6. Routes:**
```typescript
// routes/userRoutes.ts
const router = Router();
const controller = new UserController();

router.post('/',
    SSOMiddlewareClient.verifyLoginStatus,
    async (req, res) => controller.createUser(req, res)
);

export default router;
```

**7. app.ts에 등록:**
```typescript
// app.ts
import userRoutes from './routes/userRoutes';

app.use('/api/users', userRoutes);
```

**전체 요청 흐름:**
```
POST /api/users
  ↓
userRoutes가 /와 매칭
  ↓
SSOMiddleware가 인증
  ↓
controller.createUser 호출
  ↓
Zod로 유효성 검사
  ↓
userService.create 호출
  ↓
비즈니스 규칙 확인
  ↓
userRepository.create 호출
  ↓
Prisma가 사용자 생성
  ↓
역순으로 응답 반환
  ↓
Controller가 응답 포맷
  ↓
200/201을 클라이언트로 전송
```

---

**관련 파일:**
- [SKILL.md](SKILL.md)
- [routing-and-controllers.md](routing-and-controllers.md)
- [services-and-repositories.md](services-and-repositories.md)
- [validation-patterns.md](validation-patterns.md)
