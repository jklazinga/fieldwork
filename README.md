![Fieldwork](assets/fieldwork-banner.png)

# Fieldwork

AI coding agents are good at building. They are not good at deciding what to build, writing specs that hold up, or knowing when a feature is actually done.

Fieldwork fixes that. It's a PM workflow for AI coding agents that covers the full feature lifecycle: discovery, spec, engineering handoff, GTM, launch, and retro. Skills trigger automatically based on where you are in the process, so you don't have to manage the workflow yourself.

Most PM tools for AI stop at strategy or stop at code. Fieldwork connects them. The discovery output feeds the spec. The spec feeds the implementation plan. The plan feeds your task tracker. The retro closes the loop against the prediction you made at discovery.

Every skill reads your actual codebase and product context. The outputs are grounded in what you know about your users, your OKRs, and your constraints, not generic templates.

---

## Installation

### Claude Code (recommended)

Run these inside any Claude Code session:

```
/plugin marketplace add jklazinga/fieldwork-marketplace
/plugin install fieldwork@fieldwork-marketplace
```

Fieldwork is active in that project.

### Cursor

Tell Cursor Agent:

```
Fetch and follow the instructions at https://raw.githubusercontent.com/jklazinga/fieldwork/refs/heads/main/.cursor/INSTALL.md
```

### Codex

Tell Codex:

```
Fetch and follow instructions from https://raw.githubusercontent.com/jklazinga/fieldwork/refs/heads/main/.codex/INSTALL.md
```

### OpenCode

Tell OpenCode:

```
Fetch and follow instructions from https://raw.githubusercontent.com/jklazinga/fieldwork/refs/heads/main/.opencode/INSTALL.md
```

---

## First run

After install, describe your project or idea to the agent. If context files are missing, the `onboard` skill runs automatically. It will:

1. Read your codebase
2. Ask a short set of questions about your product, users, and constraints
3. Write three context files that every other skill reads from

Once onboarding is done, describe a feature or opportunity. The agent takes it from there.

---

## Three paths

Not every idea needs a spec. Match the process to the bet.

### Spike
Small or reversible bet. Key assumption testable by building. Use when the fastest way to learn is to make something, not document something.

`spike` → debrief → (if validated) `discover` or `write-spec`

### Feature
Opportunity understood well enough to commit engineering time. Medium-sized bet.

`discover` → `write-spec` → `review-spec` → `write-plan` → `scaffold-tasks` → `write-gtm` → `write-launch-brief` → `close-feature`

### Initiative
Large, cross-cutting, or high-stakes. All gates required.

Same as Feature - no steps skipped.

---

## The Full Workflow

1. **onboard** - Activates when context files are missing or stale. Reads your codebase, asks about your product and users, writes the three context files everything else depends on.

2. **spike** - Activates for small bets or when a key assumption is testable by building. Produces a test plan, not a spec. Time-boxed. Feeds back into discover if validated.

3. **discover** - Activates when you describe a new idea or opportunity. Asks clarifying questions, frames the problem, surfaces assumptions. Ends with a prediction: if this works, what will we observe? Saves an opportunity brief and assumptions log.

4. **write-spec** - Activates after opportunity approval. Writes a full product spec grounded in your context files. Structured for agent handoff - the spec is the execution brief.

5. **review-spec** - Activates after the spec is drafted. Reviews for gaps, contradictions, and missing edge cases. Asks: does this spec give us a real shot at the outcome we predicted in discovery? Flags issues before they become rework.

6. **write-plan** - Activates after spec approval. Produces a sequenced implementation plan grounded in the actual codebase.

7. **scaffold-tasks** - Activates after plan approval. Creates tracked tasks in GitHub Issues, Linear, or a local task file. MCP-aware - checks what's available before attempting any calls.

8. **write-gtm** - Activates after spec approval, in parallel with execution. Writes a go-to-market plan covering positioning, channels, and launch sequencing.

9. **write-marketing** - Activates after GTM approval. Writes a marketing brief for the feature: messaging, audience, and channel-specific copy direction.

10. **write-launch-brief** - Activates as launch approaches. Consolidates everything into a single launch brief for stakeholders and comms.

11. **close-feature** - Activates after the feature ships. Runs a structured retro that compares against the prediction made at discovery. Did we solve the customer problem?

---

## What's Inside

### Discovery
- **onboard** - Project and product context setup
- **spike** - Time-boxed prototype to answer one question before committing
- **discover** - Opportunity framing and assumption mapping

### Spec
- **write-spec** - Full product spec from opportunity brief
- **review-spec** - Spec review for gaps and edge cases

### Engineering handoff
- **write-plan** - Implementation plan
- **scaffold-tasks** - GitHub Issues, Linear tickets, or task file

### Go-to-market
- **write-gtm** - GTM plan: positioning, channels, sequencing
- **write-marketing** - Marketing brief: messaging and copy direction
- **write-launch-brief** - Launch brief for stakeholders and comms

### Retrospective
- **close-feature** - Structured post-ship retro

### Meta
- **writing-skills** - Create custom skills for your workflow

---

## Context files

Three files that all skills read. Generated by `onboard`, updated by you.

- **`context/project-context.md`** - Codebase, tech stack, architecture, MCP availability
- **`context/product-context.md`** - OKRs, personas, positioning, roadmap, channels
- **`context/constraints.md`** - Team, deadlines, tech debt, non-negotiables

These are **committed by default**. The grounding mechanism only works if context survives across sessions. If a context file contains secrets, move those to `.env` and reference them by name - don't put secrets in context files.

Templates are in `context/`.

### Custom context

Drop `.md` files into `context/custom/` to inject project-specific instructions into every Fieldwork session. Fieldwork loads them automatically at session start alongside the three core context files.

Use this for anything that should apply consistently across your workflow but is specific to your project or team:

- **Tool integrations:** e.g. "We have Figma MCP available. During discovery and spec, always check Figma for existing designs before asking the PM."
- **Team conventions:** naming standards, review process, who owns what
- **Stage overrides:** e.g. "During GTM, pull copy from `docs/gtm-templates/` and include it in the launch brief."
- **Workflow preferences:** any standing instructions you want applied every session

The `onboard` skill scaffolds `context/custom/` with a README when it runs. The directory is yours. Fieldwork will never overwrite files in it.

---

## Integrations

### Superpowers
If you use [Superpowers](https://github.com/obra/superpowers), the plan output from `write-plan` is compatible with `superpowers:executing-plans`. This is optional. You can work from the plan file directly or scaffold tasks without it.

---

## Example

See `example/` for a complete worked example showing every skill output from opportunity through retro.

---

## Philosophy

- **PM-first** - Outputs are written for product managers and go-to-market teams, not just engineers
- **Context-grounded** - Every skill reads your actual codebase and product context. No generic templates, no generic advice
- **Output-chained** - Each skill's output is the next skill's input. The chain runs from opportunity brief to retro without you having to wire it together
- **No lock-in** - Works in Claude Code, Cursor, or any agent that reads markdown. MCP integrations are optional and detected automatically

---

## Status

| Field | Value |
|---|---|
| Phase | Active development |
| Current focus | Skill implementation and example content |
| Blocked on | Nothing |
| As of | 2026-03-05 |

---

## License

MIT
