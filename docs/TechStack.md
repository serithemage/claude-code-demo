# RealWorld (Conduit) - Tech Stack

## 1. Overview

This project uses a monorepo structure to integrate Frontend and Backend management.
We adopt a proven toolchain with efficient development with Claude Code as the top priority.

---

## 2. Frontend Tech Stack

### 2.1 Core Framework

| Technology | Version | Purpose |
|------------|---------|---------|
| **React** | 19.x | UI Framework |
| **TypeScript** | 5.x | Type Safety |
| **Vite** | 6.x | Build Tool & Dev Server |

### 2.2 State Management & Data Fetching

| Technology | Version | Purpose |
|------------|---------|---------|
| **TanStack Query** | 5.x | Server State Management, Data Fetching |
| **TanStack Router** | 1.x | File-based Routing |

### 2.3 UI Library

| Technology | Version | Purpose |
|------------|---------|---------|
| **MUI (Material UI)** | 7.x | UI Components |
| **@emotion/react** | 11.x | CSS-in-JS |
| **@emotion/styled** | 11.x | Styled Components |

### 2.4 Utilities

| Technology | Purpose |
|------------|---------|
| **react-markdown** | Markdown Rendering |
| **date-fns** | Date Formatting |
| **zod** | Schema Validation |

### 2.5 Selection Rationale

- **React 19**: Leverage latest Suspense, concurrent rendering features
- **TanStack Query**: Declarative data fetching with useSuspenseQuery
- **TanStack Router**: Type-safe file-based routing
- **MUI v7**: Latest design system, good accessibility
- **Vite**: Fast HMR, optimized builds

---

## 3. Backend Tech Stack

### 3.1 Runtime & Framework

| Technology | Version | Purpose |
|------------|---------|---------|
| **Node.js** | 20.x LTS | JavaScript Runtime |
| **Express** | 4.x | Web Framework |
| **TypeScript** | 5.x | Type Safety |

### 3.2 Database & ORM

| Technology | Version | Purpose |
|------------|---------|---------|
| **SQLite** | 3.x | Database |
| **Prisma** | 5.x | ORM |

### 3.3 Authentication & Security

| Technology | Purpose |
|------------|---------|
| **jsonwebtoken** | JWT Token Generation & Verification |
| **bcryptjs** | Password Hashing |
| **cors** | CORS Configuration |
| **helmet** | Security Headers |

### 3.4 Validation & Utilities

| Technology | Purpose |
|------------|---------|
| **zod** | Request Validation |
| **slugify** | URL Slug Generation |
| **uuid** | Unique Identifier Generation |

### 3.5 Architecture Pattern

Adopting **Layered Architecture**:

```
Routes → Controllers → Services → Repositories → Database
```

| Layer | Responsibility |
|-------|----------------|
| **Routes** | Endpoint Definition, Middleware Application |
| **Controllers** | Request/Response Processing, Validation |
| **Services** | Business Logic |
| **Repositories** | Data Access (Prisma Operations) |

### 3.6 Selection Rationale

- **Express**: Simple and flexible, rich ecosystem
- **Prisma**: Type-safe ORM, migration management
- **SQLite**: Simplified setup for development environment
- **Layered Architecture**: Separation of concerns, improved testability

---

## 4. Common Tools

### 4.1 Development Tools

| Technology | Purpose |
|------------|---------|
| **pnpm** | Package Manager (Workspace Support) |
| **ESLint** | Code Quality Check |
| **Prettier** | Code Formatting |
| **tsx** | TypeScript Execution (Development) |

### 4.2 Testing

| Technology | Purpose |
|------------|---------|
| **Vitest** | Unit Tests & Integration Tests |
| **Testing Library** | React Component Testing |
| **Supertest** | API Endpoint Testing |

### 4.3 Process Management & Monitoring

| Technology | Purpose |
|------------|---------|
| **PM2** | Process Management, Log Monitoring |
| **Sentry** | Error Tracking, Performance Monitoring |

---

## 5. Directory Structure

```
claude-code-demo/
├── docs/                          # Project Documentation
│   ├── PRD.md
│   ├── TechStack.md
│   ├── Architecture.md
│   └── API-Spec.md
│
├── frontend/                      # Frontend Application
│   ├── src/
│   │   ├── features/              # Feature-based Modules
│   │   │   ├── auth/              # Authentication Feature
│   │   │   ├── articles/          # Articles Feature
│   │   │   ├── comments/          # Comments Feature
│   │   │   ├── profiles/          # Profiles Feature
│   │   │   └── tags/              # Tags Feature
│   │   ├── components/            # Shared Components
│   │   │   ├── layout/            # Layout
│   │   │   └── ui/                # UI Parts
│   │   ├── hooks/                 # Custom Hooks
│   │   ├── lib/                   # Utilities
│   │   │   ├── api/               # API Client
│   │   │   └── utils/             # Helper Functions
│   │   ├── routes/                # Route Definitions (TanStack Router)
│   │   ├── types/                 # Type Definitions
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── public/
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── package.json
│
├── backend/                       # Backend Application
│   ├── src/
│   │   ├── routes/                # Route Definitions
│   │   ├── controllers/           # Controllers
│   │   ├── services/              # Business Logic
│   │   ├── repositories/          # Data Access
│   │   ├── middleware/            # Middleware
│   │   │   ├── auth.ts            # Authentication Middleware
│   │   │   ├── errorHandler.ts    # Error Handling
│   │   │   └── validation.ts      # Validation
│   │   ├── lib/                   # Utilities
│   │   │   ├── prisma.ts          # Prisma Client
│   │   │   └── jwt.ts             # JWT Utilities
│   │   ├── types/                 # Type Definitions
│   │   └── index.ts               # Entry Point
│   ├── prisma/
│   │   ├── schema.prisma          # Database Schema
│   │   └── migrations/            # Migrations
│   ├── tsconfig.json
│   └── package.json
│
├── .claude/                       # Claude Code Configuration (Existing)
├── dev/                           # Dev Docs (Existing)
├── pnpm-workspace.yaml            # Workspace Configuration
├── package.json                   # Root package.json
├── .prettierrc                    # Prettier Configuration
├── .eslintrc.js                   # ESLint Configuration
└── CLAUDE.md                      # Project Configuration (Existing)
```

---

## 6. Development Environment Setup

### 6.1 Prerequisites

- Node.js 20.x or higher
- pnpm 8.x or higher

### 6.2 Initial Setup

```bash
# Install dependencies
pnpm install

# Database migration
pnpm --filter backend prisma migrate dev

# Start development server (Frontend + Backend)
pnpm dev
```

### 6.3 Available Scripts

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start development server (all) |
| `pnpm --filter frontend dev` | Start frontend only |
| `pnpm --filter backend dev` | Start backend only |
| `pnpm build` | Production build |
| `pnpm test` | Run tests |
| `pnpm lint` | Lint check |
| `pnpm format` | Code formatting |

---

## 7. Environment Variables

### 7.1 Backend (.env)

```env
# Database
DATABASE_URL="file:./dev.db"

# JWT
JWT_SECRET="your-secret-key"
JWT_EXPIRES_IN="7d"

# Server
PORT=3000
NODE_ENV="development"

# Sentry (Optional)
SENTRY_DSN=""
```

### 7.2 Frontend (.env)

```env
# API
VITE_API_URL="http://localhost:3000/api"
```

---

## 8. Dependency List

### 8.1 Frontend

```json
{
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "@tanstack/react-query": "^5.0.0",
    "@tanstack/react-router": "^1.0.0",
    "@mui/material": "^7.0.0",
    "@emotion/react": "^11.0.0",
    "@emotion/styled": "^11.0.0",
    "react-markdown": "^9.0.0",
    "date-fns": "^3.0.0",
    "zod": "^3.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vite": "^6.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "vitest": "^2.0.0",
    "@testing-library/react": "^16.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0"
  }
}
```

### 8.2 Backend

```json
{
  "dependencies": {
    "express": "^4.21.0",
    "@prisma/client": "^5.0.0",
    "jsonwebtoken": "^9.0.0",
    "bcryptjs": "^2.4.0",
    "cors": "^2.8.0",
    "helmet": "^8.0.0",
    "zod": "^3.0.0",
    "slugify": "^1.6.0",
    "uuid": "^10.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "prisma": "^5.0.0",
    "tsx": "^4.0.0",
    "vitest": "^2.0.0",
    "supertest": "^7.0.0",
    "@types/express": "^5.0.0",
    "@types/node": "^22.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0"
  }
}
```

---

## 9. Claude Code Integration

### 9.1 Using Existing Skills

This project integrates with existing Claude Code skills:

- **backend-dev-guidelines**: Express/TypeScript patterns
- **frontend-dev-guidelines**: React/MUI patterns
- **error-tracking**: Sentry integration
- **route-tester**: API testing

### 9.2 Development Flow

1. Review functional requirements (refer to PRD)
2. API design (refer to API-Spec)
3. Backend implementation (Layered Architecture)
4. Frontend implementation (Feature-based modules)
5. Run tests
6. Update Dev Docs
