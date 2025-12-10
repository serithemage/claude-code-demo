# Claude Code Demo - Agentic Coding 학습 프로젝트

[Claude Code is a Beast](https://www.youtube.com/watch?v=...) 영상에서 소개된 고급 설정들을 실제 애플리케이션 구현을 통해 경험하는 학습 프로젝트입니다.

## 이 프로젝트의 목적

**[RealWorld (Conduit)](https://realworld-docs.netlify.app/)** 앱 - Medium.com 클론 블로그 플랫폼 - 을 구현하면서 다양한 Claude Code 기능을 실습합니다:

- **Skills**: context 기반 자동 활성화 가이드라인
- **Hooks**: 도구 사용 전후 자동 실행 스크립트
- **Custom Agents**: 특화된 작업을 위한 커스텀 에이전트
- **Slash Commands**: 반복 작업을 위한 커스텀 명령어

---

## 프로젝트 구조

```
claude-code-demo/
├── .claude/                    # Claude Code 설정
│   ├── settings.json           # 권한, hooks 설정
│   ├── skills/                 # 자동 활성화 skills
│   │   ├── backend-dev-guidelines/
│   │   ├── frontend-dev-guidelines/
│   │   ├── route-tester/
│   │   ├── error-tracking/
│   │   └── skill-rules.json    # Skill 활성화 규칙
│   ├── hooks/                  # 자동화 hook 스크립트
│   ├── agents/                 # 커스텀 에이전트 정의
│   └── commands/               # Slash command 정의
├── docs/                       # 상위 레벨 설계 문서
│   ├── PRD.md                  # 제품 요구사항
│   ├── TechStack.md            # 기술 스택
│   ├── Architecture.md         # 시스템 아키텍처
│   └── API-Spec.md             # API 명세
├── frontend/                   # React 19 + TypeScript + MUI v7
├── backend/                    # Express + TypeScript + Prisma
└── CLAUDE.md                   # Claude Code 메인 가이드라인
```

### docs/와 .claude/skills/의 관계

| 폴더 | 역할 | 대상 |
|------|------|------|
| `docs/` | 상위 레벨 설계 문서 | 사람이 읽는 문서 |
| `.claude/skills/` | 에이전트 실행 가이드라인 | Claude를 위한 가이드라인 |

`docs/`의 내용은 `.claude/skills/`의 리소스 문서로 변환되어 Claude가 코딩 시 참조합니다.

---

## 핵심 개념

### 1. Skills

**Skills**는 Claude가 특정 작업을 수행할 때 자동으로 로드되는 지식 베이스입니다.

```
.claude/skills/
├── backend-dev-guidelines/     # 백엔드 개발 가이드라인
│   ├── SKILL.md                # 메인 skill 파일
│   └── resources/              # 상세 리소스 문서
│       ├── architecture-overview.md
│       ├── routing-and-controllers.md
│       └── ...
├── frontend-dev-guidelines/    # 프론트엔드 개발 가이드라인
└── skill-rules.json            # Skill 활성화 규칙
```

**자동 활성화 방법** (`skill-rules.json`):
- `pathPatterns`: 특정 파일 편집 시 (`backend/**/*.ts`)
- `keywords`: 프롬프트에 키워드 포함 시 (`"backend"`, `"API"`)
- `intentPatterns`: 사용자 의도 패턴 매칭

### 2. Hooks

**Hooks**는 특정 이벤트 발생 시 자동 실행되는 스크립트입니다.

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

**이 프로젝트의 Hooks**:
- `skill-activation-prompt.sh`: 프롬프트 분석 후 관련 skill 추천
- `post-tool-use-tracker.sh`: 파일 편집 추적
- `tsc-check.sh`: TypeScript 컴파일 검사

### 3. Custom Agents

**Custom Agents**는 특화된 작업을 위해 정의된 AI 페르소나입니다.

```
.claude/agents/
├── auth-route-debugger.md      # 인증 문제 디버깅
├── code-refactor-master.md     # 코드 리팩토링
├── frontend-error-fixer.md     # 프론트엔드 에러 수정
└── ...
```

### 4. Slash Commands

**Slash Commands**는 반복 작업을 위한 커스텀 명령어입니다.

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
# 저장소 복제
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

# 또는 특정 작업과 함께 시작
claude "백엔드에 새 API 엔드포인트 추가해줘"
```

---

## 학습 가이드

### 레벨 1: 기본 구조 이해

1. **CLAUDE.md 읽기**: 전체 프로젝트 가이드라인 확인
2. **docs/ 탐색**: 상위 레벨 설계 문서 이해
3. **settings.json 확인**: 권한 및 hook 설정 이해

### 레벨 2: Skills 경험

1. `backend/` 폴더에서 `.ts` 파일 편집 시도
2. `backend-dev-guidelines` skill이 자동 활성화되는지 확인
3. `skill-rules.json`에서 활성화 조건 분석

```bash
# 예시: 백엔드 파일 편집
claude "backend/src/routes/에 새 route 추가해줘"
# → backend-dev-guidelines skill 자동 활성화
```

### 레벨 3: Hooks 분석

1. `.claude/hooks/` 폴더의 스크립트 분석
2. `settings.json`의 hooks 설정과 매핑 이해
3. Hook 실행 로그 관찰

### 레벨 4: 커스터마이징

1. 새 skill 추가 시도
2. `skill-rules.json`에 활성화 규칙 추가
3. 커스텀 slash command 생성

---

## 기술 스택

| 영역 | 기술 |
|------|------|
| **프론트엔드** | React 19, TypeScript, MUI v7, TanStack Query/Router, Vite |
| **백엔드** | Node.js 20, Express 4, TypeScript, Prisma 5, SQLite |
| **인증** | JWT (localStorage 저장) |
| **테스팅** | Vitest, Testing Library, Supertest |

---

## 주요 파일 설명

| 파일 | 설명 |
|------|------|
| `CLAUDE.md` | 프로젝트 작업 시 Claude Code가 참조하는 메인 가이드라인 |
| `.claude/settings.json` | 권한, MCP 서버, hook 설정 |
| `.claude/skills/skill-rules.json` | Skill 자동 활성화 규칙 정의 |
| `docs/Architecture.md` | 시스템 아키텍처 (Mermaid 다이어그램 포함) |

---

## 유용한 명령어

```bash
# 개발 서버
pnpm dev                        # 전체 (프론트엔드 + 백엔드)
pnpm --filter frontend dev      # 프론트엔드만
pnpm --filter backend dev       # 백엔드만

# 테스팅
pnpm test                       # 전체 테스트
pnpm --filter backend test      # 백엔드 테스트만

# 빌드
pnpm build                      # 프로덕션 빌드

# 린트/포맷
pnpm lint
pnpm format
```

---

## 언어 버전

이 프로젝트는 다국어로 제공됩니다:

| 브랜치 | 언어 | 링크 |
|--------|------|------|
| `main` | English | [View](https://github.com/serithemage/claude-code-demo/tree/main) |
| `korean` | 한국어 (현재) | [View](https://github.com/serithemage/claude-code-demo/tree/korean) |
| `japanese` | 日本語 | [View](https://github.com/serithemage/claude-code-demo/tree/japanese) |

---

## 참고 자료

- [Claude Code 공식 문서](https://docs.anthropic.com/claude-code)
- [RealWorld 사양](https://realworld-docs.netlify.app/)
- [Skills 상세 가이드](.claude/skills/README.md)
- [Hooks 설정 가이드](.claude/hooks/README.md)
- [Agents 가이드](.claude/agents/README.md)

---

## 라이선스

MIT License

---

## 기여하기

Issue와 PR을 환영합니다. Agentic Coding 학습에 도움이 될 개선 사항을 제안해 주세요.
