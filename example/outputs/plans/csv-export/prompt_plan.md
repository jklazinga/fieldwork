---
feature: csv-export
date: 2026-03-04
spec: outputs/specs/csv-export/spec.md
superpowers-compatible: true
---

# Implementation Plan: CSV Export

> **For Claude:** This plan is designed for task-by-task execution.
> - If Superpowers is installed: use `superpowers:executing-plans`
> - If not: use `fieldwork:scaffold-tasks` to create trackable issues
> Do NOT implement multiple tasks in one step. Do NOT skip verification steps.

**Goal:** Export a project's tasks to CSV with one click, scoped to the authenticated team, with audit logging.
**Architecture:** New API route (`app/api/projects/[id]/export/route.ts`) streams a CSV response. New client component (`ExportButton.tsx`) triggers download via fetch + blob URL. Minimal changes to existing files.
**Tech stack:** Next.js App Router API routes, Prisma, `csv-stringify` for serialisation, NextAuth for session.
**Test approach:** TDD for the API route. Component tested with Testing Library (click → fetch mock → download triggered).

---

### Task 1: Install csv-stringify

**Files:**
- Modify: `package.json` (via npm)

**Steps:**
1. Run: `npm install csv-stringify`
2. Run: `npm install --save-dev @types/csv-stringify` (if types not bundled)

**Verification:** `node -e "require('csv-stringify')"` exits 0.
**Depends on:** none

---

### Task 2: Write failing API route test - auth check

**Files:**
- Create: `tests/api/projects/export.test.ts`

**Steps:**
1. Create the test file with a test: `GET /api/projects/[id]/export returns 403 when user is not in team`
2. Mock `getServerSession` to return a user not in the project's team
3. Run: `npm test tests/api/projects/export.test.ts` - confirm it fails (route doesn't exist yet)

**Verification:** Test output shows 1 failing test, failure reason is "Cannot find module" or 404 (not a false pass).
**Depends on:** Task 1

---

### Task 3: Write failing API route test - CSV output

**Files:**
- Modify: `tests/api/projects/export.test.ts`

**Steps:**
1. Add test: `GET /api/projects/[id]/export returns CSV with correct headers`
2. Add test: `GET /api/projects/[id]/export returns 200 with headers-only CSV for empty project`
3. Add test: `GET /api/projects/[id]/export escapes task names with commas and quotes`
4. Run: `npm test tests/api/projects/export.test.ts` - confirm all new tests fail

**Verification:** 4 failing tests total. No passing tests yet.
**Depends on:** Task 2

---

### Task 4: Implement the export API route

**Files:**
- Create: `app/api/projects/[id]/export/route.ts`

**Steps:**
1. Create the file with a `GET` handler
2. Get session with `getServerSession` - return 403 if no session
3. Query team membership - return 403 if user not in project's team
4. Query all tasks for the project via Prisma: `id, name, status, assignee { name }, dueDate, priority, createdAt`
5. Use `csv-stringify` to serialise rows with columns: `Task name, Status, Assignee, Due date, Priority, Created at`
6. Set response headers: `Content-Type: text/csv`, `Content-Disposition: attachment; filename="{project-name}-tasks-{YYYY-MM-DD}.csv"`
7. Return streamed response
8. Call `lib/audit.ts` with `{ action: "csv_export", projectId, userId }` on success

**Verification:** `npm test tests/api/projects/export.test.ts` - all 4 tests pass.
**Depends on:** Task 3

---

### Task 5: Add csv_export to audit log (if missing)

**Files:**
- Modify: `lib/audit.ts`

**Steps:**
1. Read `lib/audit.ts` - check if `csv_export` is a valid action type
2. If missing, add it to the action type union/enum
3. Run: `npm test` - confirm no regressions in audit-related tests

**Verification:** `lib/audit.ts` accepts `action: "csv_export"` without TypeScript error. `npm test` passes.
**Depends on:** Task 4

---

### Task 6: Write failing ExportButton component test

**Files:**
- Create: `tests/components/ExportButton.test.tsx`

**Steps:**
1. Create test: `ExportButton renders an "Export CSV" button`
2. Create test: `ExportButton triggers a fetch to /api/projects/{id}/export on click`
3. Mock `fetch` to return a blob
4. Run: `npm test tests/components/ExportButton.test.tsx` - confirm tests fail (component doesn't exist)

**Verification:** 2 failing tests. Failure is "Cannot find module", not a false pass.
**Depends on:** Task 1

---

### Task 7: Implement ExportButton component

**Files:**
- Create: `components/ExportButton.tsx`

**Steps:**
1. Create a `"use client"` component that accepts `projectId: string` prop
2. On click: fetch `/api/projects/${projectId}/export`, get blob, create object URL, trigger download via `<a>` click, revoke URL
3. Show loading state while fetch is in progress
4. Run: `npm test tests/components/ExportButton.test.tsx` - confirm both tests pass

**Verification:** Both component tests pass. Button renders, click triggers fetch.
**Depends on:** Task 6

---

### Task 8: Add ExportButton to project detail page

**Files:**
- Modify: `app/(dashboard)/projects/[id]/page.tsx`

**Steps:**
1. Import `ExportButton` from `components/ExportButton`
2. Add `<ExportButton projectId={id} />` to the page header, alongside existing header actions
3. Run: `npm run dev` - navigate to a project page, confirm button appears
4. Click button - confirm CSV downloads with correct filename and content

**Verification:** Button visible on project page. CSV downloads. Filename matches `{project-name}-tasks-{YYYY-MM-DD}.csv`.
**Depends on:** Task 5, Task 7

---

### Task 9: Run full test suite and commit

**Files:**
- none

**Steps:**
1. Run: `npm test` - confirm all tests pass, no regressions
2. Run: `npm run build` - confirm no TypeScript errors
3. Commit: `git add -A && git commit -m "feat: CSV export for project tasks"`

**Verification:** `npm test` exits 0. `npm run build` exits 0. Commit created.
**Depends on:** Task 8
