---
name: close-initiative
description: "Triggers after all features inside an initiative have shipped and been closed. Evaluates the initiative thesis against what was observed. Distinct from close-feature - operates at the strategic level, not the feature level. Also triggers if the user says 'close the initiative', 'initiative retro', or 'did the initiative work'."
---

# Close Initiative

## Overview

Run a strategic retrospective after all initiative features have shipped. The per-feature retros captured feature-level learnings. This skill asks the initiative-level question: did the thesis prove out?

**Announce at start:** "I'm using the Fieldwork close-initiative skill to close out this initiative."

## Pre-flight check

1. Load `outputs/initiatives/{initiative-name}/initiative-brief.md`.
2. Confirm all features listed in the brief have a corresponding retro in `outputs/retros/`.
3. If any feature retros are missing, flag them. Do not proceed until they exist - the initiative close depends on them.

## Checklist

### 1. Read the thesis and prediction aloud
Load the initiative brief. Read aloud:
- "The strategic thesis was: [thesis]."
- "At the start, you predicted: [initiative-level prediction]. Let's start there."

### 2. Evaluate the thesis
- Did the prediction come true? What's the evidence?
- Across all the feature retros, what pattern emerges about whether the thesis was right?
- If the thesis was wrong, when did you first know - and did you act on it?

### 3. Evaluate the stop condition
- Did the stop condition ever get close to triggering?
- In hindsight, was it the right stop condition? What would a better one have looked like?

### 4. Evaluate the minimum viable initiative
- Did you ship the MVI, or more?
- If more: was the extra scope justified by what you learned, or did it happen by default?
- If the MVI alone had shipped, would you have learned what you needed to know?

### 5. Cross-cutting assumption audit
Load the cross-cutting assumptions from the initiative brief. For each:
- Was it true?
- If false: which features were affected, and what would you have done differently?

### 6. Extract strategic learnings
Identify 3-5 learnings at the initiative level - not feature-level observations, but things that change how you'd frame the next initiative. Each must be concrete enough to act on.

### 7. Update project context
Add strategic learnings to `context/product-context.md` under a "Strategic Learnings" section. Do not rewrite the whole file - append only.

### 8. Write the initiative retro
Save to `outputs/initiatives/{initiative-name}/initiative-retro.md`.

## Output: outputs/initiatives/{initiative-name}/initiative-retro.md

```markdown
---
initiative: {initiative-name}
status: closed
retro-date: YYYY-MM-DD
---

# Initiative Retro: {Initiative Name}

## Thesis vs. reality
**Thesis:** [The strategic thesis from the initiative brief]
**Predicted:** [The initiative-level prediction]
**Observed:** [What actually happened - evidence across all feature retros]
**Verdict:** [Did the thesis prove out? Partially? Not at all?]

## Stop condition
**Defined as:** [The stop condition from the initiative brief]
**Did it trigger?** [Yes / No / Close]
**In hindsight:** [Was it the right stop condition?]

## Minimum viable initiative
**Defined as:** [The MVI from the initiative brief]
**What shipped:** [What actually shipped]
**Was the extra scope justified?** [Yes / No / Partially - and why]

## Cross-cutting assumptions
| Assumption | True? | Impact |
|---|---|---|
| {assumption} | Yes / No | {which features were affected, if false} |

## Strategic learnings
_Concrete enough to change how you frame the next initiative._
- {learning}

## What this initiative enables next
[Follow-on opportunities, unresolved questions, or new strategic bets surfaced by this initiative]
```

## After close-initiative

"Initiative retro saved to `outputs/initiatives/{initiative-name}/initiative-retro.md`.
Strategic learnings added to `context/product-context.md`.

Initiative closed."
