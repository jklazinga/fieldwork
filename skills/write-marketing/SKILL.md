---
name: write-marketing
description: "Triggers after a GTM plan is approved (status: approved). Produces marketing copy and asset briefs: email, in-app messages, social posts, blog outline, and sales enablement. Does NOT trigger if GTM plan status is draft or missing."
---

# Write Marketing Brief

## Overview

Produce ready-to-use marketing copy and asset briefs grounded in the GTM plan, spec, and product context. Not generic templates - actual draft copy the team can use or adapt. Written for marketing leads, content writers, and designers.

**Announce at start:** "I'm using the Fieldwork write-marketing skill to produce marketing assets."

<HARD-GATE>
Do NOT write marketing assets if:
- gtm-plan.md status is not `approved` - run write-gtm first
- context/product-context.md is missing or empty - run onboard first
</HARD-GATE>

## Pre-flight check

1. Load `outputs/gtm/{feature-name}/gtm-plan.md` - confirm `status: approved`. If not, stop.
2. Load `outputs/specs/{feature-name}/spec.md`.
3. Load `context/product-context.md` - read positioning, personas, channels, existing tone.
4. Load `context/project-context.md`.
5. If existing marketing copy is available in the codebase or docs, read it to match tone.
6. Create directory `outputs/gtm/{feature-name}/` if it does not exist (it should already).

## Copy principles

Apply to every asset:
- Lead with the user benefit, not the feature name
- Use the language from opportunity.md - how users describe the problem, not how engineers describe the solution
- Be specific - avoid "powerful", "seamless", "intuitive", "game-changing"
- One CTA per asset
- Match the product's existing tone (read existing copy if available)
- Reference personas from product-context.md - write for a specific person, not a generic user

## Asset menu

Present this list and ask which to produce. Do not produce all assets by default.

- [ ] **Email announcement** - subject line + body (existing customers)
- [ ] **In-app message** - tooltip, modal, or banner copy
- [ ] **Social posts** - LinkedIn + Twitter/X (3 variants each)
- [ ] **Blog post outline** - headline, subheads, key points, CTA
- [ ] **Sales one-pager** - what it is, who it's for, how to pitch it
- [ ] **Changelog entry** - short, user-facing release note
- [ ] **Press release outline** - if public launch

For each selected asset: draft it, present it, get approval, then move to the next. Do not batch all assets and present at the end.

## Output: outputs/gtm/{feature-name}/marketing-brief.md

```markdown
---
feature: {feature-name}
date: YYYY-MM-DD
gtm-plan: outputs/gtm/{feature-name}/gtm-plan.md
---

# Marketing Brief: {Feature Name}

## Messaging foundation
**Headline:** [One sentence - the thing we want people to remember]
**Value prop:** [Two sentences - problem + solution, in user language]
**Proof point:** [One concrete example, stat, or user outcome]

---

## [Asset: Email announcement]
**Subject line:** [Subject]
**Preview text:** [Preview]

[Body copy]

**CTA:** [Button text + destination]
**Notes for designer:** [Any specific guidance - layout, imagery, tone]

---

## [Asset: In-app message]
**Type:** tooltip | modal | banner
**Trigger:** [When does this appear]
**Copy:** [Message text]
**CTA:** [Button text]

---

## [Asset: Social - LinkedIn]
**Variant 1:** [Post copy]
**Variant 2:** [Post copy]
**Variant 3:** [Post copy]
**Hashtags:** [If relevant]

---

## [Asset: Social - Twitter/X]
**Variant 1:** [Post copy - 280 chars max]
**Variant 2:**
**Variant 3:**

---

## [Asset: Blog post outline]
**Headline:** [Title]
**Subheads:**
1. [Section 1]
2. [Section 2]
3. [Section 3]
**Key points per section:** [Bullets]
**CTA:** [What the reader should do]

---

## [Asset: Sales one-pager]
**What it is:** [One sentence]
**Who it's for:** [Persona name + context]
**The problem it solves:** [Two sentences]
**How to pitch it:** [Three talking points]
**Objection handling:** [Top 2 objections + responses]

---

## [Asset: Changelog entry]
[Short, user-facing release note - 2-4 sentences, benefit-first]

---

## [Asset: Press release outline]
**Headline:** [Title]
**Dateline:** [City, Date]
**Lead paragraph:** [Who, what, why it matters]
**Quote:** [PM or CEO quote]
**Boilerplate:** [Standard company description]
```

## After write-marketing

"Marketing brief saved to `outputs/gtm/{feature-name}/marketing-brief.md`.

Next: run `write-launch-brief` when you're approaching the launch date."
