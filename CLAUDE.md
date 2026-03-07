# Fieldwork

Fieldwork skills are in `skills/`. The `using-fieldwork` skill is loaded at session start via the hooks system - it defines which skills exist, when to invoke them, and how to chain them. Read it before doing anything else.

Before writing any spec, plan, or code - a Fieldwork skill applies. YOU MUST invoke it. Do not skip. Do not proceed without it.

## Skill index

| Skill                | When to trigger                                                          |
| -------------------- | ------------------------------------------------------------------------ |
| `onboard`              | First session in a new project, or when project context is missing/stale |
| `spike`                | Small/reversible bet, or key assumption testable by building             |
| `synthesise-research`  | User has raw research to make sense of - interviews, surveys, support tickets, NPS |
| `analyse-competitors`  | User wants to understand the competitive landscape or what alternatives exist |
| `discover`             | User describes a new idea, feature request, or opportunity               |
| `write-spec`           | An opportunity has been validated and approved                           |
| `review-spec`          | A spec has been written and needs critique                               |
| `write-plan`           | A spec is approved and implementation planning is needed                 |
| `scaffold-tasks`       | A plan is approved and tracked tasks need to be created                  |
| `write-gtm`            | A spec is approved and go-to-market planning is needed                   |
| `write-marketing`      | A GTM plan exists and marketing assets are needed                        |
| `write-launch-brief`   | A feature is approaching launch                                          |
| `close-feature`        | A feature has shipped                                                    |
| `writing-skills`       | User wants to create a new Fieldwork skill or extend an existing one     |

## Hard rules

- Match the process to the bet. A spike does not need a spec. A feature does. An initiative needs everything.
- Never write code or scaffold implementation before a spec exists and is approved (except during a spike - spikes are explicitly throwaway).
- Never write a GTM plan before a spec exists.
- Always read all three context files before invoking any skill (spike is the exception - it can run with minimal context).
- If any context file does not exist or contains unfilled template text, run `onboard` first.
- Output files go in `outputs/` - never in the project source tree.

## Skill priority

Fieldwork skills run **before** Superpowers skills for all PM workflow steps:
- Discovery → spec → review → GTM → marketing → launch → retro

Superpowers skills run **during** implementation (executing-plans, TDD, git workflow).

If both Fieldwork and Superpowers are installed, the handoff point is the approved spec → `superpowers:executing-plans`.

## MCP usage

Use available MCPs to enrich outputs:
- GitHub MCP → scaffold tasks as Issues
- Linear MCP → scaffold tasks as Linear tickets
- Slack MCP → draft launch announcements
- Confluence MCP → publish specs and GTM plans

If an MCP is not available, write output to file and note where it should be published.

## Context files

Always load these before any skill runs:
- `context/project-context.md` - codebase, tech stack, architecture
- `context/product-context.md` - OKRs, personas, positioning, roadmap
- `context/constraints.md` - stakeholders, tech debt, non-negotiables

If any context file is missing or marked stale, run `onboard` before proceeding.

## Output directory

All skill outputs go to `outputs/`:
```
outputs/
  spikes/{spike-name}/spike-plan.md
  research/{research-name}/synthesis.md
  competitive/{analysis-name}/competitive-analysis.md
  opportunities/{feature-name}/opportunity.md
  opportunities/{feature-name}/assumptions.md
  specs/{feature-name}/spec.md
  specs/{feature-name}/spec-review.md
  plans/{feature-name}/plan.md
  plans/{feature-name}/tasks.md (local fallback if no MCP)
  gtm/{feature-name}/gtm-plan.md
  gtm/{feature-name}/marketing-brief.md
  gtm/{feature-name}/launch-brief.md
  retros/{feature-name}/retro.md
```

Create the feature directory before writing any output file.

## Status values

All output documents use a `status` frontmatter field. Valid values:
- `draft` - in progress, not approved
- `approved` - PM has explicitly approved this document
- `shipped` - feature has launched

Skills that depend on upstream documents **must** check the status field and halt if it is not `approved`.

## Fieldwork discover compatibility

If `outputs/opportunities/` already contains files from Fieldwork discover, load them directly. Do not re-run `discover` - ask the PM to confirm the existing opportunity before proceeding to `write-spec`.

If `outputs/opportunities/{feature-name}/assumptions.md` exists (Fieldwork discover output), load it into `write-spec` as the riskiest assumptions section.

## Superpowers compatibility

The approved plan (not spec) is the handoff point to `superpowers:executing-plans`. After plan approval, check if Superpowers is installed. If yes, offer to invoke `superpowers:executing-plans` directly. If not, run `scaffold-tasks`.
