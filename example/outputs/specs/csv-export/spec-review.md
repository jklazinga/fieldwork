---
feature: csv-export
status: approved
date: 2026-03-03
---

# Spec Review: CSV Export

## Summary
Solid spec. Two issues need resolution before engineering starts. Three minor gaps to address.

## Issues (must fix before approval)

### 1. RFC 4180 compliance not testable as written
AC #8 says "correctly escaped per RFC 4180" but doesn't specify which library or approach. If the engineer hand-rolls CSV serialisation, edge cases will be missed. **Fix:** Specify use of a CSV library (e.g. `csv-stringify`) or add explicit test cases for comma, quote, and newline in task names.

### 2. Streaming approach conflicts with test strategy
The spec says "stream" but the test file is `export.test.ts` with no guidance on how to test a streaming response. Supertest handles streams but needs explicit setup. **Fix:** Add a note in the spec or test file scaffold on how to assert streaming CSV responses.

## Minor gaps (address in spec or implementation)

### 3. Content-Type header not specified
The API route should return `Content-Type: text/csv` and `Content-Disposition: attachment; filename="..."`. Not mentioned in spec. Add to AC or files-to-create notes.

### 4. Auth check placement
Spec says "API route returns 403 if user not in team" but doesn't say where the auth check lives. Middleware? Route handler? Given the existing pattern in `app/api/`, this should be explicit. **Fix:** Add one line: "Auth check uses `getServerSession` + team membership query, same pattern as `app/api/projects/[id]/tasks/route.ts`."

### 5. Audit log action type
Spec says "confirm `csv_export` action type is handled (add if missing)" - this is vague. Either confirm it exists now or add it as a concrete task. Don't leave it as a conditional.

## What the spec gets right
- Acceptance criteria are specific and testable
- Edge cases are thorough (null assignee, null due date, empty project)
- Scope is tight - no feature creep
- Vercel constraint is called out explicitly

## Decision
Approved after fixes to issues #1 and #2. Issues #3–5 can be resolved during implementation.
