# Async 패턴과 에러 처리

async/await 패턴과 커스텀 에러 처리에 대한 완전한 가이드입니다.

## 목차

- [Async/Await 모범 사례](#asyncawait-모범-사례)
- [Promise 에러 처리](#promise-에러-처리)
- [커스텀 에러 타입](#커스텀-에러-타입)
- [asyncErrorWrapper 유틸리티](#asyncerrorwrapper-유틸리티)
- [에러 전파](#에러-전파)
- [일반적인 Async 함정](#일반적인-async-함정)

---

## Async/Await 모범 사례

### 항상 Try-Catch 사용

```typescript
// ❌ 절대 금지: 처리되지 않은 async 에러
async function fetchData() {
    const data = await database.query(); // 던지면 처리 안 됨!
    return data;
}

// ✅ 항상: try-catch로 감싸기
async function fetchData() {
    try {
        const data = await database.query();
        return data;
    } catch (error) {
        Sentry.captureException(error);
        throw error;
    }
}
```

### .then() 체인 피하기

```typescript
// ❌ 피하기: Promise 체인
function processData() {
    return fetchData()
        .then(data => transform(data))
        .then(transformed => save(transformed))
        .catch(error => {
            console.error(error);
        });
}

// ✅ 선호: Async/await
async function processData() {
    try {
        const data = await fetchData();
        const transformed = await transform(data);
        return await save(transformed);
    } catch (error) {
        Sentry.captureException(error);
        throw error;
    }
}
```

---

## Promise 에러 처리

### 병렬 작업

```typescript
// ✅ Promise.all에서 에러 처리
try {
    const [users, profiles, settings] = await Promise.all([
        userService.getAll(),
        profileService.getAll(),
        settingsService.getAll(),
    ]);
} catch (error) {
    // 하나가 실패하면 전체 실패
    Sentry.captureException(error);
    throw error;
}

// ✅ Promise.allSettled로 개별 에러 처리
const results = await Promise.allSettled([
    userService.getAll(),
    profileService.getAll(),
    settingsService.getAll(),
]);

results.forEach((result, index) => {
    if (result.status === 'rejected') {
        Sentry.captureException(result.reason, {
            tags: { operation: ['users', 'profiles', 'settings'][index] }
        });
    }
});
```

---

## 커스텀 에러 타입

### 커스텀 에러 정의

```typescript
// 기본 에러 클래스
export class AppError extends Error {
    constructor(
        message: string,
        public code: string,
        public statusCode: number,
        public isOperational: boolean = true
    ) {
        super(message);
        this.name = this.constructor.name;
        Error.captureStackTrace(this, this.constructor);
    }
}

// 특정 에러 타입들
export class ValidationError extends AppError {
    constructor(message: string) {
        super(message, 'VALIDATION_ERROR', 400);
    }
}

export class NotFoundError extends AppError {
    constructor(message: string) {
        super(message, 'NOT_FOUND', 404);
    }
}

export class ForbiddenError extends AppError {
    constructor(message: string) {
        super(message, 'FORBIDDEN', 403);
    }
}

export class ConflictError extends AppError {
    constructor(message: string) {
        super(message, 'CONFLICT', 409);
    }
}
```

### 사용법

```typescript
// 특정 에러 던지기
if (!user) {
    throw new NotFoundError('User not found');
}

if (user.age < 18) {
    throw new ValidationError('User must be 18+');
}

// Error boundary가 처리
function errorBoundary(error, req, res, next) {
    if (error instanceof AppError) {
        return res.status(error.statusCode).json({
            error: {
                message: error.message,
                code: error.code
            }
        });
    }

    // 알 수 없는 에러
    Sentry.captureException(error);
    res.status(500).json({ error: { message: 'Internal server error' } });
}
```

---

## asyncErrorWrapper 유틸리티

### 패턴

```typescript
export function asyncErrorWrapper(
    handler: (req: Request, res: Response, next: NextFunction) => Promise<any>
) {
    return async (req: Request, res: Response, next: NextFunction) => {
        try {
            await handler(req, res, next);
        } catch (error) {
            next(error);
        }
    };
}
```

### 사용법

```typescript
// wrapper 없이 - 에러가 처리되지 않을 수 있음
router.get('/users', async (req, res) => {
    const users = await userService.getAll(); // 던지면 처리 안 됨!
    res.json(users);
});

// wrapper 사용 - 에러가 캐치됨
router.get('/users', asyncErrorWrapper(async (req, res) => {
    const users = await userService.getAll();
    res.json(users);
}));
```

---

## 에러 전파

### 적절한 에러 체인

```typescript
// ✅ 스택을 따라 에러 전파
async function repositoryMethod() {
    try {
        return await PrismaService.main.user.findMany();
    } catch (error) {
        Sentry.captureException(error, { tags: { layer: 'repository' } });
        throw error; // service로 전파
    }
}

async function serviceMethod() {
    try {
        return await repositoryMethod();
    } catch (error) {
        Sentry.captureException(error, { tags: { layer: 'service' } });
        throw error; // controller로 전파
    }
}

async function controllerMethod(req, res) {
    try {
        const result = await serviceMethod();
        res.json(result);
    } catch (error) {
        this.handleError(error, res, 'controllerMethod'); // 최종 핸들러
    }
}
```

---

## 일반적인 Async 함정

### Fire and Forget (나쁜 패턴)

```typescript
// ❌ 절대 금지: Fire and forget
async function processRequest(req, res) {
    sendEmail(user.email); // async로 실행, 에러 처리 안 됨!
    res.json({ success: true });
}

// ✅ 항상: await 또는 처리
async function processRequest(req, res) {
    try {
        await sendEmail(user.email);
        res.json({ success: true });
    } catch (error) {
        Sentry.captureException(error);
        res.status(500).json({ error: 'Failed to send email' });
    }
}

// ✅ 또는: 의도적인 백그라운드 작업
async function processRequest(req, res) {
    sendEmail(user.email).catch(error => {
        Sentry.captureException(error);
    });
    res.json({ success: true });
}
```

### 처리되지 않은 Rejection

```typescript
// ✅ 처리되지 않은 rejection을 위한 글로벌 핸들러
process.on('unhandledRejection', (reason, promise) => {
    Sentry.captureException(reason, {
        tags: { type: 'unhandled_rejection' }
    });
    console.error('Unhandled Rejection:', reason);
});

process.on('uncaughtException', (error) => {
    Sentry.captureException(error, {
        tags: { type: 'uncaught_exception' }
    });
    console.error('Uncaught Exception:', error);
    process.exit(1);
});
```

---

**관련 파일:**
- [SKILL.md](SKILL.md)
- [sentry-and-monitoring.md](sentry-and-monitoring.md)
- [complete-examples.md](complete-examples.md)
