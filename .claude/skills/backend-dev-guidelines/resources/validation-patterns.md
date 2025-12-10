# 유효성 검사 패턴 - Zod를 사용한 입력 검증

타입 안전 유효성 검사를 위한 Zod 스키마 사용에 대한 완전한 가이드입니다.

## 목차

- [Zod를 사용하는 이유](#zod를-사용하는-이유)
- [기본 Zod 패턴](#기본-zod-패턴)
- [코드베이스 스키마 예시](#코드베이스-스키마-예시)
- [Route 레벨 유효성 검사](#route-레벨-유효성-검사)
- [Controller 유효성 검사](#controller-유효성-검사)
- [DTO 패턴](#dto-패턴)
- [에러 처리](#에러-처리)
- [고급 패턴](#고급-패턴)

---

## Zod를 사용하는 이유

### Joi/다른 라이브러리 대비 장점

**타입 안전성:**
- ✅ 완전한 TypeScript 추론
- ✅ 런타임 + 컴파일 타임 유효성 검사
- ✅ 자동 타입 생성

**개발자 경험:**
- ✅ 직관적인 API
- ✅ 조합 가능한 스키마
- ✅ 훌륭한 에러 메시지

**성능:**
- ✅ 빠른 유효성 검사
- ✅ 작은 번들 사이즈
- ✅ Tree-shakeable

### Joi에서 마이그레이션

현대적인 유효성 검사는 Joi 대신 Zod를 사용합니다:

```typescript
// ❌ 구식 - Joi (단계적 폐지 중)
const schema = Joi.object({
    email: Joi.string().email().required(),
    name: Joi.string().min(3).required(),
});

// ✅ 신식 - Zod (선호)
const schema = z.object({
    email: z.string().email(),
    name: z.string().min(3),
});
```

---

## 기본 Zod 패턴

### 기본 타입

```typescript
import { z } from 'zod';

// 문자열
const nameSchema = z.string();
const emailSchema = z.string().email();
const urlSchema = z.string().url();
const uuidSchema = z.string().uuid();
const minLengthSchema = z.string().min(3);
const maxLengthSchema = z.string().max(100);

// 숫자
const ageSchema = z.number().int().positive();
const priceSchema = z.number().positive();
const rangeSchema = z.number().min(0).max(100);

// 불리언
const activeSchema = z.boolean();

// 날짜
const dateSchema = z.string().datetime(); // ISO 8601 문자열
const nativeDateSchema = z.date(); // 네이티브 Date 객체

// 열거형
const roleSchema = z.enum(['admin', 'operations', 'user']);
const statusSchema = z.enum(['PENDING', 'APPROVED', 'REJECTED']);
```

### 객체

```typescript
// 간단한 객체
const userSchema = z.object({
    email: z.string().email(),
    name: z.string(),
    age: z.number().int().positive(),
});

// 중첩 객체
const addressSchema = z.object({
    street: z.string(),
    city: z.string(),
    zipCode: z.string().regex(/^\d{5}$/),
});

const userWithAddressSchema = z.object({
    name: z.string(),
    address: addressSchema,
});

// 선택적 필드
const userSchema = z.object({
    name: z.string(),
    email: z.string().email().optional(),
    phone: z.string().optional(),
});

// Nullable 필드
const userSchema = z.object({
    name: z.string(),
    middleName: z.string().nullable(),
});
```

### 배열

```typescript
// 기본 타입 배열
const rolesSchema = z.array(z.string());
const numbersSchema = z.array(z.number());

// 객체 배열
const usersSchema = z.array(
    z.object({
        id: z.string(),
        name: z.string(),
    })
);

// 제약 조건이 있는 배열
const tagsSchema = z.array(z.string()).min(1).max(10);
const nonEmptyArray = z.array(z.string()).nonempty();
```

---

## 코드베이스 스키마 예시

### 폼 유효성 검사 스키마

**파일:** `/form/src/helpers/zodSchemas.ts`

```typescript
import { z } from 'zod';

// 질문 타입 열거형
export const questionTypeSchema = z.enum([
    'input',
    'textbox',
    'editor',
    'dropdown',
    'autocomplete',
    'checkbox',
    'radio',
    'upload',
]);

// 업로드 타입
export const uploadTypeSchema = z.array(
    z.enum(['pdf', 'image', 'excel', 'video', 'powerpoint', 'word']).nullable()
);

// 입력 타입
export const inputTypeSchema = z
    .enum(['date', 'number', 'input', 'currency'])
    .nullable();

// 질문 옵션
export const questionOptionSchema = z.object({
    id: z.number().int().positive().optional(),
    controlTag: z.string().max(150).nullable().optional(),
    label: z.string().max(100).nullable().optional(),
    order: z.number().int().min(0).default(0),
});

// 질문 스키마
export const questionSchema = z.object({
    id: z.number().int().positive().optional(),
    formID: z.number().int().positive(),
    sectionID: z.number().int().positive().optional(),
    options: z.array(questionOptionSchema).optional(),
    label: z.string().max(500),
    description: z.string().max(5000).optional(),
    type: questionTypeSchema,
    uploadTypes: uploadTypeSchema.optional(),
    inputType: inputTypeSchema.optional(),
    tags: z.array(z.string().max(150)).optional(),
    required: z.boolean(),
    isStandard: z.boolean().optional(),
    deprecatedKey: z.string().nullable().optional(),
    maxLength: z.number().int().positive().nullable().optional(),
    isOptionsSorted: z.boolean().optional(),
});

// 폼 섹션 스키마
export const formSectionSchema = z.object({
    id: z.number().int().positive(),
    formID: z.number().int().positive(),
    questions: z.array(questionSchema).optional(),
    label: z.string().max(500),
    description: z.string().max(5000).optional(),
    isStandard: z.boolean(),
});

// 폼 생성 스키마
export const createFormSchema = z.object({
    id: z.number().int().positive(),
    label: z.string().max(150),
    description: z.string().max(6000).nullable().optional(),
    isPhase: z.boolean().optional(),
    username: z.string(),
});

// 순서 업데이트 스키마
export const updateOrderSchema = z.object({
    source: z.object({
        index: z.number().int().min(0),
        sectionID: z.number().int().min(0),
    }),
    destination: z.object({
        index: z.number().int().min(0),
        sectionID: z.number().int().min(0),
    }),
});

// Controller 전용 유효성 검사 스키마
export const createQuestionValidationSchema = z.object({
    formID: z.number().int().positive(),
    sectionID: z.number().int().positive(),
    question: questionSchema,
    index: z.number().int().min(0).nullable().optional(),
    username: z.string(),
});

export const updateQuestionValidationSchema = z.object({
    questionID: z.number().int().positive(),
    username: z.string(),
    question: questionSchema,
});
```

### 프록시 관계 스키마

```typescript
// 프록시 관계 유효성 검사
const createProxySchema = z.object({
    originalUserID: z.string().min(1),
    proxyUserID: z.string().min(1),
    startsAt: z.string().datetime(),
    expiresAt: z.string().datetime(),
});

// 커스텀 유효성 검사 포함
const createProxySchemaWithValidation = createProxySchema.refine(
    (data) => new Date(data.expiresAt) > new Date(data.startsAt),
    {
        message: 'expiresAt must be after startsAt',
        path: ['expiresAt'],
    }
);
```

### Workflow 유효성 검사

```typescript
// Workflow 시작 스키마
const startWorkflowSchema = z.object({
    workflowCode: z.string().min(1),
    entityType: z.enum(['Post', 'User', 'Comment']),
    entityID: z.number().int().positive(),
    dryRun: z.boolean().optional().default(false),
});

// Workflow 단계 완료 스키마
const completeStepSchema = z.object({
    stepInstanceID: z.number().int().positive(),
    answers: z.record(z.string(), z.any()),
    dryRun: z.boolean().optional().default(false),
});
```

---

## Route 레벨 유효성 검사

### 패턴 1: 인라인 유효성 검사

```typescript
// routes/proxyRoutes.ts
import { z } from 'zod';

const createProxySchema = z.object({
    originalUserID: z.string().min(1),
    proxyUserID: z.string().min(1),
    startsAt: z.string().datetime(),
    expiresAt: z.string().datetime(),
});

router.post(
    '/',
    SSOMiddlewareClient.verifyLoginStatus,
    async (req, res) => {
        try {
            // Route 레벨에서 유효성 검사
            const validated = createProxySchema.parse(req.body);

            // Service로 위임
            const proxy = await proxyService.createProxyRelationship(validated);

            res.status(201).json({ success: true, data: proxy });
        } catch (error) {
            if (error instanceof z.ZodError) {
                return res.status(400).json({
                    success: false,
                    error: {
                        message: 'Validation failed',
                        details: error.errors,
                    },
                });
            }
            handler.handleException(res, error);
        }
    }
);
```

**장점:**
- 빠르고 간단
- 단순한 routes에 좋음

**단점:**
- 유효성 검사 로직이 routes에 있음
- 테스트하기 어려움
- 재사용 불가

---

## Controller 유효성 검사

### 패턴 2: Controller 유효성 검사 (권장)

```typescript
// validators/userSchemas.ts
import { z } from 'zod';

export const createUserSchema = z.object({
    email: z.string().email(),
    name: z.string().min(2).max(100),
    roles: z.array(z.enum(['admin', 'operations', 'user'])),
    isActive: z.boolean().default(true),
});

export const updateUserSchema = z.object({
    email: z.string().email().optional(),
    name: z.string().min(2).max(100).optional(),
    roles: z.array(z.enum(['admin', 'operations', 'user'])).optional(),
    isActive: z.boolean().optional(),
});

export type CreateUserDTO = z.infer<typeof createUserSchema>;
export type UpdateUserDTO = z.infer<typeof updateUserSchema>;
```

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

    async createUser(req: Request, res: Response): Promise<void> {
        try {
            // 입력 유효성 검사
            const validated = createUserSchema.parse(req.body);

            // Service 호출
            const user = await this.userService.createUser(validated);

            this.handleSuccess(res, user, 'User created successfully', 201);
        } catch (error) {
            if (error instanceof z.ZodError) {
                // 400 상태로 유효성 검사 에러 처리
                return this.handleError(error, res, 'createUser', 400);
            }
            this.handleError(error, res, 'createUser');
        }
    }

    async updateUser(req: Request, res: Response): Promise<void> {
        try {
            // params와 body 유효성 검사
            const userId = req.params.id;
            const validated = updateUserSchema.parse(req.body);

            const user = await this.userService.updateUser(userId, validated);

            this.handleSuccess(res, user, 'User updated successfully');
        } catch (error) {
            if (error instanceof z.ZodError) {
                return this.handleError(error, res, 'updateUser', 400);
            }
            this.handleError(error, res, 'updateUser');
        }
    }
}
```

**장점:**
- 깔끔한 분리
- 재사용 가능한 스키마
- 테스트하기 쉬움
- 타입 안전 DTOs

**단점:**
- 관리할 파일이 더 많음

---

## DTO 패턴

### 스키마에서 타입 추론

```typescript
import { z } from 'zod';

// 스키마 정의
const createUserSchema = z.object({
    email: z.string().email(),
    name: z.string(),
    age: z.number().int().positive(),
});

// 스키마에서 TypeScript 타입 추론
type CreateUserDTO = z.infer<typeof createUserSchema>;

// 다음과 동일:
// type CreateUserDTO = {
//     email: string;
//     name: string;
//     age: number;
// }

// Service에서 사용
class UserService {
    async createUser(data: CreateUserDTO): Promise<User> {
        // data는 완전히 타입화됨!
        console.log(data.email); // ✅ TypeScript가 이것이 존재함을 알고 있음
        console.log(data.invalid); // ❌ TypeScript 에러!
    }
}
```

### 입력 vs 출력 타입

```typescript
// 입력 스키마 (API가 받는 것)
const createUserInputSchema = z.object({
    email: z.string().email(),
    name: z.string(),
    password: z.string().min(8),
});

// 출력 스키마 (API가 반환하는 것)
const userOutputSchema = z.object({
    id: z.string().uuid(),
    email: z.string().email(),
    name: z.string(),
    createdAt: z.string().datetime(),
    // password 제외!
});

type CreateUserInput = z.infer<typeof createUserInputSchema>;
type UserOutput = z.infer<typeof userOutputSchema>;
```

---

## 에러 처리

### Zod 에러 포맷

```typescript
try {
    const validated = schema.parse(data);
} catch (error) {
    if (error instanceof z.ZodError) {
        console.log(error.errors);
        // [
        //   {
        //     code: 'invalid_type',
        //     expected: 'string',
        //     received: 'number',
        //     path: ['email'],
        //     message: 'Expected string, received number'
        //   }
        // ]
    }
}
```

### 커스텀 에러 메시지

```typescript
const userSchema = z.object({
    email: z.string().email({ message: 'Please provide a valid email address' }),
    name: z.string().min(2, { message: 'Name must be at least 2 characters' }),
    age: z.number().int().positive({ message: 'Age must be a positive number' }),
});
```

### 포맷된 에러 응답

```typescript
// Zod 에러를 포맷하는 헬퍼 함수
function formatZodError(error: z.ZodError) {
    return {
        message: 'Validation failed',
        errors: error.errors.map((err) => ({
            field: err.path.join('.'),
            message: err.message,
            code: err.code,
        })),
    };
}

// Controller에서
catch (error) {
    if (error instanceof z.ZodError) {
        return res.status(400).json({
            success: false,
            error: formatZodError(error),
        });
    }
}

// 응답 예시:
// {
//   "success": false,
//   "error": {
//     "message": "Validation failed",
//     "errors": [
//       {
//         "field": "email",
//         "message": "Invalid email",
//         "code": "invalid_string"
//       }
//     ]
//   }
// }
```

---

## 고급 패턴

### 조건부 유효성 검사

```typescript
// 다른 필드 값에 기반한 유효성 검사
const submissionSchema = z.object({
    type: z.enum(['NEW', 'UPDATE']),
    postId: z.number().optional(),
}).refine(
    (data) => {
        // type이 UPDATE면 postId 필수
        if (data.type === 'UPDATE') {
            return data.postId !== undefined;
        }
        return true;
    },
    {
        message: 'postId is required when type is UPDATE',
        path: ['postId'],
    }
);
```

### 데이터 변환

```typescript
// 문자열을 숫자로 변환
const userSchema = z.object({
    name: z.string(),
    age: z.string().transform((val) => parseInt(val, 10)),
});

// 날짜 변환
const eventSchema = z.object({
    name: z.string(),
    date: z.string().transform((str) => new Date(str)),
});
```

### 데이터 전처리

```typescript
// 유효성 검사 전에 문자열 trim
const userSchema = z.object({
    email: z.preprocess(
        (val) => typeof val === 'string' ? val.trim().toLowerCase() : val,
        z.string().email()
    ),
    name: z.preprocess(
        (val) => typeof val === 'string' ? val.trim() : val,
        z.string().min(2)
    ),
});
```

### 유니온 타입

```typescript
// 여러 가능한 타입
const idSchema = z.union([z.string(), z.number()]);

// 구별된 유니온
const notificationSchema = z.discriminatedUnion('type', [
    z.object({
        type: z.literal('email'),
        recipient: z.string().email(),
        subject: z.string(),
    }),
    z.object({
        type: z.literal('sms'),
        phoneNumber: z.string(),
        message: z.string(),
    }),
]);
```

### 재귀 스키마

```typescript
// 트리 같은 중첩 구조용
type Category = {
    id: number;
    name: string;
    children?: Category[];
};

const categorySchema: z.ZodType<Category> = z.lazy(() =>
    z.object({
        id: z.number(),
        name: z.string(),
        children: z.array(categorySchema).optional(),
    })
);
```

### 스키마 조합

```typescript
// 기본 스키마
const timestampsSchema = z.object({
    createdAt: z.string().datetime(),
    updatedAt: z.string().datetime(),
});

const auditSchema = z.object({
    createdBy: z.string(),
    updatedBy: z.string(),
});

// 스키마 조합
const userSchema = z.object({
    id: z.string(),
    email: z.string().email(),
    name: z.string(),
}).merge(timestampsSchema).merge(auditSchema);

// 스키마 확장
const adminUserSchema = userSchema.extend({
    adminLevel: z.number().int().min(1).max(5),
    permissions: z.array(z.string()),
});

// 특정 필드 선택
const publicUserSchema = userSchema.pick({
    id: true,
    name: true,
    // email 제외
});

// 필드 제외
const userWithoutTimestamps = userSchema.omit({
    createdAt: true,
    updatedAt: true,
});
```

### Validation Middleware

```typescript
// 재사용 가능한 validation middleware 생성
import { Request, Response, NextFunction } from 'express';
import { z } from 'zod';

export function validateBody<T extends z.ZodType>(schema: T) {
    return (req: Request, res: Response, next: NextFunction) => {
        try {
            req.body = schema.parse(req.body);
            next();
        } catch (error) {
            if (error instanceof z.ZodError) {
                return res.status(400).json({
                    success: false,
                    error: {
                        message: 'Validation failed',
                        details: error.errors,
                    },
                });
            }
            next(error);
        }
    };
}

// 사용법
router.post('/users',
    validateBody(createUserSchema),
    async (req, res) => {
        // req.body가 유효성 검사되고 타입화됨!
        const user = await userService.createUser(req.body);
        res.json({ success: true, data: user });
    }
);
```

---

**관련 파일:**
- [SKILL.md](SKILL.md) - 메인 가이드
- [routing-and-controllers.md](routing-and-controllers.md) - Controller에서 유효성 검사 사용
- [services-and-repositories.md](services-and-repositories.md) - Services에서 DTOs 사용
- [async-and-errors.md](async-and-errors.md) - 에러 처리 패턴
