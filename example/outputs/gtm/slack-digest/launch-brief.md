---
feature: slack-digest
launch-date: 2026-04-07 08:00 NZDT
status: ready
---

# Launch Brief: Daily Slack Digest

## Launch summary
**Date/time:** April 7, 2026 at 08:00 NZDT
**Type:** Hard release with feature flag fallback (digest sending can be disabled via `DIGEST_ENABLED=false` env var without a full redeploy)
**Owner:** PM
**On-call engineer:** [Engineer name]
**Rollback owner:** [Engineer name]

## Pre-launch checklist
_Derived from spec acceptance criteria_

- [ ] Digest sends to correct Slack channel at configured time (verified in staging with a test team set to fire in 2 minutes)
- [ ] Teams with no Slack connected: no digest sent, no error logged
- [ ] Teams with digest disabled: no digest sent
- [ ] Teams with zero activity: `digest_logs` records `status: skipped_no_activity`, no Slack message sent
- [ ] Slack API failure: error logged to Sentry, `digest_logs` records `status: failed` with `error_message`
- [ ] Revoked Slack token (401): Sentry notified with `slack_token_revoked` tag, team's Slack connection marked disconnected
- [ ] `digest_logs` records: `team_id`, `scheduled_at`, `sent_at`, `slack_channel`, `slack_ts`, `status`
- [ ] Digest message includes correct sections; sections with zero items are omitted
- [ ] Section with >15 items truncated to 15 with "…and N more" footer
- [ ] DST boundary test passes: `America/New_York` team configured for 08:30 sends at correct UTC time on March 8 (DST start)
- [ ] Existing Slack-connected teams have `team_digest_settings` records after migration (verify count in staging DB)
- [ ] Teams with no stored default channel have `enabled: false` and `slack_channel: null` after migration
- [ ] New Slack OAuth connection creates `team_digest_settings` with defaults (verified in staging)
- [ ] Settings UI: digest toggle and time picker visible for Slack-connected teams
- [ ] Settings UI: channel picker shown when `slack_channel` is null
- [ ] `PATCH /api/teams/[teamId]/digest-settings` saves changes correctly
- [ ] Cron endpoint returns 401 for requests without valid `CRON_SECRET`
- [ ] Cron endpoint returns 200 within 5 seconds (enqueue only, no blocking Slack calls)
- [ ] Cron deduplication: second cron fire within same minute does not double-send
- [ ] `CRON_SECRET` env var set in Vercel production environment
- [ ] Vercel Cron job visible in Vercel dashboard (Settings → Cron Jobs) and showing as active
- [ ] Sentry alert configured for `slack_token_revoked` tag and for `digest_logs.status = 'failed'` rate >5%
- [ ] Baseline churn cohort data pulled and stored (segmented by Slack-connected status)
- [ ] Baseline morning login rate (7–9am) pulled and stored (for post-launch comparison)
- [ ] In-app prompt copy approved by PM
- [ ] Customer email copy approved by PM (send scheduled for April 8)
- [ ] LinkedIn post drafted and scheduled for April 9

## Launch sequence

| Time | Action | Owner |
|---|---|---|
| T-48h (April 5) | Final QA against acceptance criteria in staging | Engineering |
| T-48h | PM reviews digest message format in staging Slack workspace | PM |
| T-48h | Confirm Vercel Cron is active in staging (check Vercel dashboard) | Engineering |
| T-24h (April 6) | Confirm on-call coverage for launch window | Engineering |
| T-24h | Pull baseline churn cohort + morning login data | PM |
| T-4h | Final deploy check — staging green, all acceptance criteria met | Engineering |
| T-0 (April 7, 08:00 NZDT) | Deploy to production | Engineering |
| T+30min | Verify: check `digest_logs` in production DB — first digests should be sending for early-timezone teams | Engineering |
| T+1h | Check Sentry — no new errors | Engineering |
| T+1h | Check Vercel Cron logs — cron firing every minute | Engineering |
| T+2h | Confirm at least one digest received in a real team's Slack channel (use internal Trackr team as canary) | PM + Engineering |
| T+24h (April 8) | Send customer email announcement to Slack-connected teams | PM |
| T+48h (April 9) | Post LinkedIn announcement | PM |
| T+14d (April 21) | Review metrics: digest send rate, adoption rate, morning login rate, early churn signal | PM |
| T+14d | Run retro | PM + Engineering |

## Rollback plan
**Trigger condition:** Sentry error rate spike >5% within 2 hours of deploy, OR digest send rate <50% in first 4 hours, OR critical bug (e.g. wrong team receiving another team's digest), OR Slack API rate limit errors appearing at scale.

**Steps:**
1. Set `DIGEST_ENABLED=false` in Vercel environment variables — this disables digest sending without a full redeploy (the cron endpoint checks this flag before enqueuing jobs). Takes effect within 1 minute.
2. Notify PM immediately via Slack DM.
3. Post in #team: "Daily Slack digest temporarily disabled — investigating. Teams will not receive digests until further notice."
4. Open incident in Sentry, assign to on-call engineer.
5. If the issue is in the cron endpoint itself (not just digest sending), redeploy the previous version from Vercel dashboard.

**Estimated rollback time:** 1-2 minutes (env var change) or 5-10 minutes (full redeploy).

## Monitoring

| Metric | Baseline | Alert threshold | Owner |
|---|---|---|---|
| Sentry error rate (digest jobs) | 0 | >5 errors in 1h | Engineering |
| Slack API error rate | 0 | >3% of sends | Engineering |
| Digest send rate (`digest_logs.status = 'sent'` / total scheduled) | — | <80% in any 24h window | PM |
| Digest disable rate (teams disabling within 7 days) | — | >20% of enabled teams | PM |
| Morning Trackr logins (7–9am), digest-enabled teams | [Pull pre-launch] | — (track trend) | PM |
| 30-day churn, Slack-connected teams | 28% | — (track at 60d) | PM |

## Comms sequence

| Time | Channel | Message | Owner | Status |
|---|---|---|---|---|
| T-0 | In-app prompt (Slack-connected teams) | "Your team now gets a daily Trackr digest in Slack" | Engineering | on deploy |
| T-0 | Changelog | Digest feature entry | Engineering | on deploy |
| T-0 | Settings page tooltip | "Digest sends at 8:30am in your timezone. Change it in settings." | Engineering | on deploy |
| T+24h | Email (Slack-connected teams) | "No more manual standup prep" | PM | scheduled April 8 |
| T+48h | LinkedIn | Variant 1 post | PM | scheduled April 9 |

## Post-launch
- [ ] Confirm `digest_logs` baseline captured (T+2h)
- [ ] Check Sentry + Vercel Cron logs at T+24h
- [ ] Review digest send rate and disable rate at T+7d
- [ ] Review morning login rate at T+14d
- [ ] Review churn cohort at T+60d
- [ ] Run retro at T+14d (April 21)
