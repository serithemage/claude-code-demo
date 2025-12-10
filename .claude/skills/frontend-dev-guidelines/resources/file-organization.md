# 파일 구성

애플리케이션에서 유지보수 가능하고 확장 가능한 프론트엔드 코드를 위한 올바른 파일 및 디렉토리 구조입니다.

---

## RealWorld (Conduit) 프론트엔드 구조

이 프로젝트의 기능 기반 모듈 구조입니다:

```
frontend/src/
├── features/                    # 기능 모듈
│   ├── auth/                    # 인증 기능
│   │   ├── components/          # Login, Register, Settings
│   │   ├── hooks/               # useAuth
│   │   ├── api/                 # authApi.ts
│   │   └── types.ts
│   ├── articles/                # 게시글 기능
│   │   ├── components/          # ArticleList, ArticleDetail, Editor
│   │   ├── hooks/               # useArticles, useSuspenseArticle
│   │   ├── api/                 # articleApi.ts
│   │   └── types.ts
│   ├── comments/                # 댓글 기능
│   │   ├── components/          # CommentList, CommentForm
│   │   ├── hooks/               # useComments
│   │   └── api/                 # commentApi.ts
│   ├── profiles/                # 프로필 기능
│   │   ├── components/          # Profile, ProfileHeader
│   │   ├── hooks/               # useProfile
│   │   └── api/                 # profileApi.ts
│   └── tags/                    # 태그 기능
│       ├── components/          # TagList, PopularTags
│       └── api/                 # tagApi.ts
│
├── components/                  # 공유 컴포넌트
│   ├── layout/
│   │   ├── Header.tsx           # 네비게이션
│   │   ├── Footer.tsx
│   │   └── Layout.tsx
│   └── ui/
│       ├── Loading.tsx
│       ├── ErrorMessage.tsx
│       └── Pagination.tsx
│
├── hooks/                       # 공유 Hooks
│   ├── useAuth.ts
│   └── useLocalStorage.ts
│
├── lib/                         # 유틸리티
│   ├── api/
│   │   └── client.ts            # API 클라이언트 (axios)
│   └── utils/
│       └── formatDate.ts
│
├── routes/                      # TanStack Router
│   ├── __root.tsx
│   ├── index.tsx                # 홈 (글로벌 피드)
│   ├── login.tsx
│   ├── register.tsx
│   ├── settings.tsx
│   ├── editor/
│   │   ├── index.tsx            # 새 글 작성
│   │   └── $slug.tsx            # 글 수정
│   ├── article/
│   │   └── $slug.tsx            # 글 상세
│   └── profile/
│       └── $username.tsx        # 사용자 프로필
│
└── types/                       # 공유 타입
    ├── user.ts
    ├── article.ts
    └── common.ts
```

### 핵심 기능 설명

| 기능         | 설명          | 주요 컴포넌트                      |
| ------------ | ------------- | ---------------------------------- |
| **auth**     | 인증 및 설정  | Login, Register, Settings          |
| **articles** | 게시글 CRUD   | ArticleList, ArticleDetail, Editor |
| **comments** | 댓글 관리     | CommentList, CommentForm           |
| **profiles** | 사용자 프로필 | Profile, FollowButton              |
| **tags**     | 인기 태그     | PopularTags, TagFilter             |

---

## features/ vs components/ 구분

### features/ 디렉토리

**목적**: 자체 로직, API, 컴포넌트가 있는 도메인별 기능

**사용 시점:**

- 기능에 여러 관련 컴포넌트가 있을 때
- 기능에 자체 API 엔드포인트가 있을 때
- 도메인별 로직이 있을 때
- 커스텀 hooks/유틸리티가 있을 때

**예시:**

- `features/posts/` - 프로젝트 카탈로그/포스트 관리
- `features/blogs/` - 블로그 빌더 및 렌더링
- `features/auth/` - 인증 플로우

**구조:**

```
features/
  my-feature/
    api/
      myFeatureApi.ts         # API 서비스 레이어
    components/
      MyFeatureMain.tsx       # 메인 컴포넌트
      SubComponents/          # 관련 컴포넌트
    hooks/
      useMyFeature.ts         # 커스텀 hooks
      useSuspenseMyFeature.ts # Suspense hooks
    helpers/
      myFeatureHelpers.ts     # 유틸리티 함수
    types/
      index.ts                # TypeScript 타입
    index.ts                  # Public exports
```

### components/ 디렉토리

**목적**: 여러 기능에서 사용되는 진정한 재사용 가능 컴포넌트

**사용 시점:**

- 컴포넌트가 3개 이상의 곳에서 사용될 때
- 컴포넌트가 제네릭 (기능별 로직 없음)
- 컴포넌트가 UI 원시 또는 패턴일 때

**예시:**

- `components/SuspenseLoader/` - 로딩 wrapper
- `components/CustomAppBar/` - 애플리케이션 헤더
- `components/ErrorBoundary/` - 에러 처리
- `components/LoadingOverlay/` - 로딩 오버레이

**구조:**

```
components/
  SuspenseLoader/
    SuspenseLoader.tsx
    SuspenseLoader.test.tsx
  CustomAppBar/
    CustomAppBar.tsx
    CustomAppBar.test.tsx
```

---

## Feature 디렉토리 구조 (상세)

### 완전한 Feature 예시

`features/posts/` 구조 기반:

```
features/
  posts/
    api/
      postApi.ts              # API 서비스 레이어 (GET, POST, PUT, DELETE)

    components/
      PostTable.tsx           # 메인 컨테이너 컴포넌트
      grids/
        PostDataGrid/
          PostDataGrid.tsx
      drawers/
        ProjectPostDrawer/
          ProjectPostDrawer.tsx
      cells/
        editors/
          TextEditCell.tsx
        renderers/
          DateCell.tsx
      toolbar/
        CustomToolbar.tsx

    hooks/
      usePostQueries.ts       # 일반 쿼리
      useSuspensePost.ts      # Suspense 쿼리
      usePostMutations.ts     # Mutations
      useGridLayout.ts        # 기능별 hooks

    helpers/
      postHelpers.ts          # 유틸리티 함수
      validation.ts           # 유효성 검사 로직

    types/
      index.ts                # TypeScript 타입/인터페이스

    queries/
      postQueries.ts          # 쿼리 키 팩토리 (선택적)

    context/
      PostContext.tsx         # React context (필요한 경우)

    index.ts                  # Public API exports
```

### 하위 디렉토리 가이드라인

#### api/ 디렉토리

**목적**: 기능에 대한 중앙화된 API 호출

**파일:**

- `{feature}Api.ts` - 메인 API 서비스

**패턴:**

```typescript
// features/my-feature/api/myFeatureApi.ts
import apiClient from '@/lib/apiClient';

export const myFeatureApi = {
  getItem: async (id: number) => {
    const { data } = await apiClient.get(`/blog/items/${id}`);
    return data;
  },
  createItem: async (payload) => {
    const { data } = await apiClient.post('/blog/items', payload);
    return data;
  },
};
```

#### components/ 디렉토리

**목적**: 기능별 컴포넌트

**구성:**

- 5개 미만 컴포넌트면 평면 구조
- 5개 이상 컴포넌트면 책임별 하위 디렉토리

**예시:**

```
components/
  MyFeatureMain.tsx           # 메인 컴포넌트
  MyFeatureHeader.tsx         # 지원 컴포넌트
  MyFeatureFooter.tsx

  # 또는 하위 디렉토리:
  containers/
    MyFeatureContainer.tsx
  presentational/
    MyFeatureDisplay.tsx
  blogs/
    MyFeatureBlog.tsx
```

#### hooks/ 디렉토리

**목적**: 기능에 대한 커스텀 hooks

**네이밍:**

- `use` 접두사 (camelCase)
- 기능을 설명하는 이름

**예시:**

```
hooks/
  useMyFeature.ts               # 메인 hook
  useSuspenseMyFeature.ts       # Suspense 버전
  useMyFeatureMutations.ts      # Mutations
  useMyFeatureFilters.ts        # 필터/검색
```

#### helpers/ 디렉토리

**목적**: 기능에 특화된 유틸리티 함수

**예시:**

```
helpers/
  myFeatureHelpers.ts           # 일반 유틸리티
  validation.ts                 # 유효성 검사 로직
  transformers.ts               # 데이터 변환
  constants.ts                  # 상수
```

#### types/ 디렉토리

**목적**: TypeScript 타입 정의

**파일:**

```
types/
  index.ts                      # 메인 타입, 내보내기
  internal.ts                   # 내부 타입 (내보내지 않음)
```

---

## Import 별칭 (Vite 설정)

### 사용 가능한 별칭

`vite.config.ts` 180-185번 줄 참조:

| 별칭          | 해석             | 용도                          |
| ------------- | ---------------- | ----------------------------- |
| `@/`          | `src/`           | src 루트에서 절대 경로 import |
| `~types`      | `src/types`      | 공유 TypeScript 타입          |
| `~components` | `src/components` | 재사용 가능 컴포넌트          |
| `~features`   | `src/features`   | Feature imports               |

### 사용 예시

```typescript
// ✅ 권장 - 절대 경로 import에 별칭 사용
import { apiClient } from '@/lib/apiClient';
import { SuspenseLoader } from '~components/SuspenseLoader';
import { postApi } from '~features/posts/api/postApi';
import type { User } from '~types/user';

// ❌ 피하세요 - 깊은 중첩에서 상대 경로
import { apiClient } from '../../../lib/apiClient';
import { SuspenseLoader } from '../../../components/SuspenseLoader';
```

### 어떤 별칭을 사용할지

**@/ (일반)**:

- Lib 유틸리티: `@/lib/apiClient`
- Hooks: `@/hooks/useAuth`
- Config: `@/config/theme`
- 공유 서비스: `@/services/authService`

**~types (타입 Import)**:

```typescript
import type { Post } from '~types/post';
import type { User, UserRole } from '~types/user';
```

**~components (재사용 가능 컴포넌트)**:

```typescript
import { SuspenseLoader } from '~components/SuspenseLoader';
import { CustomAppBar } from '~components/CustomAppBar';
import { ErrorBoundary } from '~components/ErrorBoundary';
```

**~features (Feature Imports)**:

```typescript
import { postApi } from '~features/posts/api/postApi';
import { useAuth } from '~features/auth/hooks/useAuth';
```

---

## 파일 네이밍 규칙

### 컴포넌트

**패턴**: PascalCase + `.tsx` 확장자

```
MyComponent.tsx
PostDataGrid.tsx
CustomAppBar.tsx
```

**피하세요:**

- camelCase: `myComponent.tsx` ❌
- kebab-case: `my-component.tsx` ❌
- 모두 대문자: `MYCOMPONENT.tsx` ❌

### Hooks

**패턴**: camelCase + `use` 접두사 + `.ts` 확장자

```
useMyFeature.ts
useSuspensePost.ts
useAuth.ts
useGridLayout.ts
```

### API 서비스

**패턴**: camelCase + `Api` 접미사 + `.ts` 확장자

```
myFeatureApi.ts
postApi.ts
userApi.ts
```

### Helpers/유틸리티

**패턴**: camelCase + 설명적 이름 + `.ts` 확장자

```
myFeatureHelpers.ts
validation.ts
transformers.ts
constants.ts
```

### 타입

**패턴**: camelCase, `index.ts` 또는 설명적 이름

```
types/index.ts
types/post.ts
types/user.ts
```

---

## 새 Feature 생성 시점

### 새 Feature 생성:

- 여러 관련 컴포넌트 (3개 이상)
- 자체 API 엔드포인트가 있음
- 도메인별 로직
- 시간이 지나면서 성장할 것
- 여러 routes에서 재사용

**예시:** `features/posts/`

- 20개 이상 컴포넌트
- 자체 API 서비스
- 복잡한 상태 관리
- 여러 routes에서 사용

### 기존 Feature에 추가:

- 기존 기능과 관련
- 같은 API 공유
- 논리적으로 그룹화됨
- 기존 기능 확장

**예시:** posts feature에 export dialog 추가

### 재사용 가능 컴포넌트 생성:

- 3개 이상 기능에서 사용
- 제네릭, 도메인 로직 없음
- 순수 프레젠테이션
- 공유 패턴

**예시:** `components/SuspenseLoader/`

---

## Import 구성

### Import 순서 (권장)

```typescript
// 1. React 및 React 관련
import React, { useState, useCallback, useMemo } from 'react';
import { lazy } from 'react';

// 2. 서드파티 라이브러리 (알파벳순)
import { Box, Paper, Button, Grid } from '@mui/material';
import type { SxProps, Theme } from '@mui/material';
import { useSuspenseQuery, useQueryClient } from '@tanstack/react-query';
import { createFileRoute } from '@tanstack/react-router';

// 3. 별칭 imports (@ 먼저, 그 다음 ~)
import { apiClient } from '@/lib/apiClient';
import { useAuth } from '@/hooks/useAuth';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';
import { SuspenseLoader } from '~components/SuspenseLoader';
import { postApi } from '~features/posts/api/postApi';

// 4. 타입 imports (그룹화)
import type { Post } from '~types/post';
import type { User } from '~types/user';

// 5. 상대 경로 imports (같은 feature)
import { MySubComponent } from './MySubComponent';
import { useMyFeature } from '../hooks/useMyFeature';
import { myFeatureHelpers } from '../helpers/myFeatureHelpers';
```

모든 imports에 **작은따옴표** 사용 (프로젝트 표준)

---

## Public API 패턴

### feature/index.ts

깔끔한 imports를 위해 feature에서 public API 내보내기:

```typescript
// features/my-feature/index.ts

// 메인 컴포넌트 내보내기
export { MyFeatureMain } from './components/MyFeatureMain';
export { MyFeatureHeader } from './components/MyFeatureHeader';

// Hooks 내보내기
export { useMyFeature } from './hooks/useMyFeature';
export { useSuspenseMyFeature } from './hooks/useSuspenseMyFeature';

// API 내보내기
export { myFeatureApi } from './api/myFeatureApi';

// 타입 내보내기
export type { MyFeatureData, MyFeatureConfig } from './types';
```

**사용:**

```typescript
// ✅ feature index에서 깔끔한 import
import { MyFeatureMain, useMyFeature } from '~features/my-feature';

// ❌ 깊은 imports 피하세요 (필요하면 OK)
import { MyFeatureMain } from '~features/my-feature/components/MyFeatureMain';
```

---

## 디렉토리 구조 시각화

```
src/
├── features/                    # 도메인별 기능
│   ├── posts/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── helpers/
│   │   ├── types/
│   │   └── index.ts
│   ├── blogs/
│   └── auth/
│
├── components/                  # 재사용 가능 컴포넌트
│   ├── SuspenseLoader/
│   ├── CustomAppBar/
│   ├── ErrorBoundary/
│   └── LoadingOverlay/
│
├── routes/                      # TanStack Router routes
│   ├── __root.tsx
│   ├── index.tsx
│   ├── project-catalog/
│   │   ├── index.tsx
│   │   └── create/
│   └── blogs/
│
├── hooks/                       # 공유 hooks
│   ├── useAuth.ts
│   ├── useMuiSnackbar.ts
│   └── useDebounce.ts
│
├── lib/                         # 공유 유틸리티
│   ├── apiClient.ts
│   └── utils.ts
│
├── types/                       # 공유 TypeScript 타입
│   ├── user.ts
│   ├── post.ts
│   └── common.ts
│
├── config/                      # 설정
│   └── theme.ts
│
└── App.tsx                      # 루트 컴포넌트
```

---

## 요약

**핵심 원칙:**

1. **features/** 도메인별 코드용
2. **components/** 진정한 재사용 가능 UI용
3. 하위 디렉토리 사용: api/, components/, hooks/, helpers/, types/
4. 깔끔한 imports를 위한 Import 별칭 (@/, ~types, ~components, ~features)
5. 일관된 네이밍: PascalCase 컴포넌트, camelCase 유틸리티
6. feature index.ts에서 public API 내보내기

**참고:**

- [component-patterns.md](component-patterns.md) - 컴포넌트 구조
- [data-fetching.md](data-fetching.md) - API 서비스 패턴
- [complete-examples.md](complete-examples.md) - 전체 feature 예제
