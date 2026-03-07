---
feature: csv-export
status: approved
date: 2026-03-03
opportunity: outputs/opportunities/csv-export/opportunity.md
---

# Spec: CSV Export

## Goal
Let any authenticated team member export a project's tasks to a CSV file with one click from the project view.

## Background
See opportunity doc. Approved 2026-03-03.

## In scope
- Export button on the project detail page (`app/(dashboard)/projects/[id]/page.tsx`)
- API route that generates and streams a CSV response
- Columns: Task name, Status, Assignee (name), Due date (ISO 8601), Priority, Created at
- Filename: `{project-name}-tasks-{YYYY-MM-DD}.csv`
- Audit log entry on every export (`lib/audit.ts`)
- Empty state: if project has no tasks, return a CSV with headers only (no error)

## Out of scope
- Scheduled exports
- PDF or Excel format
- Custom column selection
- Filtering before export (exports all tasks in the project)

## Acceptance criteria

1. **Export button visible** - An "Export CSV" button appears in the project detail page header for all authenticated team members.
2. **Download triggers** - Clicking the button downloads a `.csv` file named `{project-name}-tasks-{YYYY-MM-DD}.csv`.
3. **Correct columns** - CSV contains exactly: Task name, Status, Assignee, Due date, Priority, Created at. Headers match exactly (case-sensitive).
4. **Data scoped to team** - The API route returns 403 if the authenticated user does not belong to the project's team. No cross-team data is ever returned.
5. **Audit logged** - Every successful export creates an entry in the audit log with: `action: "csv_export"`, `projectId`, `userId`, `timestamp`.
6. **Empty project** - A project with zero tasks returns a CSV with headers only. No error, no empty file.
7. **Large project** - A project with 500 tasks exports successfully within the Vercel 10s limit.
8. **Special characters** - Task names containing commas, quotes, or newlines are correctly escaped per RFC 4180.

## Files to create
- `app/api/projects/[id]/export/route.ts` - GET handler, streams CSV response
- `components/ExportButton.tsx` - client component, triggers download via fetch + blob URL
- `tests/api/projects/export.test.ts` - API route tests

## Files to modify
- `app/(dashboard)/projects/[id]/page.tsx` - add `<ExportButton projectId={id} />` to page header
- `lib/audit.ts` - confirm `csv_export` action type is handled (add if missing)

## Edge cases
- Project with 0 tasks → headers-only CSV, 200 response
- User not in team → 403, no data
- Task with null assignee → "Unassigned" in CSV
- Task with null due date → empty string in CSV
- Task name with comma → wrapped in double quotes per RFC 4180
- Task name with double quote → escaped as `""` per RFC 4180

## Open questions (resolved)
- ~~Stream or buffer?~~ Stream - avoids Vercel memory limits for large projects
- ~~Include subtasks?~~ No - out of scope for v1
