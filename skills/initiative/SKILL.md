---
name: initiative
description: "Triggers when the bet is large, cross-cutting, or high-stakes - multiple teams, significant investment, or strategic importance. Runs before discovery to establish the strategic frame, stakeholder alignment, and stop condition. Does NOT replace discover - it precedes it."
---

# Initiative

## Overview

An initiative is a collection of bets that share a strategic thesis. Before any feature discovery runs, establish what the initiative is trying to prove, who needs to be aligned, and what would cause you to stop.

This skill produces one document: the initiative brief. It is not a spec. It is not a roadmap. It is the frame that all downstream feature work sits inside.

**Announce at start:** "I'm using the Fieldwork initiative skill to frame this initiative."

## The three questions

Before writing anything, get answers to these three questions. Ask them one at a time.

**1. What is the strategic thesis?**
One sentence. What do you believe is true about the market, the customer, or the product - and what does this initiative do if that belief is correct?

If the PM can't write it in one sentence, the initiative isn't ready. Help them sharpen it, but don't proceed until it exists.

**2. Who has veto power, and are they aligned on the problem framing?**
Not "who are the stakeholders" - specifically, who can stop this initiative, and do they agree on what problem is being solved? Misalignment here will surface after discovery, not before. Surface it now.

**3. What would cause us to stop?**
What signal - from users, from the market, from inside the business - would tell you this initiative is wrong? If you can't name it, you can't make a mid-initiative decision with integrity.

## Checklist

### 1. Write the initiative brief
After the three questions are answered, produce the initiative brief. See output format below.

### 2. Identify the minimum viable initiative
What is the smallest version of this initiative that proves the thesis? This is not the MVP of any individual feature - it is the minimum set of bets that, if they work, validate the strategic thesis.

Write it as: "If we only shipped [X], and it [did Y], that would be enough to know the thesis is right."

### 3. Surface cross-cutting assumptions
Ask: what has to be true for this initiative to work - across all the features inside it? These are the assumptions that, if wrong, invalidate multiple bets at once.

List them. Flag any that are testable with a spike before feature discovery begins. If one exists, recommend running the spike first.

### 4. Set the initiative-level prediction
Write one prediction at the initiative level: what will you observe in 6-12 months if the thesis is correct? This is distinct from per-feature predictions - it operates at the strategic level.

Save it to the initiative brief. It will be read aloud at initiative close.

### 5. Route to discovery
Once the brief is approved, route each feature inside the initiative through the standard feature path: `discover` → `write-spec` → `review-spec` → `write-plan` → `scaffold-tasks` → `write-gtm` → `write-launch-brief` → `close-feature`.

All gates are required. If a PM wants to skip a gate, ask why - don't just comply.

## Output: outputs/initiatives/{initiative-name}/initiative-brief.md

```markdown
---
initiative: {initiative-name}
status: draft | approved
created: YYYY-MM-DD
---

# Initiative Brief: {Initiative Name}

## Strategic thesis
_{One sentence: what you believe is true, and what this initiative does if that belief is correct}_

## Problem at initiative scale
_{What customer problem does this initiative solve - not per feature, but the whole thing}_

## Stakeholder alignment
_{Who has veto power. Are they aligned on the problem framing? What do they need to see?}_

## Stop condition
_{What signal would tell you this initiative is wrong - and cause you to stop or pivot}_

## Minimum viable initiative
_{The smallest version that proves the thesis. "If we only shipped X, and it did Y, that would be enough."}_

## Features inside this initiative
_{List of bets. Each will run through the full feature path.}_
- [ ] {feature-name} - {one-line description}

## Cross-cutting assumptions
_{Assumptions that have to be true across multiple features. Flag any testable by spike.}_
- {assumption} - [testable by spike? Y/N]

## Initiative-level prediction
_{What will you observe in 6-12 months if the thesis is correct?}_
```

## After initiative brief is approved

"Initiative brief saved to `outputs/initiatives/{initiative-name}/initiative-brief.md`.

Ready to begin feature discovery. Route each feature through the standard feature path. All gates required."
