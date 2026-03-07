---
name: write-spec
description: "Triggers after an opportunity document exists with status: approved. Reads the codebase and opportunity document to produce a grounded, engineering-ready feature spec. Does NOT trigger if opportunity status is draft or missing - run discover first."
---

# Write Spec

## Overview

Produce a feature spec grounded in the actual codebase and approved opportunity. This is not a generic PRD. It references real file paths, existing architecture, and tech stack constraints. Acceptance criteria are testable and machine-verifiable where possible. It is written for engineers and the AI agents that will implement it.

**Announce at start:** "I'm using the Fieldwork write-spec skill to write the feature spec."

<HARD-GATE>
Do NOT invoke write-gtm or any downstream skill until:
1. The spec has been written AND
2. The user has explicitly approved it (status: approved in frontmatter)

Do NOT start if opportunity status is not `approved`. Tell the user to run `discover` first.
</HARD-GATE>

## Pre-flight check

1. Load `outputs/opportunities/{feature-name}/opportunity.md` - check status is `approved`. If not, stop.
2. Load `outputs/opportunities/{feature-name}/assumptions.md` if it exists (from discover or Fieldwork discover). This becomes the riskiest assumptions section - do not re-derive from scratch.
3. Load `context/project-context.md`.
4. Load `context/product-context.md`.
5. Load `context/constraints.md`.
6. Create directory `outputs/specs/{feature-name}/` if it does not exist.

## Checklist

### 1. Read the relevant codebase
Identify all files, modules, and patterns relevant to this feature:
- Entry points (routes, controllers, API handlers)
- Data models or schema files
- Existing similar features (for pattern consistency)
- Test files for adjacent features (to understand test conventions)
- Config or feature flag infrastructure

Document the file paths you read. They go into the spec's technical notes section.

### 2. Draft the spec - section by section
Present each section to the PM before moving to the next. Do not write the whole spec and present it at the end.

**Section order:**
1. Problem statement + proposed solution
2. Scope (in / out)
3. User flows
4. Acceptance criteria
5. Technical notes
6. Riskiest assumptions (loaded from assumptions.md - do not re-derive)
7. Open questions
8. Dependencies
9. Parking Lot (always present - even if empty)

**Parking lot capture - during drafting:**
- **Explicit prefix** (`> later:`, `> parking:`, `> future:`): route directly to `## Parking Lot`. No confirmation needed.
- **Natural language signals** ("out of scope but...", "not now but...", "we should also consider...", "future state:", "down the track..."): do NOT auto-route. Instead, ask: *"That sounds like it might be a parking lot item - want to park it?"* Only move it if the PM confirms.

Do not add parked items to scope, user flows, or acceptance criteria. Capture them verbatim with a one-line note on why they were parked.

Do not ask "anything to park?" after each section. Ask once, just before step 5a: *"Before we wrap - anything to park that didn't get flagged inline?"*

### 3. Acceptance criteria rules
Every criterion must be testable. Apply this test: could an engineer write an automated test for this criterion?
- ✅ "Returns HTTP 200 with a JSON body containing `{id, name, status}`"
- ✅ "Renders the empty state component when the list has zero items"
- ✅ "Completes in under 200ms at p95 under normal load"
- ❌ "Works correctly"
- ❌ "Feels intuitive"
- ❌ "Handles errors gracefully"

If a criterion can't be made machine-verifiable, make it manually verifiable with a specific, unambiguous check.

### 4. Map back to the opportunity
Before finalising: check that every success metric from `opportunity.md` is reflected in at least one acceptance criterion. If a metric has no corresponding criterion, flag it and ask the PM to either add a criterion or remove the metric.

### 5. Resolve open questions
Ask the PM to close any open questions before finalising. Do not leave questions that engineering needs answered.

### 5a. Review the parking lot
Before saving, present the Parking Lot to the PM:
- If empty: confirm nothing was missed ("Nothing was parked - does that feel right?")
- If populated: read each item back and ask: keep (stays in parking lot), promote (add to this spec), or discard

Do not promote items without explicit PM confirmation.

### 6. Save spec
Set status to `draft`. Ask PM to review and approve.

### 7. Get approval
Present a summary: solution, scope, acceptance criteria count, top riskiest assumption.
Ask: "Does this spec capture what we're building? Ready to approve?"
On approval: update frontmatter status to `approved`.

## Output: outputs/specs/{feature-name}/spec.md

```markdown
---
feature: {feature-name}
status: draft | approved
date: YYYY-MM-DD
opportunity: outputs/opportunities/{feature-name}/opportunity.md
---

# Spec: {Feature Name}

## Problem statement
[One paragraph. Why this matters. Reference the opportunity. Do not repeat it verbatim - synthesise.]

## Proposed solution
[What we're building. Concrete, not abstract. What it does, not how it looks.]

## Scope
### In scope
- [Specific behaviour 1]
- [Specific behaviour 2]

### Out of scope
- [Explicit non-goal 1 - and why it's excluded]
- [Explicit non-goal 2]

## User flows
### Flow 1: [Name]
1. User does X
2. System does Y
3. User sees Z

### Error / edge cases
- [What happens when X fails]
- [What happens when the list is empty]
- [What happens when the user has no permission]

## Acceptance criteria
- [ ] [Testable criterion - specific, unambiguous, verifiable]
- [ ] [Testable criterion]
- [ ] [Testable criterion]

_Each criterion maps to a success metric from the opportunity doc._

## Technical notes
### Files to create
- `exact/path/to/new-file` - [purpose]

### Files to modify
- `exact/path/to/existing-file` - [what changes and why]

### Patterns to follow
- [Existing pattern in the codebase this feature should match]

### Known constraints
- [Tech debt or fragile areas to be careful around]
- [Performance requirements]
- [Security considerations]

## Riskiest assumptions
_Loaded from outputs/opportunities/{feature-name}/assumptions.md_

| Assumption | Dimension | Evidence | Mitigation |
|---|---|---|---|
| [Assumption] | Desirability / Viability / Feasibility / Usability | [What we know] | [How we reduce risk] |

## Open questions
_All questions must be resolved before engineering starts._
- [Question - owner]

## Dependencies
- [Other feature, service, or decision this depends on]
- [External API or third-party dependency]

## Parking Lot
_Ideas that surfaced during spec drafting but are out of scope for this feature. Triage separately._

| Idea | Why parked |
|---|---|
| [Idea captured verbatim] | [Out of scope / future phase / separate initiative] |
```

## After write-spec

"Spec saved to `outputs/specs/{feature-name}/spec.md`.

Two paths from here:
1. **Review first** - I run `review-spec` to critique the spec before handing off (recommended)
2. **Proceed** - I run `write-gtm` to plan go-to-market in parallel with execution

Which would you like?"
