---
name: measure
description: "Triggers when the user returns after showing a prototype to a user or stakeholder. Phrases like 'I tested it', 'I showed it', 'here's what happened', 'we ran the prototype past someone'. Produces a structured verdict and pre-fills the next skill invocation. Does NOT trigger after launch (use retro) or after a technical spike (use spike debrief). This is: we built something, a human reacted to it, now what?"
---

# Measure

## Overview

Turn a prototype observation into a structured decision. The output is not a research report — it is a verdict with a clear next action. Scoped to prototype evaluation only: something was built, a real person reacted to it, and now the team needs to decide whether to continue, pivot, or stop.

**Announce at start:** "I'm using the Fieldwork measure skill to capture what you learned."

## When to use this

Use measure when:
- A prototype was shown to a user, stakeholder, or customer
- The user returns with observations and needs to decide what to do next
- An assumption from the opportunity doc was tested through a prototype

Do NOT use measure when:
- The feature has launched (use `retro`)
- The spike was a technical question, not a user or stakeholder reaction (use spike debrief)
- No prototype was built — this is raw research (use `synthesise-research`)

## Pre-flight check

1. Load `outputs/opportunities/{feature-name}/opportunity.md` if it exists — specifically the Prediction field and the assumption map.
2. Load `outputs/opportunities/{feature-name}/assumptions.md` if it exists.
3. Load `outputs/spikes/{spike-name}/spike-plan.md` if the prototype came from a spike.
4. Create directory `outputs/opportunities/{feature-name}/` if it does not exist.

## Checklist

### 1. Establish context
Ask one question at a time. Do not present a form.

- "Who saw the prototype, and in what context?" (customer, internal stakeholder, usability session, hallway test — and how many people)
- "What did you expect them to do or say?"
- "What actually happened — what did they do or say that surprised you?"
- "What does this change about what you build next?"

Do not interpret on behalf of the PM. Capture what was observed, not what it means. Ask the PM to draw the conclusion.

### 2. Surface the hypothesis
If an opportunity doc exists, read back the Prediction field: "You were testing whether [prediction]. Does what you observed confirm, refute, or leave that open?"

If no opportunity doc exists, ask: "What were you trying to learn from this prototype?"

### 3. Render the verdict
Based on the PM's answers, propose one of three verdicts:

- **Continue** — the prototype validated the direction. Move to `write-spec` or refine the current spec.
- **Pivot** — the prototype revealed a better framing. Return to `discover` with updated assumptions.
- **Stop** — the prototype invalidated the opportunity. Close it out.

Present the verdict with one sentence of reasoning. Ask the PM to confirm or override.

### 4. Update the assumption map
If `assumptions.md` exists, update any assumptions that were tested:
- Mark confirmed assumptions as `confirmed` with the observation as evidence
- Mark refuted assumptions as `refuted` with the observation as evidence
- Leave untested assumptions unchanged

### 5. Write the next prompt
If verdict is **continue**: pre-fill the opening line for `write-spec`:
> "Write a spec for {feature-name}. The prototype confirmed [what was learned]. The key constraint is [what the observation revealed]."

If verdict is **pivot**: pre-fill the opening line for `discover`:
> "Re-open discovery for {feature-name}. The prototype showed [what was observed]. The original framing assumed [assumption] — that needs revisiting."

If verdict is **stop**: draft a one-line close-out note for the opportunity doc.

### 6. Save the measure doc
Save to `outputs/opportunities/{feature-name}/measure.md`.

Present a summary: who was tested, what was observed, verdict, next step.

Ask: "Does this capture what you learned? Ready to move to [next skill]?"

## Output: outputs/opportunities/{feature-name}/measure.md

```markdown
---
feature: {feature-name}
date: YYYY-MM-DD
prototype: {link, description, or spike reference}
tested-with: {who saw it — role, number of people, context}
verdict: continue | pivot | stop
---

# Measure: {Feature Name}

## Hypothesis tested
[The Prediction from the opportunity doc, or the question the PM was trying to answer]

## Who saw it
[Role, number of people, context — e.g. "2 retail store managers, unmoderated walkthrough"]

## What you observed
[Specific behaviours or reactions — not interpretations. What did they do? What did they say?]

## What changed
[What this observation updates — assumptions, framing, scope, or confidence]

## Verdict
**continue | pivot | stop** — [one sentence of reasoning]

## Next prompt
[Pre-filled opening line for the next discover or write-spec invocation]
```

## After measure

"Measure saved to `outputs/opportunities/{feature-name}/measure.md`.

[Route based on verdict:]"

**Continue:**
"The prototype validated the direction. Ready to move to `write-spec`? Here's your opening prompt:
> [pre-filled prompt]"

**Pivot:**
"The prototype revealed a better framing. Ready to re-open `discover`? Here's your opening prompt:
> [pre-filled prompt]"

**Stop:**
"The prototype invalidated this opportunity. I'll update the opportunity doc status to `closed` and add a close-out note. Want me to do that now?"
