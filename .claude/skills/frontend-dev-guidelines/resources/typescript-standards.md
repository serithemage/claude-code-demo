# TypeScript 표준

React 프론트엔드 코드의 타입 안전성과 유지보수성을 위한 TypeScript 모범 사례입니다.

---

## Strict 모드

### 설정

프로젝트에서 TypeScript strict 모드가 **활성화**되어 있습니다:

```json
// tsconfig.json
{
    "compilerOptions": {
        "strict": true,
        "noImplicitAny": true,
        "strictNullChecks": true
    }
}
```

**의미:**
- 암시적 `any` 타입 금지
- Null/undefined를 명시적으로 처리해야 함
- 타입 안전성 강제

---

## `any` 타입 금지

### 규칙

```typescript
// ❌ 절대 any 사용 금지
function handleData(data: any) {
    return data.something;
}

// ✅ 특정 타입 사용
interface MyData {
    something: string;
}

function handleData(data: MyData) {
    return data.something;
}

// ✅ 또는 진짜 알 수 없는 데이터에는 unknown 사용
function handleUnknown(data: unknown) {
    if (typeof data === 'object' && data !== null && 'something' in data) {
        return (data as MyData).something;
    }
}
```

**타입을 정말 모르는 경우:**
- `unknown` 사용 (타입 체크 강제)
- Type guards로 좁히기
- 타입이 알 수 없는 이유 문서화

---

## 명시적 반환 타입

### 함수 반환 타입

```typescript
// ✅ 올바름 - 명시적 반환 타입
function getUser(id: number): Promise<User> {
    return apiClient.get(`/users/${id}`);
}

function calculateTotal(items: Item[]): number {
    return items.reduce((sum, item) => sum + item.price, 0);
}

// ❌ 피하세요 - 암시적 반환 타입 (덜 명확)
function getUser(id: number) {
    return apiClient.get(`/users/${id}`);
}
```

### 컴포넌트 반환 타입

```typescript
// React.FC가 이미 반환 타입 제공 (ReactElement)
export const MyComponent: React.FC<Props> = ({ prop }) => {
    return <div>{prop}</div>;
};

// 커스텀 hooks의 경우
function useMyData(id: number): { data: Data; isLoading: boolean } {
    const [data, setData] = useState<Data | null>(null);
    const [isLoading, setIsLoading] = useState(true);

    return { data: data!, isLoading };
}
```

---

## 타입 Imports

### 'type' 키워드 사용

```typescript
// ✅ 올바름 - 타입 import 명시적으로 표시
import type { User } from '~types/user';
import type { Post } from '~types/post';
import type { SxProps, Theme } from '@mui/material';

// ❌ 피하세요 - 값과 타입 import 혼합
import { User } from '~types/user';  // 타입인지 값인지 불명확
```

**장점:**
- 타입과 값을 명확히 분리
- 더 나은 tree-shaking
- 순환 의존성 방지
- TypeScript 컴파일러 최적화

---

## 컴포넌트 Prop 인터페이스

### 인터페이스 패턴

```typescript
/**
 * MyComponent의 Props
 */
interface MyComponentProps {
    /** 표시할 사용자 ID */
    userId: number;

    /** 액션 완료 시 선택적 콜백 */
    onComplete?: () => void;

    /** 컴포넌트 표시 모드 */
    mode?: 'view' | 'edit';

    /** 추가 CSS 클래스 */
    className?: string;
}

export const MyComponent: React.FC<MyComponentProps> = ({
    userId,
    onComplete,
    mode = 'view',  // 기본값
    className,
}) => {
    return <div>...</div>;
};
```

**핵심 포인트:**
- props를 위한 별도 인터페이스
- 각 prop에 JSDoc 주석
- 선택적 props는 `?` 사용
- 구조분해에서 기본값 제공

### Children이 있는 Props

```typescript
interface ContainerProps {
    children: React.ReactNode;
    title: string;
}

// React.FC는 자동으로 children 타입 포함하지만, 명시적으로 작성
export const Container: React.FC<ContainerProps> = ({ children, title }) => {
    return (
        <div>
            <h2>{title}</h2>
            {children}
        </div>
    );
};
```

---

## 유틸리티 타입

### Partial<T>

```typescript
// 모든 속성을 선택적으로
type UserUpdate = Partial<User>;

function updateUser(id: number, updates: Partial<User>) {
    // updates는 User 속성의 어떤 부분집합도 가능
}
```

### Pick<T, K>

```typescript
// 특정 속성 선택
type UserPreview = Pick<User, 'id' | 'name' | 'email'>;

const preview: UserPreview = {
    id: 1,
    name: 'John',
    email: 'john@example.com',
    // 다른 User 속성 허용 안 됨
};
```

### Omit<T, K>

```typescript
// 특정 속성 제외
type UserWithoutPassword = Omit<User, 'password' | 'passwordHash'>;

const publicUser: UserWithoutPassword = {
    id: 1,
    name: 'John',
    email: 'john@example.com',
    // password와 passwordHash 허용 안 됨
};
```

### Required<T>

```typescript
// 모든 속성을 필수로
type RequiredConfig = Required<Config>;  // 모든 선택적 props가 필수로
```

### Record<K, V>

```typescript
// 타입 안전 객체/맵
const userMap: Record<string, User> = {
    'user1': { id: 1, name: 'John' },
    'user2': { id: 2, name: 'Jane' },
};

// 스타일용
import type { SxProps, Theme } from '@mui/material';

const styles: Record<string, SxProps<Theme>> = {
    container: { p: 2 },
    header: { mb: 1 },
};
```

---

## Type Guards

### 기본 Type Guards

```typescript
function isUser(data: unknown): data is User {
    return (
        typeof data === 'object' &&
        data !== null &&
        'id' in data &&
        'name' in data
    );
}

// 사용
if (isUser(response)) {
    console.log(response.name);  // TypeScript가 User로 인식
}
```

### Discriminated Unions

```typescript
type LoadingState =
    | { status: 'idle' }
    | { status: 'loading' }
    | { status: 'success'; data: Data }
    | { status: 'error'; error: Error };

function Component({ state }: { state: LoadingState }) {
    // TypeScript가 status 기반으로 타입 좁히기
    if (state.status === 'success') {
        return <Display data={state.data} />;  // 여기서 data 사용 가능
    }

    if (state.status === 'error') {
        return <Error error={state.error} />;  // 여기서 error 사용 가능
    }

    return <Loading />;
}
```

---

## 제네릭 타입

### 제네릭 함수

```typescript
function getById<T>(items: T[], id: number): T | undefined {
    return items.find(item => (item as any).id === id);
}

// 타입 추론과 함께 사용
const users: User[] = [...];
const user = getById(users, 123);  // 타입: User | undefined
```

### 제네릭 컴포넌트

```typescript
interface ListProps<T> {
    items: T[];
    renderItem: (item: T) => React.ReactNode;
}

export function List<T>({ items, renderItem }: ListProps<T>): React.ReactElement {
    return (
        <div>
            {items.map((item, index) => (
                <div key={index}>{renderItem(item)}</div>
            ))}
        </div>
    );
}

// 사용
<List<User>
    items={users}
    renderItem={(user) => <UserCard user={user} />}
/>
```

---

## 타입 단언 (신중하게 사용)

### 사용해야 할 때

```typescript
// ✅ OK - TypeScript보다 더 알고 있을 때
const element = document.getElementById('my-element') as HTMLInputElement;
const value = element.value;

// ✅ OK - 검증한 API 응답
const response = await api.getData();
const user = response.data as User;  // 형태를 알고 있음
```

### 사용하면 안 되는 경우

```typescript
// ❌ 피하세요 - 타입 안전성 우회
const data = getData() as any;  // 잘못됨 - TypeScript 무력화

// ❌ 피하세요 - 안전하지 않은 단언
const value = unknownValue as string;  // 실제로 string이 아닐 수 있음
```

---

## Null/Undefined 처리

### Optional Chaining

```typescript
// ✅ 올바름
const name = user?.profile?.name;

// 동등한 코드:
const name = user && user.profile && user.profile.name;
```

### Nullish Coalescing

```typescript
// ✅ 올바름
const displayName = user?.name ?? 'Anonymous';

// null 또는 undefined일 때만 기본값 사용
// (||와 다름 - ''나 0, false에서는 트리거 안 됨)
```

### Non-Null Assertion (신중하게 사용)

```typescript
// ✅ OK - 값이 존재한다고 확신할 때
const data = queryClient.getQueryData<Data>(['data'])!;

// ⚠️ 주의 - null이 아님을 알 때만 사용
// 명시적 체크가 더 나음:
const data = queryClient.getQueryData<Data>(['data']);
if (data) {
    // data 사용
}
```

---

## 요약

**TypeScript 체크리스트:**
- ✅ Strict 모드 활성화
- ✅ `any` 타입 금지 (필요하면 `unknown` 사용)
- ✅ 함수에 명시적 반환 타입
- ✅ 타입 imports에 `import type` 사용
- ✅ prop 인터페이스에 JSDoc 주석
- ✅ 유틸리티 타입 (Partial, Pick, Omit, Required, Record)
- ✅ 좁히기를 위한 Type guards
- ✅ Optional chaining과 nullish coalescing
- ❌ 꼭 필요한 경우 아니면 타입 단언 피하기

**참고:**
- [component-patterns.md](component-patterns.md) - 컴포넌트 타이핑
- [data-fetching.md](data-fetching.md) - API 타이핑
