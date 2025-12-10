# Sentry 통합 및 모니터링

Sentry v8을 사용한 에러 추적 및 성능 모니터링에 대한 완전한 가이드입니다.

## 목차

- [핵심 원칙](#핵심-원칙)
- [Sentry 초기화](#sentry-초기화)
- [에러 캡처 패턴](#에러-캡처-패턴)
- [성능 모니터링](#성능-모니터링)
- [Cron Job 모니터링](#cron-job-모니터링)
- [에러 컨텍스트 모범 사례](#에러-컨텍스트-모범-사례)
- [일반적인 실수](#일반적인-실수)

---

## 핵심 원칙

**필수**: 모든 에러는 반드시 Sentry에 캡처되어야 합니다. 예외 없음.

**모든 에러는 캡처되어야 함** - 모든 서비스에서 포괄적인 에러 추적과 함께 Sentry v8을 사용하세요.

---

## Sentry 초기화

### instrument.ts 패턴

**위치:** `src/instrument.ts` (server.ts와 모든 cron jobs에서 첫 번째 import여야 함)

**마이크로서비스용 템플릿:**

```typescript
import * as Sentry from '@sentry/node';
import * as fs from 'fs';
import * as path from 'path';
import * as ini from 'ini';

const sentryConfigPath = path.join(__dirname, '../sentry.ini');
const sentryConfig = ini.parse(fs.readFileSync(sentryConfigPath, 'utf-8'));

Sentry.init({
    dsn: sentryConfig.sentry?.dsn,
    environment: process.env.NODE_ENV || 'development',
    tracesSampleRate: parseFloat(sentryConfig.sentry?.tracesSampleRate || '0.1'),
    profilesSampleRate: parseFloat(sentryConfig.sentry?.profilesSampleRate || '0.1'),

    integrations: [
        ...Sentry.getDefaultIntegrations({}),
        Sentry.extraErrorDataIntegration({ depth: 5 }),
        Sentry.localVariablesIntegration(),
        Sentry.requestDataIntegration({
            include: {
                cookies: false,
                data: true,
                headers: true,
                ip: true,
                query_string: true,
                url: true,
                user: { id: true, email: true, username: true },
            },
        }),
        Sentry.consoleIntegration(),
        Sentry.contextLinesIntegration(),
        Sentry.prismaIntegration(),
    ],

    beforeSend(event, hint) {
        // 헬스 체크 필터링
        if (event.request?.url?.includes('/healthcheck')) {
            return null;
        }

        // 민감한 헤더 스크럽
        if (event.request?.headers) {
            delete event.request.headers['authorization'];
            delete event.request.headers['cookie'];
        }

        // PII를 위한 이메일 마스킹
        if (event.user?.email) {
            event.user.email = event.user.email.replace(/^(.{2}).*(@.*)$/, '$1***$2');
        }

        return event;
    },

    ignoreErrors: [
        /^Invalid JWT/,
        /^JWT expired/,
        'NetworkError',
    ],
});

// 서비스 컨텍스트 설정
Sentry.setTags({
    service: 'form',
    version: '1.0.1',
});

Sentry.setContext('runtime', {
    node_version: process.version,
    platform: process.platform,
});
```

**중요 포인트:**
- PII 보호 내장 (beforeSend)
- 비중요 에러 필터링
- 포괄적인 통합
- Prisma 계측
- 서비스별 태깅

---

## 에러 캡처 패턴

### 1. BaseController 패턴

```typescript
// BaseController.handleError 사용
protected handleError(error: unknown, res: Response, context: string, statusCode = 500): void {
    Sentry.withScope((scope) => {
        scope.setTag('controller', this.constructor.name);
        scope.setTag('operation', context);
        scope.setUser({ id: res.locals?.claims?.userId });
        Sentry.captureException(error);
    });

    res.status(statusCode).json({
        success: false,
        error: { message: error instanceof Error ? error.message : 'Error occurred' }
    });
}
```

### 2. Workflow 에러 처리

```typescript
import { SentryHelper } from '../utils/sentryHelper';

try {
    await businessOperation();
} catch (error) {
    SentryHelper.captureOperationError(error, {
        operationType: 'POST_CREATION',
        entityId: 123,
        userId: 'user-123',
        operation: 'createPost',
    });
    throw error;
}
```

### 3. Service 계층 에러 처리

```typescript
try {
    await someOperation();
} catch (error) {
    Sentry.captureException(error, {
        tags: {
            service: 'form',
            operation: 'someOperation'
        },
        extra: {
            userId: currentUser.id,
            entityId: 123
        }
    });
    throw error;
}
```

---

## 성능 모니터링

### 데이터베이스 성능 추적

```typescript
import { DatabasePerformanceMonitor } from '../utils/databasePerformance';

const result = await DatabasePerformanceMonitor.withPerformanceTracking(
    'findMany',
    'UserProfile',
    async () => {
        return await PrismaService.main.userProfile.findMany({ take: 5 });
    }
);
```

### API 엔드포인트 Spans

```typescript
router.post('/operation', async (req, res) => {
    return await Sentry.startSpan({
        name: 'operation.execute',
        op: 'http.server',
        attributes: {
            'http.method': 'POST',
            'http.route': '/operation'
        }
    }, async () => {
        const result = await performOperation();
        res.json(result);
    });
});
```

---

## Cron Job 모니터링

### 필수 패턴

```typescript
#!/usr/bin/env node
import '../instrument'; // shebang 이후 첫 번째 줄
import * as Sentry from '@sentry/node';

async function main() {
    return await Sentry.startSpan({
        name: 'cron.job-name',
        op: 'cron',
        attributes: {
            'cron.job': 'job-name',
            'cron.startTime': new Date().toISOString(),
        }
    }, async () => {
        try {
            // Cron job 로직
        } catch (error) {
            Sentry.captureException(error, {
                tags: {
                    'cron.job': 'job-name',
                    'error.type': 'execution_error'
                }
            });
            console.error('[Cron] Error:', error);
            process.exit(1);
        }
    });
}

main().then(() => {
    console.log('[Cron] Completed successfully');
    process.exit(0);
}).catch((error) => {
    console.error('[Cron] Fatal error:', error);
    process.exit(1);
});
```

---

## 에러 컨텍스트 모범 사례

### 풍부한 컨텍스트 예시

```typescript
Sentry.withScope((scope) => {
    // 사용자 컨텍스트
    scope.setUser({
        id: user.id,
        email: user.email,
        username: user.username
    });

    // 필터링을 위한 태그
    scope.setTag('service', 'form');
    scope.setTag('endpoint', req.path);
    scope.setTag('method', req.method);

    // 구조화된 컨텍스트
    scope.setContext('operation', {
        type: 'workflow.complete',
        workflowId: 123,
        stepId: 456
    });

    // 타임라인을 위한 breadcrumbs
    scope.addBreadcrumb({
        category: 'workflow',
        message: 'Starting step completion',
        level: 'info',
        data: { stepId: 456 }
    });

    Sentry.captureException(error);
});
```

---

## 일반적인 실수

```typescript
// ❌ 에러 삼키기
try {
    await riskyOperation();
} catch (error) {
    // 조용한 실패
}

// ❌ 일반적인 에러 메시지
throw new Error('Error occurred');

// ❌ 민감한 데이터 노출
Sentry.captureException(error, {
    extra: { password: user.password } // 절대 금지
});

// ❌ 누락된 async 에러 처리
async function bad() {
    fetchData().then(data => processResult(data)); // 처리되지 않음
}

// ✅ 적절한 async 처리
async function good() {
    try {
        const data = await fetchData();
        processResult(data);
    } catch (error) {
        Sentry.captureException(error);
        throw error;
    }
}
```

---

**관련 파일:**
- [SKILL.md](SKILL.md)
- [routing-and-controllers.md](routing-and-controllers.md)
- [async-and-errors.md](async-and-errors.md)
