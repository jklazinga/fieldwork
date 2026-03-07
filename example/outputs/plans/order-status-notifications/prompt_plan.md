---
feature: order-status-notifications
date: 2026-03-05
spec: outputs/specs/order-status-notifications/spec.md
superpowers-compatible: true
---

# Implementation Plan: Order Status Notifications

> **For Claude:** This plan is designed for task-by-task execution.
> - If Superpowers is installed: use `superpowers:executing-plans`
> - If not: use `fieldwork:scaffold-tasks` to create trackable issues
> Do NOT implement multiple tasks in one step. Do NOT skip verification steps.

**Goal:** Send automated email notifications to customers when order status changes.
**Architecture:** New `notifications` feature module. Thin hook in `order.service.ts`. Async send via existing Resend client. New `order_notifications` DB table for delivery tracking.
**Tech stack:** TypeScript, Prisma, Resend, Vitest
**Test approach:** TDD for notification.service.ts. Integration test for the status-change API endpoint.

---

### Task 1: Add OrderNotification model to Prisma schema

**Files:**
- Modify: `prisma/schema.prisma`

**Steps:**
1. Add model:
```prisma
model OrderNotification {
  id               String   @id @default(cuid())
  orderId          String
  notificationType String   // order_confirmed | work_started | ready_for_pickup
  sentAt           DateTime @default(now())
  resendMessageId  String?
  order            Order    @relation(fields: [orderId], references: [id])
}
```
2. Add relation to existing `Order` model: `notifications OrderNotification[]`
3. Run: `pnpm prisma migrate dev --name add_order_notifications`
4. Run: `pnpm prisma generate`

**Verification:** `pnpm prisma studio` - confirm `OrderNotification` table exists with correct columns.
**Depends on:** none

---

### Task 2: Write failing tests for notification.service.ts

**Files:**
- Create: `src/features/notifications/notification.service.test.ts`

**Steps:**
1. Write tests for:
   - `sendOrderNotification(orderId, type)` - sends email and logs to DB
   - `sendOrderNotification` with no customer email - skips silently, no error
   - `sendOrderNotification` with Resend failure - logs to Sentry, queues retry
   - Duplicate status (same type already sent) - does not send again
2. Run: `pnpm test src/features/notifications/notification.service.test.ts`

**Verification:** All tests fail with "Cannot find module" or similar - confirms RED state.
**Depends on:** Task 1

---

### Task 3: Create notification types and email templates

**Files:**
- Create: `src/features/notifications/notification.types.ts`
- Create: `src/features/notifications/templates/order-confirmed.ts`
- Create: `src/features/notifications/templates/work-started.ts`
- Create: `src/features/notifications/templates/ready-for-pickup.ts`

**Steps:**
1. `notification.types.ts`:
```typescript
export type NotificationType = 'order_confirmed' | 'work_started' | 'ready_for_pickup'

export interface NotificationPayload {
  orderId: string
  customerEmail: string
  customerName: string
  orderReference: string
  shopName: string
}
```
2. Each template exports a function `(payload: NotificationPayload) => { subject: string; html: string; text: string }`
3. Copy (from spec): order_confirmed = "Your order is confirmed", work_started = "Work has started on your order", ready_for_pickup = "Your order is ready for pickup"

**Verification:** TypeScript compiles with no errors: `pnpm tsc --noEmit`
**Depends on:** Task 1

---

### Task 4: Implement notification.service.ts

**Files:**
- Create: `src/features/notifications/notification.service.ts`

**Steps:**
1. Implement `sendOrderNotification(orderId: string, type: NotificationType)`:
   - Load order + customer email from DB via Prisma
   - If no customer email: log warning, return early
   - Check `order_notifications` for existing record with same orderId + type - if exists, return early (dedup)
   - Select template by type
   - Call `resend.emails.send()` (async, non-blocking - see `src/lib/email.ts` pattern)
   - On success: write to `order_notifications` table
   - On Resend failure: log to Sentry, enqueue retry via `src/lib/queue.ts`
2. Run: `pnpm test src/features/notifications/notification.service.test.ts`

**Verification:** All tests pass (GREEN).
**Depends on:** Tasks 2, 3

---

### Task 5: Hook notification into order.service.ts

**Files:**
- Modify: `src/features/orders/order.service.ts`

**Steps:**
1. Find the `updateOrderStatus` function (do not refactor the god file - add a thin hook only)
2. After the status update succeeds, call `sendOrderNotification(orderId, mapStatusToNotificationType(newStatus))` - fire and forget (do not await)
3. Add `mapStatusToNotificationType` helper that maps order status strings to `NotificationType` (return null for statuses that don't trigger notifications)
4. Run: `pnpm test`

**Verification:** All existing order tests still pass. No new test failures.
**Depends on:** Task 4

---

### Task 6: Write integration test for status-change endpoint

**Files:**
- Modify: `tests/features/orders/order-status.test.ts` (or create if it doesn't exist)

**Steps:**
1. Add test: PATCH `/orders/:id/status` with valid status → response 200, notification logged in DB
2. Add test: PATCH `/orders/:id/status` with no customer email → response 200, no notification logged
3. Run: `pnpm test tests/features/orders/`

**Verification:** Both new tests pass.
**Depends on:** Task 5

---

### Task 7: Add toast confirmation to order status UI

**Files:**
- Modify: `src/features/orders/components/OrderStatusSelect.tsx` (or equivalent)

**Steps:**
1. After successful status update API call, check response for `notificationSent: true` flag
2. If true: show toast "Customer notified by email" using `src/components/Toast.tsx`
3. If false (no customer email): show no toast (silent)
4. Add `notificationSent` boolean to the API response in `order.service.ts`

**Verification:** Manual test - update order status in dev, confirm toast appears. Update order with no customer email, confirm no toast.
**Depends on:** Task 5

---

### Task 8: Commit

**Steps:**
1. `git add -A`
2. `git commit -m "feat: order status notifications via Resend"`
3. Push to feature branch

**Verification:** CI passes.
**Depends on:** Tasks 6, 7
