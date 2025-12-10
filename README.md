# Claude Code Demo - Agentic Coding 학습 프로젝트

[Claude Code is a Beast](https://www.youtube.com/watch?v=...) 영상에서 소개된 고급 설정들을 실제 애플리케이션 구현을 통해 체험해 볼 수 있는 학습용 프로젝트입니다.

## 이 프로젝트의 목적

**RealWorld (Conduit)** 앱 - Medium.com 클론 블로그 플랫폼 - 을 구현하면서 Claude Code의 다양한 기능을 실습합니다:

- **Skills**: 컨텍스트 기반 자동 활성화 지침
- **Hooks**: 도구 사용 전/후 자동 실행 스크립트
- **Custom Agents**: 특화된 작업을 위한 커스텀 에이전트
- **Slash Commands**: 반복 작업을 위한 커스텀 명령어

---

## 프로젝트 구조

```
claude-code-demo/
├── .claude/                    # Claude Code 설정
│   ├── settings.json           # 권한, hooks 설정
│   ├── skills/                 # 자동 활성화 스킬들
│   │   ├── backend-dev-guidelines/
│   │   ├── frontend-dev-guidelines/
│   │   ├── route-tester/
│   │   ├── error-tracking/
│   │   └── skill-rules.json    # 스킬 활성화 규칙
│   ├── hooks/                  # 자동화 훅 스크립트
│   ├── agents/                 # 커스텀 에이전트 정의
│   └── commands/               # 슬래시 커맨드 정의
├── docs/                       # 하이레벨 설계 문서
│   ├── PRD.md                  # 제품 요구사항
│   ├── TechStack.md            # 기술 스택
│   ├── Architecture.md         # 시스템 아키텍처
│   └── API-Spec.md             # API 명세
├── frontend/                   # React 19 + TypeScript + MUI v7
├── backend/                    # Express + TypeScript + Prisma
└── CLAUDE.md                   # Claude Code 메인 지침
```

### docs/ vs .claude/skills/ 의 관계

| 폴더 | 역할 | 대상 |
|------|------|------|
| `docs/` | 하이레벨 설계 문서 | 사람이 읽는 문서 |
| `.claude/skills/` | 에이전트 실행 지침 | Claude가 사용하는 지침 |

`docs/`의 설계 내용을 `.claude/skills/`의 리소스 문서로 변환하여 Claude가 코딩 시 참조할 수 있게 합니다.

---

## 핵심 개념

### 1. Skills (스킬)

**스킬**은 Claude가 특정 작업을 수행할 때 자동으로 로드되는 지식 베이스입니다.

```
.claude/skills/
├── backend-dev-guidelines/     # 백엔드 개발 가이드라인
│   ├── SKILL.md                # 메인 스킬 파일
│   └── resources/              # 상세 리소스 문서
│       ├── architecture-overview.md
│       ├── routing-and-controllers.md
│       └── ...
├── frontend-dev-guidelines/    # 프론트엔드 개발 가이드라인
└── skill-rules.json            # 스킬 활성화 규칙
```

**자동 활성화 방식** (`skill-rules.json`):
- `pathPatterns`: 특정 파일 편집 시 (`backend/**/*.ts`)
- `keywords`: 프롬프트에 키워드 포함 시 (`"backend"`, `"API"`)
- `intentPatterns`: 사용자 의도 패턴 매칭

### 2. Hooks (훅)

**훅**은 특정 이벤트 발생 시 자동으로 실행되는 스크립트입니다.

```json
// .claude/settings.json
{
  "hooks": {
    "UserPromptSubmit": [...],  // 프롬프트 제출 시
    "PostToolUse": [...],       // 도구 사용 후
    "Stop": [...]               // 작업 완료 시
  }
}
```

**이 프로젝트의 훅들**:
- `skill-activation-prompt.sh`: 프롬프트 분석 후 관련 스킬 추천
- `post-tool-use-tracker.sh`: 파일 편집 추적
- `tsc-check.sh`: TypeScript 컴파일 체크

### 3. Custom Agents (커스텀 에이전트)

**커스텀 에이전트**는 특화된 작업을 위해 정의된 AI 페르소나입니다.

```
.claude/agents/
├── auth-route-debugger.md      # 인증 문제 디버깅
├── code-refactor-master.md     # 코드 리팩토링
├── frontend-error-fixer.md     # 프론트엔드 에러 수정
└── ...
```

### 4. Slash Commands (슬래시 커맨드)

**슬래시 커맨드**는 반복 작업을 위한 커스텀 명령어입니다.

```
.claude/commands/
├── dev-docs.md                 # 개발 문서 생성
├── dev-docs-update.md          # 개발 문서 업데이트
└── route-research-for-testing.md
```

사용법: `/dev-docs 인증 시스템 구현 계획`

---

## 시작하기

### 사전 요구사항

- Node.js 20+
- pnpm
- Claude Code CLI ([설치 가이드](https://claude.ai/code))

### 설치

```bash
# 저장소 클론
git clone https://github.com/serithemage/claude-code-demo.git
cd claude-code-demo

# 의존성 설치
pnpm install

# 데이터베이스 마이그레이션
pnpm --filter backend prisma migrate dev

# 개발 서버 시작
pnpm dev
```

### Claude Code 실행

```bash
# 프로젝트 디렉토리에서
claude

# 또는 특정 작업 시작
claude "백엔드에 새로운 API 엔드포인트 추가해줘"
```

---

## 학습 가이드

### 레벨 1: 기본 구조 이해

1. **CLAUDE.md 읽기**: 프로젝트 전체 지침 확인
2. **docs/ 탐색**: 하이레벨 설계 문서 이해
3. **settings.json 확인**: 권한과 훅 설정 이해

### 레벨 2: Skills 체험

1. `backend/` 폴더에서 `.ts` 파일 편집 시도
2. 자동으로 `backend-dev-guidelines` 스킬이 활성화되는지 확인
3. `skill-rules.json`에서 활성화 조건 분석

```bash
# 예시: 백엔드 파일 편집
claude "backend/src/routes/ 에 새로운 라우트 추가해줘"
# → backend-dev-guidelines 스킬 자동 활성화
```

### 레벨 3: Hooks 분석

1. `.claude/hooks/` 폴더의 스크립트 분석
2. `settings.json`의 hooks 설정과 매핑 이해
3. 훅 실행 로그 관찰

### 레벨 4: 커스터마이징

1. 새로운 스킬 추가해보기
2. `skill-rules.json`에 활성화 규칙 추가
3. 커스텀 슬래시 커맨드 만들기

---

## 기술 스택

| 영역 | 기술 |
|------|------|
| **Frontend** | React 19, TypeScript, MUI v7, TanStack Query/Router, Vite |
| **Backend** | Node.js 20, Express 4, TypeScript, Prisma 5, SQLite |
| **인증** | JWT (localStorage 저장) |
| **테스트** | Vitest, Testing Library, Supertest |

---

## 주요 파일 설명

| 파일 | 설명 |
|------|------|
| `CLAUDE.md` | Claude Code가 프로젝트 작업 시 참조하는 메인 지침 |
| `.claude/settings.json` | 권한, MCP 서버, 훅 설정 |
| `.claude/skills/skill-rules.json` | 스킬 자동 활성화 규칙 정의 |
| `docs/Architecture.md` | 시스템 아키텍처 (Mermaid 다이어그램 포함) |

---

## 유용한 명령어

```bash
# 개발 서버
pnpm dev                        # 전체 (프론트엔드 + 백엔드)
pnpm --filter frontend dev      # 프론트엔드만
pnpm --filter backend dev       # 백엔드만

# 테스트
pnpm test                       # 전체 테스트
pnpm --filter backend test      # 백엔드 테스트만

# 빌드
pnpm build                      # 프로덕션 빌드

# 린트/포맷
pnpm lint
pnpm format
```

---

## 참고 자료

- [Claude Code 공식 문서](https://docs.anthropic.com/claude-code)
- [RealWorld 스펙](https://realworld-docs.netlify.app/)
- [Skills 상세 가이드](.claude/skills/README.md)
- [Hooks 설정 가이드](.claude/hooks/README.md)
- [Agents 가이드](.claude/agents/README.md)

---

## 라이선스

MIT License

---

## 기여

이슈와 PR을 환영합니다. Agentic Coding 학습에 도움이 되는 개선사항이 있다면 제안해주세요.
