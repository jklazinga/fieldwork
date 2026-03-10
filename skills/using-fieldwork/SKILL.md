---
name: using-fieldwork
description: Use when starting any conversation - establishes how to find and use Fieldwork skills, requiring skill invocation before ANY feature work including clarifying questions
---

<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a Fieldwork skill applies to what you are doing, you ABSOLUTELY MUST invoke the skill.

FOR ANY NEW FEATURE, IDEA, OR INITIATIVE: YOU START WITH `fieldwork:discover`. YOU DO NOT HAVE A CHOICE.

This is not negotiable. This is not optional. You cannot rationalize your way out of it.

The only valid exceptions are:
1. The user explicitly says "skip discover" - confirm it, note it, proceed
2. The task is a quick question with no feature intent
3. The bet is small and testable - route to `fieldwork:spike` instead (not brainstorming, not coding)

If you are about to write code, write a spec, or write a plan WITHOUT having run `fieldwork:discover` first - stop. You are doing it wrong.
</EXTREMELY-IMPORTANT>

## Red flags - do not rationalise these

| Thought | Why it's wrong |
|---|---|
| "I'll just brainstorm first" | Brainstorming is not discovery. Run `fieldwork:discover`. |
| "This is a small feature, discover is overkill" | Small features still need a clear problem statement. Run `fieldwork:discover`. |
| "The user said write a spec, so I'll skip discover" | Instructions say WHAT, not HOW. Check upstream status first. |
| "Let me check the codebase first" | Codebase exploration is not discovery. Run `fieldwork:discover`. |
| "The user seems to know what they want" | That's an assumption. Run `fieldwork:discover`. |
| "I'll just ask a few clarifying questions first" | Those questions ARE discover. Run `fieldwork:discover`. |
| "We already talked about this idea" | Talking is not an approved opportunity document. Run `fieldwork:discover`. |
| "The user said skip it" | This is the one valid exception. Confirm explicitly, note it, proceed. |

## The entry point

Before routing to any skill, ask one question: **how big is this bet?**

The answer determines the path. Not every idea needs a spec. Not every spec needs a GTM plan. Match the process to the decision being made.

## Three paths

### Spike
**When:** The bet is small or reversible, OR a key assumption is testable by building, OR the user says "prototype", "spike", "quick experiment", "let's just try it."
**Path:** `spike` → (if shown to a user or stakeholder) `measure` → (continue) `write-spec` | (pivot) `discover` | (stop) close
**What it produces:** A test, not a document. One question answered. Use `measure` when a human has reacted to the prototype — not for purely technical spikes.

### Feature
**When:** The opportunity is understood well enough to commit engineering time. Medium-sized bet. Clear problem, plausible solution, team ready to build.
**Path:** `discover` → (optional) `spike` → (if prototype shown to a human) `measure` → `write-spec` → `review-spec` → `write-plan` → `scaffold-tasks` → `write-gtm` → `write-marketing` → `write-launch-brief` → `close-feature`
**Required gates:** discover approval, spec approval, review-spec sign-off. Other steps can be skipped with explicit PM decision.
**Optional pre-discover steps:** `analyse-competitors` (to inform framing before or during discovery), `synthesise-research` (if raw user signal exists that hasn't been processed).
**GTM sequence:** `write-gtm` runs first (strategy), then `write-marketing` (brief), then `write-launch-brief` (execution). All three are optional but must run in that order if used.

### Initiative
**When:** Large, cross-cutting, or high-stakes bet. Multiple teams, significant investment, or strategic importance.
**Path:** `initiative` → (per feature) `discover` → (optional) `spike` → (if prototype shown to a human) `measure` → `write-spec` → `review-spec` → `write-plan` → `scaffold-tasks` → `write-gtm` → `write-marketing` → `write-launch-brief` → `close-feature` → (at end) `close-initiative`
**Required gates:** All of them. The `initiative` skill runs first - it establishes the strategic thesis, stakeholder alignment, and stop condition before any feature discovery begins. If a PM wants to skip a gate, ask why - don't just comply.
**Optional pre-discover steps:** `analyse-competitors` (to inform framing before or during discovery), `synthesise-research` (if raw user signal exists that hasn't been processed). Same as Feature path.

**Note on initiative path:** `initiative` is not a bigger version of `discover`. It runs before discovery to set the frame - thesis, stop condition, who has veto power. Discovery then runs per feature inside that frame.

## How to route

When a user brings a new idea or task:

1. Read what they've said. Do you already know the tier? Route directly.
2. If unclear, ask: "Is this a quick experiment to test an idea, or are you ready to commit to building it?"
3. If they say "just a quick question" - that's fine. Answer it. Not everything is a task.

Use judgment. The goal is to match the process to the decision, not to invoke a skill for its own sake.

## Skill map

| Situation | Skill |
|---|---|
| Small bet, uncertain, testable by building | `fieldwork:spike` |
| Large, cross-cutting, or high-stakes bet | `fieldwork:initiative` |
| Raw user research to make sense of | `fieldwork:synthesise-research` |
| Understanding the competitive landscape | `fieldwork:analyse-competitors` |
| New idea, feature request, or problem to explore | `fieldwork:discover` |
| Opportunity approved, ready to spec | `fieldwork:write-spec` |
| Spec written, want structured review | `fieldwork:review-spec` |
| Spec approved, ready to plan implementation | `fieldwork:write-plan` |
| Plan approved, need tracked tasks | `fieldwork:scaffold-tasks` |
| Spec approved, need GTM plan | `fieldwork:write-gtm` |
| GTM plan approved, need marketing brief | `fieldwork:write-marketing` |
| GTM plan approved, need launch brief | `fieldwork:write-launch-brief` |
| Prototype shown to a user or stakeholder, need to capture what was learned and decide next step | `fieldwork:measure` |
| Feature shipped, need retro | `fieldwork:close-feature` |
| All initiative features shipped, need initiative retro | `fieldwork:close-initiative` |

**Note on measure verdicts:** `measure` produces one of three verdicts — continue, pivot, or stop. If stop, run `close-feature` next to formally close the opportunity.
| First time setup, context files missing | `fieldwork:onboard` |
| Creating or extending a Fieldwork skill | `fieldwork:writing-skills` |

## Skill types

**Rigid** (discover, write-spec, review-spec, write-plan): Follow exactly. The structure exists for a reason.

**Flexible** (spike, write-gtm, write-marketing, write-launch-brief, scaffold-tasks): Adapt to the project context.

The skill itself tells you which type it is.

## Context files

Load these before any skill runs:
- `context/project-context.md` - codebase, tech stack, architecture
- `context/product-context.md` - OKRs, personas, positioning, roadmap
- `context/constraints.md` - stakeholders, tech debt, non-negotiables

If any context file is missing or stale, run `fieldwork:onboard` before proceeding.

Spike is the exception: it can run with minimal context. Ask for what's needed, don't require full onboard.

## User instructions

Instructions say WHAT, not HOW. "Write a spec for X" doesn't mean skip `discover`. "Plan the GTM" doesn't mean skip `write-spec`. Check upstream status before proceeding downstream.

But also: if a user explicitly says "skip discover, I know the opportunity" - that's a valid PM decision. Confirm it, note it, proceed.
