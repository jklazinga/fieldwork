---
feature: slack-digest
status: approved
date: 2026-03-07
opportunity: outputs/opportunities/slack-digest/opportunity.md
---

# Spec: Daily Slack Digest

## Problem statement
Team leads spend 10-20 minutes every morning manually reviewing Trackr and copying a summary into Slack for standup. There is no automated way to get project activity into Slack. This friction is a leading indicator of churn: teams that stop doing the morning Trackr check tend to stop using Trackr within 2-3 weeks. We're building a daily Slack digest that replaces this manual ritual — sent automatically at a team-configured time, covering the four things Sam needs for standup: tasks completed yesterday, tasks due today, overdue tasks, and active blockers.

## Proposed solution
A Vercel Cron job fires every minute and checks for teams whose configured digest send time matches the current UTC minute. For each matching team, it enqueues a background job that queries the team's task data, formats a Slack Block Kit message, and sends it to the team's configured Slack channel via the existing Slack OAuth integration. The digest is on by default for all teams with Slack connected. Teams can disable it or change the send time in Trackr settings.

## Scope

### In scope
- Daily digest sent to a single configured Slack channel per team
- Digest content: tasks completed yesterday, tasks due today, overdue tasks, active blockers
- Configurable send time per team (default: 8:30am in team's timezone)
- On by default for teams with Slack connected at time of deploy; new teams get it on by default when they connect Slack
- Can be disabled in Trackr settings (Settings → Integrations → Slack → Daily Digest)
- Digest skipped (not sent) if the team has no Slack workspace connected
- Digest skipped if the team has zero activity to report (no completed, no due today, no overdue, no blockers) — silent skip, no error
- Delivery logged to `digest_logs` table: team_id, scheduled_at, sent_at, slack_channel, status, slack_ts
- Vercel Cron job at `api/cron/send-digest` — fires every minute, checks for due teams

### Out of scope
- Per-user DM digests (v2)
- Custom digest content selection (v2)
- Digest preview in Trackr UI (v2)
- Weekly digest cadence (v2)
- Multiple channels per team (v2)
- Digest for specific projects only (v2)
- Slack slash command for on-demand digest (v2)

## User flows

### Flow 1: Digest sends at configured time
1. Vercel Cron fires at the top of each minute, hitting `POST /api/cron/send-digest`
2. Endpoint queries `team_digest_settings` for teams where `enabled = true`, `slack_connected = true`, and the current UTC time matches the team's configured send time (within the current minute window)
3. For each matching team, enqueue a `send-digest` job via `src/lib/queue.ts`
4. Background job runs:
   a. Query task data for the team (completed yesterday, due today, overdue, blockers)
   b. If all four lists are empty, skip — log to `digest_logs` with `status: 'skipped_no_activity'`
   c. Format Slack Block Kit message via `src/features/digest/digest.formatter.ts`
   d. Send to configured Slack channel via `src/integrations/slack/digest.slack.ts`
   e. Log result to `digest_logs` (status: `sent` or `failed`)
5. Team sees the digest in their Slack channel at the configured time

### Flow 2: Team enables/disables digest in settings
1. Sam opens Settings → Integrations → Slack
2. Sam sees "Daily Digest" toggle (on by default if Slack is connected)
3. Sam can toggle off, or change the send time using a time picker
4. On save, `team_digest_settings` is updated
5. Change takes effect from the next scheduled send (not same-day if the time has already passed)

### Flow 3: Team connects Slack for the first time
1. Team completes existing Slack OAuth flow (`src/integrations/slack/`)
2. On OAuth callback success, create a `team_digest_settings` record with defaults: `enabled: true`, `send_time: '08:30'`, `timezone: team.timezone` (fall back to `'UTC'` if not set)
3. Digest will send at 8:30am team timezone starting the next day

### Edge cases
- **Team has no Slack connected:** Skip silently. Cron query filters these out via `slack_connected = true` join. No log entry.
- **Team has no activity to report:** Skip silently. Log to `digest_logs` with `status: 'skipped_no_activity'`. Do not send an empty digest.
- **Slack API failure (non-2xx):** Log error to Sentry. Log to `digest_logs` with `status: 'failed'`, `error_message` populated. Do not retry automatically in v1 — the next day's digest will send normally.
- **Slack token revoked (401):** Log to Sentry with tag `slack_token_revoked`. Mark team's Slack connection as disconnected in `slack_installations` table. Do not send further digests until team reconnects.
- **Timezone edge case — DST transition:** Store send time as a wall-clock time string (`'08:30'`) and timezone name (`'America/New_York'`). Compute the UTC equivalent at job dispatch time using the `date-fns-tz` library. This handles DST automatically — the digest always fires at 8:30am local time, even across DST boundaries.
- **Vercel Cron fires late (>1 minute delay):** Accept this. The cron checks for teams whose send time falls within the current UTC minute. If the cron fires 90 seconds late, those teams' digests are missed for the day. This is acceptable for v1 — document in constraints.
- **Team changes send time mid-day:** New time takes effect from the next calendar day. If the new time has already passed today, the digest does not send today.

## Acceptance criteria
- [ ] When a team has Slack connected, digest enabled, and the current time matches their configured send time (±1 minute), a digest is sent to their configured Slack channel
- [ ] When a team has no Slack workspace connected, no digest is sent and no error is logged
- [ ] When a team has Slack connected but digest disabled, no digest is sent
- [ ] When a team has zero activity (no completed, no due today, no overdue, no blockers), no digest is sent and `digest_logs` records `status: 'skipped_no_activity'`
- [ ] When the Slack API returns a non-2xx response, the error is logged to Sentry and `digest_logs` records `status: 'failed'` with `error_message`
- [ ] When the Slack token is revoked (401), Sentry is notified with tag `slack_token_revoked` and the team's Slack connection is marked disconnected
- [ ] Each sent digest is recorded in `digest_logs` with: `team_id`, `scheduled_at`, `sent_at`, `slack_channel`, `slack_ts`, `status`
- [ ] The digest Slack message includes four sections: Tasks Completed Yesterday, Due Today, Overdue, Active Blockers — sections with zero items are omitted from the message
- [ ] The digest send time is stored as a wall-clock time + timezone name and correctly handles DST transitions (verified with a test using America/New_York across a DST boundary date)
- [ ] Teams with Slack connected at time of deploy have a `team_digest_settings` record created by the migration with `enabled: true` and `send_time: '08:30'`
- [ ] New teams that connect Slack get a `team_digest_settings` record created automatically with defaults
- [ ] Sam can toggle the digest on/off and change the send time in Settings → Integrations → Slack; changes persist and take effect from the next scheduled send
- [ ] The Vercel Cron job endpoint (`POST /api/cron/send-digest`) returns 200 within 5 seconds (it only enqueues jobs — it does not wait for Slack sends)
- [ ] All digest logic is covered by unit tests; the cron endpoint is covered by an integration test

## Technical notes

### Files to create
- `src/features/digest/digest.service.ts` — core digest orchestration: query tasks, check for activity, call formatter, call Slack sender, log result
- `src/features/digest/digest.queries.ts` — Prisma queries for digest content (completed yesterday, due today, overdue, blockers)
- `src/features/digest/digest.formatter.ts` — formats task data into a Slack Block Kit message payload
- `src/features/digest/digest.types.ts` — types for digest content, digest settings, log entries
- `src/features/digest/digest.service.test.ts` — unit tests for digest.service.ts
- `src/features/digest/digest.formatter.test.ts` — unit tests for digest.formatter.ts
- `src/integrations/slack/digest.slack.ts` — sends a formatted Block Kit payload to a Slack channel; wraps the existing `src/lib/slack.ts` client
- `src/jobs/send-digest.ts` — background job handler: receives team_id, calls digest.service.ts
- `src/app/api/cron/send-digest/route.ts` — Vercel Cron endpoint; queries due teams, enqueues jobs
- `src/app/(app)/settings/integrations/slack/page.tsx` — add digest toggle + time picker to existing Slack settings page (or create if it doesn't exist)
- `src/components/DigestSettings.tsx` — digest enable/disable toggle and time picker component
- `prisma/migrations/YYYYMMDD_add_digest_tables/migration.sql` — creates `team_digest_settings` and `digest_logs` tables; backfills `team_digest_settings` for existing Slack-connected teams

### Files to modify
- `prisma/schema.prisma` — add `TeamDigestSettings` and `DigestLog` models; add relations to `Team`
- `src/integrations/slack/index.ts` — export `digest.slack.ts` functions
- `src/app/(app)/settings/integrations/slack/page.tsx` — add `DigestSettings` component to existing Slack settings page
- `vercel.json` — add cron job: `{ "path": "/api/cron/send-digest", "schedule": "* * * * *" }`
- `src/integrations/slack/oauth-callback.ts` (or equivalent) — create `team_digest_settings` record with defaults on successful OAuth

### Prisma schema additions
```prisma
model TeamDigestSettings {
  id          String   @id @default(cuid())
  teamId      String   @unique
  enabled     Boolean  @default(true)
  sendTime    String   @default("08:30")   // wall-clock HH:mm
  timezone    String   @default("UTC")     // IANA timezone name
  slackChannel String                      // Slack channel ID
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  team        Team     @relation(fields: [teamId], references: [id])
}

model DigestLog {
  id           String   @id @default(cuid())
  teamId       String
  scheduledAt  DateTime
  sentAt       DateTime?
  slackChannel String
  slackTs      String?  // Slack message timestamp (for threading/updates later)
  status       String   // sent | failed | skipped_no_activity
  errorMessage String?
  team         Team     @relation(fields: [teamId], references: [id])
}
```

### Patterns to follow
- Background job pattern: see `src/lib/queue.ts` — use `queue.add('send-digest', { teamId })`, do not call Slack synchronously in the cron endpoint
- Slack API calls: see `src/lib/slack.ts` — use the existing `WebClient` instance; do not create a new client
- Timezone handling: use `date-fns-tz` (`toZonedTime`, `fromZonedTime`) — already in package.json
- Settings page pattern: see `src/app/(app)/settings/integrations/` for existing integration settings pages
- Cron endpoint auth: Vercel Cron sends a `Authorization: Bearer` header with `CRON_SECRET` env var — validate this in the endpoint before processing

### Known constraints
- No synchronous Slack API calls in the request lifecycle — all Slack sends must go through `src/lib/queue.ts`
- Vercel Cron has 1-minute precision — do not promise sub-minute delivery accuracy
- No end-to-end tests — write unit tests for `digest.service.ts` and `digest.formatter.ts`; write integration test for the cron endpoint
- Slack Block Kit message has a 50-block limit — if a team has >15 overdue tasks, truncate the list and add a "…and N more" line. Test this boundary.

## Riskiest assumptions
_Loaded from outputs/opportunities/slack-digest/assumptions.md_

| Assumption | Dimension | Evidence | Mitigation |
|---|---|---|---|
| Teams will keep Slack connected and not disconnect after seeing the digest | Viability / Retention | No data on disconnection triggers | Monitor Slack disconnection rate for digest-enabled teams in week 1; alert if it spikes |
| The digest replaces the manual Trackr check (not supplements it) | Desirability / Usability | Session recordings show the ritual; no evidence a digest would replace it | Instrument morning logins (7–9am) by digest-enabled status; follow-up survey at 2 weeks |
| Vercel Cron is reliable enough for time-sensitive delivery | Feasibility | No production data at our scale | Log every cron invocation; alert if scheduled sends are missed >5% of the time |

## Open questions
_All resolved before spec approved_
- **Send empty digest or skip?** → Skip silently. Log `skipped_no_activity`. An empty digest is noise and may cause teams to disable it.
- **Slack sender identity?** → Trackr bot user (the OAuth app's bot token). The bot is named "Trackr" in Slack. This is already how the existing Slack integration sends messages.
- **Same-day time change?** → New send time takes effect from the next calendar day. Simpler to implement and avoids double-sends if a team changes the time after it has already fired today.

## Dependencies
- Slack OAuth integration (exists — `src/lib/slack.ts`, `src/integrations/slack/`)
- Background job queue (exists — `src/lib/queue.ts`)
- `date-fns-tz` for timezone handling (exists — in `package.json`)
- Vercel Pro plan (required for Vercel Cron — confirm before deploy)
- `CRON_SECRET` environment variable (add to Vercel env vars before deploy)

## Parking Lot
_Items explicitly deferred — do not implement in v1_

| Item | Reason deferred | Signal to revisit |
|---|---|---|
| Per-user DM digests | Scope. Requires per-user Slack identity mapping. | User requests or churn interviews citing "I want my own digest" |
| Custom digest content selection | Scope. Adds significant settings UI complexity. | >20% of teams disable the digest (may indicate content mismatch) |
| Digest preview in Trackr UI | Scope. Nice-to-have for settings page. | User feedback that they don't know what the digest looks like before enabling |
