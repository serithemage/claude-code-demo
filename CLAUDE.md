# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RealWorld (Conduit) - A Medium.com clone social blogging platform. Monorepo structure integrating frontend and backend.

## Development Commands

```bash
# Install dependencies
pnpm install

# Database migration
pnpm --filter backend prisma migrate dev

# Start development server (all)
pnpm dev

# Individual startup
pnpm --filter frontend dev    # Frontend only
pnpm --filter backend dev     # Backend only

# Testing
pnpm test                     # All tests
pnpm --filter backend test    # Backend tests only
pnpm --filter frontend test   # Frontend tests only

# Lint & Format
pnpm lint
pnpm format

# Build
pnpm build
```

## Architecture

### Monorepo Structure
```
├── frontend/          # React 19 + TypeScript + MUI v7 + TanStack Query/Router
├── backend/           # Express + TypeScript + Prisma + SQLite
└── docs/              # Project documentation (PRD, TechStack, Architecture, API-Spec)
```

### Backend - Layered Architecture
```
Routes → Controllers → Services → Repositories → Database (Prisma)
```

Each layer's responsibility:
- **Routes**: Endpoint definition, middleware application
- **Controllers**: Request/response processing, validation
- **Services**: Business logic
- **Repositories**: Data access via Prisma

### Frontend - Feature-Based Modules
```
src/features/     # auth, articles, comments, profiles, tags
src/components/   # Shared components (layout, ui)
src/routes/       # TanStack Router route definitions
src/lib/          # API client, utilities
```

## Tech Stack

| Area | Technology |
|------|------------|
| Frontend | React 19, TypeScript, MUI v7, TanStack Query 5, TanStack Router, Vite |
| Backend | Node.js 20, Express 4, TypeScript, Prisma 5, SQLite |
| Authentication | JWT (localStorage storage) |
| Testing | Vitest, Testing Library, Supertest |

## Claude Code Skills Integration

### Auto-Activation Skills
- **backend-dev-guidelines**: When editing `backend/**/*.ts` files
- **frontend-dev-guidelines**: When editing `frontend/src/**/*.tsx` files (block setting)
- **error-tracking**: Sentry integration patterns
- **route-tester**: API route testing

### Important Constraints
- MUI v7 required: Grid uses `size={{}}` prop (xs, sm forbidden), makeStyles forbidden
- Backend must follow layered architecture
- All Prisma operations executed in Repository layer

## Project Documentation

See `docs/` folder for details:
- `PRD.md` - Functional requirements, screen design
- `TechStack.md` - Technology selection rationale, directory structure
- `Architecture.md` - System design, ERD, authentication flow
- `API-Spec.md` - API endpoint specifications

## Language Setting

- All documentation for this project should be written in English
- Communicate in English
