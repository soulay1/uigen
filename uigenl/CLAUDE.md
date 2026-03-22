# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start dev server with Turbopack
npm run build        # Build for production
npm run lint         # Run ESLint
npm run test         # Run all Vitest tests
npm run setup        # Install deps + generate Prisma client + run migrations
npm run db:reset     # Reset SQLite database
```

Run a single test file:
```bash
npx vitest run src/components/chat/__tests__/ChatInterface.test.tsx
```

## Architecture

**UIGen** is a Next.js 15 App Router app that generates React components via Claude AI with a live preview.

### Core Data Flow

1. User sends prompt in the chat panel
2. `POST /api/chat` streams a Claude response using Vercel AI SDK
3. Claude calls `str_replace_editor` or `file_manager` tools to write files
4. The in-memory `VirtualFileSystem` is updated via `FileSystemContext`
5. `PreviewFrame` transforms JSX to HTML via Babel Standalone in an `<iframe>`
6. On completion, project is saved to SQLite via Prisma (authenticated users only)

### Key Directories

- `src/app/` — Next.js routes. `api/chat/route.ts` is the AI streaming endpoint.
- `src/components/` — UI split into `chat/`, `editor/`, `preview/`, `auth/` subdirectories plus `ui/` (shadcn).
- `src/lib/` — Core logic:
  - `file-system.ts` — `VirtualFileSystem` class (in-memory, serialized to JSON for DB)
  - `contexts/` — React Context for file system state and chat messages
  - `tools/` — AI tool implementations (`str-replace.ts`, `file-manager.ts`)
  - `transform/jsx-transformer.ts` — Converts JSX files to runnable HTML with import maps
  - `prompts/generation.tsx` — System prompt for component generation
  - `provider.ts` — LLM abstraction (Anthropic or Mock when no API key)
- `src/actions/` — Next.js Server Actions for auth and project CRUD
- `prisma/schema.prisma` — SQLite schema with `User` and `Project` models

### State Management

React Context only (no Redux/Zustand):
- `FileSystemContext` — virtual file tree, currently open file, file operations
- `ChatContext` — message list and streaming state

### AI Model

Uses `claude-haiku-4-5` via `@ai-sdk/anthropic`. A mock provider returns static code when `ANTHROPIC_API_KEY` is not set.

### Authentication

Server Actions (`src/actions/index.ts`) with bcrypt password hashing and JWT sessions via `jose`. Middleware (`src/middleware.ts`) protects project routes. Anonymous users can generate components but projects are not persisted.

### Database

Toujours se baser sur `prisma/schema.prisma` comme source de vérité pour le modèle de données. SQLite via Prisma. `Project.data` stores the serialized `VirtualFileSystem` JSON. `Project.messages` stores serialized chat history.

### Path Alias

`@/*` maps to `./src/*`.

## Code Style

N'utilise les commentaires que pour du code complexe ou non évident.
