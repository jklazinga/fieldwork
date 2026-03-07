# Project Context
_Last updated: 2026-03-01_

## Product
Trackr is a lightweight project tracking SaaS for small engineering teams (2-20 people). Teams use it to manage tasks, track time, and report progress to stakeholders. Currently in open beta with ~200 teams.

## Tech stack
TypeScript, Next.js 14 (App Router), PostgreSQL via Prisma, Tailwind CSS, React Query, Jest + Testing Library, Vercel.

## Architecture
Monorepo. Next.js handles both frontend and API routes. Database access goes through a Prisma client in `lib/db.ts`. Auth is NextAuth with GitHub OAuth. All API routes live in `app/api/`. UI components in `components/`, page-level logic in `app/`.

## Test setup
Jest with Testing Library for components. API routes tested with supertest. Run with `npm test`. Coverage enforced at 80% on CI.

## Key files
- `app/api/` - all API routes
- `lib/db.ts` - Prisma client singleton
- `prisma/schema.prisma` - data models
- `components/` - shared UI components
- `app/(dashboard)/` - authenticated app pages

## MCP integrations available
- GitHub: yes
- Linear: no
- Slack: no
- Confluence: no
