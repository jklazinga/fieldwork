---
feature: slack-digest
date: 2026-03-07
spec: outputs/specs/slack-digest/spec.md
superpowers-compatible: true
---

# Implementation Plan: Daily Slack Digest

> **For Claude:** This plan is designed for task-by-task execution.
> - If Superpowers is installed: use `superpowers:executing-plans`
> - If not: use `fieldwork:scaffold-tasks` to create trackable issues
> Do NOT implement multiple tasks in one step. Do NOT skip verification steps.

**Goal:** Send a daily Slack digest of project activity (completed, due today, overdue, blockers) to each team's configured Slack channel at their configured time.
**Architecture:** Vercel Cron fires every minute → enqueues `send-digest` jobs via `src/lib/queue.ts` → background job queries tasks, formats Block Kit message, sends via existing Slack client. New `team_digest_settings` and `digest_logs` DB tables. Digest settings UI in existing Slack settings page.
**Tech stack:** TypeScript, Prisma, Next.js 14 App Router, `date-fns-tz`, Slack Web API (`@slack/web-api`)
**Test approach:** TDD for `digest.service.ts` and `digest.formatter.ts`. Integration test for the cron endpoint. No E2E tests.

---

### Task 1: Prisma schema + migration

**Files:**
- Modify: `prisma/schema.prisma`
- Create: `prisma/migrations/YYYYMMDD_add_digest_tables/migration.sql`

**Steps:**
1. Add models to `prisma/schema.prisma`:
```prisma
model TeamDigestSettings {
  id           String   @id @default(cuid())
  teamId       String   @unique
  enabled      Boolean  @default(true)
  sendTime     String   @default("08:30")  // wall-clock HH:mm
  timezone     String   @default("UTC")    // IANA timezone name
  slackChannel String?                     // null until team configures
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
  team         Team     @relation(fields: [teamId], references: [id])
}

model DigestLog {
  id           String    @id @default(cuid())
  teamId       String
  scheduledAt  DateTime  // team's configured send time in UTC for this day
  sentAt       DateTime?
  slackChannel String
  slackTs      String?
  status       String    // sent | failed | skipped_no_activity
  errorMessage String?
  team         Team      @relation(fields: [teamId], references: [id])

  @@unique([teamId, scheduledAt])  // deduplication guard
}
```
2. Add relations to existing `Team` model:
```prisma
digestSettings TeamDigestSettings?
digestLogs     DigestLog[]
```
3. Run: `pnpm prisma migrate dev --name add_digest_tables`
4. The migration SQL should also backfill `team_digest_settings` for existing Slack-connected teams:
```sql
INSERT INTO "TeamDigestSettings" (id, "teamId", enabled, "sendTime", timezone, "slackChannel", "createdAt", "updatedAt")
SELECT
  gen_random_uuid(),
  t.id,
  CASE WHEN si."defaultChannel" IS NOT NULL THEN true ELSE false END,
  '08:30',
  COALESCE(t.timezone, 'UTC'),
  si."defaultChannel",
  NOW(),
  NOW()
FROM "Team" t
INNER JOIN "SlackInstallation" si ON si."teamId" = t.id
ON CONFLICT ("teamId") DO NOTHING;
```
5. Run: `pnpm prisma generate`

**Verification:** `pnpm prisma studio` — confirm `TeamDigestSettings` and `DigestLog` tables exist with correct columns and the unique constraint on `(teamId, scheduledAt)`.
**Depends on:** none

---

### Task 2: Write failing tests for digest.service.ts and digest.formatter.ts

**Files:**
- Create: `src/features/digest/digest.service.test.ts`
- Create: `src/features/digest/digest.formatter.test.ts`

**Steps:**
1. `digest.service.test.ts` — write tests for:
   - `sendDigestForTeam(teamId)` — queries tasks, formats message, calls Slack sender, logs `sent` to `digest_logs`
   - `sendDigestForTeam` with no activity — skips send, logs `skipped_no_activity`
   - `sendDigestForTeam` with Slack API failure — logs `failed` with `errorMessage`, logs to Sentry
   - `sendDigestForTeam` with revoked Slack token (401) — logs to Sentry with `slack_token_revoked` tag, marks Slack connection disconnected
   - `sendDigestForTeam` called twice for same `(teamId, scheduledAt)` — second call exits early (dedup via unique constraint)

2. `digest.formatter.test.ts` — write tests for:
   - `formatDigestMessage(content)` — returns valid Block Kit payload with all four sections
   - Sections with zero items are omitted from the output
   - Section with >15 items is truncated to 15 with "…and N more" footer
   - DST boundary: `getScheduledAtUtc('08:30', 'America/New_York', new Date('2026-03-08'))` returns correct UTC time (DST starts March 8 2026 in the US)

3. Run: `pnpm test src/features/digest/`

**Verification:** All tests fail with "Cannot find module" — confirms RED state.
**Depends on:** Task 1

---

### Task 3: Digest content queries

**Files:**
- Create: `src/features/digest/digest.queries.ts`
- Create: `src/features/digest/digest.types.ts`

**Steps:**
1. `digest.types.ts`:
```typescript
export interface DigestTask {
  id: string
  title: string
  assignee?: string
  dueDate?: Date
}

export interface DigestContent {
  completedYesterday: DigestTask[]
  dueToday: DigestTask[]
  overdue: DigestTask[]
  blockers: DigestTask[]
}

export type DigestStatus = 'sent' | 'failed' | 'skipped_no_activity'

export interface DigestSettings {
  teamId: string
  enabled: boolean
  sendTime: string      // HH:mm
  timezone: string      // IANA
  slackChannel: string
}
```

2. `digest.queries.ts` — implement four Prisma queries, all scoped to `teamId`:
```typescript
// completedYesterday: tasks with status = 'done' and updatedAt between
// start-of-yesterday and end-of-yesterday (in team's timezone)
export async function getCompletedYesterday(teamId: string, teamTimezone: string): Promise<DigestTask[]>

// dueToday: tasks with dueDate = today (in team's timezone) and status != 'done'
export async function getDueToday(teamId: string, teamTimezone: string): Promise<DigestTask[]>

// overdue: tasks with dueDate < today (in team's timezone) and status != 'done'
export async function getOverdue(teamId: string, teamTimezone: string): Promise<DigestTask[]>

// blockers: tasks with isBlocked = true and status != 'done'
export async function getBlockers(teamId: string): Promise<DigestTask[]>
```
3. Use `date-fns-tz` (`startOfDay`, `endOfDay`, `toZonedTime`) for all timezone-aware date boundaries. Do not use raw `new Date()` for day boundaries.

**Verification:** `pnpm tsc --noEmit` — no TypeScript errors.
**Depends on:** Task 1

---

### Task 4: Slack message formatter (Block Kit)

**Files:**
- Create: `src/features/digest/digest.formatter.ts`

**Steps:**
1. Implement `formatDigestMessage(content: DigestContent): KnownBlock[]`
2. Message structure:
   - Header block: "📋 Trackr Daily Digest — {date}"
   - Divider
   - For each non-empty section: section header (bold) + bullet list of tasks (max 15, then "…and N more")
   - Section order: ✅ Completed Yesterday → 📅 Due Today → 🔴 Overdue → 🚧 Blockers
   - Footer: "Manage digest settings → [Trackr Settings]" (link to `{APP_URL}/settings/integrations/slack`)
3. Truncation logic:
```typescript
function renderTaskList(tasks: DigestTask[], max = 15): string {
  const visible = tasks.slice(0, max)
  const rest = tasks.length - visible.length
  const lines = visible.map(t => `• ${t.title}${t.assignee ? ` (${t.assignee})` : ''}`)
  if (rest > 0) lines.push(`_…and ${rest} more_`)
  return lines.join('\n')
}
```
4. Return type is `KnownBlock[]` from `@slack/web-api` — do not use `any`.

**Verification:** `pnpm test src/features/digest/digest.formatter.test.ts` — all formatter tests pass (GREEN).
**Depends on:** Tasks 2, 3

---

### Task 5: Implement digest.service.ts

**Files:**
- Create: `src/features/digest/digest.service.ts`
- Create: `src/integrations/slack/digest.slack.ts`

**Steps:**
1. `digest.slack.ts` — thin wrapper around the existing Slack `WebClient`:
```typescript
import { getSlackClientForTeam } from '@/lib/slack'
import type { KnownBlock } from '@slack/web-api'

export async function sendDigestToChannel(
  teamId: string,
  channel: string,
  blocks: KnownBlock[]
): Promise<{ ts: string }> {
  const client = await getSlackClientForTeam(teamId)
  const result = await client.chat.postMessage({ channel, blocks, text: 'Trackr Daily Digest' })
  if (!result.ok || !result.ts) throw new Error(result.error ?? 'Slack postMessage failed')
  return { ts: result.ts }
}
```

2. `digest.service.ts` — implement `sendDigestForTeam(teamId: string, scheduledAt: Date)`:
   - Load `TeamDigestSettings` for team — if not found or `enabled: false`, return early
   - Attempt to insert `DigestLog` record with `status: 'sent'` and `scheduledAt` — if unique constraint violation, return early (dedup)
   - Query all four content lists via `digest.queries.ts`
   - If all four are empty, update log to `status: 'skipped_no_activity'`, return
   - Format message via `digest.formatter.ts`
   - Call `sendDigestToChannel` — on success, update log with `sentAt`, `slackTs`, `status: 'sent'`
   - On Slack API error: update log with `status: 'failed'`, `errorMessage`; log to Sentry
   - On 401 (token revoked): additionally mark `slack_installations` record as disconnected; log to Sentry with tag `slack_token_revoked`

3. Run: `pnpm test src/features/digest/digest.service.test.ts`

**Verification:** All service tests pass (GREEN).
**Depends on:** Tasks 2, 3, 4

---

### Task 6: Vercel Cron job + send-digest endpoint

**Files:**
- Create: `src/jobs/send-digest.ts`
- Create: `src/app/api/cron/send-digest/route.ts`
- Modify: `vercel.json`

**Steps:**
1. `send-digest.ts` — background job handler:
```typescript
import { sendDigestForTeam } from '@/features/digest/digest.service'

export async function handleSendDigestJob(payload: { teamId: string; scheduledAt: string }) {
  await sendDigestForTeam(payload.teamId, new Date(payload.scheduledAt))
}
```
Register with `src/lib/queue.ts` following the existing job registration pattern.

2. `route.ts` — Vercel Cron endpoint:
```typescript
import { NextRequest, NextResponse } from 'next/server'
import { prisma } from '@/lib/prisma'
import { queue } from '@/lib/queue'
import { getScheduledAtUtc } from '@/features/digest/digest.formatter'

export async function POST(req: NextRequest) {
  // Validate CRON_SECRET
  const auth = req.headers.get('authorization')
  if (auth !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const now = new Date()

  // Find teams whose digest is due this minute
  const dueTeams = await prisma.teamDigestSettings.findMany({
    where: {
      enabled: true,
      slackChannel: { not: null },
      team: { slackInstallation: { isNot: null } },
    },
  })

  const toSend = dueTeams.filter(settings => {
    const scheduledAt = getScheduledAtUtc(settings.sendTime, settings.timezone, now)
    const diffMs = Math.abs(scheduledAt.getTime() - now.getTime())
    return diffMs < 60_000  // within the current minute window
  })

  await Promise.all(
    toSend.map(settings =>
      queue.add('send-digest', {
        teamId: settings.teamId,
        scheduledAt: getScheduledAtUtc(settings.sendTime, settings.timezone, now).toISOString(),
      })
    )
  )

  return NextResponse.json({ queued: toSend.length })
}
```

3. `vercel.json` — add cron:
```json
{
  "crons": [
    {
      "path": "/api/cron/send-digest",
      "schedule": "* * * * *"
    }
  ]
}
```

4. Write integration test for the cron endpoint:
   - Valid `CRON_SECRET` + teams due → 200, jobs enqueued
   - Missing/invalid `CRON_SECRET` → 401
   - No teams due → 200, `{ queued: 0 }`

**Verification:** `pnpm test src/app/api/cron/send-digest/` — integration tests pass.
**Depends on:** Task 5

---

### Task 7: Digest settings UI

**Files:**
- Create: `src/components/DigestSettings.tsx`
- Modify: `src/app/(app)/settings/integrations/slack/page.tsx`

**Steps:**
1. `DigestSettings.tsx` — React component:
   - Props: `settings: DigestSettings | null`, `onSave: (settings: Partial<DigestSettings>) => Promise<void>`
   - If `settings.slackChannel` is null: render channel picker (fetches `conversations.list` via `GET /api/integrations/slack/channels`) + explanatory text "Choose a Slack channel to enable your daily digest"
   - If channel is set: render toggle (enabled/disabled) + time picker (HTML `<input type="time">`) + timezone display (read-only, from team settings)
   - On save: call `PATCH /api/teams/[teamId]/digest-settings` with `{ enabled, sendTime, slackChannel }`
   - Show success toast on save; show error toast on failure
   - Use React Query for data fetching (consistent with existing settings pages)

2. Add `DigestSettings` to the existing Slack settings page below the connection status section.

3. Create API route `src/app/api/teams/[teamId]/digest-settings/route.ts`:
   - `GET` — returns current `TeamDigestSettings` for the team
   - `PATCH` — updates `enabled`, `sendTime`, `slackChannel`; validates `sendTime` is a valid HH:mm string; validates `slackChannel` is a non-empty string if provided

**Verification:** Manual test in dev — connect Slack, open Settings → Integrations → Slack, confirm digest toggle and time picker appear. Toggle off, save, confirm `team_digest_settings.enabled = false` in DB. Change time to 09:00, save, confirm `send_time = '09:00'` in DB.
**Depends on:** Task 1

---

### Task 8: Integration test + commit

**Files:**
- Create: `tests/features/digest/digest.integration.test.ts`

**Steps:**
1. Write end-to-end integration test (unit-level, no real Slack calls — mock `sendDigestToChannel`):
   - Seed DB: team with Slack connected, `team_digest_settings` with `enabled: true`, `send_time: '08:30'`, `timezone: 'Pacific/Auckland'`, tasks in various states
   - Call `sendDigestForTeam(teamId, scheduledAt)`
   - Assert: `digest_logs` record created with `status: 'sent'`
   - Assert: `sendDigestToChannel` called with correct `channel` and a Block Kit payload containing the expected sections
   - Assert: calling `sendDigestForTeam` again with the same `scheduledAt` exits early (dedup)

2. Run full test suite: `pnpm test`

3. Commit:
```
git add -A
git commit -m "feat: daily Slack digest via Vercel Cron"
```

4. Push to feature branch and open PR.

**Verification:** CI passes. All tests green. No TypeScript errors (`pnpm tsc --noEmit`).
**Depends on:** Tasks 5, 6, 7
