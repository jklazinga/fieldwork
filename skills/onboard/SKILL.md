---
name: onboard
description: "Run at the start of any new project session when context/project-context.md is missing, empty, or stale. Also triggers if the user says 'set up context', 'initialise fieldwork', or 'update project context'. Does NOT trigger if all three context files exist and are current."
---

# Onboard

## Overview

Build the three context files that every other Fieldwork skill depends on. Read the codebase for technical context. Ask the PM for product and business context. Do not proceed with any other skill until all three files exist.

**Announce at start:** "I'm running the Fieldwork onboard skill to build project context."

<HARD-GATE>
Do NOT invoke any other Fieldwork skill until all three context files have been written and confirmed:
- context/project-context.md
- context/product-context.md
- context/constraints.md

If any file is missing, complete onboard first.
</HARD-GATE>

## Pre-flight check

Before starting, check:
1. Does `context/project-context.md` exist? If yes, show last-updated date and ask: "Context exists from [date] - update it or use as-is?"
2. Does `context/product-context.md` exist? If yes, same check.
3. Does `context/constraints.md` exist? If yes, same check.
4. Does `fieldwork.config.json` exist? If yes, load it. If no, create it from the template.

## Checklist

Complete in order:

### 1. Read the codebase
- Use Repomix if available, otherwise read: README, package.json / pyproject.toml / go.mod, src/ or app/ structure, existing docs/, CLAUDE.md or .cursorrules if present
- Identify: language, framework, key dependencies, architecture pattern, test setup

### 2. Check for MCP availability
- Load `fieldwork.config.json`. Check the `mcp` section for each integration.
- If an integration is set to `true`, confirm it's reachable with a lightweight call. If the call fails, set it back to `false` and note it.
- If an integration is not configured (missing or `false`), ask the user: "Do you have [GitHub / Linear / Slack / Confluence] MCP available in this environment?"
- Record confirmed results in `context/project-context.md` under "MCP integrations available" and update `fieldwork.config.json` accordingly.

### 3. Ask for product context - one question at a time
- What does this product do? Who uses it?
- What's the current stage? (pre-launch / live / scaling)
- What are the top 2-3 OKRs or goals this quarter?
- Who are the primary user personas? (name, role, key pain)
- How do you currently position this vs. alternatives?
- What channels do you have for reaching users? (email, in-app, social, sales, press)

### 4. Ask for constraints - one question at a time
- Any areas of the codebase that are fragile or need care?
- Any non-negotiables - architecture decisions, compliance, third-party contracts?
- Who has input or approval authority on product decisions?

### 5. Write config
- Load `fieldwork.config.json`. Write the following fields:
  - `project.name` - use the product name from step 3 (ask if unclear)
  - `project.description` - one sentence from step 3
  - `mcp` - update with confirmed results from step 2
- Save the updated config. This is what downstream skills read - do not leave it blank.

### 6. Write context files
Write all three files. Do not skip any.

Tell the user: "Context files are committed to the repo by default - this means every session starts with full context. If any file contains secrets (API keys, credentials), move those to `.env` and reference them by name. Do not put secrets in context files."

### 7. Confirm with user
Present a one-paragraph summary of each file. Ask: "Does this look right? Anything missing?"

## Output: context/project-context.md

```markdown
# Project Context
_Last updated: YYYY-MM-DD_

## Product
[What it does, who uses it, current stage]

## Tech stack
[Languages, frameworks, key dependencies]

## Architecture
[High-level structure - 3-5 sentences. Key patterns, module boundaries, data flow.]

## Test setup
[Test framework, coverage approach, how to run tests]

## Key files
[README, main entry points, config files, existing specs or ADRs]

## MCP integrations available
- GitHub: [yes/no]
- Linear: [yes/no]
- Slack: [yes/no]
- Confluence: [yes/no]
```

## Output: context/product-context.md

```markdown
# Product Context
_Last updated: YYYY-MM-DD_

## Vision
[One sentence]

## OKRs / current quarter goals
- [Objective 1]: [Key result]
- [Objective 2]: [Key result]

## Personas
### [Persona name]
- **Who:** [Role, context]
- **Goal:** [What they're trying to accomplish]
- **Pain:** [What frustrates them today]

## Positioning
[One paragraph - who this is for, what it does, why it's different]

## Roadmap context
[What's shipping this quarter, what's next, what's explicitly not on the roadmap]

## Channels
[Email list, in-app, social, blog, sales team, press contacts - what exists]

## Competitors
[Key alternatives - 2-3 sentences each]
```

## Output: context/constraints.md

```markdown
# Constraints
_Last updated: YYYY-MM-DD_

## Stakeholders
[Who has input or approval authority on product decisions]

## Tech debt
[Known fragile areas]

## Non-negotiables
[Architecture decisions, compliance, contracts]
```

### 8. Scaffold custom context directory

Create `context/custom/` if it does not already exist. Write `context/custom/README.md` with the following content (do not overwrite if it already exists):

```markdown
# Custom Context

Drop any `.md` files in this directory to inject project-specific context into every Fieldwork session.

Fieldwork loads all `.md` files here at session start (this README is excluded).

## Use cases

- **Tool integrations** — e.g. "We use Figma MCP. When running `discover` or `spec`, always check Figma for existing designs before asking the PM."
- **Team conventions** — e.g. naming standards, review process, who owns what.
- **Stage overrides** — e.g. "During GTM, pull copy from `docs/gtm-templates/` and include it in the launch brief."
- **Workflow preferences** — any other instructions you want applied consistently across all sessions.

## Tips

- One file per concern (e.g. `figma.md`, `gtm.md`, `team.md`).
- Keep files focused — broad context is fine, but avoid contradicting core Fieldwork skill behaviour.
- This directory is yours. It is not managed by Fieldwork and will not be overwritten on plugin updates.
```

## After onboard

"Context files written:
- `context/project-context.md`
- `context/product-context.md`
- `context/constraints.md`

Custom context directory created at `context/custom/` — drop `.md` files there to inject project-specific instructions into every session.

You can now describe a feature or opportunity and I'll use the `discover` skill to explore it."
