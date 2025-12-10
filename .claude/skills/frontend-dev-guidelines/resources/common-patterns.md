# 공통 패턴

forms, 인증, DataGrid, dialogs 및 기타 일반 UI 요소에 자주 사용되는 패턴입니다.

---

## useAuth를 통한 인증

### 현재 사용자 가져오기

```typescript
import { useAuth } from '@/hooks/useAuth';

export const MyComponent: React.FC = () => {
    const { user } = useAuth();

    // 사용 가능한 속성:
    // - user.id: string
    // - user.email: string
    // - user.username: string
    // - user.roles: string[]

    return (
        <div>
            <p>Logged in as: {user.email}</p>
            <p>Username: {user.username}</p>
            <p>Roles: {user.roles.join(', ')}</p>
        </div>
    );
};
```

**인증을 위해 직접 API 호출하지 마세요** - 항상 `useAuth` hook을 사용하세요.

---

## React Hook Form을 사용한 Forms

### 기본 Form

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { TextField, Button } from '@mui/material';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

// 유효성 검사를 위한 Zod 스키마
const formSchema = z.object({
    username: z.string().min(3, 'Username must be at least 3 characters'),
    email: z.string().email('Invalid email address'),
    age: z.number().min(18, 'Must be 18 or older'),
});

type FormData = z.infer<typeof formSchema>;

export const MyForm: React.FC = () => {
    const { showSuccess, showError } = useMuiSnackbar();

    const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
        resolver: zodResolver(formSchema),
        defaultValues: {
            username: '',
            email: '',
            age: 18,
        },
    });

    const onSubmit = async (data: FormData) => {
        try {
            await api.submitForm(data);
            showSuccess('Form submitted successfully');
        } catch (error) {
            showError('Failed to submit form');
        }
    };

    return (
        <form onSubmit={handleSubmit(onSubmit)}>
            <TextField
                {...register('username')}
                label='Username'
                error={!!errors.username}
                helperText={errors.username?.message}
            />

            <TextField
                {...register('email')}
                label='Email'
                error={!!errors.email}
                helperText={errors.email?.message}
                type='email'
            />

            <TextField
                {...register('age', { valueAsNumber: true })}
                label='Age'
                error={!!errors.age}
                helperText={errors.age?.message}
                type='number'
            />

            <Button type='submit' variant='contained'>
                Submit
            </Button>
        </form>
    );
};
```

---

## Dialog 컴포넌트 패턴

### 표준 Dialog 구조

BEST_PRACTICES.md에서 - 모든 dialogs는 다음을 포함해야 합니다:
- 제목에 아이콘
- 닫기 버튼 (X)
- 하단에 액션 버튼

```typescript
import { Dialog, DialogTitle, DialogContent, DialogActions, Button, IconButton } from '@mui/material';
import { Close, Info } from '@mui/icons-material';

interface MyDialogProps {
    open: boolean;
    onClose: () => void;
    onConfirm: () => void;
}

export const MyDialog: React.FC<MyDialogProps> = ({ open, onClose, onConfirm }) => {
    return (
        <Dialog open={open} onClose={onClose} maxWidth='sm' fullWidth>
            <DialogTitle>
                <Box sx={{ display: 'flex', alignItems: 'center', justifyContent: 'space-between' }}>
                    <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
                        <Info color='primary' />
                        Dialog Title
                    </Box>
                    <IconButton onClick={onClose} size='small'>
                        <Close />
                    </IconButton>
                </Box>
            </DialogTitle>

            <DialogContent>
                {/* 콘텐츠 */}
            </DialogContent>

            <DialogActions>
                <Button onClick={onClose}>Cancel</Button>
                <Button onClick={onConfirm} variant='contained'>
                    Confirm
                </Button>
            </DialogActions>
        </Dialog>
    );
};
```

---

## DataGrid Wrapper 패턴

### Wrapper 컴포넌트 계약

BEST_PRACTICES.md에서 - DataGrid wrappers는 다음을 받아야 합니다:

**필수 Props:**
- `rows`: 데이터 배열
- `columns`: 컬럼 정의
- Loading/error 상태

**선택적 Props:**
- Toolbar 컴포넌트
- 커스텀 액션
- 초기 상태

```typescript
import { DataGridPro } from '@mui/x-data-grid-pro';
import type { GridColDef } from '@mui/x-data-grid-pro';

interface DataGridWrapperProps {
    rows: any[];
    columns: GridColDef[];
    loading?: boolean;
    toolbar?: React.ReactNode;
    onRowClick?: (row: any) => void;
}

export const DataGridWrapper: React.FC<DataGridWrapperProps> = ({
    rows,
    columns,
    loading = false,
    toolbar,
    onRowClick,
}) => {
    return (
        <DataGridPro
            rows={rows}
            columns={columns}
            loading={loading}
            slots={{ toolbar: toolbar ? () => toolbar : undefined }}
            onRowClick={(params) => onRowClick?.(params.row)}
            // 표준 설정
            pagination
            pageSizeOptions={[25, 50, 100]}
            initialState={{
                pagination: { paginationModel: { pageSize: 25 } },
            }}
        />
    );
};
```

---

## Mutation 패턴

### 캐시 Invalidation이 있는 Update

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

export const useUpdateEntity = () => {
    const queryClient = useQueryClient();
    const { showSuccess, showError } = useMuiSnackbar();

    return useMutation({
        mutationFn: ({ id, data }: { id: number; data: any }) =>
            api.updateEntity(id, data),

        onSuccess: (result, variables) => {
            // 영향받는 쿼리 invalidate
            queryClient.invalidateQueries({ queryKey: ['entity', variables.id] });
            queryClient.invalidateQueries({ queryKey: ['entities'] });

            showSuccess('Entity updated');
        },

        onError: () => {
            showError('Failed to update entity');
        },
    });
};

// 사용
const updateEntity = useUpdateEntity();

const handleSave = () => {
    updateEntity.mutate({ id: 123, data: { name: 'New Name' } });
};
```

---

## 상태 관리 패턴

### 서버 상태를 위한 TanStack Query (주요)

**모든 서버 데이터**에 TanStack Query 사용:
- Fetching: useSuspenseQuery
- Mutations: useMutation
- 캐싱: 자동
- 동기화: 내장

```typescript
// ✅ 올바름 - 서버 데이터에 TanStack Query
const { data: users } = useSuspenseQuery({
    queryKey: ['users'],
    queryFn: () => userApi.getUsers(),
});
```

### UI 상태를 위한 useState

**로컬 UI 상태에만** `useState` 사용:
- Form 입력 (uncontrolled)
- Modal 열림/닫힘
- 선택된 탭
- 임시 UI 플래그

```typescript
// ✅ 올바름 - UI 상태에 useState
const [modalOpen, setModalOpen] = useState(false);
const [selectedTab, setSelectedTab] = useState(0);
```

### 전역 클라이언트 상태를 위한 Zustand (최소한으로)

**전역 클라이언트 상태에만** Zustand 사용:
- 테마 선호
- 사이드바 접힘 상태
- 사용자 선호 (서버에서 오지 않는)

```typescript
import { create } from 'zustand';

interface AppState {
    sidebarOpen: boolean;
    toggleSidebar: () => void;
}

export const useAppState = create<AppState>((set) => ({
    sidebarOpen: true,
    toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
}));
```

**prop drilling 피하기** - 대신 context나 Zustand 사용.

---

## 요약

**공통 패턴:**
- ✅ 현재 사용자를 위한 useAuth hook (id, email, roles, username)
- ✅ forms를 위한 React Hook Form + Zod
- ✅ 아이콘 + 닫기 버튼이 있는 Dialog
- ✅ DataGrid wrapper 계약
- ✅ 캐시 invalidation이 있는 Mutations
- ✅ 서버 상태를 위한 TanStack Query
- ✅ UI 상태를 위한 useState
- ✅ 전역 클라이언트 상태를 위한 Zustand (최소한으로)

**참고:**
- [data-fetching.md](data-fetching.md) - TanStack Query 패턴
- [component-patterns.md](component-patterns.md) - 컴포넌트 구조
- [loading-and-error-states.md](loading-and-error-states.md) - 에러 처리
