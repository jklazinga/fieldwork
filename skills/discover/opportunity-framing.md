# Opportunity Framing Reference

## What is a framing?

A framing is a way of scoping the opportunity - where in the system you intervene, and how broadly. Different framings have different cost, risk, and value trade-offs. The PM picks the framing before any spec work begins.

## How to generate framings

Produce 2-3 framings. Each framing should:
1. Name the intervention point (data layer, API layer, UI layer, workflow layer, etc.)
2. State the scope (narrow vs. broad)
3. State the trade-off (speed vs. completeness, risk vs. coverage)

## Framing patterns

| Pattern | When to use | Trade-off |
|---|---|---|
| **Data layer** | Problem is about what data exists or how it's structured | Broad impact, high complexity, long lead time |
| **API / service layer** | Problem is about how data is exposed or processed | Medium complexity, enables multiple surfaces |
| **UI layer** | Problem is about how users interact with existing data | Fast to ship, narrow impact, may not fix root cause |
| **Workflow layer** | Problem is about the sequence of steps users take | Requires deep user research, high value if right |
| **Notification / trigger layer** | Problem is about when users are informed | Low complexity, high leverage for engagement problems |
| **Integration layer** | Problem is about connecting to external systems | Depends on partner APIs, medium complexity |

## Example framings

**Problem:** Users don't know when a record has been updated by a colleague.

- **Framing A - Notification layer (narrow, fast):** Add an in-app notification when a record is edited. Ships in 1 sprint. Doesn't solve the underlying visibility problem.
- **Framing B - Activity feed (medium, 2-3 sprints):** Add a per-record activity log. Solves visibility but requires schema changes.
- **Framing C - Real-time collaboration (broad, 2+ quarters):** Live presence indicators and conflict resolution. Solves the root problem but high complexity.

## Presenting framings to the PM

Present each framing as:
> "We could solve this at the [layer] level. That means [what we'd build]. The upside is [benefit]. The downside is [cost/risk]. This would take roughly [complexity]."

Then ask: "Which framing fits your constraints best, or do you want to propose a different scope?"
