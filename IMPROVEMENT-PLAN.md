# Fieldwork Improvement Plan
_Borrowing execution mechanics from Superpowers_

---

## Change 1: Explicit sub-skill chaining

**What:** Skills that hand off to other skills declare the handoff as a directive, not a suggestion.

**Why:** Currently handoffs are buried in "After X" sections. Superpowers uses `REQUIRED SUB-SKILL: Use superpowers:X` as a directive. Making these explicit reduces the chance an agent skips the handoff or substitutes its own approach.

**Three handoffs to fix:**

### `write-plan` → Superpowers
Replace the current compatibility footnote with a proper integration section:

```markdown
## Integration

**If Superpowers is installed:**
- REQUIRED SUB-SKILL: Use `superpowers:executing-plans` to execute this plan in a separate session, OR
- REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` to execute in the current session

Do not execute the plan manually if Superpowers is available — the subagent review loops are the quality gate.
```

### `discover` → `spike`
The spike check currently says "stop and trigger spike". Replace with:

```
REQUIRED SUB-SKILL: Use `fieldwork:spike` — do not proceed to write-spec until spike is complete and the PM has reviewed the outcome.
```

### `close-feature` → `discover`
Drop the weak "want me to run discover?" suggestion. Replace with a structured step after the product-context writeback:

```
Extract any candidate opportunities surfaced by the retro as named items — one line each. Present them to the PM: "This retro surfaced the following potential opportunities: [list]. Which if any do you want to pursue?" If one is selected, trigger `fieldwork:discover` immediately.
```

**Effort:** Low. Targeted edits to 3 skills, one commit.

---

## Change 2: Reviewer subagent for `review-spec`

**What:** `review-spec` dispatches a fresh subagent to run the critique instead of the main agent self-reviewing.

**Why:** The main agent critiques a spec it may have just written — same context, same attachments. A fresh subagent has no attachment to the original work. This is the closest Fieldwork gets to Superpowers' quality gate pattern, and it's in the highest-stakes skill.

**How it works:**

1. Main agent loads all required files (spec, opportunity, assumptions, constraints).
2. Main agent dispatches a reviewer subagent via the `Task()` tool. Constructs the prompt by reading `skills/review-spec/reviewer-prompt.md` and appending all file contents inline under clearly labelled headings. The subagent does not read files — all content is inline.
3. Subagent returns findings grouped by severity: BLOCKER, IMPORTANT, MINOR.
4. Main agent presents findings to PM and runs the triage loop as normal.

**New file: `skills/review-spec/reviewer-prompt.md`**

Contains:
- Role framing: "You are a spec reviewer. Find problems before engineering starts. Be direct. Do not soften findings. Do not suggest fixes — identify problems and state what's needed to resolve them."
- Input section: describes the four inline documents the main agent will append.
- Full critique dimensions (moved verbatim from `review-spec` SKILL.md).
- Output format: findings grouped as BLOCKER / IMPORTANT / MINOR, plus a one-paragraph summary.

**Changes to `review-spec` SKILL.md:**

Replace the "Critique dimensions" section with a dispatch instruction:

```markdown
## Dispatch reviewer subagent

After pre-flight, do NOT run the critique yourself. Instead:

1. Read `skills/review-spec/reviewer-prompt.md`.
2. Construct the subagent prompt: paste reviewer-prompt.md in full, then append each loaded document inline under a clearly labelled heading (e.g. `## SPEC`, `## OPPORTUNITY`, `## ASSUMPTIONS`, `## CONSTRAINTS`).
3. Dispatch via the Task() tool. Wait for findings.
4. Present findings to the PM and proceed with the triage loop below.

The critique dimensions are in reviewer-prompt.md — do not run them yourself.
```

**Open question before implementing:** Confirm `Task()` tool is available in the Claude Code plugin context and that subagent output is returned to the main agent synchronously. If not, fallback is a structured self-review with explicit "pretend you did not write this spec" framing — weaker but implementable without Task().

**Effort:** Medium. New file + targeted edit to `review-spec`. Separate commit from Change 1.

---

## Sequencing

| Order | Change | Effort | Value |
|---|---|---|---|
| 1 | Explicit sub-skill chaining | Low | Medium |
| 2 | Reviewer subagent for `review-spec` | Medium | High |

**Note:** Announce-at-start was originally listed as Change 1. Dropped — all 16 skills already have announce lines. Nothing to do.

---

## What this doesn't fix

- Install/first-run friction. Still high.
- `initiative` skill being underspecified.
- The fundamental constraint: skills are instructions, not enforced logic. The reviewer subagent (Change 2) is the closest we get to enforcement — everything else remains advisory.
