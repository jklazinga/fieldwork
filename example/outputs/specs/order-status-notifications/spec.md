---
feature: order-status-notifications
status: approved
date: 2026-03-05
opportunity: outputs/opportunities/order-status-notifications/opportunity.md
---

# Spec: Order Status Notifications

## Problem statement
Shop owners spend 30-60 minutes per day answering "is my order ready?" calls. Customers have no visibility into order progress. This is the #1 NPS complaint. We're building automated email notifications triggered by order status changes to reduce inbound contacts and improve customer experience.

## Proposed solution
When a smith updates an order's status in Acme Anvils, the system automatically sends a transactional email to the customer associated with that order. Three notification types in v1: order confirmed, work started, ready for pickup. Notifications are on by default. Copy is PM-defined (not shop-owner customisable in v1).

## Scope

### In scope
- Three email notification types: `order_confirmed`, `work_started`, `ready_for_pickup`
- Triggered on order status change (existing status field in `orders` table)
- Sent to the customer email on the order record
- PM-defined email templates (not customisable by shop owner in v1)
- Notification sent via Resend (existing email provider)
- Basic delivery tracking: sent timestamp logged to `order_notifications` table
- Notifications on by default for all new orders; existing orders opt-in via a one-time migration prompt

### Out of scope
- SMS notifications
- Customer self-service portal
- Shop owner notification template customisation
- Notification preferences UI (v1 uses defaults)
- Internal notifications to smiths
- Push notifications
- Notification analytics dashboard

## User flows

### Flow 1: Status update triggers notification
1. Smith opens an order in Acme Anvils
2. Smith changes order status (e.g. "In Progress" → "Ready for Pickup")
3. System detects status change
4. System looks up customer email on the order
5. System sends the appropriate email template via Resend
6. System logs notification to `order_notifications` table (order_id, notification_type, sent_at, resend_message_id)
7. Smith sees a confirmation toast: "Customer notified by email"

### Flow 2: Order created (confirmed notification)
1. PM or smith creates a new order with a customer email
2. On save, system sends `order_confirmed` email
3. Notification logged

### Edge cases
- **No customer email on order:** Skip notification silently. Log a warning. Do not block the status update.
- **Resend API failure:** Log the error to Sentry. Do not block the status update. Retry once after 60 seconds via a background job.
- **Duplicate status update (same status set twice):** Do not send duplicate notification. Check last notification type before sending.
- **Order deleted before notification sends:** Cancel the pending notification.

## Acceptance criteria
- [ ] When an order status changes to `confirmed`, `in_progress`, or `ready_for_pickup`, an email is sent to the customer email on the order within 30 seconds
- [ ] When no customer email exists on the order, no notification is sent and the status update succeeds without error
- [ ] When Resend returns a non-2xx response, the error is logged to Sentry and a retry is queued for 60 seconds later
- [ ] A duplicate status update (same status set twice in a row) does not trigger a second notification
- [ ] Each sent notification is recorded in `order_notifications` with: `order_id`, `notification_type`, `sent_at`, `resend_message_id`
- [ ] The smith sees a toast confirmation "Customer notified by email" after a status change that triggers a notification
- [ ] All three email templates render correctly in Gmail, Apple Mail, and Outlook (verified via Resend preview)
- [ ] Mobile-responsive email layout (verified at 375px viewport)

## Technical notes

### Files to create
- `src/features/notifications/notification.service.ts` - core notification logic
- `src/features/notifications/notification.types.ts` - types for notification events
- `src/features/notifications/templates/order-confirmed.ts` - email template
- `src/features/notifications/templates/work-started.ts` - email template
- `src/features/notifications/templates/ready-for-pickup.ts` - email template
- `prisma/migrations/YYYYMMDD_add_order_notifications/migration.sql` - new table

### Files to modify
- `src/features/orders/order.service.ts` - add notification trigger on status change (note: this is the 900-line god file - add a thin hook, do not expand it further)
- `prisma/schema.prisma` - add `OrderNotification` model
- `src/lib/email.ts` - extend with notification-specific send function

### Patterns to follow
- Email sending pattern: see `src/lib/email.ts` - use `resend.emails.send()`, do not add synchronous calls (see constraints)
- Background job pattern: see `src/lib/queue.ts` for retry queue implementation
- Toast notifications: see `src/components/Toast.tsx` - use existing component

### Known constraints
- `order.service.ts` is a 900-line god file - add a thin hook only, do not expand
- Email sending must be async (non-blocking) - existing constraint, see constraints.md
- No end-to-end tests exist - write unit tests for notification.service.ts and integration tests for the API endpoint

## Riskiest assumptions
_Loaded from outputs/opportunities/order-status-notifications/assumptions.md_

| Assumption | Dimension | Evidence | Mitigation |
|---|---|---|---|
| Shop owners will consistently update order status | Feasibility / Usability | No data on current update rate | Check DB before launch - if <20% of orders have status changes, consider improving the status update UX first |
| Customers want email (not SMS) | Desirability | Assumed from NPS comments | Ship email v1, add channel preference in v2 based on actual usage data |

## Open questions
_All resolved before spec approved_
- **On by default for existing orders?** → Yes, but show a one-time prompt to shop owners explaining the change. PM to write prompt copy.
- **Who controls copy?** → PM-defined templates in v1. Shop owner customisation deferred to v2.

## Dependencies
- Resend account and API key (exists - in production env)
- Background job queue (exists - `src/lib/queue.ts`)
- Customer email field on orders (exists - `orders.customer_email` in schema)
