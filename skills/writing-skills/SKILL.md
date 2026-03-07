---
name: writing-skills
description: "Triggers when the user asks to create a new Fieldwork skill, extend an existing skill, or add a custom skill to their project. Guides the creation of a well-structured SKILL.md file."
---

# Writing Skills

## Overview

Create a new Fieldwork skill or extend an existing one. A skill is a single SKILL.md file with YAML frontmatter and a structured prompt. The frontmatter description is the trigger mechanism - get it right and the agent will use the skill automatically. Get it wrong and the skill will never fire, or will fire at the wrong time.

**Announce at start:** "I'm using the Fieldwork writing-skills guide to create a new skill."

## Skill anatomy

Every skill has three parts:

### 1. Frontmatter (the trigger)
```yaml
---
name: skill-name
description: "Triggers when [specific condition]. Does NOT trigger when [exclusion condition]."
---
```

The `description` is what the agent reads to decide whether to trigger. It must be:
- **Specific:** "Triggers when the user describes a new idea and no opportunity document exists" - not "Triggers for discovery"
- **Exclusive:** Include a "Does NOT trigger when..." clause to prevent false positives
- **Action-oriented:** Start with "Triggers when..." not "This skill..."

### 2. Overview (the announcement)
```markdown
## Overview
[One paragraph - what this skill does and why]

**Announce at start:** "I'm using the Fieldwork [skill-name] skill to [action]."
```

The announcement tells the user which skill is running. Always include it.

### 3. Body (the instructions)

Structure:
1. **HARD-GATE** (if needed) - conditions that must be true before proceeding
2. **Pre-flight check** - files to load, directories to create, upstream status to verify
3. **Checklist** - ordered steps, one at a time
4. **Output format** - exact file path and markdown template
5. **After [skill-name]** - what to tell the user when done, what to offer next

## HARD-GATE rules

Use a HARD-GATE when skipping a step would produce incorrect or dangerous output:
- Spec not approved → don't write the plan
- Context files missing → don't run discovery
- GTM plan not approved → don't write marketing assets

Format:
```markdown
<HARD-GATE>
Do NOT [action] if:
- [Condition 1]
- [Condition 2]
</HARD-GATE>
```

Do not use HARD-GATEs for soft preferences. Only use them for conditions that would cause real harm to the output.

## Output file rules

Every skill that produces a file must:
1. Specify the exact output path: `outputs/{type}/{feature-name}/{filename}.md`
2. Create the directory if it doesn't exist (include this as a step)
3. Use YAML frontmatter with at minimum: `feature`, `status`, `date`
4. Use `status: draft` until the user explicitly approves

## Status values

All output documents use the same status vocabulary:
- `draft` - in progress, not approved
- `approved` - PM has explicitly approved
- `shipped` - feature has launched

Skills that depend on upstream documents must check the status field in frontmatter before proceeding.

## Chaining rules

Skills chain via status checks, not via direct invocation. A skill should:
1. Check that upstream documents exist and have the right status
2. Offer to invoke the next skill at the end - but not automatically

Never invoke another skill without asking the user first.

## Testing a new skill

Before shipping a skill:
1. **Trigger test:** Describe the scenario in plain language. Does the agent trigger the skill? Does it trigger when it shouldn't?
2. **Gate test:** Try to skip a required upstream step. Does the HARD-GATE stop you?
3. **Output test:** Run the skill end-to-end. Is the output file correct? Is the frontmatter right?
4. **Chain test:** Does the skill correctly offer the next step? Does it check upstream status?

## Skill file location

Custom skills for a specific project go in `.fieldwork/skills/{skill-name}/SKILL.md` in the project repo.

Shared skills that could apply to any project go in the Fieldwork repo at `skills/{skill-name}/SKILL.md`.

## Skill template

```markdown
---
name: {skill-name}
description: "Triggers when [specific condition]. Does NOT trigger when [exclusion]."
---

# {Skill Name}

## Overview

[One paragraph - what this skill does and why]

**Announce at start:** "I'm using the Fieldwork {skill-name} skill to [action]."

<HARD-GATE>
Do NOT [action] if:
- [Condition]
</HARD-GATE>

## Pre-flight check

1. Load `[upstream file]` - confirm `status: [required status]`. If not, stop.
2. Create directory `outputs/{type}/{feature-name}/` if it does not exist.

## Checklist

### 1. [Step name]
[Instructions]

### 2. [Step name]
[Instructions]

## Output: outputs/{type}/{feature-name}/{filename}.md

\`\`\`markdown
---
feature: {feature-name}
status: draft
date: YYYY-MM-DD
---

# [Title]

[Content]
\`\`\`

## After {skill-name}

"[What to tell the user. What file was saved. What to offer next.]"
```
