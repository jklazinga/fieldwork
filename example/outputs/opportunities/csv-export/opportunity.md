---
feature: csv-export
status: approved
date: 2026-03-02
---

# Opportunity: CSV Export

## Problem
Stakeholders (Alexes) want to see project status without logging into Trackr. Engineering leads (Sams) are manually copying task lists into spreadsheets or Slack messages to share progress. This is happening weekly for ~30% of active teams based on support tickets and user interviews.

## Who it affects
- **Primary:** Engineering leads who report to non-technical stakeholders
- **Secondary:** Stakeholders who receive the exports (they never log in)

## Current workarounds
- Manual copy-paste from Trackr into Google Sheets
- Screenshots of the task board
- Verbal updates in Slack

## Opportunity
Let users export any project's tasks to CSV with one click. The CSV should be immediately usable in Google Sheets or Excel - no cleanup required.

## Why now
- Three enterprise trial teams cited "no export" as a blocker to upgrading
- Competitors (Linear, Jira) all have this - it's table stakes for teams with non-technical stakeholders
- Low implementation complexity relative to impact

## Success metrics
- 20% of active teams use export within 30 days of launch
- Reduction in "how do I share this?" support tickets (currently ~5/week)
- At least 2 enterprise trial conversions cite export as a factor

## Out of scope
- Scheduled/automated exports
- PDF export
- Custom column selection (ship with sensible defaults first)
- Excel (.xlsx) format - CSV opens in Excel fine

## Assumptions (riskiest)
1. Teams will use CSV export for stakeholder reporting, not just data backup
2. Default column set (task name, status, assignee, due date, priority) covers 80% of use cases
3. Vercel 10s limit won't be hit for projects up to 500 tasks
