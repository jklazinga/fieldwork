---
name: scaffold-tasks
description: "Triggers after a plan is approved (status: approved) and the user wants to create tracked tasks in GitHub, Linear, or a local task file. Does NOT trigger if plan status is draft or missing - run write-plan first. Also triggers if the user says 'create issues', 'scaffold tasks', or 'set up the board'."
---

# Scaffold Tasks

## Overview

Turn the approved implementation plan into tracked tasks - GitHub issues, Linear tickets, or a local `tasks.md` file. MCP-aware: checks what's available before attempting any calls. Falls back gracefully.

**Announce at start:** "I'm using the Fieldwork scaffold-tasks skill to create tracked tasks."

<HARD-GATE>
Do NOT scaffold tasks if:
- plan.md status is not `approved` - check frontmatter, do not rely on the user's word

Do NOT attempt MCP calls without first checking availability in context/project-context.md.
Tell the user upfront which system will be used before creating anything.
</HARD-GATE>

## Pre-flight check

1. Load `outputs/plans/{feature-name}/plan.md` - confirm `status: approved`. If not, stop.
2. Load `context/project-context.md` - check "MCP integrations available" section.
3. Determine target system (see routing below).
4. Tell the user: "I'll create tasks in [system]. [N] tasks across [N] phases. Proceed?"

## MCP routing

Check `context/project-context.md` → "MCP integrations available":

| Available | Action |
|---|---|
| GitHub MCP | Create issues with labels, milestone, and task body |
| Linear MCP | Create issues with project, priority, and description |
| Both | Ask user which to use |
| Neither | Fall back to local `tasks.md` - tell the user explicitly |

Do not attempt a GitHub or Linear call if the MCP is not confirmed available. Do not silently fall back - always tell the user what you're doing and why.

## Task creation rules

For each task in the plan:
- **Title:** `[Phase name] - [Task name]`
- **Body:** Include "What", "Done when", "Files", "Depends on" from the plan
- **Labels (GitHub):** `fieldwork`, phase name (e.g. `phase:data-layer`), feature name
- **Priority (Linear):** High for Phase 1 tasks, Medium for later phases
- **Milestone / Project:** Feature name

Create tasks in dependency order. If task B depends on task A, create A first and reference it in B's body.

## Local fallback: tasks.md

If no MCP is available, write `outputs/plans/{feature-name}/tasks.md`:

```markdown
---
feature: {feature-name}
date: YYYY-MM-DD
source: outputs/plans/{feature-name}/plan.md
---

# Tasks: {Feature Name}

## Phase 1: [Name]

- [ ] **[Task name]**
  Files: `exact/path/to/file`
  What: [Specific change]
  Done when: [Testable condition]
  Depends on: [Task name or "nothing"]

## Phase 2: [Name]

- [ ] ...
```

## After scaffold-tasks

If GitHub/Linear:
"[N] tasks created in [system].
Links:
- [Task 1 title] - [URL]
- [Task 2 title] - [URL]
...

Ready to start building. Run `close-feature` after the feature ships."

If local fallback:
"Tasks written to `outputs/plans/{feature-name}/tasks.md`.
No GitHub or Linear MCP was available - tasks are local only.
To connect to GitHub or Linear, update `context/project-context.md` → MCP integrations and re-run this skill."
