# Phase 1: 모노레포 구조 설정 - Context

## SESSION PROGRESS (2025-12-10)

### ✅ 완료
- 구현 계획 문서 작성 (`docs/implementation-plan.md`)
- Phase 1 plan.md 작성 완료
- `try1` 브랜치 생성

### 🟡 진행 중
- Phase 1 dev docs 작성 (context.md, tasks.md)

### ⏳ 대기 중
- Step 1: pnpm-workspace.yaml 생성
- Step 2: 루트 package.json 생성
- Step 3: .gitignore 업데이트
- Step 4: .prettierrc 생성
- Step 5: eslint.config.js 생성
- Step 6: frontend/package.json 생성
- Step 7: backend/package.json 생성

### ⚠️ 블로커
- 없음

---

## 중요 파일

### 참조 문서

**docs/implementation-plan.md**
- 전체 구현 계획 (15 Phases, 50 Tasks)
- Phase 간 의존성 그래프 포함
- 현재 Phase 1을 진행 중

**docs/TechStack.md**
- 기술 스택 정의
- 의존성 목록 참조용
- 디렉토리 구조 참조용

**docs/Architecture.md**
- 시스템 아키텍처
- 백엔드 레이어드 아키텍처 패턴
- 프론트엔드 기능 기반 모듈 구조

### 생성 예정 파일

| 파일 | 목적 | 상태 |
|------|------|------|
| `pnpm-workspace.yaml` | 워크스페이스 설정 | ⏳ |
| `package.json` | 루트 스크립트 | ⏳ |
| `.gitignore` | Git 무시 파일 | 업데이트 필요 |
| `.prettierrc` | 코드 포맷팅 | ⏳ |
| `eslint.config.js` | 린트 규칙 | ⏳ |
| `frontend/package.json` | 프론트엔드 워크스페이스 | ⏳ |
| `backend/package.json` | 백엔드 워크스페이스 | ⏳ |

---

## 중요 결정 사항

### 1. 패키지 매니저
- **결정**: pnpm 사용
- **이유**: 워크스페이스 지원, 빠른 설치, 디스크 공간 효율

### 2. 워크스페이스 구조
- **결정**: frontend/, backend/ 2개 워크스페이스
- **이유**: 관심사 분리, 독립적 배포 가능

### 3. ESLint 버전
- **결정**: ESLint 9+ (Flat config)
- **이유**: 최신 설정 방식, 향후 호환성

### 4. TypeScript 공유
- **결정**: 루트에 TypeScript 설치, 각 워크스페이스에서 tsconfig 확장
- **이유**: 버전 일관성, 중복 방지

---

## 기술적 제약

1. **Node.js 버전**: 20.x LTS 이상 필요
2. **pnpm 버전**: 8.x 이상 필요
3. **ESLint**: Flat config 형식 사용 (eslint.config.js)

---

## 빠른 재개 지침

**이 작업을 계속하려면:**

1. 이 파일과 `phase1-monorepo-setup-plan.md` 읽기
2. `phase1-monorepo-setup-tasks.md`에서 다음 미완료 태스크 확인
3. 각 태스크의 수락 기준 충족 확인
4. 완료 후 이 context 파일 업데이트

**현재 위치:** dev docs 작성 완료 후 Step 1부터 구현 시작

---

## 관련 링크

- [전체 구현 계획](../../../docs/implementation-plan.md)
- [기술 스택](../../../docs/TechStack.md)
- [아키텍처](../../../docs/Architecture.md)
