---
name: analyse-competitors
description: "Triggers when the user wants to understand the competitive landscape - what alternatives exist, how they're positioned, where the gaps are, and what this means for the product. Also triggers if the user says 'competitive analysis', 'what are competitors doing', 'how do we compare', or 'who else does this'. Can run standalone or feed into discover, write-spec, or write-gtm."
---

# Analyse Competitors

## Overview

Understand the competitive landscape well enough to make sharper product decisions. This is not a feature comparison matrix - it is an analysis of how competitors frame the problem, who they're built for, and where they fall short. The output feeds directly into positioning, opportunity framing, and GTM.

**Announce at start:** "I'm using the Fieldwork analyse-competitors skill to map the competitive landscape."

## When to use this

Use when:
- Starting discovery on a new opportunity and need to understand what already exists
- Writing a GTM plan and need to sharpen positioning against alternatives
- A competitor has just shipped something relevant
- The user asks "who else does this?" or "how do we compare?"

Do NOT use when:
- The user wants to copy a competitor's feature - that's a discovery problem, not a competitive analysis
- The user wants a full market sizing exercise - that's out of scope

## Pre-flight check

1. Load `context/product-context.md` - read the Competitors section. What's already known?
2. Ask: "What's the context for this analysis - discovery, GTM, or a specific competitor move?"
3. Ask: "Which competitors are you most concerned about? Any you want to make sure we cover?"
4. If connected to an opportunity: load `outputs/opportunities/{feature-name}/opportunity.md`

## Checklist

### 1. Identify the alternatives
List every realistic alternative a user might choose instead of this product for this specific problem. Include:
- Direct competitors (same category, same problem)
- Indirect competitors (different category, same job-to-be-done)
- The "do nothing" option - what users do today without any tool

Do not limit to named competitors. "Spreadsheet + email" is a competitor.

### 2. Analyse each alternative - one at a time
For each competitor, answer:
- **Who is it built for?** (Not the marketing claim - the actual user it serves best)
- **What problem does it solve best?** (Where it genuinely wins)
- **Where does it fall short?** (Specific gaps, not generic weaknesses)
- **How does it position itself?** (The story it tells - tagline, homepage, sales pitch)
- **What does it cost?** (Pricing model and rough tier)
- **What do users say about it?** (Reviews, complaints, praise - if available)

Keep each analysis tight. Three sentences per dimension is enough.

### 3. Find the gaps
After analysing all alternatives, ask:
- What problem is nobody solving well?
- What user segment is underserved?
- What positioning angle is unclaimed?
- What do users consistently complain about across all alternatives?

These gaps are where differentiation lives.

### 4. Identify the threat
Ask: "Which competitor is most likely to eat into this product's position in the next 12 months, and why?"

This is the competitive risk. It belongs in the GTM plan and in `context/constraints.md`.

### 5. Derive positioning implications
Based on the gap analysis, suggest 1-3 positioning angles that are:
- Defensible (not easily copied)
- Grounded in a real user need (not just a feature claim)
- Distinct from what competitors are already saying

### 6. Write the analysis
Save to `outputs/competitive/{analysis-name}/competitive-analysis.md`.

Update `context/product-context.md` Competitors section - append new findings, do not rewrite.

If connected to an opportunity: note competitive context in `outputs/opportunities/{feature-name}/opportunity.md` under a "Competitive context" section.

## Output: outputs/competitive/{analysis-name}/competitive-analysis.md

```markdown
---
analysis: {analysis-name}
date: YYYY-MM-DD
context: [discovery | gtm | standalone | competitor-move]
linked-opportunity: outputs/opportunities/{feature-name}/opportunity.md (if exists)
---

# Competitive Analysis: {Analysis Name}

## Scope
[What problem or opportunity this analysis covers - one sentence]

## Alternatives map

### [Competitor / Alternative name]
- **Built for:** [Who it actually serves]
- **Wins at:** [Where it genuinely beats alternatives]
- **Falls short:** [Specific gaps]
- **Positioning:** [How it describes itself]
- **Pricing:** [Model and rough tier]
- **User signal:** [What users say - reviews, complaints, praise]

_(repeat for each alternative)_

## The "do nothing" option
[What users do today without any tool - manual process, workaround, or just living with the problem]

## Gaps
[What nobody is solving well - specific, not generic]

## Competitive threat
[Which competitor is most likely to move into this space, and what that looks like]

## Positioning implications
[1-3 angles that are defensible, user-grounded, and unclaimed]

## Recommended next step
[Feed into discover / update GTM positioning / note as constraint / monitor and revisit]
```

## After analyse-competitors

"Competitive analysis saved to `outputs/competitive/{analysis-name}/competitive-analysis.md`.
`context/product-context.md` updated with new competitive context.

[If connected to opportunity]: Competitive framing added to opportunity.md.

The clearest gap is: [gap]. The strongest positioning angle is: [angle].

[Route based on context]:
- Discovery context: 'Ready to continue with `discover` using this competitive framing?'
- GTM context: 'This feeds directly into the positioning section of the GTM plan.'
- Standalone: 'Want to explore any of these gaps as a new opportunity?'"
