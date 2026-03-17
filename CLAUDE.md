# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a mono-repo that collects hands-on projects from [Anthropic Academy](https://anthropic.com) courses. Each course lives in its own numbered folder.

| Folder | Course |
|---|---|
| `000-Claude-Code-in-Action/` | Claude Code in Action |

Each course folder may contain its own `CLAUDE.md` with project-specific guidance. Always read the subfolder CLAUDE.md when working inside a course project.

## Course 000 — UIGen (`000-Claude-Code-in-Action/uigen/`)

The main hands-on project for course 000. Full guidance is in `000-Claude-Code-in-Action/uigen/CLAUDE.md`. Key commands:

```bash
cd 000-Claude-Code-in-Action/uigen

npm run setup          # first-time: install, generate Prisma client, run migrations
npm run dev            # dev server (Turbopack)
npm test               # all tests
npm test -- src/path/to/file.test.tsx  # single test file
npm run lint
npx prisma studio      # browse database
npm run db:reset       # wipe and re-run all migrations
```

Requires `ANTHROPIC_API_KEY` in `000-Claude-Code-in-Action/uigen/.env`. Without it, the app falls back to a `MockLanguageModel` returning static demo components.

## Adding New Courses

New courses follow the same pattern: a numbered folder (`001-Course-Name/`) containing a README.md that describes the course and subfolders for any hands-on projects. Add a row to the table above and create a CLAUDE.md inside the project subfolder.
