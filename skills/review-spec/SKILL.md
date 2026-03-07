---
name: review-spec
description: "Triggers after a spec has been written (status: draft). Runs a structured critique to find ambiguities, missing edge cases, and assumption violations before engineering begins. Does NOT trigger on an already-approved spec unless the user explicitly asks to re-review."
---

# Review Spec

## Overview

Critique the spec before engineering starts. Find the gaps, ambiguities, and hidden assumptions that will cause rework later. Be direct. Do not soften findings. Ask the PM to resolve blockers before proceeding.

**Announce at start:** "I'm using the Fieldwork review-spec skill to critique the spec."

<HARD-GATE>
Do NOT mark the spec as approved until all BLOCKER issues are resolved.
Do NOT invoke write-gtm or any downstream skill until spec status is `approved`.
</HARD-GATE>

## Pre-flight check

1. Load `outputs/specs/{feature-name}/spec.md` - confirm it exists and status is `draft`.
2. Load `outputs/opportunities/{feature-name}/opportunity.md` - for alignment checks.
3. Load `outputs/opportunities/{feature-name}/assumptions.md` - for assumption checks.
4. Load `context/constraints.md` - for feasibility checks.
5. Create directory `outputs/specs/{feature-name}/` if it does not exist (it should already).

## Critique dimensions

Run all dimensions. Do not skip any.

### Clarity
- Are acceptance criteria testable? Flag any that use vague language ("works correctly", "handles gracefully", "feels intuitive").
- Are user flows complete? Check: what happens on error? On empty state? On permission denied? On network failure?
- Is scope unambiguous? Could an engineer interpret any in-scope item two different ways?
- Are all terms defined? (No undefined jargon, no assumed shared knowledge)

### Completeness
- Are all user types covered? (Check personas from product-context.md)
- Are all entry points to this feature documented? (Direct navigation, deep links, API calls, background jobs)
- Are dependencies identified and resolved? (No "assumes X exists" without confirming X exists)
- Are all error states documented?

### Assumption alignment
- Load `assumptions.md`. For each high-importance / weak-evidence assumption: does the spec acknowledge it? Does it have a mitigation?
- Does the spec assume existing infrastructure that may not exist? (Check against project-context.md)
- Does the spec assume user behaviour that hasn't been validated?

### Feasibility
- Do the technical notes reference actual files in the codebase? (Not invented paths)
- Does the spec ignore known tech debt or fragile areas from constraints.md?
- Are performance requirements realistic given the current architecture?
- Are there security implications not addressed?

### Opportunity alignment
- Does the proposed solution actually solve the problem stated in opportunity.md?
- Is every success metric from opportunity.md reflected in at least one acceptance criterion? Flag any metric with no corresponding criterion.
- Does the scope boundary in the spec match the scope boundary in the opportunity?
- Does a "Prediction" exist in opportunity.md? If yes: does this spec give us a real shot at achieving it? If the spec as written would not plausibly produce the predicted outcome, flag it as a BLOCKER.

### Spec quality
- Is the out-of-scope section explicit? (Not just empty)
- Are open questions listed? Are they actually open, or have they been answered in the spec body?
- Is the spec self-contained? Could an engineer implement this without asking the PM any questions?

## Severity levels

- **BLOCKER** - Must be resolved before engineering starts. Creates ambiguity that will cause rework or incorrect implementation.
- **IMPORTANT** - Should be resolved. Creates rework risk if left. Engineering can start but will likely need to come back.
- **MINOR** - Nice to fix. Low risk if left. Note it and move on.

## Presenting findings

Group by severity. For each issue:
- State the problem clearly
- State what's needed to resolve it
- Do not suggest the fix - ask the PM to resolve it

After presenting all findings, state the summary: "There are [N] blockers, [N] important issues, and [N] minor issues."

Then immediately begin the triage loop below - do not wait for the PM to ask.

## Triage loop

Work through issues in order: blockers first, then important, then minor.

**For blockers and important issues - no opt-out:**
- Present the issue (one at a time)
- Wait for PM resolution
- Confirm the resolution is sufficient
- Update the spec directly (do not create a separate "fixed" file)
- Check the issue off in the review file
- Move to the next issue

**For minor issues - offer an opt-out:**
After all blockers and important issues are resolved, say:
"[N] minor issues remain - want to go through them, or is this good enough to proceed?"
Proceed based on PM's answer.

**If there are no blockers and no important issues:**
Say: "[N] minor issues found. Spec looks solid - want to address these or proceed to engineering?"

**When all required issues are resolved:**
- Ask: "All blockers resolved. One more question before we approve: is this spec complete enough that rework would be more expensive than starting? Or is there anything you'd rather resolve now?"
- On approval: update `spec.md` frontmatter status to `approved`
- Update `spec-review.md` with resolution notes

## Output: outputs/specs/{feature-name}/spec-review.md

```markdown
---
feature: {feature-name}
date: YYYY-MM-DD
spec: outputs/specs/{feature-name}/spec.md
reviewer: fieldwork:review-spec
---

# Spec Review: {Feature Name}

## Blockers
- [ ] [Issue - what's wrong - what's needed to resolve]

## Important
- [ ] [Issue - what's wrong]

## Minor
- [ ] [Issue]

## Assumption gaps
[Any high-importance / weak-evidence assumptions not addressed in the spec]

## Opportunity alignment
[Any success metrics from opportunity.md not reflected in acceptance criteria]

## Summary
[One paragraph overall assessment - be direct]

---
## Resolution log
_Updated as issues are resolved_

- [Issue] - resolved [date]: [how it was resolved]
```

## After review-spec

Once spec status is `approved`: "Spec approved. Ready to proceed with `write-gtm`?"
