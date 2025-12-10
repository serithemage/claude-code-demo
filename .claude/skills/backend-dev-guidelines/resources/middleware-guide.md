# Middleware 가이드 - Express Middleware 패턴

백엔드 마이크로서비스에서 middleware 생성 및 사용에 대한 완전한 가이드입니다.

## 목차

- [인증 Middleware](#인증-middleware)
- [AsyncLocalStorage를 사용한 Audit Middleware](#asynclocalstorage를-사용한-audit-middleware)
- [Error Boundary Middleware](#error-boundary-middleware)
- [Validation Middleware](#validation-middleware)
- [조합 가능한 Middleware](#조합-가능한-middleware)
- [Middleware 순서](#middleware-순서)

---

## 인증 Middleware

### SSOMiddleware 패턴

**파일:** `/form/src/middleware/SSOMiddleware.ts`

```typescript
export class SSOMiddlewareClient {
    static verifyLoginStatus(req: Request, res: Response, next: NextFunction): void {
        const token = req.cookies.refresh_token;

        if (!token) {
            return res.status(401).json({ error: 'Not authenticated' });
        }

        try {
            const decoded = jwt.verify(token, config.tokens.jwt);
            res.locals.claims = decoded;
            res.locals.effectiveUserId = decoded.sub;
            next();
        } catch (error) {
            res.status(401).json({ error: 'Invalid token' });
        }
    }
}
```

---

## AsyncLocalStorage를 사용한 Audit Middleware

### Blog API의 훌륭한 패턴

**파일:** `/form/src/middleware/auditMiddleware.ts`

```typescript
import { AsyncLocalStorage } from 'async_hooks';

export interface AuditContext {
    userId: string;
    userName?: string;
    impersonatedBy?: string;
    sessionId?: string;
    timestamp: Date;
    requestId: string;
}

export const auditContextStorage = new AsyncLocalStorage<AuditContext>();

export function auditMiddleware(req: Request, res: Response, next: NextFunction): void {
    const context: AuditContext = {
        userId: res.locals.effectiveUserId || 'anonymous',
        userName: res.locals.claims?.preferred_username,
        impersonatedBy: res.locals.isImpersonating ? res.locals.originalUserId : undefined,
        timestamp: new Date(),
        requestId: req.id || uuidv4(),
    };

    auditContextStorage.run(context, () => {
        next();
    });
}

// 현재 컨텍스트 getter
export function getAuditContext(): AuditContext | null {
    return auditContextStorage.getStore() || null;
}
```

**장점:**
- 전체 요청에 걸쳐 컨텍스트 전파
- 모든 함수에 컨텍스트를 전달할 필요 없음
- services, repositories에서 자동으로 사용 가능
- 타입 안전 컨텍스트 액세스

**Services에서 사용:**
```typescript
import { getAuditContext } from '../middleware/auditMiddleware';

async function someOperation() {
    const context = getAuditContext();
    console.log('Operation by:', context?.userId);
}
```

---

## Error Boundary Middleware

### 포괄적인 에러 핸들러

**파일:** `/form/src/middleware/errorBoundary.ts`

```typescript
export function errorBoundary(
    error: Error,
    req: Request,
    res: Response,
    next: NextFunction
): void {
    // 상태 코드 결정
    const statusCode = getStatusCodeForError(error);

    // Sentry에 캡처
    Sentry.withScope((scope) => {
        scope.setLevel(statusCode >= 500 ? 'error' : 'warning');
        scope.setTag('error_type', error.name);
        scope.setContext('error_details', {
            message: error.message,
            stack: error.stack,
        });
        Sentry.captureException(error);
    });

    // 사용자 친화적 응답
    res.status(statusCode).json({
        success: false,
        error: {
            message: getUserFriendlyMessage(error),
            code: error.name,
        },
        requestId: Sentry.getCurrentScope().getPropagationContext().traceId,
    });
}

// Async wrapper
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

---

## 조합 가능한 Middleware

### withAuthAndAudit 패턴

```typescript
export function withAuthAndAudit(...authMiddleware: any[]) {
    return [
        ...authMiddleware,
        auditMiddleware,
    ];
}

// 사용법
router.post('/:formID/submit',
    ...withAuthAndAudit(SSOMiddlewareClient.verifyLoginStatus),
    async (req, res) => controller.submit(req, res)
);
```

---

## Middleware 순서

### 중요한 순서 (반드시 따라야 함)

```typescript
// 1. Sentry request handler (첫 번째)
app.use(Sentry.Handlers.requestHandler());

// 2. Body 파싱
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// 3. Cookie 파싱
app.use(cookieParser());

// 4. 인증 초기화
app.use(SSOMiddleware.initialize());

// 5. 여기서 Routes 등록
app.use('/api/users', userRoutes);

// 6. 에러 핸들러 (routes 이후)
app.use(errorBoundary);

// 7. Sentry 에러 핸들러 (마지막)
app.use(Sentry.Handlers.errorHandler());
```

**규칙:** 에러 핸들러는 반드시 모든 routes 이후에 등록해야 합니다!

---

**관련 파일:**
- [SKILL.md](SKILL.md)
- [routing-and-controllers.md](routing-and-controllers.md)
- [async-and-errors.md](async-and-errors.md)
