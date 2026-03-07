---
feature: csv-export
status: shipped
shipped-date: 2026-03-28
retro-date: 2026-04-04
---

# Feature Retro: CSV Export

## What shipped vs. what was specced

### Acceptance criteria
- [x] Export button visible on project detail page
- [x] Download triggers with correct filename
- [x] Correct columns (Task name, Status, Assignee, Due date, Priority, Created at)
- [x] Data scoped to team - 403 for non-members
- [x] Audit logged on every export
- [x] Empty project returns headers-only CSV
- [x] 500-task project exports within 10s
- [x] Special characters escaped per RFC 4180

### Scope changes
**Added (not in spec):**
- Loading spinner on the Export button - added during implementation because the fetch took 1-2s on large projects and the button felt broken without feedback

**Cut (was in spec):**
- Nothing cut

## Surprises
- `csv-stringify` streaming API was more complex than expected with Next.js App Router - had to use `TransformStream` to bridge the two. Took Jordan an extra half-day.
- The 3 enterprise trial teams responded to Alex's personal email within hours. One converted same day.

## Spec quality
**What the spec got right:**
- Edge cases (null assignee, null due date, empty project) were all hit in testing - good they were in the spec
- Vercel 10s constraint was the right call - a 500-task project takes ~3s, well within limit

**What the spec got wrong:**
- Didn't specify streaming implementation approach - left the engineer to figure out `TransformStream` compatibility with App Router. Should have said "use TransformStream + ReadableStream" or linked to a reference.
- "Confirm `csv_export` action type is handled (add if missing)" was vague - it was missing, and finding that out mid-implementation broke flow. Should have been a concrete task.

## GTM execution
**What happened vs. the plan:**
- In-app changelog: went out same day ✓
- Email to active teams: sent day +1, 34% open rate, 12% click-through ✓
- Discord: posted, 8 reactions, 2 questions answered ✓
- Twitter: posted, 47 impressions, 3 likes - low signal as expected ✓
- Enterprise outreach: 3 emails sent, 1 conversion same day, 1 trial extended, 1 no response ✓

**Early user signal:**
- 31 teams exported in the first week (15% of active teams)
- 4 support tickets about export - all "how do I find the button?" → button placement could be more prominent
- One user asked for filtered export (by status) - noted for v2

## Learnings

1. **Streaming in Next.js App Router needs explicit documentation.** Any spec that involves streaming a response should specify the exact approach (`TransformStream` + `ReadableStream`), not just say "stream it." The compatibility surface is non-obvious.
2. **"Add if missing" tasks cause mid-implementation interruptions.** Conditional tasks ("do X if Y is missing") should be resolved before the plan is written. Check the condition during spec review, not during implementation.
3. **Personal outreach to churning trials works.** Alex's direct email converted one trial same day. For features that unblock specific customers, direct outreach should be the first GTM action, not an afterthought.

## What to carry forward
- Filtered export (by status, assignee, or date range) - 3 requests in first week, add to backlog
- Button placement: consider moving "Export CSV" to a more prominent position (currently in header, users expect it in a menu)
- Streaming pattern: document `TransformStream` approach in `context/project-context.md` under "Patterns to follow"
