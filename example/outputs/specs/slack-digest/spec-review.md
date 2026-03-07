---
feature: slack-digest
date: 2026-03-07
spec: outputs/specs/slack-digest/spec.md
reviewer: fieldwork:review-spec
---

# Spec Review: Daily Slack Digest

## Blockers

- [ ] **The cron endpoint has no deduplication guard.** If Vercel Cron fires twice within the same minute (which Vercel's documentation says can happen under retry conditions), the same team could receive two digests. The spec describes queuing a job per matching team but does not describe how to prevent double-enqueue. Before engineering starts, the spec needs to define a deduplication strategy — either a unique constraint on `digest_logs (team_id, scheduled_at)` that causes the second insert to fail gracefully, or a Redis-based lock in the queue layer. This is a correctness issue, not a nice-to-have.

- [ ] **The migration backfill for existing Slack-connected teams is underspecified.** The spec says "backfills `team_digest_settings` for existing Slack-connected teams" but does not specify which Slack channel to use as the default. Teams may have multiple Slack channels available via their OAuth token. The spec needs to define the fallback: use the channel stored in `slack_installations` (if one is stored), or require teams to configure the channel before the digest activates. If the channel is unknown, `enabled` should default to `false` until the team configures it — otherwise the migration may fail for teams with no stored channel.

- [ ] **Block Kit 50-block limit is mentioned but the truncation behaviour is not in the acceptance criteria.** The spec notes "if a team has >15 overdue tasks, truncate the list and add a '…and N more' line" in the technical notes, but this is not reflected in any acceptance criterion. If this edge case is not tested, it will be missed. Add an acceptance criterion: "When any digest section contains more than 15 items, the section is truncated to 15 items with a '…and N more' footer line."

## Important

- [ ] **`CRON_SECRET` validation is mentioned in technical notes but not in acceptance criteria.** If the cron endpoint does not validate the `CRON_SECRET` header, it is publicly callable by anyone who knows the URL. This is a security issue. Add an acceptance criterion: "Requests to `POST /api/cron/send-digest` without a valid `Authorization: Bearer <CRON_SECRET>` header return 401."

- [ ] **The settings UI flow for the initial channel selection is missing.** Flow 2 (team enables/disables digest) assumes a channel is already configured. But for teams that connected Slack before this feature shipped, the channel may not be set. The spec needs a sub-flow: what does Sam see in Settings → Integrations → Slack if no channel is configured? A channel picker should appear before the digest toggle is actionable.

## Minor

- [ ] **`digest_logs.scheduledAt` type is `DateTime` but the cron fires every minute.** Clarify whether `scheduledAt` is the exact UTC time the cron fired, or the team's configured send time rounded to the minute. This matters for deduplication and for debugging missed sends. Suggest: store the team's configured send time (in UTC) as `scheduledAt`, not the cron fire time.

- [ ] **No mention of what happens to `DigestLog` records over time.** At 200 teams × 365 days, that's 73,000 rows per year. Not a problem now, but worth noting a retention policy (e.g. delete logs older than 90 days) as a follow-up task.

## Assumption gaps
- The spec correctly loads all three riskiest assumptions from `assumptions.md` and includes mitigations. The Vercel Cron reliability assumption is appropriately flagged. No gaps.

## Opportunity alignment
- The primary success metric (30-day churn reduction for Slack-connected teams) is not directly testable from acceptance criteria — it's a lagging indicator. This is expected and correct. The proxy metric (morning login rate) is instrumentable and should be added to the launch brief monitoring table. ✅
- All four digest content sections (completed yesterday, due today, overdue, blockers) are in scope and reflected in acceptance criteria. ✅
- The "on by default for Slack-connected teams" requirement is in scope and reflected in the migration spec. ✅ (pending resolution of the channel selection blocker above)

## Summary
Strong spec with well-defined edge cases and accurate file paths. Three blockers to resolve before engineering starts: cron deduplication, migration channel fallback, and the Block Kit truncation acceptance criterion. The security issue (CRON_SECRET validation) is important but quick to add. Once these are resolved, this spec is ready.

---
## Resolution log

- **Cron deduplication** — resolved 2026-03-07: Add a unique constraint on `digest_logs (team_id, scheduled_at)`. The job handler attempts to insert the log record first; if the insert fails due to the unique constraint, the job exits early without sending. This is the simplest approach and avoids a Redis dependency. Spec updated.

- **Migration channel fallback** — resolved 2026-03-07: For existing Slack-connected teams, use the `default_channel` stored in `slack_installations` if present. If no channel is stored, create the `team_digest_settings` record with `enabled: false` and `slack_channel: null`. The settings page will show a "Choose a channel to enable your digest" prompt for these teams. Spec updated.

- **Block Kit truncation acceptance criterion** — resolved 2026-03-07: Added to acceptance criteria: "When any digest section contains more than 15 items, the section is truncated to 15 items with a '…and N more' footer line." Unit test added to `digest.formatter.test.ts` scope.

- **CRON_SECRET validation** — resolved 2026-03-07: Added to acceptance criteria. Engineering to implement as middleware on the cron route. `CRON_SECRET` added to the dependencies section and pre-launch checklist.

- **Channel picker for unconfigured teams** — resolved 2026-03-07: Settings page will render a Slack channel picker (using the Slack API `conversations.list` endpoint, called server-side) when `slack_channel` is null. Digest toggle is disabled until a channel is selected. Added to Flow 2 in the spec.
