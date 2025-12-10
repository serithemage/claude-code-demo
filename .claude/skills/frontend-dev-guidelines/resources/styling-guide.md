# 스타일링 가이드

MUI v7 sx prop, 인라인 스타일, 테마 통합을 사용한 최신 스타일링 패턴입니다.

---

## 인라인 vs 별도 스타일

### 결정 기준

**<100줄: 컴포넌트 상단에 인라인 스타일**

```typescript
import type { SxProps, Theme } from '@mui/material';

const componentStyles: Record<string, SxProps<Theme>> = {
    container: {
        p: 2,
        display: 'flex',
        flexDirection: 'column',
    },
    header: {
        mb: 2,
        borderBottom: '1px solid',
        borderColor: 'divider',
    },
    // ... 더 많은 스타일
};

export const MyComponent: React.FC = () => {
    return (
        <Box sx={componentStyles.container}>
            <Box sx={componentStyles.header}>
                <h2>Title</h2>
            </Box>
        </Box>
    );
};
```

**>100줄: 별도 `.styles.ts` 파일**

```typescript
// MyComponent.styles.ts
import type { SxProps, Theme } from '@mui/material';

export const componentStyles: Record<string, SxProps<Theme>> = {
    container: { ... },
    header: { ... },
    // ... 100줄 이상의 스타일
};

// MyComponent.tsx
import { componentStyles } from './MyComponent.styles';

export const MyComponent: React.FC = () => {
    return <Box sx={componentStyles.container}>...</Box>;
};
```

### 실제 예시: UnifiedForm.tsx

**48-126번 줄**: 78줄의 인라인 스타일 (허용됨)

```typescript
const formStyles: Record<string, SxProps<Theme>> = {
    gridContainer: {
        height: '100%',
        maxHeight: 'calc(100vh - 220px)',
    },
    section: {
        height: '100%',
        maxHeight: 'calc(100vh - 220px)',
        overflow: 'auto',
        p: 4,
    },
    // ... 15개 이상의 스타일 객체
};
```

**가이드라인**: 사용자는 ~80줄 인라인이 편함. 100줄 근처에서 판단해서 결정.

---

## sx Prop 패턴

### 기본 사용법

```typescript
<Box sx={{ p: 2, mb: 3, display: 'flex' }}>
    Content
</Box>
```

### 테마 액세스와 함께

```typescript
<Box
    sx={{
        p: 2,
        backgroundColor: (theme) => theme.palette.primary.main,
        color: (theme) => theme.palette.primary.contrastText,
        borderRadius: (theme) => theme.shape.borderRadius,
    }}
>
    Themed Box
</Box>
```

### 반응형 스타일

```typescript
<Box
    sx={{
        p: { xs: 1, sm: 2, md: 3 },
        width: { xs: '100%', md: '50%' },
        flexDirection: { xs: 'column', md: 'row' },
    }}
>
    Responsive Layout
</Box>
```

### Pseudo-Selectors

```typescript
<Box
    sx={{
        p: 2,
        '&:hover': {
            backgroundColor: 'rgba(0,0,0,0.05)',
        },
        '&:active': {
            backgroundColor: 'rgba(0,0,0,0.1)',
        },
        '& .child-class': {
            color: 'primary.main',
        },
    }}
>
    Interactive Box
</Box>
```

---

## MUI v7 패턴

### Grid 컴포넌트 (v7 문법)

```typescript
import { Grid } from '@mui/material';

// ✅ 올바름 - size prop이 있는 v7 문법
<Grid container spacing={2}>
    <Grid size={{ xs: 12, md: 6 }}>
        Left Column
    </Grid>
    <Grid size={{ xs: 12, md: 6 }}>
        Right Column
    </Grid>
</Grid>

// ❌ 잘못됨 - 구식 v6 문법
<Grid container spacing={2}>
    <Grid xs={12} md={6}>  {/* 구식 - 사용 금지 */}
        Content
    </Grid>
</Grid>
```

**핵심 변경**: `xs={12} md={6}` 대신 `size={{ xs: 12, md: 6 }}`

### 반응형 Grid

```typescript
<Grid container spacing={3}>
    <Grid size={{ xs: 12, sm: 6, md: 4, lg: 3 }}>
        Responsive Column
    </Grid>
</Grid>
```

### 중첩 Grids

```typescript
<Grid container spacing={2}>
    <Grid size={{ xs: 12, md: 8 }}>
        <Grid container spacing={1}>
            <Grid size={{ xs: 12, sm: 6 }}>
                Nested 1
            </Grid>
            <Grid size={{ xs: 12, sm: 6 }}>
                Nested 2
            </Grid>
        </Grid>
    </Grid>

    <Grid size={{ xs: 12, md: 4 }}>
        Sidebar
    </Grid>
</Grid>
```

---

## 타입 안전 스타일

### 스타일 객체 타입

```typescript
import type { SxProps, Theme } from '@mui/material';

// 타입 안전 스타일
const styles: Record<string, SxProps<Theme>> = {
    container: {
        p: 2,
        // 자동완성과 타입 체크가 여기서 작동
    },
};

// 또는 개별 스타일
const containerStyle: SxProps<Theme> = {
    p: 2,
    display: 'flex',
};
```

### 테마 인식 스타일

```typescript
const styles: Record<string, SxProps<Theme>> = {
    primary: {
        color: (theme) => theme.palette.primary.main,
        backgroundColor: (theme) => theme.palette.primary.light,
        '&:hover': {
            backgroundColor: (theme) => theme.palette.primary.dark,
        },
    },
    customSpacing: {
        padding: (theme) => theme.spacing(2),
        margin: (theme) => theme.spacing(1, 2), // 위/아래: 1, 좌/우: 2
    },
};
```

---

## 사용하지 말아야 할 것

### ❌ makeStyles (MUI v4 패턴)

```typescript
// ❌ 피하세요 - 구식 Material-UI v4 패턴
import { makeStyles } from '@mui/styles';

const useStyles = makeStyles((theme) => ({
    root: {
        padding: theme.spacing(2),
    },
}));
```

**피해야 하는 이유**: deprecated, v7에서 잘 지원 안 됨

### ❌ styled() 컴포넌트

```typescript
// ❌ 피하세요 - styled-components 패턴
import { styled } from '@mui/material/styles';

const StyledBox = styled(Box)(({ theme }) => ({
    padding: theme.spacing(2),
}));
```

**피해야 하는 이유**: sx prop이 더 유연하고 새 컴포넌트를 생성하지 않음

### ✅ 대신 sx Prop 사용

```typescript
// ✅ 권장
<Box
    sx={{
        p: 2,
        backgroundColor: 'primary.main',
    }}
>
    Content
</Box>
```

---

## 코드 스타일 표준

### 들여쓰기

**4 스페이스** (2가 아님, 탭이 아님)

```typescript
const styles: Record<string, SxProps<Theme>> = {
    container: {
        p: 2,
        display: 'flex',
        flexDirection: 'column',
    },
};
```

### 따옴표

**작은따옴표** 문자열용 (프로젝트 표준)

```typescript
// ✅ 올바름
const color = 'primary.main';
import { Box } from '@mui/material';

// ❌ 잘못됨
const color = "primary.main";
import { Box } from "@mui/material";
```

### 후행 쉼표

객체와 배열에 **항상 후행 쉼표** 사용

```typescript
// ✅ 올바름
const styles = {
    container: { p: 2 },
    header: { mb: 1 },  // 후행 쉼표
};

const items = [
    'item1',
    'item2',  // 후행 쉼표
];

// ❌ 잘못됨 - 후행 쉼표 없음
const styles = {
    container: { p: 2 },
    header: { mb: 1 }  // 쉼표 누락
};
```

---

## 일반적인 스타일 패턴

### Flexbox 레이아웃

```typescript
const styles = {
    flexRow: {
        display: 'flex',
        flexDirection: 'row',
        alignItems: 'center',
        gap: 2,
    },
    flexColumn: {
        display: 'flex',
        flexDirection: 'column',
        gap: 1,
    },
    spaceBetween: {
        display: 'flex',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
};
```

### 간격

```typescript
// Padding
p: 2           // 모든 면
px: 2          // 가로 (좌 + 우)
py: 2          // 세로 (위 + 아래)
pt: 2, pr: 1   // 특정 면

// Margin
m: 2, mx: 2, my: 2, mt: 2, mr: 1

// 단위: 1 = 8px (theme.spacing(1))
p: 2  // = 16px
p: 0.5  // = 4px
```

### 포지셔닝

```typescript
const styles = {
    relative: {
        position: 'relative',
    },
    absolute: {
        position: 'absolute',
        top: 0,
        right: 0,
    },
    sticky: {
        position: 'sticky',
        top: 0,
        zIndex: 1000,
    },
};
```

---

## 요약

**스타일링 체크리스트:**
- ✅ MUI 스타일링에 `sx` prop 사용
- ✅ `SxProps<Theme>`로 타입 안전
- ✅ <100줄: 인라인; >100줄: 별도 파일
- ✅ MUI v7 Grid: `size={{ xs: 12 }}`
- ✅ 4 스페이스 들여쓰기
- ✅ 작은따옴표
- ✅ 후행 쉼표
- ❌ makeStyles나 styled() 사용 금지

**참고:**
- [component-patterns.md](component-patterns.md) - 컴포넌트 구조
- [complete-examples.md](complete-examples.md) - 전체 스타일링 예제
