# Services와 Repositories - 비즈니스 로직 계층

Services로 비즈니스 로직을 구성하고 repositories로 데이터 액세스를 관리하는 완전한 가이드입니다.

## 목차

- [Service 계층 개요](#service-계층-개요)
- [의존성 주입 패턴](#의존성-주입-패턴)
- [Singleton 패턴](#singleton-패턴)
- [Repository 패턴](#repository-패턴)
- [Service 설계 원칙](#service-설계-원칙)
- [캐싱 전략](#캐싱-전략)
- [Services 테스트](#services-테스트)

---

## Service 계층 개요

### Services의 목적

**Services는 비즈니스 로직을 포함합니다** - 애플리케이션의 '무엇'과 '왜':

```
Controller 질문: "이것을 해야 하나요?"
Service 답변: "예/아니오, 이유는 이것이고, 이것이 발생합니다"
Repository 실행: "요청한 데이터입니다"
```

**Services의 책임:**
- ✅ 비즈니스 규칙 적용
- ✅ 여러 repositories 오케스트레이션
- ✅ 트랜잭션 관리
- ✅ 복잡한 계산
- ✅ 외부 서비스 통합
- ✅ 비즈니스 유효성 검사

**Services가 하면 안 되는 것:**
- ❌ HTTP 알기 (Request/Response)
- ❌ Prisma 직접 액세스 (repositories 사용)
- ❌ Route 전용 로직 처리
- ❌ HTTP 응답 포맷팅

---

## 의존성 주입 패턴

### 의존성 주입을 사용하는 이유

**장점:**
- 테스트하기 쉬움 (mock 주입)
- 명확한 의존성
- 유연한 설정
- 느슨한 결합 촉진

### 훌륭한 예시: NotificationService

**파일:** `/blog-api/src/services/NotificationService.ts`

```typescript
// 명확성을 위한 의존성 인터페이스 정의
export interface NotificationServiceDependencies {
    prisma: PrismaClient;
    batchingService: BatchingService;
    emailComposer: EmailComposer;
}

// 의존성 주입이 있는 Service
export class NotificationService {
    private prisma: PrismaClient;
    private batchingService: BatchingService;
    private emailComposer: EmailComposer;
    private preferencesCache: Map<string, { preferences: UserPreference; timestamp: number }> = new Map();
    private CACHE_TTL = (notificationConfig.preferenceCacheTTLMinutes || 5) * 60 * 1000;

    // 생성자를 통해 의존성 주입
    constructor(dependencies: NotificationServiceDependencies) {
        this.prisma = dependencies.prisma;
        this.batchingService = dependencies.batchingService;
        this.emailComposer = dependencies.emailComposer;
    }

    /**
     * 알림 생성 및 적절한 라우팅
     */
    async createNotification(params: CreateNotificationParams) {
        const { recipientID, type, title, message, link, context = {}, channel = 'both', priority = NotificationPriority.NORMAL } = params;

        try {
            // 템플릿 가져와서 콘텐츠 렌더링
            const template = getNotificationTemplate(type);
            const rendered = renderNotificationContent(template, context);

            // 인앱 알림 레코드 생성
            const notificationId = await createNotificationRecord({
                instanceId: parseInt(context.instanceId || '0', 10),
                template: type,
                recipientUserId: recipientID,
                channel: channel === 'email' ? 'email' : 'inApp',
                contextData: context,
                title: finalTitle,
                message: finalMessage,
                link: finalLink,
            });

            // 채널에 따라 알림 라우팅
            if (channel === 'email' || channel === 'both') {
                await this.routeNotification({
                    notificationId,
                    userId: recipientID,
                    type,
                    priority,
                    title: finalTitle,
                    message: finalMessage,
                    link: finalLink,
                    context,
                });
            }

            return notification;
        } catch (error) {
            ErrorLogger.log(error, {
                context: {
                    '[NotificationService] createNotification': {
                        type: params.type,
                        recipientID: params.recipientID,
                    },
                },
            });
            throw error;
        }
    }

    /**
     * 사용자 설정에 따라 알림 라우팅
     */
    private async routeNotification(params: { notificationId: number; userId: string; type: string; priority: NotificationPriority; title: string; message: string; link?: string; context?: Record<string, any> }) {
        // 캐싱과 함께 사용자 설정 가져오기
        const preferences = await this.getUserPreferences(params.userId);

        // 배치할지 즉시 보낼지 확인
        if (this.shouldBatchEmail(preferences, params.type, params.priority)) {
            await this.batchingService.queueNotificationForBatch({
                notificationId: params.notificationId,
                userId: params.userId,
                userPreference: preferences,
                priority: params.priority,
            });
        } else {
            // EmailComposer를 통해 즉시 전송
            await this.sendImmediateEmail({
                userId: params.userId,
                title: params.title,
                message: params.message,
                link: params.link,
                context: params.context,
                type: params.type,
            });
        }
    }

    /**
     * 이메일을 배치해야 하는지 결정
     */
    shouldBatchEmail(preferences: UserPreference, notificationType: string, priority: NotificationPriority): boolean {
        // HIGH 우선순위는 항상 즉시
        if (priority === NotificationPriority.HIGH) {
            return false;
        }

        // 배치 모드 확인
        const batchMode = preferences.emailBatchMode || BatchMode.IMMEDIATE;
        return batchMode !== BatchMode.IMMEDIATE;
    }

    /**
     * 캐싱과 함께 사용자 설정 가져오기
     */
    async getUserPreferences(userId: string): Promise<UserPreference> {
        // 먼저 캐시 확인
        const cached = this.preferencesCache.get(userId);
        if (cached && Date.now() - cached.timestamp < this.CACHE_TTL) {
            return cached.preferences;
        }

        const preference = await this.prisma.userPreference.findUnique({
            where: { userID: userId },
        });

        const finalPreferences = preference || DEFAULT_PREFERENCES;

        // 캐시 업데이트
        this.preferencesCache.set(userId, {
            preferences: finalPreferences,
            timestamp: Date.now(),
        });

        return finalPreferences;
    }
}
```

**Controller에서 사용:**

```typescript
// 의존성과 함께 인스턴스화
const notificationService = new NotificationService({
    prisma: PrismaService.main,
    batchingService: new BatchingService(PrismaService.main),
    emailComposer: new EmailComposer(),
});

// Controller에서 사용
const notification = await notificationService.createNotification({
    recipientID: 'user-123',
    type: 'AFRLWorkflowNotification',
    context: { workflowName: 'AFRL Monthly Report' },
});
```

**핵심 포인트:**
- 생성자를 통해 의존성 전달
- 명확한 인터페이스가 필요한 의존성 정의
- 테스트하기 쉬움 (mock 주입)
- 캡슐화된 캐싱 로직
- HTTP와 분리된 비즈니스 규칙

---

## Singleton 패턴

### Singleton을 사용해야 할 때

**사용 대상:**
- 비싼 초기화가 있는 Services
- 공유 상태가 있는 Services (캐싱)
- 여러 곳에서 액세스되는 Services
- Permission services
- Configuration services

### 예시: PermissionService (Singleton)

**파일:** `/blog-api/src/services/permissionService.ts`

```typescript
import { PrismaClient } from '@prisma/client';

class PermissionService {
    private static instance: PermissionService;
    private prisma: PrismaClient;
    private permissionCache: Map<string, { canAccess: boolean; timestamp: number }> = new Map();
    private CACHE_TTL = 5 * 60 * 1000; // 5분

    // private 생성자로 직접 인스턴스화 방지
    private constructor() {
        this.prisma = PrismaService.main;
    }

    // Singleton 인스턴스 가져오기
    public static getInstance(): PermissionService {
        if (!PermissionService.instance) {
            PermissionService.instance = new PermissionService();
        }
        return PermissionService.instance;
    }

    /**
     * 사용자가 workflow 단계를 완료할 수 있는지 확인
     */
    async canCompleteStep(userId: string, stepInstanceId: number): Promise<boolean> {
        const cacheKey = `${userId}:${stepInstanceId}`;

        // 캐시 확인
        const cached = this.permissionCache.get(cacheKey);
        if (cached && Date.now() - cached.timestamp < this.CACHE_TTL) {
            return cached.canAccess;
        }

        try {
            const post = await this.prisma.post.findUnique({
                where: { id: postId },
                include: {
                    author: true,
                    comments: {
                        include: {
                            user: true,
                        },
                    },
                },
            });

            if (!post) {
                return false;
            }

            // 사용자에게 권한이 있는지 확인
            const canEdit = post.authorId === userId ||
                await this.isUserAdmin(userId);

            // 결과 캐시
            this.permissionCache.set(cacheKey, {
                canAccess: isAssigned,
                timestamp: Date.now(),
            });

            return isAssigned;
        } catch (error) {
            console.error('[PermissionService] Error checking step permission:', error);
            return false;
        }
    }

    /**
     * 사용자 캐시 클리어
     */
    clearUserCache(userId: string): void {
        for (const [key] of this.permissionCache) {
            if (key.startsWith(`${userId}:`)) {
                this.permissionCache.delete(key);
            }
        }
    }

    /**
     * 전체 캐시 클리어
     */
    clearCache(): void {
        this.permissionCache.clear();
    }
}

// Singleton 인스턴스 export
export const permissionService = PermissionService.getInstance();
```

**사용:**

```typescript
import { permissionService } from '../services/permissionService';

// 코드베이스 어디서든 사용
const canComplete = await permissionService.canCompleteStep(userId, stepId);

if (!canComplete) {
    throw new ForbiddenError('You do not have permission to complete this step');
}
```

---

## Repository 패턴

### Repositories의 목적

**Repositories는 데이터 액세스를 추상화합니다** - 데이터 작업의 '어떻게':

```
Service: "이름순으로 정렬된 모든 활성 사용자를 주세요"
Repository: "이것을 수행하는 Prisma 쿼리입니다"
```

**Repositories의 책임:**
- ✅ 모든 Prisma 작업
- ✅ 쿼리 구성
- ✅ 쿼리 최적화 (select, include)
- ✅ 데이터베이스 에러 처리
- ✅ 데이터베이스 결과 캐싱

**Repositories가 하면 안 되는 것:**
- ❌ 비즈니스 로직 포함
- ❌ HTTP 알기
- ❌ 결정 내리기 (그건 service 계층)

### Repository 템플릿

```typescript
// repositories/UserRepository.ts
import { PrismaService } from '@project-lifecycle-portal/database';
import type { User, Prisma } from '@project-lifecycle-portal/database';

export class UserRepository {
    /**
     * 최적화된 쿼리로 ID로 사용자 찾기
     */
    async findById(userId: string): Promise<User | null> {
        try {
            return await PrismaService.main.user.findUnique({
                where: { userID: userId },
                select: {
                    userID: true,
                    email: true,
                    name: true,
                    isActive: true,
                    roles: true,
                    createdAt: true,
                    updatedAt: true,
                },
            });
        } catch (error) {
            console.error('[UserRepository] Error finding user by ID:', error);
            throw new Error(`Failed to find user: ${userId}`);
        }
    }

    /**
     * 모든 활성 사용자 찾기
     */
    async findActive(options?: { orderBy?: Prisma.UserOrderByWithRelationInput }): Promise<User[]> {
        try {
            return await PrismaService.main.user.findMany({
                where: { isActive: true },
                orderBy: options?.orderBy || { name: 'asc' },
                select: {
                    userID: true,
                    email: true,
                    name: true,
                    roles: true,
                },
            });
        } catch (error) {
            console.error('[UserRepository] Error finding active users:', error);
            throw new Error('Failed to find active users');
        }
    }

    /**
     * 이메일로 사용자 찾기
     */
    async findByEmail(email: string): Promise<User | null> {
        try {
            return await PrismaService.main.user.findUnique({
                where: { email },
            });
        } catch (error) {
            console.error('[UserRepository] Error finding user by email:', error);
            throw new Error(`Failed to find user with email: ${email}`);
        }
    }

    /**
     * 새 사용자 생성
     */
    async create(data: Prisma.UserCreateInput): Promise<User> {
        try {
            return await PrismaService.main.user.create({ data });
        } catch (error) {
            console.error('[UserRepository] Error creating user:', error);
            throw new Error('Failed to create user');
        }
    }

    /**
     * 사용자 업데이트
     */
    async update(userId: string, data: Prisma.UserUpdateInput): Promise<User> {
        try {
            return await PrismaService.main.user.update({
                where: { userID: userId },
                data,
            });
        } catch (error) {
            console.error('[UserRepository] Error updating user:', error);
            throw new Error(`Failed to update user: ${userId}`);
        }
    }

    /**
     * 사용자 삭제 (isActive = false로 soft delete)
     */
    async delete(userId: string): Promise<User> {
        try {
            return await PrismaService.main.user.update({
                where: { userID: userId },
                data: { isActive: false },
            });
        } catch (error) {
            console.error('[UserRepository] Error deleting user:', error);
            throw new Error(`Failed to delete user: ${userId}`);
        }
    }

    /**
     * 이메일 존재 여부 확인
     */
    async emailExists(email: string): Promise<boolean> {
        try {
            const count = await PrismaService.main.user.count({
                where: { email },
            });
            return count > 0;
        } catch (error) {
            console.error('[UserRepository] Error checking email exists:', error);
            throw new Error('Failed to check if email exists');
        }
    }
}

// Singleton 인스턴스 export
export const userRepository = new UserRepository();
```

**Service에서 Repository 사용:**

```typescript
// services/userService.ts
import { userRepository } from '../repositories/UserRepository';
import { ConflictError, NotFoundError } from '../utils/errors';

export class UserService {
    /**
     * 비즈니스 규칙과 함께 새 사용자 생성
     */
    async createUser(data: { email: string; name: string; roles: string[] }): Promise<User> {
        // 비즈니스 규칙: 이메일이 이미 존재하는지 확인
        const emailExists = await userRepository.emailExists(data.email);
        if (emailExists) {
            throw new ConflictError('Email already exists');
        }

        // 비즈니스 규칙: 역할 검증
        const validRoles = ['admin', 'operations', 'user'];
        const invalidRoles = data.roles.filter((role) => !validRoles.includes(role));
        if (invalidRoles.length > 0) {
            throw new ValidationError(`Invalid roles: ${invalidRoles.join(', ')}`);
        }

        // Repository를 통해 사용자 생성
        return await userRepository.create({
            email: data.email,
            name: data.name,
            roles: data.roles,
            isActive: true,
        });
    }

    /**
     * ID로 사용자 가져오기
     */
    async getUser(userId: string): Promise<User> {
        const user = await userRepository.findById(userId);

        if (!user) {
            throw new NotFoundError(`User not found: ${userId}`);
        }

        return user;
    }
}
```

---

## Service 설계 원칙

### 1. 단일 책임

각 service는 하나의 명확한 목적을 가져야 합니다:

```typescript
// ✅ 좋음 - 단일 책임
class UserService {
    async createUser() {}
    async updateUser() {}
    async deleteUser() {}
}

class EmailService {
    async sendEmail() {}
    async sendBulkEmails() {}
}

// ❌ 나쁨 - 너무 많은 책임
class UserService {
    async createUser() {}
    async sendWelcomeEmail() {}  // EmailService여야 함
    async logUserActivity() {}   // AuditService여야 함
    async processPayment() {}    // PaymentService여야 함
}
```

### 2. 명확한 메서드 이름

메서드 이름은 무엇을 하는지 설명해야 합니다:

```typescript
// ✅ 좋음 - 명확한 의도
async createNotification()
async getUserPreferences()
async shouldBatchEmail()
async routeNotification()

// ❌ 나쁨 - 모호하거나 오해의 소지
async process()
async handle()
async doIt()
async execute()
```

### 3. 반환 타입

항상 명시적 반환 타입 사용:

```typescript
// ✅ 좋음 - 명시적 타입
async createUser(data: CreateUserDTO): Promise<User> {}
async findUsers(): Promise<User[]> {}
async deleteUser(id: string): Promise<void> {}

// ❌ 나쁨 - 암묵적 any
async createUser(data) {}  // 타입 없음!
```

### 4. 에러 처리

Services는 의미 있는 에러를 던져야 합니다:

```typescript
// ✅ 좋음 - 의미 있는 에러
if (!user) {
    throw new NotFoundError(`User not found: ${userId}`);
}

if (emailExists) {
    throw new ConflictError('Email already exists');
}

// ❌ 나쁨 - 일반적인 에러
if (!user) {
    throw new Error('Error');  // 무슨 에러?
}
```

### 5. God Services 피하기

모든 것을 하는 services 만들지 마세요:

```typescript
// ❌ 나쁨 - God service
class WorkflowService {
    async startWorkflow() {}
    async completeStep() {}
    async assignRoles() {}
    async sendNotifications() {}  // NotificationService여야 함
    async validatePermissions() {}  // PermissionService여야 함
    async logAuditTrail() {}  // AuditService여야 함
    // ... 50개 이상의 메서드
}

// ✅ 좋음 - 집중된 services
class WorkflowService {
    constructor(
        private notificationService: NotificationService,
        private permissionService: PermissionService,
        private auditService: AuditService
    ) {}

    async startWorkflow() {
        // 다른 services 오케스트레이션
        await this.permissionService.checkPermission();
        await this.workflowRepository.create();
        await this.notificationService.notify();
        await this.auditService.log();
    }
}
```

---

## 캐싱 전략

### 1. In-Memory 캐싱

```typescript
class UserService {
    private cache: Map<string, { user: User; timestamp: number }> = new Map();
    private CACHE_TTL = 5 * 60 * 1000; // 5분

    async getUser(userId: string): Promise<User> {
        // 캐시 확인
        const cached = this.cache.get(userId);
        if (cached && Date.now() - cached.timestamp < this.CACHE_TTL) {
            return cached.user;
        }

        // 데이터베이스에서 가져오기
        const user = await userRepository.findById(userId);

        // 캐시 업데이트
        if (user) {
            this.cache.set(userId, { user, timestamp: Date.now() });
        }

        return user;
    }

    clearUserCache(userId: string): void {
        this.cache.delete(userId);
    }
}
```

### 2. 캐시 무효화

```typescript
class UserService {
    async updateUser(userId: string, data: UpdateUserDTO): Promise<User> {
        // 데이터베이스에서 업데이트
        const user = await userRepository.update(userId, data);

        // 캐시 무효화
        this.clearUserCache(userId);

        return user;
    }
}
```

---

## Services 테스트

### 단위 테스트

```typescript
// tests/userService.test.ts
import { UserService } from '../services/userService';
import { userRepository } from '../repositories/UserRepository';
import { ConflictError } from '../utils/errors';

// Repository 모킹
jest.mock('../repositories/UserRepository');

describe('UserService', () => {
    let userService: UserService;

    beforeEach(() => {
        userService = new UserService();
        jest.clearAllMocks();
    });

    describe('createUser', () => {
        it('이메일이 존재하지 않으면 사용자를 생성해야 한다', async () => {
            // Arrange
            const userData = {
                email: 'test@example.com',
                name: 'Test User',
                roles: ['user'],
            };

            (userRepository.emailExists as jest.Mock).mockResolvedValue(false);
            (userRepository.create as jest.Mock).mockResolvedValue({
                userID: '123',
                ...userData,
            });

            // Act
            const user = await userService.createUser(userData);

            // Assert
            expect(user).toBeDefined();
            expect(user.email).toBe(userData.email);
            expect(userRepository.emailExists).toHaveBeenCalledWith(userData.email);
            expect(userRepository.create).toHaveBeenCalled();
        });

        it('이메일이 존재하면 ConflictError를 던져야 한다', async () => {
            // Arrange
            const userData = {
                email: 'existing@example.com',
                name: 'Test User',
                roles: ['user'],
            };

            (userRepository.emailExists as jest.Mock).mockResolvedValue(true);

            // Act & Assert
            await expect(userService.createUser(userData)).rejects.toThrow(ConflictError);
            expect(userRepository.create).not.toHaveBeenCalled();
        });
    });
});
```

---

**관련 파일:**
- [SKILL.md](SKILL.md) - 메인 가이드
- [routing-and-controllers.md](routing-and-controllers.md) - Services를 사용하는 Controllers
- [database-patterns.md](database-patterns.md) - Prisma와 repository 패턴
- [complete-examples.md](complete-examples.md) - 전체 service/repository 예제
