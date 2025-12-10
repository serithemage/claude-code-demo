# Phase 1: 모노레포 구조 설정 - Task 체크리스트

## 개요

RealWorld (Conduit) 프로젝트를 위한 pnpm 기반 모노레포 구조 설정.

---

## Step 1: pnpm-workspace.yaml 생성 ⏳ 미시작

- [ ] `pnpm-workspace.yaml` 파일 생성
  - 수락 기준: frontend, backend 워크스페이스 정의됨
  - 참조: plan.md Step 1

**파일 내용:**
```yaml
packages:
  - 'frontend'
  - 'backend'
```

---

## Step 2: 루트 package.json 생성 ⏳ 미시작

- [ ] `package.json` 파일 생성
  - name: "claude-code-demo"
  - private: true

- [ ] scripts 정의
  - [ ] dev: 전체 개발 서버 (concurrently)
  - [ ] dev:frontend: 프론트엔드만
  - [ ] dev:backend: 백엔드만
  - [ ] build: 프로덕션 빌드
  - [ ] test: 전체 테스트
  - [ ] lint: ESLint 실행
  - [ ] format: Prettier 포맷팅

- [ ] devDependencies 정의
  - [ ] concurrently
  - [ ] prettier
  - [ ] eslint
  - [ ] typescript

**수락 기준:**
- `pnpm install` 성공
- 모든 scripts 정의됨

---

## Step 3: .gitignore 업데이트 ⏳ 미시작

- [ ] `.gitignore` 파일 업데이트

**추가 항목:**
- [ ] node_modules/
- [ ] dist/, build/
- [ ] .env, .env.local, .env.*.local
- [ ] .idea/, .vscode/, *.swp, *.swo
- [ ] .DS_Store, Thumbs.db
- [ ] *.log, npm-debug.log*, pnpm-debug.log*
- [ ] *.db, *.sqlite
- [ ] coverage/

**수락 기준:**
- node_modules, .env, dist 등 제외됨

---

## Step 4: .prettierrc 생성 ⏳ 미시작

- [ ] `.prettierrc` 파일 생성

**설정 내용:**
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

## Step 5: ESLint 설정 생성 ⏳ 미시작

- [ ] `eslint.config.js` 파일 생성 (Flat config)
- [ ] TypeScript 지원 설정
- [ ] React 지원 설정 (프론트엔드)
- [ ] 프로젝트 공통 규칙 정의

**수락 기준:**
- `pnpm lint` 실행 가능

---

## Step 6: frontend/package.json 생성 ⏳ 미시작

- [ ] `frontend/` 디렉토리 생성
- [ ] `frontend/package.json` 파일 생성

**파일 내용:**
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
- `pnpm -r list`에서 frontend 표시

---

## Step 7: backend/package.json 생성 ⏳ 미시작

- [ ] `backend/` 디렉토리 생성
- [ ] `backend/package.json` 파일 생성

**파일 내용:**
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
- `pnpm -r list`에서 backend 표시

---

## 최종 검증 ⏳ 미시작

- [ ] `pnpm install` 에러 없이 완료
- [ ] `pnpm -r list`에서 frontend, backend 표시
- [ ] `pnpm format` 실행 가능
- [ ] `pnpm lint` 실행 가능

---

## 빠른 재개

**다음 태스크 찾기:**
1. 위 체크리스트에서 첫 번째 미완료 항목 (`[ ]`) 찾기
2. 해당 Step의 수락 기준 확인
3. 완료 후 체크표시 (`[x]`)로 업데이트

**현재 진행 상황:** Step 1부터 시작 예정
