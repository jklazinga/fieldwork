---
feature: slack-digest
date: 2026-03-07
---

# Assumption Map: Daily Slack Digest

## High importance / weak evidence (riskiest — validate first)

| Assumption | Dimension | Evidence so far |
|---|---|---|
| Teams will keep Slack connected and not disconnect after seeing the digest | Viability / Retention | No data. We don't know what triggers Slack disconnection. No churn interviews have asked about integration behaviour. |
| The digest content is useful enough that team leads don't still check Trackr manually | Desirability / Usability | Assumed from session recordings showing the morning Trackr-to-Slack ritual. No direct research on whether a digest would replace or supplement that behaviour. |
| Teams want a fixed daily send time, not an on-demand summary | Desirability | Assumed. No research on whether teams prefer scheduled delivery vs. a `/trackr digest` Slack command. |

## High importance / strong evidence (monitor)

| Assumption | Dimension | Evidence so far |
|---|---|---|
| Team leads manually copy Trackr updates into Slack before standup | Desirability | Observed in session recordings for teams in the 5-15 person range. Consistent pattern across multiple teams. |
| Slack is the primary async communication channel for our target teams | Viability | ~68% of Trackr teams have connected a Slack workspace (internal data). Strong signal. |
| Reducing morning admin overhead is a top pain point for Sam-type users | Desirability | Mentioned in 4/6 customer interviews in Q1 2026. Consistent with session recording behaviour. |

## Low importance / weak evidence (low priority)

| Assumption | Dimension | Evidence so far |
|---|---|---|
| Jordan (IC) will read the digest and act on blocker information before standup | Usability | No evidence. Low importance because the primary value is for Sam — Jordan's engagement is a bonus. |
| Priya (PM) will use the digest as a substitute for logging into Trackr | Desirability | No evidence. Priya's use case is secondary to Sam's. |
| Teams will want to customise digest content in v2 | Viability | Assumed from general product customisation trends. Not validated. Low priority until v1 ships. |

## Low importance / strong evidence (accepted)

| Assumption | Dimension | Evidence so far |
|---|---|---|
| Vercel Cron is sufficient for scheduling at our current scale (~200 teams) | Feasibility | Vercel Cron supports up to 1 job per minute per project on Pro plan. At 200 teams, we can batch sends within a single cron invocation. Acceptable for v1. |
| Slack Block Kit is the right message format (vs. plain text) | Usability | Slack's own documentation recommends Block Kit for structured messages. Block Kit renders well on mobile and desktop. |

## Validation ideas

**For "teams will keep Slack connected after seeing the digest":**
- Before launch, pull the Slack disconnection rate for teams that have connected Slack in the last 90 days. If >20% disconnect within 30 days of connecting, investigate why before shipping a feature that depends on the connection staying live.
- Post-launch: monitor Slack disconnection events for digest-enabled teams vs. non-digest teams. Alert if disconnection rate spikes in week 1.

**For "the digest replaces the manual Trackr check (not supplements it)":**
- Instrument morning Trackr logins (7am–9am) by team, segmented by digest-enabled status. If logins don't drop after digest is enabled, the digest is not replacing the behaviour — it's adding to it.
- Consider a 2-week follow-up survey to 10 Sam-type users: "Since the digest started, how often do you still open Trackr before standup?" — qualitative signal before the quantitative data is conclusive.

**For "teams want a fixed daily send time (not on-demand)":**
- This can be validated cheaply post-launch by monitoring whether teams change their configured send time. If <10% of teams change the default (8:30am), the assumption holds. If many teams change it or request a Slack command, reconsider the model.
