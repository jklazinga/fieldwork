---
name: discover
description: "Triggers when the user describes a new idea, feature request, problem, or opportunity and no approved opportunity document exists for it yet. Explores it through structured questions before any spec or plan is written. Does NOT trigger if an opportunity document already exists for this feature with status: approved - in that case, ask the user if they want to proceed to write-spec."
---

# Discover

## Overview

Turn a rough idea into a validated opportunity through structured dialogue. Ask questions one at a time. Explore the problem before the solution. Surface and map assumptions. End with a written opportunity document the PM approves before any spec work begins.

**Announce at start:** "I'm using the Fieldwork discover skill to explore this opportunity."

<HARD-GATE>
Do NOT invoke write-spec or any downstream skill until:
1. The opportunity document has been written AND
2. The user has explicitly approved it (status: approved in frontmatter)

If context/project-context.md is missing, run onboard first. Do not proceed without it.
</HARD-GATE>

## Supporting files

Read these on demand using the Read tool - do not read all at once. Use the path relative to the project root (where CLAUDE.md lives):
- `skills/discover/question-bank.md` - full question library (read when exploring the problem)
- `skills/discover/opportunity-framing.md` - framing patterns and examples (read when proposing framings)
- `skills/discover/assumption-mapping.md` - 2x2 map, dimension definitions, validation ideas (read when surfacing assumptions)

If the Fieldwork plugin is installed via Claude Code plugin system, these files are at `{plugin_root}/skills/discover/{filename}`. Use the Read tool with the absolute path if relative paths fail.

## Pre-flight check

0. **Bet-sizing check:** Before starting, confirm this is a Feature or Initiative tier bet (not a Spike). If the user is uncertain whether the opportunity is worth committing to, or if the key assumption is testable by building, suggest `spike` first. Ask: "Is this something you're ready to commit engineering time to, or do you want to test the idea first?"
1. Does `context/project-context.md` exist? If no - stop, run `onboard` first.
2. Does `context/product-context.md` exist? If no - stop, run `onboard` first.
3. Does `outputs/opportunities/{feature-name}/opportunity.md` already exist?
   - If yes and status is `approved` - tell the user: "An approved opportunity already exists. Want to proceed to `write-spec`, or revisit the opportunity?"
   - If yes and status is `draft` - load it and continue from where it left off.
   - If no - start fresh.
4. Does `outputs/opportunities/{feature-name}/assumptions.md` exist (Fieldwork discover output)?
   - If yes - load it. Check whether the opportunity document has been revised since the assumptions file was last updated (compare dates in frontmatter). If the opportunity was revised after the assumptions were written, tell the user: "Your assumptions file predates the latest opportunity revision - some assumptions may be stale. Want to re-run assumption mapping or proceed with the existing file?" Do not skip this check. If dates match or no revision has occurred, use the file directly and skip the assumption questions below.

## Checklist

### 1. Load context
- [ ] Load `context/project-context.md`
- [ ] Load `context/product-context.md`
- [ ] Load `context/constraints.md` if it exists

### 2. Understand the problem - one question at a time
Read `skills/discover/question-bank.md` using the Read tool. Pick the most relevant questions for this idea. Do not ask all of them. Start with:
- [ ] What problem does this solve? For whom?
- [ ] How do users currently deal with this problem?
- [ ] How often does this occur? How painful is it?
- [ ] What does success look like - what would be measurably different?
- [ ] What's the smallest version that would be valuable?

### 3. Propose framings
Read `skills/discover/opportunity-framing.md` using the Read tool.
- [ ] Propose 2-3 framings with trade-offs
- [ ] Ask the PM to pick a framing or propose their own

### 4. Surface and map assumptions
Read `skills/discover/assumption-mapping.md` using the Read tool.
- [ ] Ask about assumptions across all four dimensions (one at a time): Desirability, Viability, Feasibility, Usability
- [ ] For each assumption: rate importance (high/medium/low) and evidence (strong/weak/none)
- [ ] Produce the 2x2 map
- [ ] Identify validation ideas for high-importance / weak-evidence assumptions

### 5. Write opportunity document
- [ ] Create directory `outputs/opportunities/{feature-name}/` if it does not exist
- [ ] Write opportunity document with `status: draft`

### 6. Write assumptions document
- [ ] Save assumption map to `outputs/opportunities/{feature-name}/assumptions.md`

### 7. Get approval
- [ ] Present summary: problem, proposed opportunity, top 3 riskiest assumptions, success metrics
- [ ] Ask: "Does this capture the opportunity correctly? Any changes before I mark it approved?"
- [ ] **Decision gate:** Ask: "Is this opportunity worth an engineering investment? What would have to be true for it not to be?" Wait for a real answer - do not accept "yes" without the PM articulating the condition under which they'd stop.
- [ ] **Spike check:** Read `outputs/opportunities/{feature-name}/assumptions.md`. Identify any assumptions rated high-importance AND weak-evidence or none. If any exist, name them explicitly: "Before I mark this approved — these assumptions are high-importance / weak-evidence: [list each one]. Do any of these need a spike to validate before we commit to a spec?" Wait for a real answer. If yes, stop and trigger `spike`. If no, record the PM's reasoning in the opportunity document under "Assumptions accepted without validation".
- [ ] **Write a prediction:** Ask: "If this works, what will we observe in 30/60/90 days? Be specific." Record the answer in the opportunity document under "Prediction". This is what the retro will compare against.
- [ ] On approval: update frontmatter status to `approved`

## Output: outputs/opportunities/{feature-name}/opportunity.md

```markdown
---
feature: {feature-name}
status: draft | approved
date: YYYY-MM-DD
---

# Opportunity: {Feature Name}

## Problem
[What problem exists, for whom, how often, how painful]

## Current behaviour
[How users deal with this today - workarounds, competing tools, manual steps]

## Proposed opportunity
[What we could build - framed as an opportunity, not a solution]

## Success metrics
[How we'll know this worked - specific and measurable]

## Riskiest assumptions
[Top 3 from the assumption map - the ones that could kill this if wrong]

## Scope boundary
[What this is NOT - explicit non-goals]

## Open questions
[What we don't know yet]

## Prediction
_Written at approval. Compared against at retro._
[If this works, what will we observe at 30 / 60 / 90 days? Specific and measurable.]
```

## Output: outputs/opportunities/{feature-name}/assumptions.md

See `skills/discover/assumption-mapping.md` for the full output format.

## After discover

"Opportunity saved to `outputs/opportunities/{feature-name}/opportunity.md`.
Assumption map saved to `outputs/opportunities/{feature-name}/assumptions.md`.

Three paths from here:
1. **Spike first** - if any high-importance / weak-evidence assumptions exist, run `spike` to test before committing to a spec. After the spike is shown to a user or stakeholder, run `measure`.
2. **Write the spec** - opportunity is solid, ready to commit.
3. **Research first** - if raw user signal exists that hasn't been synthesised, run `synthesise-research` and feed findings back into this opportunity.

Which would you like?"

**If the PM chooses path 2**, ask before starting:

"How do you want to run this?
1. **Step by step** — I'll run each skill and check in with you between stages
2. **Full run** — I'll chain `write-spec → review-spec → write-plan → scaffold-tasks` autonomously and only stop if I hit a blocker I can't resolve on my own"

**If step by step (2a):** Proceed with `write-spec` as normal. Each skill hands off to the next only when the PM explicitly proceeds.

**If full run (2b):**
- Run `write-spec` — do not stop for PM approval between sections; run the hill-climbing pass; present spec summary and proceed without waiting for PM sign-off unless the PM has set `working-style: 1` (section by section) earlier in this session.
- Run `review-spec` — dispatch reviewer subagent; resolve any IMPORTANT or MINOR issues autonomously where the resolution is unambiguous; **stop and wait for PM input only if a BLOCKER requires a decision that cannot be resolved from the spec, opportunity, or constraints files alone**.
- Run `write-plan` — proceed autonomously.
- Run `scaffold-tasks` — proceed autonomously.
- When complete, present a summary: "Full run complete. Here's what was produced: [list outputs]. [N] decisions were made autonomously. [List any BLOCKERs that required PM input, or 'No blockers required your input.']"

**Full run rules:**
- Never skip a required gate silently. If a gate cannot be cleared without PM input, surface it clearly and wait.
- Never invent resolutions to BLOCKERs. If a BLOCKER requires a judgment call the agent cannot make from available context, stop.
- Proceed past each skill only when its output meets the quality bar the skill defines — not just because the skill ran.
