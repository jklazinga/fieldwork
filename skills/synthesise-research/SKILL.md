---
name: synthesise-research
description: "Triggers when the user has raw research to make sense of - interview transcripts, survey responses, support tickets, session recordings, NPS comments, or any unstructured user signal. Extracts patterns, surfaces assumptions, and produces a synthesis that feeds directly into discover or an existing opportunity document. Does NOT trigger if the user is describing an idea from their own head - that's discover."
---

# Synthesise Research

## Overview

Turn raw user signal into structured insight. The output is not a summary - it is a set of patterns, assumptions, and open questions that sharpen the problem framing. Every finding feeds into either a new opportunity or an existing one.

**Announce at start:** "I'm using the Fieldwork synthesise-research skill to make sense of this research."

## When to use this

Use when you have:
- Interview transcripts (recorded or written notes)
- Survey responses or NPS comments
- Support tickets or customer complaints
- Session recordings or usability test notes
- Sales call notes or win/loss interviews
- Any unstructured signal from real users

Do NOT use when:
- The user is describing their own intuition or hypothesis - that's `discover`
- The research is a competitor analysis - that's `analyse-competitors`
- The research is quantitative analytics data - ask the user to interpret the numbers first, then synthesise

## Pre-flight check

1. Ask: "What do you have? Paste it in, share a file, or describe what you've collected."
2. Ask: "Is this connected to an existing opportunity, or are we starting fresh?"
   - If connected: load `outputs/opportunities/{feature-name}/opportunity.md` and `assumptions.md`
   - If fresh: this synthesis may trigger a new `discover` run after it completes
3. Ask: "How many sources are we working with?" (one interview vs. twenty survey responses changes the approach)

## Checklist

### 1. Read everything first
Read all provided material before drawing any conclusions. Do not pattern-match on the first quote you see.

### 2. Extract raw observations
Pull out direct quotes and specific behaviours. Not interpretations - observations. Format:

> "[Direct quote or behaviour description]" - [Source type, e.g. Interview 3, NPS comment, support ticket]

Aim for 10-30 observations depending on volume. More is not better - pick the ones that are specific, surprising, or repeated.

### 3. Cluster into patterns
Group observations into 3-7 patterns. A pattern must appear in at least two independent sources to count. Name each pattern in plain language - not "users struggle with X" but "users work around X by doing Y."

For each pattern:
- How many sources show it?
- Is it consistent or contradictory across sources?
- Is it a problem, a behaviour, a workaround, or a desire?

### 4. Surface assumptions
For each pattern, ask: "What would have to be true for this to matter?" These become assumptions. Map them against the existing assumption map if one exists, or create a new one.

Flag any pattern that contradicts an existing assumption - these are the most valuable findings.

### 5. Identify the strongest signal
Ask: "If you could only take one thing from this research, what changes your thinking the most?"

This is the synthesis headline. It should be a single sentence that a PM could act on.

### 6. Identify open questions
What did the research not answer? What would you need to know next? These feed into the next round of discovery or a spike.

### 7. Write the synthesis
Save to `outputs/research/{research-name}/synthesis.md`.

If connected to an existing opportunity:
- Update `outputs/opportunities/{feature-name}/assumptions.md` with new evidence
- Note in `opportunity.md` that new research has been synthesised (append, do not rewrite)

If not connected to an existing opportunity:
- Ask: "This research points to [pattern]. Want to run `discover` to explore it as an opportunity?"

## Output: outputs/research/{research-name}/synthesis.md

```markdown
---
research: {research-name}
date: YYYY-MM-DD
sources: [list of source types and counts, e.g. "5 interviews, 23 NPS comments"]
linked-opportunity: outputs/opportunities/{feature-name}/opportunity.md (if exists)
---

# Research Synthesis: {Research Name}

## Headline finding
[One sentence - the single most important thing this research tells us]

## Patterns

### [Pattern name]
**What we observed:** [2-3 specific observations with source attribution]
**How many sources:** [N of M]
**Type:** [Problem / Behaviour / Workaround / Desire]
**What it implies:** [One sentence - what this means for the product]

_(repeat for each pattern)_

## Assumption updates
_Changes to existing assumptions based on this research_

| Assumption | Previous confidence | New confidence | Evidence |
|---|---|---|---|
| [Assumption text] | [High/Med/Low] | [High/Med/Low] | [Quote or observation] |

## Contradictions
[Any finding that contradicts an existing assumption or prior belief - flag these explicitly]

## Open questions
[What this research didn't answer - what to explore next]

## Recommended next step
[Discover a new opportunity / Update existing opportunity / Run a spike on X / Talk to more users about Y]
```

## After synthesise-research

"Synthesis saved to `outputs/research/{research-name}/synthesis.md`.

[If connected to opportunity]: Assumption map updated. The strongest new signal is: [headline finding].

[If not connected]: This research points to [pattern]. Want to run `discover` to explore it as an opportunity, or does it update something you're already working on?"
