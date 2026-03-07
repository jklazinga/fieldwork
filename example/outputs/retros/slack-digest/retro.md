---
feature: slack-digest
shipped-date: 2026-04-07
retro-date: 2026-04-21
---

# Feature Retro: Daily Slack Digest

## What shipped vs. what was specced

### Acceptance criteria
- [x] When a team has Slack connected, digest enabled, and the current time matches their configured send time (±1 minute), a digest is sent to their configured Slack channel
- [x] When a team has no Slack workspace connected, no digest is sent and no error is logged
- [x] When a team has Slack connected but digest disabled, no digest is sent
- [x] When a team has zero activity, no digest is sent and `digest_logs` records `status: 'skipped_no_activity'`
- [x] When the Slack API returns a non-2xx response, the error is logged to Sentry and `digest_logs` records `status: 'failed'` with `error_message`
- [x] When the Slack token is revoked (401), Sentry is notified with tag `slack_token_revoked` and the team's Slack connection is marked disconnected
- [x] Each sent digest is recorded in `digest_logs` with all required fields
- [x] Sections with zero items are omitted from the Slack message
- [x] DST handling verified with `America/New_York` test across a DST boundary date
- [x] Existing Slack-connected teams backfilled with `team_digest_settings` at deploy
- [x] New teams connecting Slack get `team_digest_settings` created with defaults
- [x] Sam can toggle digest on/off and change send time in Settings → Integrations → Slack
- [x] Cron endpoint returns 200 within 5 seconds
- [ ] All digest logic covered by unit tests; cron endpoint covered by integration test — **partial miss**: `digest.formatter.ts` unit tests were written but the Block Kit truncation boundary (>15 overdue tasks → "…and N more") was not tested. Hotfix added the test 3 days post-launch after a team with 22 overdue tasks triggered a malformed message.

### Scope changes
**Added (not in spec):**
- Block Kit message truncation at 15 items per section was specced as a constraint but the spec did not define the exact truncation format. The "…and N more overdue tasks" line was added during implementation — minor, but it required a design decision that should have been in the spec.
- `CRON_SECRET` validation was added to the cron endpoint as specced, but the spec did not mention that Vercel requires the secret to be set in the Vercel dashboard *and* in `.env.local` for local testing. This caused a 2-hour debugging session during development.

**Cut (was in spec):**
- Nothing cut.

## Surprises

- **Vercel Cron has 1-minute precision, not second-level.** The spec documented this correctly. What it didn't document: Vercel Cron can fire up to 30 seconds late under load. In practice, teams configured for 8:30am sometimes receive their digest at 8:30:28am. This is fine for standup prep but worth communicating in the settings UI ("Digest arrives around your configured time").
- **Slack Block Kit 50-block limit hit in production on day 3.** A team with 22 overdue tasks and 8 blockers hit the limit. The spec mentioned the 50-block limit and the 15-item truncation rule, but the formatter test didn't cover this boundary. The malformed message was caught by Sentry (Slack returned a 400). Hotfix shipped within 4 hours: added the "…and N more" truncation and the missing test.
- **Timezone backfill was messier than expected.** The migration backfilled `team_digest_settings` for existing Slack-connected teams using `team.timezone`. About 12% of teams had `timezone: null` in the database — they fell back to UTC as specced, but their 8:30am UTC digest arrived at unexpected local times. 3 support tickets in week 1. The spec's fallback to UTC was correct but the user-facing impact wasn't considered.
- **Opt-out rate was lower than feared.** We expected ~15% of teams to disable the digest in week 1. Actual opt-out rate: 4%. The "on by default" decision was the right call.
- **Morning login rate dropped measurably.** 7–9am logins by digest-enabled teams dropped 31% in week 2 vs. the two weeks prior. This is the signal we were looking for — the digest is replacing the manual check, not supplementing it.

## Spec quality

**What the spec got right:**
- Edge cases were comprehensive and all needed to be handled: no Slack connected, no activity, Slack API failure, token revocation, DST transitions, late Vercel Cron fires, same-day time changes
- File paths were accurate — no surprises in the codebase
- The "enqueue, don't call synchronously" constraint was clearly stated and saved a production incident (the cron endpoint would have timed out if it called Slack directly)
- The Prisma schema additions were correct and complete — no schema changes needed during implementation
- The `skipped_no_activity` log status was the right call — it gave us visibility into how many teams have quiet days without generating noise

**What the spec got wrong:**
- Block Kit truncation format was left as an implementation detail — it should have been specced explicitly (format of the "…and N more" line, which sections get truncated first if multiple are long)
- No mention of the `CRON_SECRET` local development setup — this is a standard Vercel Cron gotcha and should be in the dependencies section
- Timezone null handling was specced (fall back to UTC) but the user-facing impact of UTC fallback wasn't flagged as a risk. Should have been: "Teams with no timezone set will receive the digest at 8:30am UTC — this may be unexpected. Consider prompting teams to set their timezone before enabling the digest."
- The settings UI section was thin — "add digest toggle + time picker to existing Slack settings page" left too much to interpretation. The time picker UX (dropdown vs. free text vs. native time input) was debated during implementation and cost half a day.

## GTM execution

**What happened vs. the plan:**
- Feature deployed April 7 as planned
- Customer email sent April 8 — open rate 52% (above average; Slack integrations reliably drive opens)
- LinkedIn post published April 9 — 2.1x normal engagement
- Metrics review scheduled April 21 — on track

**Early user signal:**
- 4 unsolicited positive Slack messages from team leads in week 1 ("this is exactly what I needed before standup")
- 1 support ticket: team lead confused about why the digest arrived at 1:30am (their timezone was null → UTC fallback). Resolved by prompting them to set their timezone in account settings.
- 2 support tickets: teams asking how to change the digest channel (currently only configurable at workspace level, not per-channel). This is a v2 item but came up faster than expected.
- 0 tickets about the digest content being wrong or missing tasks — the query logic held up.

## Learnings

1. **Spec the truncation format, not just the rule.** "Truncate at 15 items" is not enough. The exact format of the overflow line, which sections get truncated first, and whether truncation is tested at the boundary all need to be explicit. A missing boundary test caused a production incident on day 3.
2. **Timezone null is a user-facing risk, not just a fallback.** When a spec says "fall back to UTC if timezone not set," flag the user-facing impact. UTC fallback at 8:30am is 1:30am in New Zealand. Add a pre-launch check: how many teams have `timezone: null`? Consider prompting them before the feature goes live.
3. **Vercel Cron local development setup is a standard gotcha.** `CRON_SECRET` must be in both Vercel dashboard and `.env.local`. Add this to the dependencies section of any spec that uses Vercel Cron. It's not obvious and it cost 2 hours.
4. **"On by default" was the right call — don't second-guess it.** We debated opt-in vs. opt-out. Opt-out (on by default) drove 96% retention of the feature in week 1. If we'd gone opt-in, adoption would have been a fraction of that.
5. **Settings UI decisions need more spec detail.** "Add a time picker" is not a spec. Specify the input type (dropdown with 30-minute increments? native `<input type="time">`?), the save behaviour (immediate vs. save button), and the confirmation state. Half a day of implementation debate is avoidable.

## What to carry forward

- **Deferred: Per-user DM digests (v2)** — 2 support tickets in week 1 asking for individual digests. Signal is there. Add to next quarter's roadmap.
- **Deferred: Custom digest content selection (v2)** — no user requests yet. Hold.
- **Deferred: Digest preview in Trackr UI (v2)** — 1 support ticket asking "what will the digest look like before I enable it?" Low volume but easy win. Consider adding a static preview screenshot to the settings page as a stopgap.
- **Follow-up: Prompt teams with `timezone: null` to set their timezone** — small spec, can ship next sprint. Prevents the 1:30am digest problem recurring as more teams enable the feature.
- **Follow-up: Surface digest channel in settings** — teams are asking how to change which Slack channel receives the digest. Currently only set at OAuth time. Add a channel selector to the digest settings UI.
- **Open question: Should we add a "send now" button to the settings page?** Team leads occasionally want an on-demand digest before an unscheduled meeting. Low effort, high value for power users. Evaluate after 60-day data.
