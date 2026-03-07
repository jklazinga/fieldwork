# Constraints
_Last updated: 2026-03-01_

## Team
2 engineers (Sam full-time, Jordan 60% - also handles infra). No dedicated QA. PM is also the founder (Alex, non-technical).

## Deadlines
- CSV export: soft target end of March (mentioned in a customer call, not a hard commitment)
- Public status pages: no date set
- No external events or conferences this quarter

## Tech debt
- `app/api/projects/[id]/tasks/route.ts` - large file, mixed concerns, slow to test. Avoid adding to it without refactoring first.
- Prisma migrations have drifted in staging - always test migrations against a fresh DB before merging.
- No end-to-end tests yet. Jest coverage is unit/integration only.

## Non-negotiables
- All data exports must be scoped to the authenticated team - no cross-team data leakage.
- Vercel free tier limits: no long-running serverless functions (max 10s). Any heavy processing must be chunked or offloaded.
- GDPR: any export feature must be logged in the audit trail (`lib/audit.ts`).
