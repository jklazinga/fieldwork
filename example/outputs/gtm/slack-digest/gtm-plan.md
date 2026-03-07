---
feature: slack-digest
status: approved
date: 2026-03-07
spec: outputs/specs/slack-digest/spec.md
---

# GTM Plan: Daily Slack Digest

## Positioning
Trackr now sends your team a daily Slack summary of what's done, what's due, and what's blocked — so you walk into standup already knowing the score.

## Target audience

### Primary: Sam (Engineering Lead)
Sam is the one doing the manual morning ritual — open Trackr, scan the board, copy-paste into Slack. This feature directly eliminates that ritual. He'll notice the change on day one. Lead with the time saved and the standup prep angle, not the technology. Sam doesn't care that it's a Vercel Cron job; he cares that he doesn't have to open Trackr at 8:25am anymore.

### Secondary: Priya (PM)
Priya wants visibility without logging in. The digest lands in the team's Slack channel, which she's already in. She gets passive awareness of project status without any behaviour change on her part. Mention this in the changelog but don't lead with it — Priya is not the one who will enable or disable the feature.

### Tertiary: Jordan (IC)
Jordan doesn't use Trackr much. The digest surfaces blockers to Jordan in Slack, where Jordan already is. No behaviour change required. Don't market to Jordan directly — the value is ambient.

## Value proposition
**Problem:** Team leads spend 10-20 minutes every morning manually reviewing Trackr and writing a Slack summary for standup.
**Before:** Open Trackr, scan the board, write the summary, paste it into Slack. Every. Single. Morning.
**After:** The digest is already in Slack when you wake up. Walk into standup. Done.

## Launch type
- [x] Customer announcement (in-app + email)
- [ ] Quiet rollout
- [ ] Internal announcement only
- [x] Public launch (LinkedIn, changelog)

## Channels & tactics

| Channel | Message | Owner | Timing |
|---|---|---|---|
| In-app prompt (Slack-connected teams only) | "Your team now gets a daily Trackr digest in Slack" | Engineering | On deploy (T-0) |
| Email to all teams with Slack connected | "No more manual standup prep" | PM | T+24h (April 8) |
| Changelog | Feature entry | Engineering | On deploy (T-0) |
| LinkedIn | Feature announcement | PM | T+48h (April 9) |
| In-app settings tooltip | "Digest sends at 8:30am in your timezone. Change it in settings." | Engineering | On deploy (T-0) |

## Timeline

| Milestone | Date | Owner |
|---|---|---|
| Feature complete | April 4, 2026 | Engineering |
| Internal QA + staging verification | April 5–6, 2026 | PM + Engineering |
| Deploy to production | April 7, 2026 | Engineering |
| In-app prompt live | April 7, 2026 (on deploy) | Engineering |
| Customer email send (Slack-connected teams) | April 8, 2026 | PM |
| LinkedIn post | April 9, 2026 | PM |
| Metrics review (2-week check) | April 21, 2026 | PM |
| Retro | April 21, 2026 | PM + Engineering |

## Success metrics
_Consistent with outputs/opportunities/slack-digest/opportunity.md_

- **Retention:** 30-day churn reduced from 28% to 20% for teams with Slack connected and digest enabled, measured at 60 days post-launch (baseline: pull churn cohort by Slack-connected status before April 7)
- **Digest send rate:** ≥85% of scheduled sends succeed (Vercel Cron fires + Slack API accepts) within 30 days of launch
- **Adoption:** ≥60% of Slack-connected teams have digest enabled (not manually disabled) at 30 days
- **Behaviour change:** Average morning Trackr logins (7–9am) drop by ≥30% for digest-enabled teams at 30 days

## Risks

- **Slack API rate limits:** At 200 teams, all digests sending within a narrow morning window could hit Slack's rate limits. Mitigation: the cron fires every minute and staggers sends based on each team's configured time. Monitor Slack API error rate in week 1; if rate limit errors appear, add per-team send delays in the queue.
- **Teams not having Slack connected (limits reach):** The email announcement and in-app prompt only reach Slack-connected teams. Teams without Slack connected won't see this feature at all. Mitigation: add a secondary in-app prompt for non-Slack-connected teams: "Connect Slack to get daily digests." This is a v1.1 item — do not block launch on it.
- **Digest arriving at wrong time (timezone bugs):** DST transitions are the most likely source of off-by-one-hour errors. Mitigation: the spec requires `date-fns-tz` for all timezone calculations and a specific DST boundary test. Verify in staging with a team configured to `America/New_York` before launch.
- **Teams disabling the digest immediately:** If the digest content is noisy or irrelevant, teams may disable it within the first week. Mitigation: monitor disable rate in week 1. If >20% of teams disable within 7 days, investigate digest content quality before the 2-week metrics review.

## Open questions
_All resolved_
- **Which teams get the email?** → Only teams with Slack connected at time of send. PM to pull the list from the DB before April 8.
- **Baseline churn data:** → PM to pull 30-day churn cohort segmented by Slack-connected status before April 7 deploy.
