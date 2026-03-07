---
feature: slack-digest
date: 2026-03-07
gtm-plan: outputs/gtm/slack-digest/gtm-plan.md
---

# Marketing Brief: Daily Slack Digest

## Messaging foundation
**Headline:** No more manual standup prep. Trackr sends the summary for you.
**Value prop:** Trackr now sends a daily digest to your team's Slack — tasks completed, what's due today, what's overdue, and what's blocked. It's already in Slack when you wake up.
**Proof point:** Team leads spend 10-20 minutes every morning reviewing Trackr and writing a Slack summary. This feature gives that time back, every day.

---

## Email announcement (teams with Slack connected)

**Subject line:** No more manual standup prep
**Preview text:** Trackr now sends the summary for you.

Hi [First name],

We just shipped something that'll change how your team starts the day.

Starting now, Trackr automatically sends a daily digest to your team's Slack channel — every morning at 8:30am in your timezone.

The digest covers:
- ✅ Tasks completed yesterday
- 📅 Tasks due today
- 🔴 Overdue tasks
- 🚧 Active blockers

No more opening Trackr before standup. No more copy-pasting updates into Slack. It's already there.

**[See it in your Slack settings →]**

The digest is on by default. You can change the send time or turn it off in Settings → Integrations → Slack.

If you have questions, reply to this email — we read every one.

— The Trackr team

**Notes for designer:** Plain text style. Single CTA. The bullet list above should render as actual bullets, not emoji-prefixed plain text — check rendering in Gmail and Apple Mail. Mobile-first layout.

---

## In-app prompt (teams with Slack connected — shown on first login after deploy)

**Type:** Banner / dismissible prompt
**Placement:** Top of the main board view
**Trigger:** First login after deploy, for teams with Slack connected and `team_digest_settings.enabled = true`
**Copy:**

> **Your team now gets a daily Trackr digest in Slack.**
> Every morning at 8:30am, we'll send a summary of completed tasks, what's due today, overdue items, and blockers — straight to your Slack channel.
> [Change settings] [Got it]

**CTA 1:** "Change settings" → links to `/settings/integrations/slack`
**CTA 2:** "Got it" → dismisses the prompt, sets a `digest_prompt_dismissed` flag so it doesn't reappear

**Notes:** Do not show this prompt to teams where `team_digest_settings.enabled = false` (i.e. teams where the migration set `enabled: false` because no default channel was found). Those teams should see the channel picker in settings instead.

---

## In-app settings tooltip

**Type:** Tooltip
**Placement:** Next to the "Daily Digest" toggle in Settings → Integrations → Slack
**Copy:** Sends a summary of completed tasks, due today, overdue, and blockers to your Slack channel each morning.
**CTA:** None (informational only)

---

## Social — LinkedIn

**Variant 1:**
We just shipped daily Slack digests for Trackr.

Every morning, your team gets a summary in Slack: what was completed yesterday, what's due today, what's overdue, and what's blocked.

No more opening Trackr before standup. No more copy-pasting updates into Slack.

It's already there.

**Variant 2:**
The most common thing we see in Trackr session recordings:

Team lead opens Trackr at 8:25am. Scans the board. Writes a summary. Pastes it into Slack. Every. Single. Morning.

We just automated that.

Daily Slack digests are now live in Trackr. On by default for teams with Slack connected.

**Variant 3:**
Small engineering teams don't need more meetings. They need better async context.

We shipped daily Slack digests in Trackr — a morning summary of what's done, what's due, what's overdue, and what's blocking your team.

Lands in Slack at 8:30am. No login required. No manual prep.

For the team leads who've been doing this by hand every morning: this one's for you.

---

## Changelog entry

**Daily Slack Digest**

Trackr now sends a daily summary to your team's Slack channel — tasks completed yesterday, tasks due today, overdue tasks, and active blockers. The digest sends at 8:30am in your team's timezone by default. You can change the send time or turn it off in Settings → Integrations → Slack. Requires a connected Slack workspace.
