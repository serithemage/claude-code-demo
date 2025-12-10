# 완전한 예제

React.FC, lazy loading, Suspense, useSuspenseQuery, 스타일링, 라우팅, 에러 처리 등 모든 최신 패턴을 결합한 전체 작동 예제입니다.

---

## 예제 1: 완전한 최신 컴포넌트

결합: React.FC, useSuspenseQuery, cache-first, useCallback, 스타일링, 에러 처리

```typescript
/**
 * 사용자 프로필 표시 컴포넌트
 * Suspense와 TanStack Query를 사용한 최신 패턴 시연
 */
import React, { useState, useCallback, useMemo } from 'react';
import { Box, Paper, Typography, Button, Avatar } from '@mui/material';
import type { SxProps, Theme } from '@mui/material';
import { useSuspenseQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userApi } from '../api/userApi';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';
import type { User } from '~types/user';

// 스타일 객체
const componentStyles: Record<string, SxProps<Theme>> = {
    container: {
        p: 3,
        maxWidth: 600,
        margin: '0 auto',
    },
    header: {
        display: 'flex',
        alignItems: 'center',
        gap: 2,
        mb: 3,
    },
    content: {
        display: 'flex',
        flexDirection: 'column',
        gap: 2,
    },
    actions: {
        display: 'flex',
        gap: 1,
        mt: 2,
    },
};

interface UserProfileProps {
    userId: string;
    onUpdate?: () => void;
}

export const UserProfile: React.FC<UserProfileProps> = ({ userId, onUpdate }) => {
    const queryClient = useQueryClient();
    const { showSuccess, showError } = useMuiSnackbar();
    const [isEditing, setIsEditing] = useState(false);

    // Suspense 쿼리 - isLoading 불필요!
    const { data: user } = useSuspenseQuery({
        queryKey: ['user', userId],
        queryFn: () => userApi.getUser(userId),
        staleTime: 5 * 60 * 1000,
    });

    // Update mutation
    const updateMutation = useMutation({
        mutationFn: (updates: Partial<User>) =>
            userApi.updateUser(userId, updates),

        onSuccess: () => {
            queryClient.invalidateQueries({ queryKey: ['user', userId] });
            showSuccess('Profile updated');
            setIsEditing(false);
            onUpdate?.();
        },

        onError: () => {
            showError('Failed to update profile');
        },
    });

    // Memoized 계산 값
    const fullName = useMemo(() => {
        return `${user.firstName} ${user.lastName}`;
    }, [user.firstName, user.lastName]);

    // useCallback을 사용한 이벤트 핸들러
    const handleEdit = useCallback(() => {
        setIsEditing(true);
    }, []);

    const handleSave = useCallback(() => {
        updateMutation.mutate({
            firstName: user.firstName,
            lastName: user.lastName,
        });
    }, [user, updateMutation]);

    const handleCancel = useCallback(() => {
        setIsEditing(false);
    }, []);

    return (
        <Paper sx={componentStyles.container}>
            <Box sx={componentStyles.header}>
                <Avatar sx={{ width: 64, height: 64 }}>
                    {user.firstName[0]}{user.lastName[0]}
                </Avatar>
                <Box>
                    <Typography variant='h5'>{fullName}</Typography>
                    <Typography color='text.secondary'>{user.email}</Typography>
                </Box>
            </Box>

            <Box sx={componentStyles.content}>
                <Typography>Username: {user.username}</Typography>
                <Typography>Roles: {user.roles.join(', ')}</Typography>
            </Box>

            <Box sx={componentStyles.actions}>
                {!isEditing ? (
                    <Button variant='contained' onClick={handleEdit}>
                        Edit Profile
                    </Button>
                ) : (
                    <>
                        <Button
                            variant='contained'
                            onClick={handleSave}
                            disabled={updateMutation.isPending}
                        >
                            {updateMutation.isPending ? 'Saving...' : 'Save'}
                        </Button>
                        <Button onClick={handleCancel}>
                            Cancel
                        </Button>
                    </>
                )}
            </Box>
        </Paper>
    );
};

export default UserProfile;
```

**사용:**
```typescript
<SuspenseLoader>
    <UserProfile userId='123' onUpdate={() => console.log('Updated')} />
</SuspenseLoader>
```

---

## 예제 2: 완전한 Feature 구조

`features/posts/` 기반 실제 예시:

```
features/
  users/
    api/
      userApi.ts                # API 서비스 레이어
    components/
      UserProfile.tsx           # 메인 컴포넌트 (예제 1에서)
      UserList.tsx              # 리스트 컴포넌트
      UserForm.tsx              # 폼 컴포넌트
      modals/
        DeleteUserModal.tsx     # 모달 컴포넌트
    hooks/
      useSuspenseUser.ts        # Suspense 쿼리 hook
      useUserMutations.ts       # Mutation hooks
      useUserPermissions.ts     # Feature별 hook
    helpers/
      userHelpers.ts            # 유틸리티 함수
      validation.ts             # 유효성 검사 로직
    types/
      index.ts                  # TypeScript 인터페이스
    index.ts                    # Public API exports
```

### API 서비스 (userApi.ts)

```typescript
import apiClient from '@/lib/apiClient';
import type { User, CreateUserPayload, UpdateUserPayload } from '../types';

export const userApi = {
    getUser: async (userId: string): Promise<User> => {
        const { data } = await apiClient.get(`/users/${userId}`);
        return data;
    },

    getUsers: async (): Promise<User[]> => {
        const { data } = await apiClient.get('/users');
        return data;
    },

    createUser: async (payload: CreateUserPayload): Promise<User> => {
        const { data } = await apiClient.post('/users', payload);
        return data;
    },

    updateUser: async (userId: string, payload: UpdateUserPayload): Promise<User> => {
        const { data } = await apiClient.put(`/users/${userId}`, payload);
        return data;
    },

    deleteUser: async (userId: string): Promise<void> => {
        await apiClient.delete(`/users/${userId}`);
    },
};
```

### Suspense Hook (useSuspenseUser.ts)

```typescript
import { useSuspenseQuery } from '@tanstack/react-query';
import { userApi } from '../api/userApi';
import type { User } from '../types';

export function useSuspenseUser(userId: string) {
    return useSuspenseQuery<User, Error>({
        queryKey: ['user', userId],
        queryFn: () => userApi.getUser(userId),
        staleTime: 5 * 60 * 1000,
        gcTime: 10 * 60 * 1000,
    });
}

export function useSuspenseUsers() {
    return useSuspenseQuery<User[], Error>({
        queryKey: ['users'],
        queryFn: () => userApi.getUsers(),
        staleTime: 1 * 60 * 1000,  // 리스트에는 더 짧게
    });
}
```

### 타입 (types/index.ts)

```typescript
export interface User {
    id: string;
    username: string;
    email: string;
    firstName: string;
    lastName: string;
    roles: string[];
    createdAt: string;
    updatedAt: string;
}

export interface CreateUserPayload {
    username: string;
    email: string;
    firstName: string;
    lastName: string;
    password: string;
}

export type UpdateUserPayload = Partial<Omit<User, 'id' | 'createdAt' | 'updatedAt'>>;
```

### Public Exports (index.ts)

```typescript
// 컴포넌트 export
export { UserProfile } from './components/UserProfile';
export { UserList } from './components/UserList';

// Hooks export
export { useSuspenseUser, useSuspenseUsers } from './hooks/useSuspenseUser';
export { useUserMutations } from './hooks/useUserMutations';

// API export
export { userApi } from './api/userApi';

// 타입 export
export type { User, CreateUserPayload, UpdateUserPayload } from './types';
```

---

## 예제 3: Lazy Loading이 있는 완전한 Route

```typescript
/**
 * 사용자 프로필 route
 * 경로: /users/:userId
 */

import { createFileRoute } from '@tanstack/react-router';
import { lazy } from 'react';
import { SuspenseLoader } from '~components/SuspenseLoader';

// UserProfile 컴포넌트 lazy load
const UserProfile = lazy(() =>
    import('@/features/users/components/UserProfile').then(
        (module) => ({ default: module.UserProfile })
    )
);

export const Route = createFileRoute('/users/$userId')({
    component: UserProfilePage,
    loader: ({ params }) => ({
        crumb: `User ${params.userId}`,
    }),
});

function UserProfilePage() {
    const { userId } = Route.useParams();

    return (
        <SuspenseLoader>
            <UserProfile
                userId={userId}
                onUpdate={() => console.log('Profile updated')}
            />
        </SuspenseLoader>
    );
}

export default UserProfilePage;
```

---

## 예제 4: 검색과 필터링이 있는 리스트

```typescript
import React, { useState, useMemo } from 'react';
import { Box, TextField, List, ListItem } from '@mui/material';
import { useDebounce } from 'use-debounce';
import { useSuspenseQuery } from '@tanstack/react-query';
import { userApi } from '../api/userApi';

export const UserList: React.FC = () => {
    const [searchTerm, setSearchTerm] = useState('');
    const [debouncedSearch] = useDebounce(searchTerm, 300);

    const { data: users } = useSuspenseQuery({
        queryKey: ['users'],
        queryFn: () => userApi.getUsers(),
    });

    // Memoized 필터링
    const filteredUsers = useMemo(() => {
        if (!debouncedSearch) return users;

        return users.filter(user =>
            user.name.toLowerCase().includes(debouncedSearch.toLowerCase()) ||
            user.email.toLowerCase().includes(debouncedSearch.toLowerCase())
        );
    }, [users, debouncedSearch]);

    return (
        <Box>
            <TextField
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
                placeholder='Search users...'
                fullWidth
                sx={{ mb: 2 }}
            />

            <List>
                {filteredUsers.map(user => (
                    <ListItem key={user.id}>
                        {user.name} - {user.email}
                    </ListItem>
                ))}
            </List>
        </Box>
    );
};
```

---

## 예제 5: 유효성 검사가 있는 Form

```typescript
import React from 'react';
import { Box, TextField, Button, Paper } from '@mui/material';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { userApi } from '../api/userApi';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

const userSchema = z.object({
    username: z.string().min(3).max(50),
    email: z.string().email(),
    firstName: z.string().min(1),
    lastName: z.string().min(1),
});

type UserFormData = z.infer<typeof userSchema>;

interface CreateUserFormProps {
    onSuccess?: () => void;
}

export const CreateUserForm: React.FC<CreateUserFormProps> = ({ onSuccess }) => {
    const queryClient = useQueryClient();
    const { showSuccess, showError } = useMuiSnackbar();

    const { register, handleSubmit, formState: { errors }, reset } = useForm<UserFormData>({
        resolver: zodResolver(userSchema),
        defaultValues: {
            username: '',
            email: '',
            firstName: '',
            lastName: '',
        },
    });

    const createMutation = useMutation({
        mutationFn: (data: UserFormData) => userApi.createUser(data),

        onSuccess: () => {
            queryClient.invalidateQueries({ queryKey: ['users'] });
            showSuccess('User created successfully');
            reset();
            onSuccess?.();
        },

        onError: () => {
            showError('Failed to create user');
        },
    });

    const onSubmit = (data: UserFormData) => {
        createMutation.mutate(data);
    };

    return (
        <Paper sx={{ p: 3, maxWidth: 500 }}>
            <form onSubmit={handleSubmit(onSubmit)}>
                <Box sx={{ display: 'flex', flexDirection: 'column', gap: 2 }}>
                    <TextField
                        {...register('username')}
                        label='Username'
                        error={!!errors.username}
                        helperText={errors.username?.message}
                        fullWidth
                    />

                    <TextField
                        {...register('email')}
                        label='Email'
                        type='email'
                        error={!!errors.email}
                        helperText={errors.email?.message}
                        fullWidth
                    />

                    <TextField
                        {...register('firstName')}
                        label='First Name'
                        error={!!errors.firstName}
                        helperText={errors.firstName?.message}
                        fullWidth
                    />

                    <TextField
                        {...register('lastName')}
                        label='Last Name'
                        error={!!errors.lastName}
                        helperText={errors.lastName?.message}
                        fullWidth
                    />

                    <Button
                        type='submit'
                        variant='contained'
                        disabled={createMutation.isPending}
                    >
                        {createMutation.isPending ? 'Creating...' : 'Create User'}
                    </Button>
                </Box>
            </form>
        </Paper>
    );
};

export default CreateUserForm;
```

---

## 예제 6: Lazy Loading이 있는 부모 컨테이너

```typescript
import React from 'react';
import { Box } from '@mui/material';
import { SuspenseLoader } from '~components/SuspenseLoader';

// 무거운 컴포넌트 lazy load
const UserList = React.lazy(() => import('./UserList'));
const UserStats = React.lazy(() => import('./UserStats'));
const ActivityFeed = React.lazy(() => import('./ActivityFeed'));

export const UserDashboard: React.FC = () => {
    return (
        <Box sx={{ p: 2 }}>
            <SuspenseLoader>
                <UserStats />
            </SuspenseLoader>

            <Box sx={{ display: 'flex', gap: 2, mt: 2 }}>
                <Box sx={{ flex: 2 }}>
                    <SuspenseLoader>
                        <UserList />
                    </SuspenseLoader>
                </Box>

                <Box sx={{ flex: 1 }}>
                    <SuspenseLoader>
                        <ActivityFeed />
                    </SuspenseLoader>
                </Box>
            </Box>
        </Box>
    );
};

export default UserDashboard;
```

**장점:**
- 각 섹션이 독립적으로 로드
- 사용자가 부분 콘텐츠를 더 빨리 볼 수 있음
- 더 나은 체감 성능

---

## 예제 7: Cache-First 전략 구현

useSuspensePost.ts 기반 완전한 예제:

```typescript
import { useSuspenseQuery, useQueryClient } from '@tanstack/react-query';
import { postApi } from '../api/postApi';
import type { Post } from '../types';

/**
 * Cache-first 전략이 있는 스마트 post hook
 * 가능할 때 그리드 캐시의 데이터 재사용
 */
export function useSuspensePost(blogId: number, postId: number) {
    const queryClient = useQueryClient();

    return useSuspenseQuery<Post, Error>({
        queryKey: ['post', blogId, postId],
        queryFn: async () => {
            // 전략 1: 그리드 캐시 먼저 확인 (API 호출 방지)
            const gridCache = queryClient.getQueryData<{ rows: Post[] }>([
                'posts-v2',
                blogId,
                'summary'
            ]) || queryClient.getQueryData<{ rows: Post[] }>([
                'posts-v2',
                blogId,
                'flat'
            ]);

            if (gridCache?.rows) {
                const cached = gridCache.rows.find(
                    (row) => row.S_ID === postId
                );

                if (cached) {
                    return cached;  // 캐시에서 반환 - API 호출 없음!
                }
            }

            // 전략 2: 캐시에 없음, API에서 fetch
            return postApi.getPost(blogId, postId);
        },
        staleTime: 5 * 60 * 1000,       // 5분 동안 fresh
        gcTime: 10 * 60 * 1000,          // 10분 동안 캐시
        refetchOnWindowFocus: false,     // 포커스 시 refetch 안 함
    });
}
```

**이 패턴을 사용하는 이유:**
- API 전에 그리드 캐시 확인
- 사용자가 그리드에서 왔으면 즉시 데이터
- 캐시에 없으면 API로 fallback
- 설정 가능한 캐시 시간

---

## 예제 8: 완전한 Route 파일

```typescript
/**
 * Project catalog route
 * 경로: /project-catalog
 */

import { createFileRoute } from '@tanstack/react-router';
import { lazy } from 'react';

// PostTable 컴포넌트 lazy load
const PostTable = lazy(() =>
    import('@/features/posts/components/PostTable').then(
        (module) => ({ default: module.PostTable })
    )
);

// Route 상수
const PROJECT_CATALOG_FORM_ID = 744;
const PROJECT_CATALOG_PROJECT_ID = 225;

export const Route = createFileRoute('/project-catalog/')({
    component: ProjectCatalogPage,
    loader: () => ({
        crumb: 'Projects',  // Breadcrumb 제목
    }),
});

function ProjectCatalogPage() {
    return (
        <PostTable
            blogId={PROJECT_CATALOG_FORM_ID}
            projectId={PROJECT_CATALOG_PROJECT_ID}
            tableType='active_projects'
            title='Form Dashboard'
        />
    );
}

export default ProjectCatalogPage;
```

---

## 예제 9: Form이 있는 Dialog

```typescript
import React from 'react';
import {
    Dialog,
    DialogTitle,
    DialogContent,
    DialogActions,
    Button,
    TextField,
    Box,
    IconButton,
} from '@mui/material';
import { Close, PersonAdd } from '@mui/icons-material';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const formSchema = z.object({
    name: z.string().min(1),
    email: z.string().email(),
});

type FormData = z.infer<typeof formSchema>;

interface AddUserDialogProps {
    open: boolean;
    onClose: () => void;
    onSubmit: (data: FormData) => Promise<void>;
}

export const AddUserDialog: React.FC<AddUserDialogProps> = ({
    open,
    onClose,
    onSubmit,
}) => {
    const { register, handleSubmit, formState: { errors }, reset } = useForm<FormData>({
        resolver: zodResolver(formSchema),
    });

    const handleClose = () => {
        reset();
        onClose();
    };

    const handleFormSubmit = async (data: FormData) => {
        await onSubmit(data);
        handleClose();
    };

    return (
        <Dialog open={open} onClose={handleClose} maxWidth='sm' fullWidth>
            <DialogTitle>
                <Box sx={{ display: 'flex', alignItems: 'center', justifyContent: 'space-between' }}>
                    <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
                        <PersonAdd color='primary' />
                        Add User
                    </Box>
                    <IconButton onClick={handleClose} size='small'>
                        <Close />
                    </IconButton>
                </Box>
            </DialogTitle>

            <form onSubmit={handleSubmit(handleFormSubmit)}>
                <DialogContent>
                    <Box sx={{ display: 'flex', flexDirection: 'column', gap: 2 }}>
                        <TextField
                            {...register('name')}
                            label='Name'
                            error={!!errors.name}
                            helperText={errors.name?.message}
                            fullWidth
                            autoFocus
                        />

                        <TextField
                            {...register('email')}
                            label='Email'
                            type='email'
                            error={!!errors.email}
                            helperText={errors.email?.message}
                            fullWidth
                        />
                    </Box>
                </DialogContent>

                <DialogActions>
                    <Button onClick={handleClose}>Cancel</Button>
                    <Button type='submit' variant='contained'>
                        Add User
                    </Button>
                </DialogActions>
            </form>
        </Dialog>
    );
};
```

---

## 예제 10: 병렬 데이터 Fetching

```typescript
import React from 'react';
import { Box, Grid, Paper } from '@mui/material';
import { useSuspenseQueries } from '@tanstack/react-query';
import { userApi } from '../api/userApi';
import { statsApi } from '../api/statsApi';
import { activityApi } from '../api/activityApi';

export const Dashboard: React.FC = () => {
    // Suspense로 모든 데이터를 병렬로 fetch
    const [statsQuery, usersQuery, activityQuery] = useSuspenseQueries({
        queries: [
            {
                queryKey: ['stats'],
                queryFn: () => statsApi.getStats(),
            },
            {
                queryKey: ['users', 'active'],
                queryFn: () => userApi.getActiveUsers(),
            },
            {
                queryKey: ['activity', 'recent'],
                queryFn: () => activityApi.getRecent(),
            },
        ],
    });

    return (
        <Box sx={{ p: 2 }}>
            <Grid container spacing={2}>
                <Grid size={{ xs: 12, md: 4 }}>
                    <Paper sx={{ p: 2 }}>
                        <h3>Stats</h3>
                        <p>Total: {statsQuery.data.total}</p>
                    </Paper>
                </Grid>

                <Grid size={{ xs: 12, md: 4 }}>
                    <Paper sx={{ p: 2 }}>
                        <h3>Active Users</h3>
                        <p>Count: {usersQuery.data.length}</p>
                    </Paper>
                </Grid>

                <Grid size={{ xs: 12, md: 4 }}>
                    <Paper sx={{ p: 2 }}>
                        <h3>Recent Activity</h3>
                        <p>Events: {activityQuery.data.length}</p>
                    </Paper>
                </Grid>
            </Grid>
        </Box>
    );
};

// Suspense와 함께 사용
<SuspenseLoader>
    <Dashboard />
</SuspenseLoader>
```

---

## 예제 11: Optimistic Update

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import type { User } from '../types';

export const useToggleUserStatus = () => {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: (userId: string) => userApi.toggleStatus(userId),

        // Optimistic update
        onMutate: async (userId) => {
            // 진행 중인 refetch 취소
            await queryClient.cancelQueries({ queryKey: ['users'] });

            // 이전 값 스냅샷
            const previousUsers = queryClient.getQueryData<User[]>(['users']);

            // Optimistically UI 업데이트
            queryClient.setQueryData<User[]>(['users'], (old) => {
                return old?.map(user =>
                    user.id === userId
                        ? { ...user, active: !user.active }
                        : user
                ) || [];
            });

            return { previousUsers };
        },

        // 에러 시 롤백
        onError: (err, userId, context) => {
            queryClient.setQueryData(['users'], context?.previousUsers);
        },

        // mutation 후 refetch
        onSettled: () => {
            queryClient.invalidateQueries({ queryKey: ['users'] });
        },
    });
};
```

---

## 요약

**핵심 포인트:**

1. **컴포넌트 패턴**: React.FC + lazy + Suspense + useSuspenseQuery
2. **Feature 구조**: 구성된 하위 디렉토리 (api/, components/, hooks/ 등)
3. **라우팅**: Lazy loading이 있는 폴더 기반
4. **데이터 Fetching**: Cache-first 전략이 있는 useSuspenseQuery
5. **Forms**: React Hook Form + Zod 유효성 검사
6. **에러 처리**: useMuiSnackbar + onError 콜백
7. **성능**: useMemo, useCallback, React.memo, debouncing
8. **스타일링**: 100줄 미만 인라인, sx prop, MUI v7 문법

**각 패턴의 자세한 설명은 다른 리소스 참조.**
