# Loading & Error States

**중요**: 적절한 loading과 error state 처리는 레이아웃 이동을 방지하고 더 나은 사용자 경험을 제공합니다.

---

## ⚠️ 핵심 규칙: Early Return 절대 금지

### 문제

```typescript
// ❌ 절대 이렇게 하지 마세요 - loading spinner로 early return
const Component = () => {
    const { data, isLoading } = useQuery();

    // 잘못됨: 레이아웃 이동과 나쁜 UX 유발
    if (isLoading) {
        return <LoadingSpinner />;
    }

    return <Content data={data} />;
};
```

**왜 나쁜가:**
1. **레이아웃 이동**: 로딩 완료 시 콘텐츠 위치가 점프
2. **CLS (Cumulative Layout Shift)**: 나쁜 Core Web Vital 점수
3. **불쾌한 UX**: 페이지 구조가 갑자기 변경
4. **스크롤 위치 손실**: 사용자가 페이지에서 위치를 잃음

### 해결책

**옵션 1: SuspenseLoader (새 컴포넌트에 권장)**

```typescript
import { SuspenseLoader } from '~components/SuspenseLoader';

const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

export const MyComponent: React.FC = () => {
    return (
        <SuspenseLoader>
            <HeavyComponent />
        </SuspenseLoader>
    );
};
```

**옵션 2: LoadingOverlay (레거시 useQuery 패턴용)**

```typescript
import { LoadingOverlay } from '~components/LoadingOverlay';

export const MyComponent: React.FC = () => {
    const { data, isLoading } = useQuery({ ... });

    return (
        <LoadingOverlay loading={isLoading}>
            <Content data={data} />
        </LoadingOverlay>
    );
};
```

---

## SuspenseLoader 컴포넌트

### 기능

- lazy 컴포넌트 로딩 중 로딩 인디케이터 표시
- 부드러운 페이드인 애니메이션
- 레이아웃 이동 방지
- 앱 전체에서 일관된 로딩 경험

### Import

```typescript
import { SuspenseLoader } from '~components/SuspenseLoader';
// 또는
import { SuspenseLoader } from '@/components/SuspenseLoader';
```

### 기본 사용법

```typescript
<SuspenseLoader>
    <LazyLoadedComponent />
</SuspenseLoader>
```

### useSuspenseQuery와 함께

```typescript
import { useSuspenseQuery } from '@tanstack/react-query';
import { SuspenseLoader } from '~components/SuspenseLoader';

const Inner: React.FC = () => {
    // isLoading 불필요!
    const { data } = useSuspenseQuery({
        queryKey: ['data'],
        queryFn: () => api.getData(),
    });

    return <Display data={data} />;
};

// 외부 컴포넌트가 Suspense로 감싸기
export const Outer: React.FC = () => {
    return (
        <SuspenseLoader>
            <Inner />
        </SuspenseLoader>
    );
};
```

### 여러 Suspense Boundary

**패턴**: 독립적인 섹션에 대해 별도의 로딩

```typescript
export const Dashboard: React.FC = () => {
    return (
        <Box>
            <SuspenseLoader>
                <Header />
            </SuspenseLoader>

            <SuspenseLoader>
                <MainContent />
            </SuspenseLoader>

            <SuspenseLoader>
                <Sidebar />
            </SuspenseLoader>
        </Box>
    );
};
```

**장점:**
- 각 섹션이 독립적으로 로드
- 사용자가 부분 콘텐츠를 더 빨리 볼 수 있음
- 더 나은 체감 성능

### 중첩 Suspense

```typescript
export const ParentComponent: React.FC = () => {
    return (
        <SuspenseLoader>
            {/* Parent가 로딩 중 suspend */}
            <ParentContent>
                <SuspenseLoader>
                    {/* child를 위한 중첩 suspense */}
                    <ChildComponent />
                </SuspenseLoader>
            </ParentContent>
        </SuspenseLoader>
    );
};
```

---

## LoadingOverlay 컴포넌트

### 사용 시점

- `useQuery`가 있는 레거시 컴포넌트 (아직 Suspense로 리팩토링 안 됨)
- 오버레이 로딩 상태가 필요할 때
- Suspense boundary를 사용할 수 없을 때

### 사용법

```typescript
import { LoadingOverlay } from '~components/LoadingOverlay';

export const MyComponent: React.FC = () => {
    const { data, isLoading } = useQuery({
        queryKey: ['data'],
        queryFn: () => api.getData(),
    });

    return (
        <LoadingOverlay loading={isLoading}>
            <Box sx={{ p: 2 }}>
                {data && <Content data={data} />}
            </Box>
        </LoadingOverlay>
    );
};
```

**기능:**
- 스피너가 있는 반투명 오버레이 표시
- 콘텐츠 영역 예약 (레이아웃 이동 없음)
- 로딩 중 상호작용 방지

---

## 에러 처리

### useMuiSnackbar Hook (필수)

**절대 react-toastify 사용 금지** - 프로젝트 표준은 MUI Snackbar

```typescript
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

export const MyComponent: React.FC = () => {
    const { showSuccess, showError, showInfo, showWarning } = useMuiSnackbar();

    const handleAction = async () => {
        try {
            await api.doSomething();
            showSuccess('Operation completed successfully');
        } catch (error) {
            showError('Operation failed');
        }
    };

    return <Button onClick={handleAction}>Do Action</Button>;
};
```

**사용 가능한 메서드:**
- `showSuccess(message)` - 녹색 성공 메시지
- `showError(message)` - 빨간색 에러 메시지
- `showWarning(message)` - 주황색 경고 메시지
- `showInfo(message)` - 파란색 정보 메시지

### TanStack Query Error Callbacks

```typescript
import { useSuspenseQuery } from '@tanstack/react-query';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

export const MyComponent: React.FC = () => {
    const { showError } = useMuiSnackbar();

    const { data } = useSuspenseQuery({
        queryKey: ['data'],
        queryFn: () => api.getData(),

        // 에러 처리
        onError: (error) => {
            showError('Failed to load data');
            console.error('Query error:', error);
        },
    });

    return <Content data={data} />;
};
```

### Error Boundaries

```typescript
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
    return (
        <Box sx={{ p: 4, textAlign: 'center' }}>
            <Typography variant='h5' color='error'>
                Something went wrong
            </Typography>
            <Typography>{error.message}</Typography>
            <Button onClick={resetErrorBoundary}>Try Again</Button>
        </Box>
    );
}

export const MyPage: React.FC = () => {
    return (
        <ErrorBoundary
            FallbackComponent={ErrorFallback}
            onError={(error) => console.error('Boundary caught:', error)}
        >
            <SuspenseLoader>
                <ComponentThatMightError />
            </SuspenseLoader>
        </ErrorBoundary>
    );
};
```

---

## 완전한 예제

### 예제 1: Suspense가 있는 최신 컴포넌트

```typescript
import React from 'react';
import { Box, Paper } from '@mui/material';
import { useSuspenseQuery } from '@tanstack/react-query';
import { SuspenseLoader } from '~components/SuspenseLoader';
import { myFeatureApi } from '../api/myFeatureApi';

// Inner 컴포넌트는 useSuspenseQuery 사용
const InnerComponent: React.FC<{ id: number }> = ({ id }) => {
    const { data } = useSuspenseQuery({
        queryKey: ['entity', id],
        queryFn: () => myFeatureApi.getEntity(id),
    });

    // data는 항상 정의됨 - isLoading 불필요!
    return (
        <Paper sx={{ p: 2 }}>
            <h2>{data.title}</h2>
            <p>{data.description}</p>
        </Paper>
    );
};

// Outer 컴포넌트가 Suspense boundary 제공
export const OuterComponent: React.FC<{ id: number }> = ({ id }) => {
    return (
        <Box>
            <SuspenseLoader>
                <InnerComponent id={id} />
            </SuspenseLoader>
        </Box>
    );
};

export default OuterComponent;
```

### 예제 2: LoadingOverlay가 있는 레거시 패턴

```typescript
import React from 'react';
import { Box } from '@mui/material';
import { useQuery } from '@tanstack/react-query';
import { LoadingOverlay } from '~components/LoadingOverlay';
import { myFeatureApi } from '../api/myFeatureApi';

export const LegacyComponent: React.FC<{ id: number }> = ({ id }) => {
    const { data, isLoading, error } = useQuery({
        queryKey: ['entity', id],
        queryFn: () => myFeatureApi.getEntity(id),
    });

    return (
        <LoadingOverlay loading={isLoading}>
            <Box sx={{ p: 2 }}>
                {error && <ErrorDisplay error={error} />}
                {data && <Content data={data} />}
            </Box>
        </LoadingOverlay>
    );
};
```

### 예제 3: Snackbar가 있는 에러 처리

```typescript
import React from 'react';
import { useSuspenseQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Button } from '@mui/material';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';
import { myFeatureApi } from '../api/myFeatureApi';

export const EntityEditor: React.FC<{ id: number }> = ({ id }) => {
    const queryClient = useQueryClient();
    const { showSuccess, showError } = useMuiSnackbar();

    const { data } = useSuspenseQuery({
        queryKey: ['entity', id],
        queryFn: () => myFeatureApi.getEntity(id),
        onError: () => {
            showError('Failed to load entity');
        },
    });

    const updateMutation = useMutation({
        mutationFn: (updates) => myFeatureApi.update(id, updates),

        onSuccess: () => {
            queryClient.invalidateQueries({ queryKey: ['entity', id] });
            showSuccess('Entity updated successfully');
        },

        onError: () => {
            showError('Failed to update entity');
        },
    });

    return (
        <Button onClick={() => updateMutation.mutate({ name: 'New' })}>
            Update
        </Button>
    );
};
```

---

## Loading State 안티패턴

### ❌ 하지 말아야 할 것

```typescript
// ❌ 절대 금지 - Early return
if (isLoading) {
    return <CircularProgress />;
}

// ❌ 절대 금지 - 조건부 렌더링
{isLoading ? <Spinner /> : <Content />}

// ❌ 절대 금지 - 레이아웃 변경
if (isLoading) {
    return (
        <Box sx={{ height: 100 }}>
            <Spinner />
        </Box>
    );
}
return (
    <Box sx={{ height: 500 }}>  // 다른 높이!
        <Content />
    </Box>
);
```

### ✅ 해야 할 것

```typescript
// ✅ 최선 - useSuspenseQuery + SuspenseLoader
<SuspenseLoader>
    <ComponentWithSuspenseQuery />
</SuspenseLoader>

// ✅ 수용 가능 - LoadingOverlay
<LoadingOverlay loading={isLoading}>
    <Content />
</LoadingOverlay>

// ✅ OK - 같은 레이아웃의 인라인 skeleton
<Box sx={{ height: 500 }}>
    {isLoading ? <Skeleton variant='rectangular' height='100%' /> : <Content />}
</Box>
```

---

## Skeleton Loading (대안)

### MUI Skeleton 컴포넌트

```typescript
import { Skeleton, Box } from '@mui/material';

export const MyComponent: React.FC = () => {
    const { data, isLoading } = useQuery({ ... });

    return (
        <Box sx={{ p: 2 }}>
            {isLoading ? (
                <>
                    <Skeleton variant='text' width={200} height={40} />
                    <Skeleton variant='rectangular' width='100%' height={200} />
                    <Skeleton variant='text' width='100%' />
                </>
            ) : (
                <>
                    <Typography variant='h5'>{data.title}</Typography>
                    <img src={data.image} />
                    <Typography>{data.description}</Typography>
                </>
            )}
        </Box>
    );
};
```

**핵심**: Skeleton은 실제 콘텐츠와 **같은 레이아웃**이어야 함 (이동 없음)

---

## 요약

**Loading States:**
- ✅ **권장**: SuspenseLoader + useSuspenseQuery (최신 패턴)
- ✅ **수용 가능**: LoadingOverlay (레거시 패턴)
- ✅ **OK**: 같은 레이아웃의 Skeleton
- ❌ **절대 금지**: Early return 또는 조건부 레이아웃

**에러 처리:**
- ✅ **항상**: 사용자 피드백에 useMuiSnackbar
- ❌ **절대 금지**: react-toastify
- ✅ queries/mutations에서 onError 콜백 사용
- ✅ 컴포넌트 레벨 에러에 Error boundaries

**참고:**
- [component-patterns.md](component-patterns.md) - Suspense 통합
- [data-fetching.md](data-fetching.md) - useSuspenseQuery 세부사항
