# 성능 최적화

React 컴포넌트 성능 최적화, 불필요한 리렌더링 방지, 메모리 누수 방지를 위한 패턴입니다.

---

## Memoization 패턴

### 비싼 계산에 useMemo 사용

```typescript
import { useMemo } from 'react';

export const DataDisplay: React.FC<{ items: Item[], searchTerm: string }> = ({
    items,
    searchTerm,
}) => {
    // ❌ 피하세요 - 매 렌더마다 실행
    const filteredItems = items
        .filter(item => item.name.includes(searchTerm))
        .sort((a, b) => a.name.localeCompare(b.name));

    // ✅ 올바름 - Memoized, 의존성 변경 시에만 재계산
    const filteredItems = useMemo(() => {
        return items
            .filter(item => item.name.toLowerCase().includes(searchTerm.toLowerCase()))
            .sort((a, b) => a.name.localeCompare(b.name));
    }, [items, searchTerm]);

    return <List items={filteredItems} />;
};
```

**useMemo를 사용해야 할 때:**
- 대규모 배열 필터링/정렬
- 복잡한 계산
- 데이터 구조 변환
- 비싼 연산 (루프, 재귀)

**useMemo를 사용하지 않아야 할 때:**
- 간단한 문자열 연결
- 기본 산술
- 조기 최적화 (먼저 프로파일링하세요!)

---

## 이벤트 핸들러에 useCallback 사용

### 문제

```typescript
// ❌ 피하세요 - 매 렌더마다 새 함수 생성
export const Parent: React.FC = () => {
    const handleClick = (id: string) => {
        console.log('Clicked:', id);
    };

    // Parent가 렌더될 때마다 Child가 리렌더됨
    // handleClick이 매번 새 함수 참조이기 때문
    return <Child onClick={handleClick} />;
};
```

### 해결책

```typescript
import { useCallback } from 'react';

export const Parent: React.FC = () => {
    // ✅ 올바름 - 안정적인 함수 참조
    const handleClick = useCallback((id: string) => {
        console.log('Clicked:', id);
    }, []); // 빈 deps = 함수가 절대 변경되지 않음

    // props가 실제로 변경될 때만 Child 리렌더
    return <Child onClick={handleClick} />;
};
```

**useCallback을 사용해야 할 때:**
- 자식에게 props로 전달되는 함수
- useEffect의 의존성으로 사용되는 함수
- memoized 컴포넌트에 전달되는 함수
- 리스트의 이벤트 핸들러

**useCallback을 사용하지 않아야 할 때:**
- 자식에게 전달되지 않는 이벤트 핸들러
- 간단한 인라인 핸들러: `onClick={() => doSomething()}`

---

## 컴포넌트 Memoization에 React.memo 사용

### 기본 사용법

```typescript
import React from 'react';

interface ExpensiveComponentProps {
    data: ComplexData;
    onAction: () => void;
}

// ✅ 비싼 컴포넌트를 React.memo로 감싸기
export const ExpensiveComponent = React.memo<ExpensiveComponentProps>(
    function ExpensiveComponent({ data, onAction }) {
        // 복잡한 렌더링 로직
        return <ComplexVisualization data={data} />;
    }
);
```

**React.memo를 사용해야 할 때:**
- 자주 렌더되는 컴포넌트
- 비싼 렌더링이 있는 컴포넌트
- props가 자주 변경되지 않을 때
- 리스트 아이템 컴포넌트
- DataGrid 셀/렌더러

**React.memo를 사용하지 않아야 할 때:**
- props가 어차피 자주 변경될 때
- 렌더링이 이미 빠를 때
- 조기 최적화

---

## Debounced 검색

### use-debounce Hook 사용

```typescript
import { useState } from 'react';
import { useDebounce } from 'use-debounce';
import { useSuspenseQuery } from '@tanstack/react-query';

export const SearchComponent: React.FC = () => {
    const [searchTerm, setSearchTerm] = useState('');

    // 300ms 동안 debounce
    const [debouncedSearchTerm] = useDebounce(searchTerm, 300);

    // 쿼리는 debounced 값 사용
    const { data } = useSuspenseQuery({
        queryKey: ['search', debouncedSearchTerm],
        queryFn: () => api.search(debouncedSearchTerm),
        enabled: debouncedSearchTerm.length > 0,
    });

    return (
        <input
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
            placeholder='Search...'
        />
    );
};
```

**최적 Debounce 타이밍:**
- **300-500ms**: 검색/필터링
- **1000ms**: 자동 저장
- **100-200ms**: 실시간 유효성 검사

---

## 메모리 누수 방지

### Timeout/Interval 정리

```typescript
import { useEffect, useState } from 'react';

export const MyComponent: React.FC = () => {
    const [count, setCount] = useState(0);

    useEffect(() => {
        // ✅ 올바름 - Interval 정리
        const intervalId = setInterval(() => {
            setCount(c => c + 1);
        }, 1000);

        return () => {
            clearInterval(intervalId);  // 정리!
        };
    }, []);

    useEffect(() => {
        // ✅ 올바름 - Timeout 정리
        const timeoutId = setTimeout(() => {
            console.log('Delayed action');
        }, 5000);

        return () => {
            clearTimeout(timeoutId);  // 정리!
        };
    }, []);

    return <div>{count}</div>;
};
```

### 이벤트 리스너 정리

```typescript
useEffect(() => {
    const handleResize = () => {
        console.log('Resized');
    };

    window.addEventListener('resize', handleResize);

    return () => {
        window.removeEventListener('resize', handleResize);  // 정리!
    };
}, []);
```

### Fetch에 Abort Controllers 사용

```typescript
useEffect(() => {
    const abortController = new AbortController();

    fetch('/api/data', { signal: abortController.signal })
        .then(response => response.json())
        .then(data => setState(data))
        .catch(error => {
            if (error.name === 'AbortError') {
                console.log('Fetch aborted');
            }
        });

    return () => {
        abortController.abort();  // 정리!
    };
}, []);
```

**참고**: TanStack Query 사용 시 이것은 자동으로 처리됩니다.

---

## 폼 성능

### 특정 필드만 Watch (전체 아님)

```typescript
import { useForm } from 'react-hook-form';

export const MyForm: React.FC = () => {
    const { register, watch, handleSubmit } = useForm();

    // ❌ 피하세요 - 모든 필드 watch, 모든 변경에 리렌더
    const formValues = watch();

    // ✅ 올바름 - 필요한 것만 watch
    const username = watch('username');
    const email = watch('email');

    // 또는 여러 특정 필드
    const [username, email] = watch(['username', 'email']);

    return (
        <form onSubmit={handleSubmit(onSubmit)}>
            <input {...register('username')} />
            <input {...register('email')} />
            <input {...register('password')} />

            {/* username/email 변경 시에만 리렌더 */}
            <p>Username: {username}, Email: {email}</p>
        </form>
    );
};
```

---

## 리스트 렌더링 최적화

### Key Prop 사용

```typescript
// ✅ 올바름 - 안정적인 고유 키
{items.map(item => (
    <ListItem key={item.id}>
        {item.name}
    </ListItem>
))}

// ❌ 피하세요 - 인덱스를 키로 (리스트가 변경되면 불안정)
{items.map((item, index) => (
    <ListItem key={index}>  // 리스트 순서 변경 시 잘못됨
        {item.name}
    </ListItem>
))}
```

### Memoized 리스트 아이템

```typescript
const ListItem = React.memo<ListItemProps>(({ item, onAction }) => {
    return (
        <Box onClick={() => onAction(item.id)}>
            {item.name}
        </Box>
    );
});

export const List: React.FC<{ items: Item[] }> = ({ items }) => {
    const handleAction = useCallback((id: string) => {
        console.log('Action:', id);
    }, []);

    return (
        <Box>
            {items.map(item => (
                <ListItem
                    key={item.id}
                    item={item}
                    onAction={handleAction}
                />
            ))}
        </Box>
    );
};
```

---

## 컴포넌트 재초기화 방지

### 문제

```typescript
// ❌ 피하세요 - 매 렌더마다 컴포넌트 재생성
export const Parent: React.FC = () => {
    // 매 렌더마다 새 컴포넌트 정의!
    const ChildComponent = () => <div>Child</div>;

    return <ChildComponent />;  // 매 렌더마다 언마운트하고 리마운트
};
```

### 해결책

```typescript
// ✅ 올바름 - 외부에 정의하거나 useMemo 사용
const ChildComponent: React.FC = () => <div>Child</div>;

export const Parent: React.FC = () => {
    return <ChildComponent />;  // 안정적인 컴포넌트
};

// ✅ 또는 동적인 경우 useMemo 사용
export const Parent: React.FC<{ config: Config }> = ({ config }) => {
    const DynamicComponent = useMemo(() => {
        return () => <div>{config.title}</div>;
    }, [config.title]);

    return <DynamicComponent />;
};
```

---

## 무거운 의존성 Lazy Loading

### 코드 스플리팅

```typescript
// ❌ 피하세요 - 최상위에서 무거운 라이브러리 import
import jsPDF from 'jspdf';  // 큰 라이브러리가 즉시 로드됨
import * as XLSX from 'xlsx';  // 큰 라이브러리가 즉시 로드됨

// ✅ 올바름 - 필요할 때 동적 import
const handleExportPDF = async () => {
    const { jsPDF } = await import('jspdf');
    const doc = new jsPDF();
    // 사용
};

const handleExportExcel = async () => {
    const XLSX = await import('xlsx');
    // 사용
};
```

---

## 요약

**성능 체크리스트:**
- ✅ 비싼 계산에 `useMemo` (filter, sort, map)
- ✅ 자식에게 전달되는 함수에 `useCallback`
- ✅ 비싼 컴포넌트에 `React.memo`
- ✅ 검색/필터 debounce (300-500ms)
- ✅ useEffect에서 timeout/interval 정리
- ✅ 특정 폼 필드만 watch (전체 아님)
- ✅ 리스트에 안정적인 키
- ✅ 무거운 라이브러리 lazy load
- ✅ React.lazy로 코드 스플리팅

**참고:**
- [component-patterns.md](component-patterns.md) - Lazy loading
- [data-fetching.md](data-fetching.md) - TanStack Query 최적화
- [complete-examples.md](complete-examples.md) - 컨텍스트 내 성능 패턴
