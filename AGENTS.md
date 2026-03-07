# Fieldwork - Agent Instructions

You have access to the Fieldwork PM skills framework.

Before any feature work, planning, or go-to-market activity, check whether a relevant Fieldwork skill applies. Skills trigger automatically - you do not need to be asked.

## Skill index

| Skill                | When to trigger                                                          |
| -------------------- | ------------------------------------------------------------------------ |
| `onboard`            | First session in a new project, or when project context is missing/stale |
| `discover`           | User describes a new idea, feature request, or opportunity               |
| `write-spec`         | An opportunity has been validated and approved                           |
| `review-spec`        | A spec has been written and needs critique                               |
| `write-gtm`          | A spec is approved and go-to-market planning is needed                   |
| `write-marketing`    | A GTM plan exists and marketing assets are needed                        |
| `write-launch-brief` | A feature is approaching launch                                          |
| `close-feature`      | A feature has shipped                                                    |
| `writing-skills`     | User wants to create a new Fieldwork skill or extend an existing one     |

## Hard rules

- Never write code or scaffold implementation before a spec exists and is approved.
- Never write a GTM plan before a spec exists.
- Always read `context/project-context.md` before invoking any skill.
- If `context/project-context.md` does not exist, run `onboard` first.
- Output files go in `outputs/` - never in the project source tree.

## MCP usage

Use available MCPs to enrich outputs:
- GitHub MCP → scaffold tasks as Issues
- Linear MCP → scaffold tasks as Linear tickets
- Slack MCP → draft launch announcements
- Confluence MCP → publish specs and GTM plans

If an MCP is not available, write output to file and note where it should be published.

## Output directory

All skill outputs go to `outputs/`:
```
outputs/
  opportunities/{feature-name}/opportunity.md
  opportunities/{feature-name}/assumptions.md
  specs/{feature-name}/spec.md
  specs/{feature-name}/spec-review.md
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

## Context files

Always load these before any skill runs:
- `context/project-context.md` - codebase, tech stack, architecture
- `context/product-context.md` - OKRs, personas, positioning, roadmap
- `context/constraints.md` - stakeholders, tech debt, non-negotiables

If any context file is missing or contains unfilled template text, run `onboard` before proceeding.

## Discover compatibility

If `outputs/opportunities/` already contains files from a previous discover run, load them directly. Do not re-run `discover` - ask the PM to confirm the existing opportunity before proceeding to `write-spec`.

If `outputs/opportunities/{feature-name}/assumptions.md` exists, load it into `write-spec` as the riskiest assumptions section.


