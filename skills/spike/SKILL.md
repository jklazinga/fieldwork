---
name: spike
description: "Triggers when the bet is small or reversible, when a key assumption is testable by building, or when the user says 'prototype', 'spike', 'quick experiment', or 'let's just try it'. Produces a spike plan: what to build, what question it answers, what result changes the decision. Does NOT produce a spec - the output is a learning, not a commitment."
---

# Spike

## Overview

Run a time-boxed prototype to answer a specific question before committing to a spec. The output is not a document - it is a test. The question drives everything: what you build, how long you spend, and what you do with the result.

**Announce at start:** "I'm using the Fieldwork spike skill to plan this experiment."

## When to use this

Use spike when:
- The bet is small or easily reversible
- A key assumption is high-importance / weak-evidence and testable by building
- The fastest way to learn is to make something, not to document something
- The user says "prototype", "spike", "quick experiment", or "let's just try it"

Do NOT use spike when:
- The opportunity is already validated and the team is ready to commit
- The question is about market fit, not technical or UX feasibility (talk to users instead)
- The spike would take longer than a proper spec + implementation

## Pre-flight check

1. Does `context/project-context.md` exist? If no - ask the user for the minimum context needed (tech stack, codebase location). Do not require full onboard for a spike.
2. Does an opportunity document already exist for this idea? If yes - load it. The spike may be testing a specific assumption from the assumption map.
3. Is there a specific assumption from `outputs/opportunities/{feature-name}/assumptions.md` that triggered this spike? If yes - load it and use it as the spike question.

## Checklist

### 1. Define the question
The spike must answer exactly one question. If the user has multiple questions, pick the most important one. The question must be:
- Specific (not "does this work?" but "can we render 10,000 rows in under 100ms with our current table component?")
- Answerable by building (not by research or user interviews)
- Binary or near-binary (yes/no, fast/slow, feasible/not feasible)

Ask: "What's the one question this spike needs to answer?"

### 2. Define the decision
What changes based on the answer?
- If yes → [next step: proceed to discover/spec, or proceed with this approach]
- If no → [next step: explore alternative, descope, or kill the idea]

If the answer won't change anything, the spike isn't worth running.

### 3. Define the timebox
Spikes are time-boxed. Default: half a day. Maximum: two days. If it takes longer, it's a feature, not a spike.

Ask: "How long are you willing to spend on this before deciding?"

### 4. Define done
What does the spike produce?
- A working prototype (even if throwaway)
- A benchmark result
- A recorded demo
- A yes/no answer with evidence

The spike is done when the question is answered - not when the prototype is polished.

### 5. Write the spike plan
Save to `outputs/spikes/{spike-name}/spike-plan.md`.

### 6. After the spike runs
When the user returns with results, run the debrief (see below). Do not skip it.

## Output: outputs/spikes/{spike-name}/spike-plan.md

```markdown
---
spike: {spike-name}
status: planned | running | complete
date: YYYY-MM-DD
timebox: {N} hours
linked-opportunity: outputs/opportunities/{feature-name}/opportunity.md (if exists)
linked-assumption: {assumption text} (if triggered by assumption map)
---

# Spike: {Spike Name}

## Question
[The one specific question this spike answers]

## Why this question
[What assumption or uncertainty is driving this - reference the opportunity or assumption map if they exist]

## Decision
**If yes:** [What we do next]
**If no:** [What we do instead]

## What to build
[Minimum thing needed to answer the question - not a feature, a test]
[Be specific: what to implement, what to ignore, what shortcuts are acceptable]

## Timebox
[N hours / N days - hard stop]

## Done when
[Specific, observable result that answers the question]

## What to ignore
[Explicitly list what is out of scope for this spike - polish, error handling, tests, etc.]
```

## Debrief (run when spike is complete)

When the user returns with results:

1. Ask: "What did you find?"
2. Ask: "Does the result change the decision?"
3. Update `spike-plan.md` status to `complete`. Add a "Result" section.
4. Route based on outcome:

**If spike validates the opportunity:**
- "The spike answered the question. Ready to move to `discover` to formalise the opportunity?" (or `write-spec` if an opportunity already exists and is approved)
- If the spike was triggered by a specific assumption from `outputs/opportunities/{feature-name}/assumptions.md`, update that assumption's evidence and confidence rating now. Do not wait for the retro.

**If spike invalidates the opportunity:**
- "The spike answered the question - this path doesn't work. Want to explore an alternative framing, or close this out?"
- If triggered by an assumption, mark it as invalidated in `assumptions.md`. This changes the opportunity framing - surface that explicitly.

**If spike is inconclusive:**
- "The spike didn't fully answer the question. Options: run another spike with a tighter question, or proceed with the uncertainty acknowledged in the assumption map."
- If triggered by an assumption, note the inconclusive result in `assumptions.md` and flag it as still high-importance / weak-evidence.

## Output: Result section (appended to spike-plan.md)

```markdown
## Result
_Added: YYYY-MM-DD_

**Answer:** [Yes / No / Inconclusive]
**Evidence:** [What was observed - benchmark numbers, demo link, screenshot, code snippet]
**Decision:** [What we're doing next - and why]
```

## After spike

"Spike complete. Result saved to `outputs/spikes/{spike-name}/spike-plan.md`.

[Route based on outcome - see debrief section above]"
