---
name: write-gtm
description: "Triggers after a spec is approved (status: approved). Produces a go-to-market plan covering positioning, audience, channels, and success metrics. Does NOT trigger if spec status is draft or missing."
---

# Write GTM Plan

## Overview

Produce a go-to-market plan grounded in the approved spec, opportunity, and product context. This is not a generic template - it references the actual product, real user segments from product-context.md, and the specific value this feature delivers. It is written for PMs and marketing leads.

**Announce at start:** "I'm using the Fieldwork write-gtm skill to create the go-to-market plan."

<HARD-GATE>
Do NOT write the GTM plan if:
- spec.md status is not `approved` - check frontmatter, do not rely on the user's word
- context/product-context.md is missing or empty - run onboard first

Do NOT proceed to write-marketing or write-launch-brief until this GTM plan has status: approved.
</HARD-GATE>

## Pre-flight check

1. Load `outputs/specs/{feature-name}/spec.md` - confirm `status: approved`. If not, stop.
2. Load `outputs/opportunities/{feature-name}/opportunity.md`.
3. Load `context/product-context.md` - read personas, positioning, channels, competitors. If empty, stop and ask the PM to fill it in before proceeding.
4. Load `context/project-context.md`.
5. Create directory `outputs/gtm/{feature-name}/` if it does not exist.

## Checklist

### 1. Derive positioning from context
Before asking any questions, draft a positioning statement using:
- The problem from opportunity.md
- The personas from product-context.md
- The existing positioning from product-context.md
- The channels already listed in product-context.md

Present the draft and ask: "Does this positioning feel right, or do you want to adjust it?"

### 2. Ask GTM questions - one at a time
Only ask questions that can't be answered from the context files:
- Is this a quiet rollout or a marketing moment?
- Are there any specific customers or partners to involve in the launch?
- Any competitive considerations? (e.g. a competitor just shipped something similar)

### 3. Draft GTM plan - section by section
Present each section for approval before moving to the next.

### 4. Get approval
Ask: "Does this GTM plan reflect how you want to launch this? Ready to approve?"
On approval: update frontmatter status to `approved`.

## Output: outputs/gtm/{feature-name}/gtm-plan.md

```markdown
---
feature: {feature-name}
status: draft | approved
date: YYYY-MM-DD
spec: outputs/specs/{feature-name}/spec.md
---

# GTM Plan: {Feature Name}

## Positioning
[One sentence: what this is, for whom, and why it matters now]

## Target audience
### Primary
[Who - use persona names from product-context.md, not generic descriptions]
[Why them first - what makes this most valuable to them]
[What they care about - the language they use to describe the problem]

### Secondary
[Who else benefits, how they find out]

## Value proposition
[What problem this solves]
[What the alternative was before this]
[What's now possible]

## Launch type
- [ ] Quiet rollout (no announcement) - channels/tactics section is minimal or empty; success metrics are usage-only; no comms sequence in launch brief
- [ ] Internal announcement (team/company only) - channels/tactics covers internal Slack/email only; no external comms; launch brief covers internal rollout sequence
- [ ] Customer announcement (email/in-app) - channels/tactics includes email and in-app messaging; launch brief includes comms sequence and timing; write-marketing produces copy briefs
- [ ] Public launch (blog, social, press) - full channels/tactics; write-marketing produces all asset briefs; launch brief includes press/social sequence, day-of runbook, and rollback plan

## Channels & tactics
_Channels drawn from context/product-context.md_

| Channel | Message | Owner |
|---|---|---|
| [Channel from product-context] | [Specific message] | [PM/Marketing] |

## Success metrics
[How we'll measure GTM success - adoption rate, activation, feature usage, support ticket volume]
[Tie back to success metrics in opportunity.md]

## Risks
[What could go wrong with the launch - not the feature]
[Competitive response, customer confusion, messaging gaps]

## Open questions
[Unresolved GTM decisions - owner for each]
```

## After write-gtm

"GTM plan saved to `outputs/gtm/{feature-name}/gtm-plan.md`.

Next steps:
1. **Marketing brief** - I run `write-marketing` to produce copy and asset briefs
2. **Launch brief** - I run `write-launch-brief` when you're closer to the date
3. **Done for now** - come back when you need marketing assets"
