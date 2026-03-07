---
name: write-launch-brief
description: "Triggers when a feature is approaching launch and the user asks for a launch brief, launch plan, or launch runbook. Produces a launch brief: final checklist, comms sequence, rollback plan, and day-of runbook. Requires an approved GTM plan."
---

# Write Launch Brief

## Overview

Produce a launch brief that coordinates everything needed to ship safely and communicate clearly. Covers the day-of sequence, who does what, how to monitor, and what to do if something goes wrong.

**Announce at start:** "I'm using the Fieldwork write-launch-brief skill to produce the launch brief."

<HARD-GATE>
Do NOT write the launch brief if:
- gtm-plan.md does not exist or status is not `approved` - run write-gtm first

If marketing-brief.md does not exist, warn the user: "No marketing brief found - the comms sequence section will be incomplete. Run write-marketing first, or continue without it."
Do NOT silently skip the comms section.
</HARD-GATE>

## Pre-flight check

1. Load `outputs/gtm/{feature-name}/gtm-plan.md` - confirm `status: approved`. If not, stop.
2. Load `outputs/specs/{feature-name}/spec.md` - acceptance criteria become the launch criteria checklist.
3. Check for `outputs/gtm/{feature-name}/marketing-brief.md` - if missing, warn the user (see HARD-GATE above).
4. Load `context/project-context.md` - check MCP availability for any monitoring integrations.
5. Create directory `outputs/gtm/{feature-name}/` if it does not exist (it should already).

## Checklist

### 1. Ask launch questions - one at a time
- What's the launch date and time (and timezone)?
- Is this a feature flag rollout, staged rollout, or hard release?
- Who needs to be notified internally before it goes live? (team, leadership, support)
- What's the rollback plan if something breaks? (flag off, revert deploy, DB migration rollback?)
- Who is on-call for the launch window?
- What monitoring is in place? (error rates, adoption metrics, support ticket volume)
- Are there any feature flags involved? What's the flag name and where is it configured?

### 2. Derive launch criteria from spec
Load acceptance criteria from spec.md. These become the pre-launch verification checklist. Do not create a separate checklist - use the spec's criteria directly.

### 3. Draft launch brief - section by section
Present each section for review before moving to the next.

### 4. Save launch brief

## Output: outputs/gtm/{feature-name}/launch-brief.md

```markdown
---
feature: {feature-name}
launch-date: YYYY-MM-DD HH:MM TZ
status: draft | approved | shipped
---

# Launch Brief: {Feature Name}

## Launch summary
**Date/time:** [When - include timezone]
**Type:** feature flag rollout | staged rollout | hard release
**Owner:** [PM name]
**On-call engineer:** [Name]
**Rollback owner:** [Name - who pulls the trigger if something goes wrong]

## Pre-launch checklist
_Derived from spec acceptance criteria_

- [ ] [Acceptance criterion 1 from spec.md]
- [ ] [Acceptance criterion 2 from spec.md]
- [ ] Feature flag configured: `[flag-name]` in `[config location]`
- [ ] Internal team notified: [who, via what channel, by when]
- [ ] Customer comms scheduled: [channel, send time]
- [ ] Monitoring/alerts configured: [what's being watched]
- [ ] Rollback plan confirmed with on-call engineer
- [ ] Support team briefed: [what they need to know]

## Launch sequence
| Time | Action | Owner |
|---|---|---|
| T-48h | Internal preview / demo | PM |
| T-24h | Final QA sign-off against acceptance criteria | Engineering |
| T-4h | Confirm on-call coverage | Engineering |
| T-0 | Enable flag / deploy | Engineering |
| T+1h | Check error rates, confirm no spike | Engineering |
| T+2h | Confirm adoption metric baseline captured | PM |
| T+24h | Send customer email (if applicable) | Marketing |
| T+7d | Review adoption metrics, schedule retro | PM |

## Rollback plan
**Trigger condition:** [What would cause a rollback - error rate threshold, critical bug, data issue]
**Steps:**
1. [Exact step - e.g. "Set feature flag `{flag-name}` to false in {config location}"]
2. [Exact step]
3. [Notify: who, via what channel]
**Estimated rollback time:** [How long to fully revert]

## Monitoring
| Metric | Baseline | Alert threshold | Owner |
|---|---|---|---|
| Error rate | [current] | [threshold] | Engineering |
| Adoption | - | - | PM |
| Support tickets | [current] | [threshold] | Support |

## Comms sequence
_Derived from marketing-brief.md_

| Time | Channel | Message | Owner | Status |
|---|---|---|---|---|
| T-0 | In-app | [tooltip/banner copy] | Engineering | scheduled |
| T+24h | Email | [subject line] | Marketing | scheduled |
| T+0 | Social | [post copy] | Marketing | scheduled |

## Post-launch
- [ ] Confirm metrics baseline captured (T+2h)
- [ ] Check error rates at T+24h
- [ ] Review adoption at T+7d
- [ ] Run `close-feature` retro at T+14d or after metrics stabilise
```

## After write-launch-brief

"Launch brief saved to `outputs/gtm/{feature-name}/launch-brief.md`.

After launch, run `close-feature` to capture learnings and close out the feature."
