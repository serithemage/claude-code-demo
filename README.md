# Claude Code Demo - Agentic Coding Learning Project

A learning project to experience the advanced configurations introduced in the [Claude Code is a Beast](https://www.youtube.com/watch?v=...) video through actual application implementation.

## Purpose of This Project

Implement the **[RealWorld (Conduit)](https://realworld-docs.netlify.app/)** app - a Medium.com clone blog platform - while practicing various Claude Code features:

- **Skills**: Context-based auto-activation guidelines
- **Hooks**: Auto-execution scripts before/after tool usage
- **Custom Agents**: Custom agents for specialized tasks
- **Slash Commands**: Custom commands for repetitive tasks

---

## Project Structure

```
claude-code-demo/
├── .claude/                    # Claude Code Configuration
│   ├── settings.json           # Permissions, hooks settings
│   ├── skills/                 # Auto-activation skills
│   │   ├── backend-dev-guidelines/
│   │   ├── frontend-dev-guidelines/
│   │   ├── route-tester/
│   │   ├── error-tracking/
│   │   └── skill-rules.json    # Skill activation rules
│   ├── hooks/                  # Automation hook scripts
│   ├── agents/                 # Custom agent definitions
│   └── commands/               # Slash command definitions
├── docs/                       # High-level design documents
│   ├── PRD.md                  # Product Requirements
│   ├── TechStack.md            # Tech Stack
│   ├── Architecture.md         # System Architecture
│   └── API-Spec.md             # API Specification
├── frontend/                   # React 19 + TypeScript + MUI v7
├── backend/                    # Express + TypeScript + Prisma
└── CLAUDE.md                   # Claude Code main guidelines
```

### Relationship between docs/ and .claude/skills/

| Folder | Role | Target |
|--------|------|--------|
| `docs/` | High-level design documents | Human-readable documents |
| `.claude/skills/` | Agent execution guidelines | Guidelines for Claude |

Content from `docs/` is transformed into resource documents in `.claude/skills/` for Claude to reference during coding.

---

## Core Concepts

### 1. Skills

**Skills** are knowledge bases that are automatically loaded when Claude performs specific tasks.

```
.claude/skills/
├── backend-dev-guidelines/     # Backend Development Guidelines
│   ├── SKILL.md                # Main skill file
│   └── resources/              # Detailed resource documents
│       ├── architecture-overview.md
│       ├── routing-and-controllers.md
│       └── ...
├── frontend-dev-guidelines/    # Frontend Development Guidelines
└── skill-rules.json            # Skill activation rules
```

**Auto-activation methods** (`skill-rules.json`):
- `pathPatterns`: When editing specific files (`backend/**/*.ts`)
- `keywords`: When prompt contains keywords (`"backend"`, `"API"`)
- `intentPatterns`: User intent pattern matching

### 2. Hooks

**Hooks** are scripts that automatically execute when specific events occur.

```json
// .claude/settings.json
{
  "hooks": {
    "UserPromptSubmit": [...],  // On prompt submission
    "PostToolUse": [...],       // After tool usage
    "Stop": [...]               // On work completion
  }
}
```

**Hooks in this project**:
- `skill-activation-prompt.sh`: Recommend relevant skills after prompt analysis
- `post-tool-use-tracker.sh`: Track file edits
- `tsc-check.sh`: TypeScript compile check

### 3. Custom Agents

**Custom Agents** are AI personas defined for specialized tasks.

```
.claude/agents/
├── auth-route-debugger.md      # Authentication issue debugging
├── code-refactor-master.md     # Code refactoring
├── frontend-error-fixer.md     # Frontend error fixing
└── ...
```

### 4. Slash Commands

**Slash Commands** are custom commands for repetitive tasks.

```
.claude/commands/
├── dev-docs.md                 # Generate dev docs
├── dev-docs-update.md          # Update dev docs
└── route-research-for-testing.md
```

Usage: `/dev-docs auth system implementation plan`

---

## Getting Started

### Prerequisites

- Node.js 20+
- pnpm
- Claude Code CLI ([Installation Guide](https://claude.ai/code))

### Installation

```bash
# Clone repository
git clone https://github.com/serithemage/claude-code-demo.git
cd claude-code-demo

# Install dependencies
pnpm install

# Database migration
pnpm --filter backend prisma migrate dev

# Start development server
pnpm dev
```

### Running Claude Code

```bash
# In project directory
claude

# Or start with specific task
claude "Add a new API endpoint to the backend"
```

---

## Learning Guide

### Level 1: Understanding Basic Structure

1. **Read CLAUDE.md**: Check overall project guidelines
2. **Explore docs/**: Understand high-level design documents
3. **Check settings.json**: Understand permissions and hook settings

### Level 2: Experiencing Skills

1. Try editing a `.ts` file in the `backend/` folder
2. Confirm that `backend-dev-guidelines` skill activates automatically
3. Analyze activation conditions in `skill-rules.json`

```bash
# Example: Edit backend file
claude "Add a new route to backend/src/routes/"
# → backend-dev-guidelines skill auto-activates
```

### Level 3: Analyzing Hooks

1. Analyze scripts in `.claude/hooks/` folder
2. Understand mapping with hooks settings in `settings.json`
3. Observe hook execution logs

### Level 4: Customization

1. Try adding a new skill
2. Add activation rules to `skill-rules.json`
3. Create custom slash commands

---

## Tech Stack

| Area | Technology |
|------|------------|
| **Frontend** | React 19, TypeScript, MUI v7, TanStack Query/Router, Vite |
| **Backend** | Node.js 20, Express 4, TypeScript, Prisma 5, SQLite |
| **Authentication** | JWT (localStorage storage) |
| **Testing** | Vitest, Testing Library, Supertest |

---

## Key File Descriptions

| File | Description |
|------|-------------|
| `CLAUDE.md` | Main guidelines Claude Code references when working on project |
| `.claude/settings.json` | Permissions, MCP servers, hook settings |
| `.claude/skills/skill-rules.json` | Skill auto-activation rule definitions |
| `docs/Architecture.md` | System architecture (includes Mermaid diagrams) |

---

## Useful Commands

```bash
# Development server
pnpm dev                        # All (frontend + backend)
pnpm --filter frontend dev      # Frontend only
pnpm --filter backend dev       # Backend only

# Testing
pnpm test                       # All tests
pnpm --filter backend test      # Backend tests only

# Build
pnpm build                      # Production build

# Lint/Format
pnpm lint
pnpm format
```

---

## Language Versions

This project is available in multiple languages:

| Branch | Language | Link |
|--------|----------|------|
| `main` | English (Current) | [View](https://github.com/serithemage/claude-code-demo/tree/main) |
| `korean` | Korean (한국어) | [View](https://github.com/serithemage/claude-code-demo/tree/korean) |
| `japanese` | Japanese (日本語) | [View](https://github.com/serithemage/claude-code-demo/tree/japanese) |

---

## References

- [Claude Code Official Documentation](https://docs.anthropic.com/claude-code)
- [RealWorld Spec](https://realworld-docs.netlify.app/)
- [Skills Detailed Guide](.claude/skills/README.md)
- [Hooks Configuration Guide](.claude/hooks/README.md)
- [Agents Guide](.claude/agents/README.md)

---

## License

MIT License

---

## Contributing

Issues and PRs are welcome. Please suggest any improvements that would help with Agentic Coding learning.
