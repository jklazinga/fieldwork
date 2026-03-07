---
name: close-feature
description: "Triggers after a feature has shipped. Compares what was specced against what was built, captures learnings, and feeds back into future discovery. Also triggers if the user says 'run a retro', 'close out this feature', or 'post-launch review'."
---

# Close Feature

## Overview

Run a structured retrospective after a feature ships. The central question is not "did we build what we specced?" but "did we make the right decision, and did it solve the customer problem?" Capture what we learned about the problem, the users, and the opportunity. Feed learnings back into the project context for future discovery.

**Announce at start:** "I'm using the Fieldwork close-feature skill to close out this feature."

## Pre-flight check

1. Load `outputs/specs/{feature-name}/spec.md`.
2. Load `outputs/gtm/{feature-name}/gtm-plan.md` if it exists.
3. Load `outputs/gtm/{feature-name}/launch-brief.md` if it exists.
4. Load `outputs/opportunities/{feature-name}/opportunity.md`.
5. Read git history using the Bash tool: `git log --oneline --since="{spec-approved-date}" --pretty=format:"%h %s"` - check commits and PR descriptions since spec was approved. If spec-approved-date is unknown, ask the user.
6. Create directory `outputs/retros/{feature-name}/` if it does not exist.

## Checklist

### 1. Load the opportunity
Load `outputs/opportunities/{feature-name}/opportunity.md`. The success metrics defined there are the primary measure of success - not spec completeness.

### 2. Check metric availability
Before asking retrospective questions, ask: "Do you have enough data to evaluate the prediction yet - usage data, user feedback, or any signal on whether this solved the problem?"

**If yes:** proceed to step 3.

**If no:** run the deferred retro path:
- Write a partial retro with what's known (what shipped, GTM signal, any early qualitative feedback)
- Set `status: deferred` in the retro frontmatter
- Add a note: "Prediction evaluation deferred - metrics not yet available. Re-run `close-feature` in [agreed timeframe]."
- Ask: "When should we revisit this? I'll note it so it doesn't get lost."
- Save the partial retro and stop. Do not mark documents as `shipped` yet - leave them as `approved`.

### 3. Retrospective questions - one at a time
- Load the "Prediction" from `outputs/opportunities/{feature-name}/opportunity.md`. Read it aloud: "At discovery, you predicted: [prediction]. Let's start there."
- Did the prediction come true? What's the evidence? (If no prediction exists, ask: "Did this solve the customer problem? What's the evidence?")
- What did users actually do with it - did it match the expected behaviour?
- Was the opportunity framing right? Would you frame it differently now?
- What assumptions turned out to be wrong?
- What did the GTM plan get right or wrong about how to reach users?
- What would you do differently in the discovery phase?

### 4. Extract learnings
Identify 3-5 concrete learnings. Each learning must be actionable - not "the spec was unclear" but "acceptance criteria for async operations need to specify timeout behaviour explicitly."

### 5. Update project context
Add learnings to `context/project-context.md` under a "Learnings" section. Add any new constraints discovered to `context/constraints.md`. Do not rewrite the whole file - append only.

### 6. Update document statuses
Update frontmatter `status` to `shipped` in:
- `outputs/specs/{feature-name}/spec.md`
- `outputs/gtm/{feature-name}/gtm-plan.md` (if exists)
- `outputs/gtm/{feature-name}/launch-brief.md` (if exists)

### 7. Write retro
Save to `outputs/retros/{feature-name}/retro.md`.

## Output: outputs/retros/{feature-name}/retro.md

```markdown
---
feature: {feature-name}
status: shipped
retro-date: YYYY-MM-DD
---

# Feature Retro: {Feature Name}

## Prediction vs. reality
_From opportunity.md → Prediction_

**Predicted:** [The prediction written at discovery approval]
**Observed:** [What actually happened - evidence: usage data, user feedback, support signal, qualitative observation]
**Verdict:** [Did the prediction come true? Partially? Not at all?]

## Did we solve the problem?
[What the opportunity said success looked like - and what actually happened. Evidence: user feedback, usage data, support signal, qualitative observation.]

## Was the opportunity framing right?
[Did we frame the problem correctly? What would a better framing have looked like?]

## Assumptions that were wrong
[Which assumptions from assumptions.md turned out to be incorrect - and what we now know instead]

## GTM signal
**What happened vs. the plan:**
- [Channel / tactic - planned vs. actual]

**Early user signal:**
- [Feedback, adoption signal, or qualitative observation]

## Discovery learnings
_What would have made the discovery phase sharper?_
- [Learning - specific enough to change behaviour next time]

## What to carry forward
[Follow-up opportunities, unresolved questions, or new problems surfaced - feed into the next discovery cycle]
```

## After close-feature

"Retro saved to `outputs/retros/{feature-name}/retro.md`.
Learnings added to `context/project-context.md`.
All output documents updated to status: shipped.

Feature closed. The learnings are now part of the project context for future features."
