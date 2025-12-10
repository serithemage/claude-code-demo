# 컴포넌트 패턴

타입 안전성, lazy loading, Suspense boundaries를 강조하는 애플리케이션의 최신 React 컴포넌트 아키텍처입니다.

---

## React.FC 패턴 (권장)

### React.FC를 사용하는 이유

모든 컴포넌트는 다음을 위해 `React.FC<Props>` 패턴을 사용합니다:
- props에 대한 명시적 타입 안전성
- 일관된 컴포넌트 시그니처
- 명확한 prop 인터페이스 문서화
- 더 나은 IDE 자동완성

### 기본 패턴

```typescript
import React from 'react';

interface MyComponentProps {
    /** 표시할 사용자 ID */
    userId: number;
    /** 액션 발생 시 선택적 콜백 */
    onAction?: () => void;
}

export const MyComponent: React.FC<MyComponentProps> = ({ userId, onAction }) => {
    return (
        <div>
            User: {userId}
        </div>
    );
};

export default MyComponent;
```

**핵심 포인트:**
- JSDoc 주석이 있는 별도의 Props 인터페이스 정의
- `React.FC<Props>`가 타입 안전성 제공
- 파라미터에서 props 구조분해
- 하단에 default export

---

## Lazy Loading 패턴

### Lazy Load해야 할 때

다음과 같은 컴포넌트는 lazy load합니다:
- 무거운 것 (DataGrid, 차트, 리치 텍스트 에디터)
- Route 레벨 컴포넌트
- Modal/dialog 콘텐츠 (초기에 표시되지 않음)
- Below-the-fold 콘텐츠

### Lazy Load 방법

```typescript
import React from 'react';

// 무거운 컴포넌트 lazy load
const PostDataGrid = React.lazy(() =>
    import('./grids/PostDataGrid')
);

// named exports의 경우
const MyComponent = React.lazy(() =>
    import('./MyComponent').then(module => ({
        default: module.MyComponent
    }))
);
```

**PostTable.tsx의 예시:**

```typescript
/**
 * 메인 post table 컨테이너 컴포넌트
 */
import React, { useState, useCallback } from 'react';
import { Box, Paper } from '@mui/material';

// 번들 크기 최적화를 위해 PostDataGrid lazy load
const PostDataGrid = React.lazy(() => import('./grids/PostDataGrid'));

import { SuspenseLoader } from '~components/SuspenseLoader';

export const PostTable: React.FC<PostTableProps> = ({ formId }) => {
    return (
        <Box>
            <SuspenseLoader>
                <PostDataGrid formId={formId} />
            </SuspenseLoader>
        </Box>
    );
};

export default PostTable;
```

---

## Suspense Boundaries

### SuspenseLoader 컴포넌트

**Import:**
```typescript
import { SuspenseLoader } from '~components/SuspenseLoader';
// 또는
import { SuspenseLoader } from '@/components/SuspenseLoader';
```

**사용:**
```typescript
<SuspenseLoader>
    <LazyLoadedComponent />
</SuspenseLoader>
```

**기능:**
- lazy 컴포넌트 로드 중 로딩 인디케이터 표시
- 부드러운 페이드인 애니메이션
- 일관된 로딩 경험
- 레이아웃 이동 방지

### Suspense Boundaries 배치 위치

**Route 레벨:**
```typescript
// routes/my-route/index.tsx
const MyPage = lazy(() => import('@/features/my-feature/components/MyPage'));

function Route() {
    return (
        <SuspenseLoader>
            <MyPage />
        </SuspenseLoader>
    );
}
```

**컴포넌트 레벨:**
```typescript
function ParentComponent() {
    return (
        <Box>
            <Header />
            <SuspenseLoader>
                <HeavyDataGrid />
            </SuspenseLoader>
        </Box>
    );
}
```

**여러 Boundaries:**
```typescript
function Page() {
    return (
        <Box>
            <SuspenseLoader>
                <HeaderSection />
            </SuspenseLoader>

            <SuspenseLoader>
                <MainContent />
            </SuspenseLoader>

            <SuspenseLoader>
                <Sidebar />
            </SuspenseLoader>
        </Box>
    );
}
```

각 섹션이 독립적으로 로드되어 더 나은 UX.

---

## 컴포넌트 구조 템플릿

### 권장 순서

```typescript
/**
 * 컴포넌트 설명
 * 기능, 사용 시점
 */
import React, { useState, useCallback, useMemo, useEffect } from 'react';
import { Box, Paper, Button } from '@mui/material';
import type { SxProps, Theme } from '@mui/material';
import { useSuspenseQuery } from '@tanstack/react-query';

// Feature imports
import { myFeatureApi } from '../api/myFeatureApi';
import type { MyData } from '~types/myData';

// 컴포넌트 imports
import { SuspenseLoader } from '~components/SuspenseLoader';

// Hooks
import { useAuth } from '@/hooks/useAuth';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

// 1. PROPS 인터페이스 (JSDoc과 함께)
interface MyComponentProps {
    /** 표시할 entity의 ID */
    entityId: number;
    /** 액션 완료 시 선택적 콜백 */
    onComplete?: () => void;
    /** 표시 모드 */
    mode?: 'view' | 'edit';
}

// 2. 스타일 (인라인이고 100줄 미만인 경우)
const componentStyles: Record<string, SxProps<Theme>> = {
    container: {
        p: 2,
        display: 'flex',
        flexDirection: 'column',
    },
    header: {
        mb: 2,
        display: 'flex',
        justifyContent: 'space-between',
    },
};

// 3. 컴포넌트 정의
export const MyComponent: React.FC<MyComponentProps> = ({
    entityId,
    onComplete,
    mode = 'view',
}) => {
    // 4. HOOKS (이 순서로)
    // - Context hooks 먼저
    const { user } = useAuth();
    const { showSuccess, showError } = useMuiSnackbar();

    // - 데이터 fetching
    const { data } = useSuspenseQuery({
        queryKey: ['myEntity', entityId],
        queryFn: () => myFeatureApi.getEntity(entityId),
    });

    // - 로컬 상태
    const [selectedItem, setSelectedItem] = useState<string | null>(null);
    const [isEditing, setIsEditing] = useState(mode === 'edit');

    // - Memoized 값
    const filteredData = useMemo(() => {
        return data.filter(item => item.active);
    }, [data]);

    // - Effects
    useEffect(() => {
        // 설정
        return () => {
            // 정리
        };
    }, []);

    // 5. 이벤트 핸들러 (useCallback과 함께)
    const handleItemSelect = useCallback((itemId: string) => {
        setSelectedItem(itemId);
    }, []);

    const handleSave = useCallback(async () => {
        try {
            await myFeatureApi.updateEntity(entityId, { /* data */ });
            showSuccess('Entity updated successfully');
            onComplete?.();
        } catch (error) {
            showError('Failed to update entity');
        }
    }, [entityId, onComplete, showSuccess, showError]);

    // 6. 렌더
    return (
        <Box sx={componentStyles.container}>
            <Box sx={componentStyles.header}>
                <h2>My Component</h2>
                <Button onClick={handleSave}>Save</Button>
            </Box>

            <Paper sx={{ p: 2 }}>
                {filteredData.map(item => (
                    <div key={item.id}>{item.name}</div>
                ))}
            </Paper>
        </Box>
    );
};

// 7. EXPORT (하단에 default export)
export default MyComponent;
```

---

## 컴포넌트 분리

### 컴포넌트를 분리해야 할 때

**여러 컴포넌트로 분리해야 할 때:**
- 컴포넌트가 300줄 초과
- 여러 개의 구분된 책임
- 재사용 가능한 섹션
- 복잡한 중첩 JSX

**예시:**

```typescript
// ❌ 피하세요 - 모놀리식
function MassiveComponent() {
    // 500줄 이상
    // 검색 로직
    // 필터 로직
    // 그리드 로직
    // 액션 패널 로직
}

// ✅ 권장 - 모듈형
function ParentContainer() {
    return (
        <Box>
            <SearchAndFilter onFilter={handleFilter} />
            <DataGrid data={filteredData} />
            <ActionPanel onAction={handleAction} />
        </Box>
    );
}
```

### 함께 유지해야 할 때

**같은 파일에 유지해야 할 때:**
- 컴포넌트 200줄 미만
- 밀접하게 결합된 로직
- 다른 곳에서 재사용 불가
- 단순 프레젠테이션 컴포넌트

---

## Export 패턴

### Named Const + Default Export (권장)

```typescript
export const MyComponent: React.FC<Props> = ({ ... }) => {
    // 컴포넌트 로직
};

export default MyComponent;
```

**이유:**
- 테스트/리팩토링을 위한 named export
- lazy loading 편의를 위한 default export
- 소비자에게 두 옵션 모두 제공

### Named Exports Lazy Loading

```typescript
const MyComponent = React.lazy(() =>
    import('./MyComponent').then(module => ({
        default: module.MyComponent
    }))
);
```

---

## 컴포넌트 통신

### Props Down, Events Up

```typescript
// 부모
function Parent() {
    const [selectedId, setSelectedId] = useState<string | null>(null);

    return (
        <Child
            data={data}                    // Props down
            onSelect={setSelectedId}       // Events up
        />
    );
}

// 자식
interface ChildProps {
    data: Data[];
    onSelect: (id: string) => void;
}

export const Child: React.FC<ChildProps> = ({ data, onSelect }) => {
    return (
        <div onClick={() => onSelect(data[0].id)}>
            {/* 콘텐츠 */}
        </div>
    );
};
```

### Prop Drilling 피하기

**깊은 중첩에는 context 사용:**
```typescript
// ❌ 피하세요 - 5단계 이상 prop drilling
<A prop={x}>
  <B prop={x}>
    <C prop={x}>
      <D prop={x}>
        <E prop={x} />  // 결국 여기서 사용
      </D>
    </C>
  </B>
</A>

// ✅ 권장 - Context 또는 TanStack Query
const MyContext = createContext<MyData | null>(null);

function Provider({ children }) {
    const { data } = useSuspenseQuery({ ... });
    return <MyContext.Provider value={data}>{children}</MyContext.Provider>;
}

function DeepChild() {
    const data = useContext(MyContext);
    // 직접 data 사용
}
```

---

## 고급 패턴

### Compound Components

```typescript
// Card.tsx
export const Card: React.FC<CardProps> & {
    Header: typeof CardHeader;
    Body: typeof CardBody;
    Footer: typeof CardFooter;
} = ({ children }) => {
    return <Paper>{children}</Paper>;
};

Card.Header = CardHeader;
Card.Body = CardBody;
Card.Footer = CardFooter;

// 사용
<Card>
    <Card.Header>Title</Card.Header>
    <Card.Body>Content</Card.Body>
    <Card.Footer>Actions</Card.Footer>
</Card>
```

### Render Props (드물지만 유용)

```typescript
interface DataProviderProps {
    children: (data: Data) => React.ReactNode;
}

export const DataProvider: React.FC<DataProviderProps> = ({ children }) => {
    const { data } = useSuspenseQuery({ ... });
    return <>{children(data)}</>;
};

// 사용
<DataProvider>
    {(data) => <Display data={data} />}
</DataProvider>
```

---

## 요약

**최신 컴포넌트 레시피:**
1. TypeScript와 함께 `React.FC<Props>`
2. 무거우면 lazy load: `React.lazy(() => import())`
3. 로딩을 위해 `<SuspenseLoader>`로 감싸기
4. 데이터에 `useSuspenseQuery` 사용
5. Import 별칭 (@/, ~types, ~components)
6. `useCallback`과 함께 이벤트 핸들러
7. 하단에 default export
8. 로딩 상태에서 early returns 금지

**참고:**
- [data-fetching.md](data-fetching.md) - useSuspenseQuery 세부사항
- [loading-and-error-states.md](loading-and-error-states.md) - Suspense 모범 사례
- [complete-examples.md](complete-examples.md) - 전체 작동 예제
