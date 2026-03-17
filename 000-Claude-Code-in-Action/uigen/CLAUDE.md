# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# First-time setup
npm run setup          # install deps, generate Prisma client, run migrations

# Development
npm run dev            # Next.js dev server with Turbopack

# Testing
npm test               # run all tests (vitest)
npm test -- src/components/chat/__tests__/ChatInterface.test.tsx  # run a single test file

# Database
npx prisma studio      # browse/edit database
npm run db:reset       # wipe and re-run all migrations

# Linting
npm run lint
```

`ANTHROPIC_API_KEY` in `.env` is required for real AI responses. Without it the app falls back to a `MockLanguageModel` that returns static demo components.

`NODE_OPTIONS="--require ./node-compat.cjs"` is prepended to all Next.js commands via `cross-env` to patch Node.js globals required by the Prisma SQLite driver on Windows.

## Architecture

UIGen is an AI-powered React component generator. Users describe components in a chat panel; Claude generates code using tools; the result renders live in a preview panel alongside a Monaco code editor.

### Key data flow

1. **Chat** → `ChatContext` (`src/lib/contexts/chat-context.tsx`) holds messages and drives requests to `POST /api/chat`.
2. **API route** (`src/app/api/chat/route.ts`) calls Claude (`claude-haiku-4-5`) via the Vercel AI SDK with two tools attached, streams the response back.
3. **Tools** mutate the virtual file system:
   - `str_replace_editor` (`src/lib/tools/str-replace.ts`) — create, view, edit, insert in files.
   - `file_manager` (`src/lib/tools/file-manager.ts`) — rename, delete files.
4. **FileSystemContext** (`src/lib/contexts/file-system-context.tsx`) owns the in-memory virtual FS (`src/lib/file-system.ts`). Tool call results are piped here.
5. **PreviewFrame** (`src/components/preview/PreviewFrame.tsx`) reads the virtual FS, transpiles JSX via Babel standalone, and renders inside an iframe.
6. On save, messages + virtual FS are serialised to JSON and persisted in the `Project.messages` / `Project.data` columns via server actions.

### Auth

JWT-based sessions using `jose` (HS256) + `bcrypt`. Sessions live in an httpOnly `auth-token` cookie (7-day expiry). Core logic in `src/lib/auth.ts`; server actions in `src/actions/index.ts`.

### Database

SQLite via Prisma. Two models: `User` and `Project`. `Project.messages` and `Project.data` are JSON strings. The Prisma client is generated into `src/generated/prisma` (not the default location).

### Testing

Vitest with jsdom + React Testing Library. Tests live in `__tests__/` directories next to the code they cover.
