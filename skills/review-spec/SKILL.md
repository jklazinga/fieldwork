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

## Dispatch reviewer subagent

After pre-flight, do NOT run the critique yourself. Instead:

1. Read `skills/review-spec/reviewer-prompt.md`.
2. Construct the subagent prompt: paste `reviewer-prompt.md` in full, then append each loaded document inline under a clearly labelled heading:
   - `## SPEC` — full content of spec.md
   - `## OPPORTUNITY` — full content of opportunity.md
   - `## ASSUMPTIONS` — full content of assumptions.md
   - `## CONSTRAINTS` — full content of constraints.md
3. Dispatch via the `Task()` tool. Wait for findings.
4. Parse findings into BLOCKER / IMPORTANT / MINOR groups.
5. Present findings to the PM and proceed with the triage loop below.

**If `Task()` is not available:** Run the critique dimensions from `reviewer-prompt.md` yourself, but frame it explicitly: "I am now reviewing this spec as if I did not write it." Do not soften findings.

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
