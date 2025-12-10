# Phase 1: 모노레포 구조 설정 - 구현 계획

## 요약

RealWorld (Conduit) 프로젝트를 위한 pnpm 기반 모노레포 구조를 설정합니다.
frontend/와 backend/ 두 개의 워크스페이스로 구성되며, 공통 개발 도구 설정을 포함합니다.

---

## 현재 상태

- `try1` 브랜치에서 작업 중
- `docs/implementation-plan.md` 작성 완료
- frontend/, backend/ 디렉토리 미존재
- 루트 package.json, pnpm-workspace.yaml 미존재

---

## 목표 상태

```
claude-code-demo/
├── pnpm-workspace.yaml          # 워크스페이스 설정
├── package.json                 # 루트 package.json (scripts)
├── .gitignore                   # 업데이트됨
├── .prettierrc                  # Prettier 설정
├── .eslintrc.js                 # ESLint 설정
├── frontend/                    # 프론트엔드 워크스페이스
│   └── package.json
├── backend/                     # 백엔드 워크스페이스
│   └── package.json
└── ...
```

---

## 구현 단계

### Step 1: pnpm-workspace.yaml 생성

**파일:** `pnpm-workspace.yaml`

```yaml
packages:
  - 'frontend'
  - 'backend'
```

**수락 기준:**

- pnpm이 frontend, backend를 워크스페이스로 인식

---

### Step 2: 루트 package.json 생성

**파일:** `package.json`

**포함 내용:**

- name: "claude-code-demo"
- private: true
- scripts:
  - `dev`: 전체 개발 서버 (concurrently)
  - `dev:frontend`: 프론트엔드만
  - `dev:backend`: 백엔드만
  - `build`: 프로덕션 빌드
  - `test`: 전체 테스트
  - `lint`: ESLint 실행
  - `format`: Prettier 포맷팅
- devDependencies:
  - concurrently (병렬 실행)
  - prettier
  - eslint
  - typescript

**수락 기준:**

- `pnpm install` 성공
- 모든 scripts 정의됨

---

### Step 3: .gitignore 업데이트

**추가 항목:**

```
# Dependencies
node_modules/

# Build outputs
dist/
build/

# Environment
.env
.env.local
.env.*.local

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*
pnpm-debug.log*

# Database
*.db
*.sqlite

# Coverage
coverage/
```

**수락 기준:**

- node_modules, .env, dist 등 제외됨

---

### Step 4: .prettierrc 생성

**파일:** `.prettierrc`

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

**수락 기준:**

- `pnpm format` 실행 가능

---

### Step 5: ESLint 설정 생성

**파일:** `eslint.config.js` (Flat config - ESLint 9+)

**설정 내용:**

- TypeScript 지원
- React 지원 (프론트엔드)
- 프로젝트 공통 규칙

**수락 기준:**

- `pnpm lint` 실행 가능

---

### Step 6: frontend/package.json 생성 (기본)

**파일:** `frontend/package.json`

```json
{
  "name": "frontend",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "echo 'Frontend not yet configured'",
    "build": "echo 'Frontend not yet configured'",
    "test": "echo 'Frontend not yet configured'"
  }
}
```

**수락 기준:**

- pnpm workspace에서 인식됨

---

### Step 7: backend/package.json 생성 (기본)

**파일:** `backend/package.json`

```json
{
  "name": "backend",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "echo 'Backend not yet configured'",
    "build": "echo 'Backend not yet configured'",
    "test": "echo 'Backend not yet configured'"
  }
}
```

**수락 기준:**

- pnpm workspace에서 인식됨

---

## 리스크 평가

| 리스크              | 영향도 | 대응책                |
| ------------------- | ------ | --------------------- |
| pnpm 미설치         | 높음   | 설치 가이드 제공      |
| Node.js 버전 불일치 | 중간   | .nvmrc 파일 추가 고려 |
| ESLint 설정 충돌    | 낮음   | 기본 설정 사용        |

---

## 성공 지표

- [ ] `pnpm install` 에러 없이 완료
- [ ] `pnpm -r list` 에서 frontend, backend 표시
- [ ] `pnpm format` 실행 가능
- [ ] `pnpm lint` 실행 가능

---

## 다음 단계

Phase 1 완료 후:

- Task 1.2: 백엔드 프로젝트 초기화 (Express, TypeScript, Prisma)
- Task 1.3: 프론트엔드 프로젝트 초기화 (Vite, React 19, MUI)
