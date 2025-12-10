# 설정 관리 - UnifiedConfig 패턴

백엔드 마이크로서비스의 설정 관리에 대한 완전한 가이드입니다.

## 목차

- [UnifiedConfig 개요](#unifiedconfig-개요)
- [process.env 직접 사용 금지](#processenv-직접-사용-금지)
- [설정 구조](#설정-구조)
- [환경별 설정](#환경별-설정)
- [시크릿 관리](#시크릿-관리)
- [마이그레이션 가이드](#마이그레이션-가이드)

---

## UnifiedConfig 개요

### UnifiedConfig를 사용하는 이유

**process.env의 문제점:**
- ❌ 타입 안전성 없음
- ❌ 유효성 검사 없음
- ❌ 테스트하기 어려움
- ❌ 코드 전체에 분산됨
- ❌ 기본값 없음
- ❌ 오타에 대한 런타임 에러

**unifiedConfig의 장점:**
- ✅ 타입 안전 설정
- ✅ 단일 진실 공급원
- ✅ 시작 시 유효성 검사
- ✅ 모킹으로 쉬운 테스트
- ✅ 명확한 구조
- ✅ 환경 변수로 폴백

---

## process.env 직접 사용 금지

### 규칙

```typescript
// ❌ 절대 이렇게 하지 마세요
const timeout = parseInt(process.env.TIMEOUT_MS || '5000');
const dbHost = process.env.DB_HOST || 'localhost';

// ✅ 항상 이렇게 하세요
import { config } from './config/unifiedConfig';
const timeout = config.timeouts.default;
const dbHost = config.database.host;
```

### 이것이 중요한 이유

**문제 예시:**
```typescript
// 환경 변수 이름 오타
const host = process.env.DB_HSOT; // undefined! 에러 없음!

// 타입 안전성
const port = process.env.PORT; // 문자열! parseInt 필요
const timeout = parseInt(process.env.TIMEOUT); // 설정 안 되면 NaN!
```

**unifiedConfig 사용 시:**
```typescript
const port = config.server.port; // number, 보장됨
const timeout = config.timeouts.default; // number, 폴백 포함
```

---

## 설정 구조

### UnifiedConfig 인터페이스

```typescript
export interface UnifiedConfig {
    database: {
        host: string;
        port: number;
        username: string;
        password: string;
        database: string;
    };
    server: {
        port: number;
        sessionSecret: string;
    };
    tokens: {
        jwt: string;
        inactivity: string;
        internal: string;
    };
    keycloak: {
        realm: string;
        client: string;
        baseUrl: string;
        secret: string;
    };
    aws: {
        region: string;
        emailQueueUrl: string;
        accessKeyId: string;
        secretAccessKey: string;
    };
    sentry: {
        dsn: string;
        environment: string;
        tracesSampleRate: number;
    };
    // ... 더 많은 섹션
}
```

### 구현 패턴

**파일:** `/blog-api/src/config/unifiedConfig.ts`

```typescript
import * as fs from 'fs';
import * as path from 'path';
import * as ini from 'ini';

const configPath = path.join(__dirname, '../../config.ini');
const iniConfig = ini.parse(fs.readFileSync(configPath, 'utf-8'));

export const config: UnifiedConfig = {
    database: {
        host: iniConfig.database?.host || process.env.DB_HOST || 'localhost',
        port: parseInt(iniConfig.database?.port || process.env.DB_PORT || '3306'),
        username: iniConfig.database?.username || process.env.DB_USER || 'root',
        password: iniConfig.database?.password || process.env.DB_PASSWORD || '',
        database: iniConfig.database?.database || process.env.DB_NAME || 'blog_dev',
    },
    server: {
        port: parseInt(iniConfig.server?.port || process.env.PORT || '3002'),
        sessionSecret: iniConfig.server?.sessionSecret || process.env.SESSION_SECRET || 'dev-secret',
    },
    // ... 더 많은 설정
};

// 중요 설정 유효성 검사
if (!config.tokens.jwt) {
    throw new Error('JWT secret not configured!');
}
```

**핵심 포인트:**
- config.ini에서 먼저 읽기
- process.env로 폴백
- 개발용 기본값
- 시작 시 유효성 검사
- 타입 안전 액세스

---

## 환경별 설정

### config.ini 구조

```ini
[database]
host = localhost
port = 3306
username = root
password = password1
database = blog_dev

[server]
port = 3002
sessionSecret = your-secret-here

[tokens]
jwt = your-jwt-secret
inactivity = 30m
internal = internal-api-token

[keycloak]
realm = myapp
client = myapp-client
baseUrl = http://localhost:8080
secret = keycloak-client-secret

[sentry]
dsn = https://your-sentry-dsn
environment = development
tracesSampleRate = 0.1
```

### 환경 오버라이드

```bash
# .env 파일 (선택적 오버라이드)
DB_HOST=production-db.example.com
DB_PASSWORD=secure-password
PORT=80
```

**우선순위:**
1. config.ini (최고 우선순위)
2. process.env 변수
3. 하드코딩된 기본값 (최저 우선순위)

---

## 시크릿 관리

### 시크릿 커밋 금지

```gitignore
# .gitignore
config.ini
.env
sentry.ini
*.pem
*.key
```

### 프로덕션에서 환경 변수 사용

```typescript
// 개발: config.ini
// 프로덕션: 환경 변수

export const config: UnifiedConfig = {
    database: {
        password: process.env.DB_PASSWORD || iniConfig.database?.password || '',
    },
    tokens: {
        jwt: process.env.JWT_SECRET || iniConfig.tokens?.jwt || '',
    },
};
```

---

## 마이그레이션 가이드

### 모든 process.env 사용 찾기

```bash
grep -r "process.env" blog-api/src/ --include="*.ts" | wc -l
```

### 마이그레이션 예시

**이전:**
```typescript
// 코드 전체에 분산됨
const timeout = parseInt(process.env.OPENID_HTTP_TIMEOUT_MS || '15000');
const keycloakUrl = process.env.KEYCLOAK_BASE_URL;
const jwtSecret = process.env.JWT_SECRET;
```

**이후:**
```typescript
import { config } from './config/unifiedConfig';

const timeout = config.keycloak.timeout;
const keycloakUrl = config.keycloak.baseUrl;
const jwtSecret = config.tokens.jwt;
```

**장점:**
- 타입 안전
- 중앙 집중화
- 테스트하기 쉬움
- 시작 시 유효성 검사

---

**관련 파일:**
- [SKILL.md](SKILL.md)
- [testing-guide.md](testing-guide.md)
