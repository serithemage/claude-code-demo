# 테스트 가이드 - 백엔드 테스트 전략

Jest와 모범 사례를 사용한 백엔드 서비스 테스트에 대한 완전한 가이드입니다.

## 목차

- [단위 테스트](#단위-테스트)
- [통합 테스트](#통합-테스트)
- [모킹 전략](#모킹-전략)
- [테스트 데이터 관리](#테스트-데이터-관리)
- [인증된 라우트 테스트](#인증된-라우트-테스트)
- [커버리지 목표](#커버리지-목표)

---

## 단위 테스트

### 테스트 구조

```typescript
// services/userService.test.ts
import { UserService } from './userService';
import { UserRepository } from '../repositories/UserRepository';

jest.mock('../repositories/UserRepository');

describe('UserService', () => {
    let service: UserService;
    let mockRepository: jest.Mocked<UserRepository>;

    beforeEach(() => {
        mockRepository = {
            findByEmail: jest.fn(),
            create: jest.fn(),
        } as any;

        service = new UserService();
        (service as any).userRepository = mockRepository;
    });

    afterEach(() => {
        jest.clearAllMocks();
    });

    describe('create', () => {
        it('이메일이 존재하면 에러를 던져야 한다', async () => {
            mockRepository.findByEmail.mockResolvedValue({ id: '123' } as any);

            await expect(
                service.create({ email: 'test@test.com' })
            ).rejects.toThrow('Email already in use');
        });

        it('이메일이 고유하면 사용자를 생성해야 한다', async () => {
            mockRepository.findByEmail.mockResolvedValue(null);
            mockRepository.create.mockResolvedValue({ id: '123' } as any);

            const user = await service.create({
                email: 'test@test.com',
                firstName: 'John',
                lastName: 'Doe',
            });

            expect(user).toBeDefined();
            expect(mockRepository.create).toHaveBeenCalledWith(
                expect.objectContaining({
                    email: 'test@test.com'
                })
            );
        });
    });
});
```

---

## 통합 테스트

### 실제 데이터베이스로 테스트

```typescript
import { PrismaService } from '@project-lifecycle-portal/database';

describe('UserService Integration', () => {
    let testUser: any;

    beforeAll(async () => {
        // 테스트 데이터 생성
        testUser = await PrismaService.main.user.create({
            data: {
                email: 'test@test.com',
                profile: { create: { firstName: 'Test', lastName: 'User' } },
            },
        });
    });

    afterAll(async () => {
        // 정리
        await PrismaService.main.user.delete({ where: { id: testUser.id } });
    });

    it('이메일로 사용자를 찾아야 한다', async () => {
        const user = await userService.findByEmail('test@test.com');
        expect(user).toBeDefined();
        expect(user?.email).toBe('test@test.com');
    });
});
```

---

## 모킹 전략

### PrismaService 모킹

```typescript
jest.mock('@project-lifecycle-portal/database', () => ({
    PrismaService: {
        main: {
            user: {
                findMany: jest.fn(),
                findUnique: jest.fn(),
                create: jest.fn(),
                update: jest.fn(),
            },
        },
        isAvailable: true,
    },
}));
```

### Services 모킹

```typescript
const mockUserService = {
    findById: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
} as jest.Mocked<UserService>;
```

---

## 테스트 데이터 관리

### Setup과 Teardown

```typescript
describe('PermissionService', () => {
    let instanceId: number;

    beforeAll(async () => {
        // 테스트 post 생성
        const post = await PrismaService.main.post.create({
            data: { title: 'Test Post', content: 'Test', authorId: 'test-user' },
        });
        instanceId = post.id;
    });

    afterAll(async () => {
        // 정리
        await PrismaService.main.post.delete({
            where: { id: instanceId },
        });
    });

    beforeEach(() => {
        // 캐시 클리어
        permissionService.clearCache();
    });

    it('권한을 확인해야 한다', async () => {
        const hasPermission = await permissionService.checkPermission(
            'user-id',
            instanceId,
            'VIEW_WORKFLOW'
        );
        expect(hasPermission).toBeDefined();
    });
});
```

---

## 인증된 라우트 테스트

### test-auth-route.js 사용

```bash
# 인증된 엔드포인트 테스트
node scripts/test-auth-route.js http://localhost:3002/form/api/users

# POST 데이터로 테스트
node scripts/test-auth-route.js http://localhost:3002/form/api/users POST '{"email":"test@test.com"}'
```

### 테스트에서 인증 모킹

```typescript
// auth middleware 모킹
jest.mock('../middleware/SSOMiddleware', () => ({
    SSOMiddlewareClient: {
        verifyLoginStatus: (req, res, next) => {
            res.locals.claims = {
                sub: 'test-user-id',
                preferred_username: 'testuser',
            };
            next();
        },
    },
}));
```

---

## 커버리지 목표

### 권장 커버리지

- **단위 테스트**: 70% 이상 커버리지
- **통합 테스트**: 중요 경로 커버
- **E2E 테스트**: 성공 경로 커버

### 커버리지 실행

```bash
npm test -- --coverage
```

---

**관련 파일:**
- [SKILL.md](SKILL.md)
- [services-and-repositories.md](services-and-repositories.md)
- [complete-examples.md](complete-examples.md)
