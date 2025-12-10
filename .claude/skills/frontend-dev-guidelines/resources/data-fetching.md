# 데이터 페칭 패턴

Suspense boundaries, cache-first 전략, 중앙화된 API 서비스를 사용한 TanStack Query 기반의 최신 데이터 페칭 패턴입니다.

---

## RealWorld API 타입 정의

### 핵심 타입

```typescript
// types/user.ts
export interface User {
  email: string;
  token: string;
  username: string;
  bio: string | null;
  image: string | null;
}

export interface Profile {
  username: string;
  bio: string | null;
  image: string | null;
  following: boolean;
}

// types/article.ts
export interface Article {
  slug: string;
  title: string;
  description: string;
  body: string;
  tagList: string[];
  createdAt: string;      // ISO 8601
  updatedAt: string;      // ISO 8601
  favorited: boolean;
  favoritesCount: number;
  author: Profile;
}

export interface Comment {
  id: number;
  createdAt: string;      // ISO 8601
  updatedAt: string;      // ISO 8601
  body: string;
  author: Profile;
}
```

### API 응답 타입

```typescript
// 사용자 응답
interface UserResponse {
  user: User;
}

// 게시글 응답
interface ArticleResponse {
  article: Article;
}

interface ArticlesResponse {
  articles: Article[];
  articlesCount: number;
}

// 댓글 응답
interface CommentsResponse {
  comments: Comment[];
}

// 태그 응답
interface TagsResponse {
  tags: string[];
}

// 에러 응답
interface ErrorResponse {
  errors: {
    [field: string]: string[];
  };
}
```

### API 서비스 예시

```typescript
// features/articles/api/articleApi.ts
import apiClient from '@/lib/api/client';
import type { Article, ArticlesResponse } from '../types';

export const articleApi = {
  // 게시글 목록 조회
  getArticles: async (params?: {
    tag?: string;
    author?: string;
    favorited?: string;
    limit?: number;
    offset?: number;
  }): Promise<ArticlesResponse> => {
    const { data } = await apiClient.get('/articles', { params });
    return data;
  },

  // 피드 조회 (팔로우한 사용자)
  getFeed: async (limit = 20, offset = 0): Promise<ArticlesResponse> => {
    const { data } = await apiClient.get('/articles/feed', {
      params: { limit, offset }
    });
    return data;
  },

  // 게시글 상세 조회
  getArticle: async (slug: string): Promise<Article> => {
    const { data } = await apiClient.get(`/articles/${slug}`);
    return data.article;
  },

  // 게시글 작성
  createArticle: async (article: {
    title: string;
    description: string;
    body: string;
    tagList?: string[];
  }): Promise<Article> => {
    const { data } = await apiClient.post('/articles', { article });
    return data.article;
  },

  // 좋아요
  favoriteArticle: async (slug: string): Promise<Article> => {
    const { data } = await apiClient.post(`/articles/${slug}/favorite`);
    return data.article;
  },

  // 좋아요 취소
  unfavoriteArticle: async (slug: string): Promise<Article> => {
    const { data } = await apiClient.delete(`/articles/${slug}/favorite`);
    return data.article;
  },
};
```

---

## 주요 패턴: useSuspenseQuery

### useSuspenseQuery를 사용하는 이유

**모든 새 컴포넌트**에서는 일반 `useQuery` 대신 `useSuspenseQuery`를 사용하세요:

**장점:**
- `isLoading` 체크가 필요 없음
- Suspense boundaries와 통합
- 더 깔끔한 컴포넌트 코드
- 일관된 로딩 UX
- Error boundaries를 통한 더 나은 에러 처리

### 기본 패턴

```typescript
import { useSuspenseQuery } from '@tanstack/react-query';
import { myFeatureApi } from '../api/myFeatureApi';

export const MyComponent: React.FC<Props> = ({ id }) => {
    // isLoading 불필요 - Suspense가 처리!
    const { data } = useSuspenseQuery({
        queryKey: ['myEntity', id],
        queryFn: () => myFeatureApi.getEntity(id),
    });

    // data는 항상 정의됨 (undefined | Data가 아님)
    return <div>{data.name}</div>;
};

// Suspense boundary로 감싸기
<SuspenseLoader>
    <MyComponent id={123} />
</SuspenseLoader>
```

### useSuspenseQuery vs useQuery

| 특징 | useSuspenseQuery | useQuery |
|------|------------------|----------|
| 로딩 상태 | Suspense가 처리 | 수동 `isLoading` 체크 |
| 데이터 타입 | 항상 정의됨 | `Data \| undefined` |
| 사용 대상 | Suspense boundaries | 전통적인 컴포넌트 |
| 권장 대상 | **새 컴포넌트** | 레거시 코드만 |
| 에러 처리 | Error boundaries | 수동 에러 상태 |

**일반 useQuery를 사용해야 할 때:**
- 레거시 코드 유지보수
- Suspense 없는 매우 단순한 케이스
- 백그라운드 업데이트가 있는 폴링

**새 컴포넌트: 항상 useSuspenseQuery 선호**

---

## Cache-First 전략

### Cache-First 패턴 예시

**스마트 캐싱**으로 React Query 캐시를 먼저 확인하여 API 호출을 줄입니다:

```typescript
import { useSuspenseQuery, useQueryClient } from '@tanstack/react-query';
import { postApi } from '../api/postApi';

export function useSuspensePost(postId: number) {
    const queryClient = useQueryClient();

    return useSuspenseQuery({
        queryKey: ['post', postId],
        queryFn: async () => {
            // 전략 1: 리스트 캐시에서 먼저 가져오기 시도
            const cachedListData = queryClient.getQueryData<{ posts: Post[] }>([
                'posts',
                'list'
            ]);

            if (cachedListData?.posts) {
                const cachedPost = cachedListData.posts.find(
                    (post) => post.id === postId
                );

                if (cachedPost) {
                    return cachedPost;  // 캐시에서 반환!
                }
            }

            // 전략 2: 캐시에 없음, API에서 fetch
            return postApi.getPost(postId);
        },
        staleTime: 5 * 60 * 1000,      // 5분 동안 fresh로 간주
        gcTime: 10 * 60 * 1000,         // 10분 동안 캐시에 유지
        refetchOnWindowFocus: false,    // 포커스 시 refetch 안 함
    });
}
```

**핵심 포인트:**
- API 호출 전 그리드/리스트 캐시 확인
- 중복 요청 방지
- `staleTime`: 데이터가 fresh로 간주되는 기간
- `gcTime`: 사용되지 않는 데이터가 캐시에 남아있는 기간
- `refetchOnWindowFocus: false`: 사용자 선호

---

## 병렬 데이터 페칭

### useSuspenseQueries

여러 독립적인 리소스를 fetch할 때:

```typescript
import { useSuspenseQueries } from '@tanstack/react-query';

export const MyComponent: React.FC = () => {
    const [userQuery, settingsQuery, preferencesQuery] = useSuspenseQueries({
        queries: [
            {
                queryKey: ['user'],
                queryFn: () => userApi.getCurrentUser(),
            },
            {
                queryKey: ['settings'],
                queryFn: () => settingsApi.getSettings(),
            },
            {
                queryKey: ['preferences'],
                queryFn: () => preferencesApi.getPreferences(),
            },
        ],
    });

    // 모든 데이터 사용 가능, Suspense가 로딩 처리
    const user = userQuery.data;
    const settings = settingsQuery.data;
    const preferences = preferencesQuery.data;

    return <Display user={user} settings={settings} prefs={preferences} />;
};
```

**장점:**
- 모든 쿼리가 병렬로 실행
- 단일 Suspense boundary
- 타입 안전한 결과

---

## Query Keys 구성

### 네이밍 규칙

```typescript
// Entity 리스트
['entities', blogId]
['entities', blogId, 'summary']    // 뷰 모드와 함께
['entities', blogId, 'flat']

// 단일 entity
['entity', blogId, entityId]

// 관련 데이터
['entity', entityId, 'history']
['entity', entityId, 'comments']

// 사용자별
['user', userId, 'profile']
['user', userId, 'permissions']
```

**규칙:**
- entity 이름으로 시작 (리스트는 복수, 단일은 단수)
- 특정성을 위해 ID 포함
- 뷰 모드 / 관계는 끝에 추가
- 앱 전체에서 일관성 유지

### Query Key 예시

```typescript
// useSuspensePost.ts에서
queryKey: ['post', blogId, postId]
queryKey: ['posts-v2', blogId, 'summary']

// Invalidation 패턴
queryClient.invalidateQueries({ queryKey: ['post', blogId] });  // form의 모든 posts
queryClient.invalidateQueries({ queryKey: ['post'] });          // 모든 posts
```

---

## API 서비스 레이어 패턴

### 파일 구조

feature별로 중앙화된 API 서비스 생성:

```
features/
  my-feature/
    api/
      myFeatureApi.ts    # 서비스 레이어
```

### 서비스 패턴 (postApi.ts 기반)

```typescript
/**
 * my-feature 작업을 위한 중앙화된 API 서비스
 * 일관된 에러 처리를 위해 apiClient 사용
 */
import apiClient from '@/lib/apiClient';
import type { MyEntity, UpdatePayload } from '../types';

export const myFeatureApi = {
    /**
     * 단일 entity fetch
     */
    getEntity: async (blogId: number, entityId: number): Promise<MyEntity> => {
        const { data } = await apiClient.get(
            `/blog/entities/${blogId}/${entityId}`
        );
        return data;
    },

    /**
     * form의 모든 entities fetch
     */
    getEntities: async (blogId: number, view: 'summary' | 'flat'): Promise<MyEntity[]> => {
        const { data } = await apiClient.get(
            `/blog/entities/${blogId}`,
            { params: { view } }
        );
        return data.rows;
    },

    /**
     * Entity 업데이트
     */
    updateEntity: async (
        blogId: number,
        entityId: number,
        payload: UpdatePayload
    ): Promise<MyEntity> => {
        const { data } = await apiClient.put(
            `/blog/entities/${blogId}/${entityId}`,
            payload
        );
        return data;
    },

    /**
     * Entity 삭제
     */
    deleteEntity: async (blogId: number, entityId: number): Promise<void> => {
        await apiClient.delete(`/blog/entities/${blogId}/${entityId}`);
    },
};
```

**핵심 포인트:**
- 메서드가 있는 단일 객체 export
- `apiClient` 사용 (`@/lib/apiClient`의 axios 인스턴스)
- 타입 안전한 파라미터와 반환값
- 각 메서드에 JSDoc 주석
- 중앙화된 에러 처리 (apiClient가 처리)

---

## Route 형식 규칙 (중요)

### 올바른 형식

```typescript
// ✅ 올바름 - 직접 서비스 경로
await apiClient.get('/blog/posts/123');
await apiClient.post('/projects/create', data);
await apiClient.put('/users/update/456', updates);
await apiClient.get('/email/templates');

// ❌ 잘못됨 - /api/ 접두사 추가하지 마세요
await apiClient.get('/api/blog/posts/123');  // 잘못됨!
await apiClient.post('/api/projects/create', data); // 잘못됨!
```

**마이크로서비스 라우팅:**
- Form 서비스: `/blog/*`
- Projects 서비스: `/projects/*`
- Email 서비스: `/email/*`
- Users 서비스: `/users/*`

**이유:** API 라우팅은 프록시 설정에서 처리됨, `/api/` 접두사 불필요.

---

## Mutations

### 기본 Mutation 패턴

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { myFeatureApi } from '../api/myFeatureApi';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

export const MyComponent: React.FC = () => {
    const queryClient = useQueryClient();
    const { showSuccess, showError } = useMuiSnackbar();

    const updateMutation = useMutation({
        mutationFn: (payload: UpdatePayload) =>
            myFeatureApi.updateEntity(blogId, entityId, payload),

        onSuccess: () => {
            // Invalidate하고 refetch
            queryClient.invalidateQueries({
                queryKey: ['entity', blogId, entityId]
            });
            showSuccess('Entity updated successfully');
        },

        onError: (error) => {
            showError('Failed to update entity');
            console.error('Update error:', error);
        },
    });

    const handleUpdate = () => {
        updateMutation.mutate({ name: 'New Name' });
    };

    return (
        <Button
            onClick={handleUpdate}
            disabled={updateMutation.isPending}
        >
            {updateMutation.isPending ? 'Updating...' : 'Update'}
        </Button>
    );
};
```

### Optimistic Updates

```typescript
const updateMutation = useMutation({
    mutationFn: (payload) => myFeatureApi.update(id, payload),

    // Optimistic update
    onMutate: async (newData) => {
        // 진행 중인 refetch 취소
        await queryClient.cancelQueries({ queryKey: ['entity', id] });

        // 현재 값 스냅샷
        const previousData = queryClient.getQueryData(['entity', id]);

        // Optimistically 업데이트
        queryClient.setQueryData(['entity', id], (old) => ({
            ...old,
            ...newData,
        }));

        // 롤백 함수 반환
        return { previousData };
    },

    // 에러 시 롤백
    onError: (err, newData, context) => {
        queryClient.setQueryData(['entity', id], context.previousData);
        showError('Update failed');
    },

    // 성공 또는 에러 후 refetch
    onSettled: () => {
        queryClient.invalidateQueries({ queryKey: ['entity', id] });
    },
});
```

---

## 고급 쿼리 패턴

### Prefetching

```typescript
export function usePrefetchEntity() {
    const queryClient = useQueryClient();

    return (blogId: number, entityId: number) => {
        return queryClient.prefetchQuery({
            queryKey: ['entity', blogId, entityId],
            queryFn: () => myFeatureApi.getEntity(blogId, entityId),
            staleTime: 5 * 60 * 1000,
        });
    };
}

// 사용: hover 시 prefetch
<div onMouseEnter={() => prefetch(blogId, id)}>
    <Link to={`/entity/${id}`}>View</Link>
</div>
```

### Fetch 없이 캐시 접근

```typescript
export function useEntityFromCache(blogId: number, entityId: number) {
    const queryClient = useQueryClient();

    // 캐시에서 가져오기, 없으면 fetch하지 않음
    const directCache = queryClient.getQueryData<MyEntity>(['entity', blogId, entityId]);

    if (directCache) return directCache;

    // 그리드 캐시 시도
    const gridCache = queryClient.getQueryData<{ rows: MyEntity[] }>(['entities-v2', blogId]);

    return gridCache?.rows.find(row => row.id === entityId);
}
```

### 의존적 쿼리

```typescript
// 사용자 먼저 fetch, 그 다음 사용자 설정
const { data: user } = useSuspenseQuery({
    queryKey: ['user', userId],
    queryFn: () => userApi.getUser(userId),
});

const { data: settings } = useSuspenseQuery({
    queryKey: ['user', userId, 'settings'],
    queryFn: () => settingsApi.getUserSettings(user.id),
    // Suspense로 인해 자동으로 user 로드 대기
});
```

---

## API 클라이언트 설정

### apiClient 사용

```typescript
import apiClient from '@/lib/apiClient';

// apiClient는 설정된 axios 인스턴스
// 자동으로 포함:
// - Base URL 설정
// - 쿠키 기반 인증
// - 에러 인터셉터
// - 응답 변환기
```

**새 axios 인스턴스 생성 금지** - 일관성을 위해 apiClient 사용.

---

## 쿼리 에러 처리

### onError 콜백

```typescript
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

const { showError } = useMuiSnackbar();

const { data } = useSuspenseQuery({
    queryKey: ['entity', id],
    queryFn: () => myFeatureApi.getEntity(id),

    // 에러 처리
    onError: (error) => {
        showError('Failed to load entity');
        console.error('Load error:', error);
    },
});
```

### Error Boundaries

종합적인 에러 처리를 위해 Error Boundaries와 결합:

```typescript
import { ErrorBoundary } from 'react-error-boundary';

<ErrorBoundary
    fallback={<ErrorDisplay />}
    onError={(error) => console.error(error)}
>
    <SuspenseLoader>
        <ComponentWithSuspenseQuery />
    </SuspenseLoader>
</ErrorBoundary>
```

---

## 완전한 예제

### 예제 1: 간단한 Entity Fetch

```typescript
import React from 'react';
import { useSuspenseQuery } from '@tanstack/react-query';
import { Box, Typography } from '@mui/material';
import { userApi } from '../api/userApi';

interface UserProfileProps {
    userId: string;
}

export const UserProfile: React.FC<UserProfileProps> = ({ userId }) => {
    const { data: user } = useSuspenseQuery({
        queryKey: ['user', userId],
        queryFn: () => userApi.getUser(userId),
        staleTime: 5 * 60 * 1000,
    });

    return (
        <Box>
            <Typography variant='h5'>{user.name}</Typography>
            <Typography>{user.email}</Typography>
        </Box>
    );
};

// Suspense와 함께 사용
<SuspenseLoader>
    <UserProfile userId='123' />
</SuspenseLoader>
```

### 예제 2: Cache-First 전략

```typescript
import { useSuspenseQuery, useQueryClient } from '@tanstack/react-query';
import { postApi } from '../api/postApi';
import type { Post } from '../types';

/**
 * Cache-first 전략이 있는 hook
 * API 호출 전 그리드 캐시 확인
 */
export function useSuspensePost(blogId: number, postId: number) {
    const queryClient = useQueryClient();

    return useSuspenseQuery<Post, Error>({
        queryKey: ['post', blogId, postId],
        queryFn: async () => {
            // 1. 그리드 캐시 먼저 확인
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
                const cached = gridCache.rows.find(row => row.S_ID === postId);
                if (cached) {
                    return cached;  // 그리드 데이터 재사용
                }
            }

            // 2. 캐시에 없음, 직접 fetch
            return postApi.getPost(blogId, postId);
        },
        staleTime: 5 * 60 * 1000,
        gcTime: 10 * 60 * 1000,
        refetchOnWindowFocus: false,
    });
}
```

**장점:**
- 중복 API 호출 방지
- 이미 로드된 경우 즉시 데이터
- 캐시에 없으면 API로 fallback

### 예제 3: 병렬 Fetching

```typescript
import { useSuspenseQueries } from '@tanstack/react-query';

export const Dashboard: React.FC = () => {
    const [statsQuery, projectsQuery, notificationsQuery] = useSuspenseQueries({
        queries: [
            {
                queryKey: ['stats'],
                queryFn: () => statsApi.getStats(),
            },
            {
                queryKey: ['projects', 'active'],
                queryFn: () => projectsApi.getActiveProjects(),
            },
            {
                queryKey: ['notifications', 'unread'],
                queryFn: () => notificationsApi.getUnread(),
            },
        ],
    });

    return (
        <Box>
            <StatsCard data={statsQuery.data} />
            <ProjectsList projects={projectsQuery.data} />
            <Notifications items={notificationsQuery.data} />
        </Box>
    );
};
```

---

## 캐시 Invalidation이 있는 Mutations

### Update Mutation

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { postApi } from '../api/postApi';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

export const useUpdatePost = () => {
    const queryClient = useQueryClient();
    const { showSuccess, showError } = useMuiSnackbar();

    return useMutation({
        mutationFn: ({ blogId, postId, data }: UpdateParams) =>
            postApi.updatePost(blogId, postId, data),

        onSuccess: (data, variables) => {
            // 특정 post invalidate
            queryClient.invalidateQueries({
                queryKey: ['post', variables.blogId, variables.postId]
            });

            // 그리드 새로고침을 위해 리스트 invalidate
            queryClient.invalidateQueries({
                queryKey: ['posts-v2', variables.blogId]
            });

            showSuccess('Post updated');
        },

        onError: (error) => {
            showError('Failed to update post');
            console.error('Update error:', error);
        },
    });
};

// 사용
const updatePost = useUpdatePost();

const handleSave = () => {
    updatePost.mutate({
        blogId: 123,
        postId: 456,
        data: { responses: { '101': 'value' } }
    });
};
```

### Delete Mutation

```typescript
export const useDeletePost = () => {
    const queryClient = useQueryClient();
    const { showSuccess, showError } = useMuiSnackbar();

    return useMutation({
        mutationFn: ({ blogId, postId }: DeleteParams) =>
            postApi.deletePost(blogId, postId),

        onSuccess: (data, variables) => {
            // 캐시에서 수동으로 제거 (optimistic)
            queryClient.setQueryData<{ rows: Post[] }>(
                ['posts-v2', variables.blogId],
                (old) => ({
                    ...old,
                    rows: old?.rows.filter(row => row.S_ID !== variables.postId) || []
                })
            );

            showSuccess('Post deleted');
        },

        onError: (error, variables) => {
            // 롤백 - 정확한 상태를 위해 refetch
            queryClient.invalidateQueries({
                queryKey: ['posts-v2', variables.blogId]
            });
            showError('Failed to delete post');
        },
    });
};
```

---

## 쿼리 설정 모범 사례

### 기본 설정

```typescript
// QueryClientProvider 설정에서
const queryClient = new QueryClient({
    defaultOptions: {
        queries: {
            staleTime: 1000 * 60 * 5,        // 5분
            gcTime: 1000 * 60 * 10,           // 10분 (이전 cacheTime)
            refetchOnWindowFocus: false,       // 포커스 시 refetch 안 함
            refetchOnMount: false,             // fresh면 마운트 시 refetch 안 함
            retry: 1,                          // 실패한 쿼리 한 번 재시도
        },
    },
});
```

### 쿼리별 오버라이드

```typescript
// 자주 변경되는 데이터 - 짧은 staleTime
useSuspenseQuery({
    queryKey: ['notifications', 'unread'],
    queryFn: () => notificationApi.getUnread(),
    staleTime: 30 * 1000,  // 30초
});

// 거의 변경되지 않는 데이터 - 긴 staleTime
useSuspenseQuery({
    queryKey: ['form', blogId, 'structure'],
    queryFn: () => formApi.getStructure(blogId),
    staleTime: 30 * 60 * 1000,  // 30분
});
```

---

## 요약

**최신 데이터 페칭 레시피:**

1. **API 서비스 생성**: apiClient를 사용하는 `features/X/api/XApi.ts`
2. **useSuspenseQuery 사용**: SuspenseLoader로 감싼 컴포넌트에서
3. **Cache-First**: API 호출 전 그리드 캐시 확인
4. **Query Keys**: 일관된 네이밍 ['entity', id]
5. **Route 형식**: `/api/blog/route`가 아닌 `/blog/route`
6. **Mutations**: 성공 후 invalidateQueries
7. **에러 처리**: onError + useMuiSnackbar
8. **타입 안전**: 모든 파라미터와 반환값 타입 지정

**참고:**
- [component-patterns.md](component-patterns.md) - Suspense 통합
- [loading-and-error-states.md](loading-and-error-states.md) - SuspenseLoader 사용
- [complete-examples.md](complete-examples.md) - 전체 작동 예제
