# Routing 가이드

TanStack Router 구현과 폴더 기반 라우팅 및 lazy loading 패턴입니다.

---

## RealWorld 페이지 라우트

### 페이지 구조

| 경로                           | 페이지   | 인증   | 설명                   |
| ------------------------------ | -------- | ------ | ---------------------- |
| `/`                            | Home     | 불필요 | 글로벌 피드, 인기 태그 |
| `/login`                       | Login    | 불필요 | 로그인 폼              |
| `/register`                    | Register | 불필요 | 가입 폼                |
| `/settings`                    | Settings | 필수   | 사용자 설정            |
| `/editor`                      | Editor   | 필수   | 새 글 작성             |
| `/editor/:slug`                | Editor   | 필수   | 글 수정                |
| `/article/:slug`               | Article  | 불필요 | 글 상세, 댓글          |
| `/profile/:username`           | Profile  | 불필요 | 사용자 프로필, 글 목록 |
| `/profile/:username/favorites` | Profile  | 불필요 | 좋아요한 글 목록       |

### 라우트 파일 구조

```
routes/
├── __root.tsx           # 루트 레이아웃 (Header, Footer)
├── index.tsx            # 홈 페이지 (/)
├── login.tsx            # 로그인 (/login)
├── register.tsx         # 가입 (/register)
├── settings.tsx         # 설정 (/settings)
├── editor/
│   ├── index.tsx        # 새 글 (/editor)
│   └── $slug.tsx        # 글 수정 (/editor/:slug)
├── article/
│   └── $slug.tsx        # 글 상세 (/article/:slug)
└── profile/
    ├── $username.tsx    # 프로필 (/profile/:username)
    └── $username/
        └── favorites.tsx  # 좋아요 목록
```

### 인증 필요 라우트 보호

```typescript
// routes/settings.tsx
import { createFileRoute, redirect } from '@tanstack/react-router';

export const Route = createFileRoute('/settings')({
  beforeLoad: ({ context }) => {
    if (!context.auth.isAuthenticated) {
      throw redirect({ to: '/login' });
    }
  },
  component: SettingsPage,
});
```

---

## TanStack Router 개요

파일 기반 라우팅이 있는 **TanStack Router**:

- 폴더 구조가 routes를 정의
- 코드 스플리팅을 위한 Lazy loading
- 타입 안전 라우팅
- Breadcrumb loaders

---

## 폴더 기반 라우팅

### 디렉토리 구조

```
routes/
  __root.tsx                    # 루트 레이아웃
  index.tsx                     # 홈 route (/)
  posts/
    index.tsx                   # /posts
    create/
      index.tsx                 # /posts/create
    $postId.tsx                 # /posts/:postId (동적)
  comments/
    index.tsx                   # /comments
```

**패턴**:

- `index.tsx` = 해당 경로의 Route
- `$param.tsx` = 동적 파라미터
- 중첩 폴더 = 중첩 routes

---

## 기본 Route 패턴

### posts/index.tsx 예시

```typescript
/**
 * Posts route 컴포넌트
 * 메인 블로그 포스트 목록 표시
 */

import { createFileRoute } from '@tanstack/react-router';
import { lazy } from 'react';

// 페이지 컴포넌트 Lazy load
const PostsList = lazy(() =>
    import('@/features/posts/components/PostsList').then(
        (module) => ({ default: module.PostsList }),
    ),
);

export const Route = createFileRoute('/posts/')(({
    component: PostsPage,
    // Breadcrumb 데이터 정의
    loader: () => ({
        crumb: 'Posts',
    }),
});

function PostsPage() {
    return (
        <PostsList
            title='All Posts'
            showFilters={true}
        />
    );
}

export default PostsPage;
```

**핵심 포인트:**

- 무거운 컴포넌트 lazy load
- route 경로와 함께 `createFileRoute`
- breadcrumb 데이터용 `loader`
- Page 컴포넌트가 콘텐츠 렌더
- Route와 컴포넌트 둘 다 export

---

## Lazy Loading Routes

### Named Export 패턴

```typescript
import { lazy } from 'react';

// named exports의 경우 .then()으로 default에 매핑
const MyPage = lazy(() =>
  import('@/features/my-feature/components/MyPage').then((module) => ({ default: module.MyPage }))
);
```

### Default Export 패턴

```typescript
import { lazy } from 'react';

// default exports의 경우 더 간단한 문법
const MyPage = lazy(() => import('@/features/my-feature/components/MyPage'));
```

### Routes를 Lazy Load하는 이유

- 코드 스플리팅 - 더 작은 초기 번들
- 더 빠른 초기 페이지 로드
- 네비게이션할 때만 route 코드 로드
- 더 나은 성능

---

## createFileRoute

### 기본 설정

```typescript
export const Route = createFileRoute('/my-route/')(({
    component: MyRoutePage,
});

function MyRoutePage() {
    return <div>My Route Content</div>;
}
```

### Breadcrumb Loader와 함께

```typescript
export const Route = createFileRoute('/my-route/')(({
    component: MyRoutePage,
    loader: () => ({
        crumb: 'My Route Title',
    }),
});
```

Breadcrumb이 네비게이션/앱 바에 자동으로 표시됩니다.

### 데이터 Loader와 함께

```typescript
export const Route = createFileRoute('/my-route/')(({
    component: MyRoutePage,
    loader: async () => {
        // 여기서 데이터 prefetch 가능
        const data = await api.getData();
        return { crumb: 'My Route', data };
    },
});
```

### Search Params와 함께

```typescript
export const Route = createFileRoute('/search/')(({
    component: SearchPage,
    validateSearch: (search: Record<string, unknown>) => {
        return {
            query: (search.query as string) || '',
            page: Number(search.page) || 1,
        };
    },
});

function SearchPage() {
    const { query, page } = Route.useSearch();
    // query와 page 사용
}
```

---

## 동적 Routes

### 파라미터 Routes

```typescript
// routes/users/$userId.tsx

export const Route = createFileRoute('/users/$userId')(({
    component: UserPage,
});

function UserPage() {
    const { userId } = Route.useParams();

    return <UserProfile userId={userId} />;
}
```

### 여러 파라미터

```typescript
// routes/posts/$postId/comments/$commentId.tsx

export const Route = createFileRoute('/posts/$postId/comments/$commentId')(({
    component: CommentPage,
});

function CommentPage() {
    const { postId, commentId } = Route.useParams();

    return <CommentEditor postId={postId} commentId={commentId} />;
}
```

---

## 네비게이션

### 프로그래매틱 네비게이션

```typescript
import { useNavigate } from '@tanstack/react-router';

export const MyComponent: React.FC = () => {
    const navigate = useNavigate();

    const handleClick = () => {
        navigate({ to: '/posts' });
    };

    return <Button onClick={handleClick}>View Posts</Button>;
};
```

### 파라미터와 함께

```typescript
const handleNavigate = () => {
  navigate({
    to: '/users/$userId',
    params: { userId: '123' },
  });
};
```

### Search Params와 함께

```typescript
const handleSearch = () => {
  navigate({
    to: '/search',
    search: { query: 'test', page: 1 },
  });
};
```

---

## Route 레이아웃 패턴

### 루트 레이아웃 (\_\_root.tsx)

```typescript
import { createRootRoute, Outlet } from '@tanstack/react-router';
import { Box } from '@mui/material';
import { CustomAppBar } from '~components/CustomAppBar';

export const Route = createRootRoute({
    component: RootLayout,
});

function RootLayout() {
    return (
        <Box>
            <CustomAppBar />
            <Box sx={{ p: 2 }}>
                <Outlet />  {/* 자식 routes가 여기서 렌더 */}
            </Box>
        </Box>
    );
}
```

### 중첩 레이아웃

```typescript
// routes/dashboard/index.tsx
export const Route = createFileRoute('/dashboard/')(({
    component: DashboardLayout,
});

function DashboardLayout() {
    return (
        <Box>
            <DashboardSidebar />
            <Box sx={{ flex: 1 }}>
                <Outlet />  {/* 중첩 routes */}
            </Box>
        </Box>
    );
}
```

---

## 완전한 Route 예시

```typescript
/**
 * User profile route
 * 경로: /users/:userId
 */

import { createFileRoute } from '@tanstack/react-router';
import { lazy } from 'react';
import { SuspenseLoader } from '~components/SuspenseLoader';

// 무거운 컴포넌트 Lazy load
const UserProfile = lazy(() =>
    import('@/features/users/components/UserProfile').then(
        (module) => ({ default: module.UserProfile })
    )
);

export const Route = createFileRoute('/users/$userId')(({
    component: UserPage,
    loader: () => ({
        crumb: 'User Profile',
    }),
});

function UserPage() {
    const { userId } = Route.useParams();

    return (
        <SuspenseLoader>
            <UserProfile userId={userId} />
        </SuspenseLoader>
    );
}

export default UserPage;
```

---

## 요약

**Routing 체크리스트:**

- ✅ 폴더 기반: `routes/my-route/index.tsx`
- ✅ 컴포넌트 Lazy load: `React.lazy(() => import())`
- ✅ route 경로와 함께 `createFileRoute` 사용
- ✅ `loader` 함수에 breadcrumb 추가
- ✅ 로딩 상태를 위해 `SuspenseLoader`로 감싸기
- ✅ 동적 params에 `Route.useParams()` 사용
- ✅ 프로그래매틱 네비게이션에 `useNavigate()` 사용

**참고:**

- [component-patterns.md](component-patterns.md) - Lazy loading 패턴
- [loading-and-error-states.md](loading-and-error-states.md) - SuspenseLoader 사용
- [complete-examples.md](complete-examples.md) - 전체 route 예제
