---
feature: slack-digest
status: approved
date: 2026-03-07
---

# Opportunity: Daily Slack Digest

## Problem
Engineering team leads at Trackr-connected teams start every morning the same way: open Trackr, scan the board, mentally summarise what's done, what's overdue, and what's blocked, then copy-paste that summary into Slack for standup. This ritual takes 10-20 minutes per day and is entirely manual. It's also error-prone — leads miss tasks, forget blockers, or skip the update entirely when they're running late. The result is standups that start without shared context, and team members (especially Jordan-type ICs) who don't know what's blocking them until the meeting starts.

This is the most common workflow we see in session recordings for teams in the 5-15 person range. It's also the workflow that, when it breaks down, drives churn: teams that stop doing the morning Trackr check tend to stop using Trackr within 2-3 weeks.

## Current behaviour
Team leads (Sam) open Trackr each morning before standup. They manually review the board — completed tasks, overdue tasks, active blockers — and write a summary in Slack. There is no automated summary, no scheduled delivery, and no way for team members to get this context without either logging into Trackr themselves or waiting for Sam to post it. Priya (PM) has no visibility unless she logs in separately. Jordan (IC) almost never logs in unprompted.

## Proposed opportunity
Trackr sends a daily digest to the team's connected Slack workspace at a configured time (default 8:30am in the team's timezone). The digest covers: tasks completed yesterday, tasks due today, overdue tasks, and active blockers. It replaces the manual morning ritual for Sam, gives Priya passive visibility, and surfaces blockers to Jordan before standup starts — without anyone having to log into Trackr.

The digest is on by default for teams with Slack already connected. No new OAuth setup required. Teams can disable it or change the delivery time in Trackr settings.

## Success metrics
- **Retention (primary):** 30-day churn reduced from 28% to 20% for teams with Slack connected and digest enabled, measured at 60 days post-launch. Baseline: pull churn cohort data by Slack-connected status before launch.
- **Engagement:** Digest send rate ≥85% of scheduled sends (i.e. Vercel Cron fires and Slack API accepts) within 30 days of launch.
- **Adoption:** ≥60% of teams with Slack connected have digest enabled (not manually disabled) at 30 days.
- **Behaviour change:** Average number of manual Trackr logins between 7am–9am drops by ≥30% for digest-enabled teams at 30 days (proxy for "Sam stopped doing the manual check").

## Riskiest assumptions
1. **Teams will keep Slack connected and not disconnect after seeing the digest** — high importance, weak evidence. We assume teams find the digest useful enough to keep the integration live. If the digest content is noisy or irrelevant, teams may disconnect Slack entirely, which would be worse than the status quo.
2. **The digest content is useful enough that team leads don't still check Trackr manually** — high importance, weak evidence. If Sam reads the digest and then opens Trackr anyway to verify, we've added noise without removing the behaviour. We need the digest to be accurate and complete enough to be the single source of truth for standup prep.
3. **Vercel Cron is reliable enough for time-sensitive delivery** — medium importance, weak evidence. Vercel Cron has a 1-minute precision limit and is not guaranteed to fire at the exact configured time. For a standup digest, a 5-minute delay is acceptable; a 30-minute delay is not. We have no production data on Vercel Cron reliability at our scale.

## Scope boundary
- **In scope (v1):** Daily digest to a single configured Slack channel per team. Digest content: tasks completed yesterday, tasks due today, overdue tasks, active blockers. Configurable send time (default 8:30am team timezone). On by default for Slack-connected teams. Can be disabled in settings.
- **Not in scope (v2):** Per-user DM digests. Custom digest content selection. Digest preview in Trackr UI. Weekly digest cadence. Multiple channels per team. Digest for specific projects only.

## Open questions
- Should the digest fire if there is no activity to report (e.g. a team with zero completed tasks, zero overdue, zero blockers)? Sending an empty digest may feel noisy. Skipping it silently may feel unreliable.
- Who is the Slack sender — a Trackr bot user, or the workspace's connected OAuth user? This affects how the message appears in Slack and whether it can be @mentioned.
- If a team changes their configured send time, does the change take effect the same day or the next day?

## Prediction
- **30 days:** Digest is sending reliably for ≥60% of Slack-connected teams. Early qualitative signal from Sam-type users: "I don't have to open Trackr before standup anymore." Churn data not yet conclusive — too early.
- **60 days:** Measurable reduction in morning Trackr logins for digest-enabled teams. Churn cohort data starts to show separation between digest-enabled and digest-disabled teams. At least one team asks for per-user DM digests (v2 signal).
- **90 days:** 30-day churn for digest-enabled teams is at or below 20%. Parking lot items (DM digests, content selection) are prioritised based on actual usage patterns. If churn hasn't moved, we investigate whether the digest content is the problem or whether the churn drivers are elsewhere.
