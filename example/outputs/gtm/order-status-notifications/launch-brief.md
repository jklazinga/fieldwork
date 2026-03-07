---
feature: order-status-notifications
launch-date: 2026-03-31 09:00 NZDT
status: ready
---

# Launch Brief: Order Status Notifications

## Launch summary
**Date/time:** March 31, 2026 at 09:00 NZDT
**Type:** Hard release (no feature flag - notifications fire on status change immediately after deploy)
**Owner:** Alex (PM)
**On-call engineer:** [Engineer name]
**Rollback owner:** [Engineer name]

## Pre-launch checklist
_Derived from spec acceptance criteria_

- [ ] Status change → email sent within 30 seconds (verified in staging)
- [ ] No customer email → status update succeeds, no notification sent, no error
- [ ] Resend failure → error logged to Sentry, retry queued
- [ ] Duplicate status → no duplicate notification sent
- [ ] `order_notifications` table records: order_id, notification_type, sent_at, resend_message_id
- [ ] Toast "Customer notified by email" appears after status change
- [ ] All three email templates verified in Gmail, Apple Mail, Outlook (PM sign-off via Resend preview)
- [ ] Mobile-responsive email layout verified at 375px
- [ ] Migration prompt copy approved by PM
- [ ] Resend sending domain verified and warmed (check Resend dashboard)
- [ ] Sentry alert configured for Resend errors
- [ ] Support team briefed: "Customers will now receive emails from Acme Anvils when order status changes. If a customer asks, it's intentional."
- [ ] Baseline support contact volume pulled from Intercom (for post-launch comparison)

## Launch sequence

| Time | Action | Owner |
|---|---|---|
| T-48h (March 29) | Final QA against acceptance criteria in staging | Engineering |
| T-48h | PM reviews email templates via Resend preview | PM |
| T-24h (March 30) | Confirm on-call coverage for launch window | Engineering |
| T-24h | Brief support team | PM |
| T-4h | Final deploy check - staging green | Engineering |
| T-0 (March 31, 09:00) | Deploy to production | Engineering |
| T+30min | Verify: create test order, update status, confirm email received | PM + Engineering |
| T+1h | Check Sentry - no new errors | Engineering |
| T+1h | Check Resend dashboard - delivery rate >95% | PM |
| T+24h (April 1) | Send customer email announcement | PM |
| T+48h (April 2) | Post LinkedIn announcement | PM |
| T+14d (April 14) | Review metrics: notification send rate, support contact volume | PM |
| T+14d | Schedule retro (close-feature) | PM |

## Rollback plan
**Trigger condition:** Error rate spike >5% in Sentry within 1 hour of deploy, OR Resend delivery rate <80%, OR critical bug reported by multiple customers.

**Steps:**
1. Revert deploy on Railway: `railway rollback` (or redeploy previous version from Railway dashboard)
2. Notify PM immediately via Slack DM
3. Post in #team: "Order status notifications rolled back - investigating. Customers will not receive notifications until further notice."
4. Open incident in Sentry, assign to on-call engineer
**Estimated rollback time:** 5-10 minutes

## Monitoring

| Metric | Baseline | Alert threshold | Owner |
|---|---|---|---|
| Sentry error rate | 0 Resend errors | >5 errors in 1h | Engineering |
| Resend delivery rate | - | <80% | PM |
| Notification send rate | - | <10% of status changes (week 1) | PM |
| Support contacts ("where's my order?") | [Pull from Intercom pre-launch] | - | PM |

## Comms sequence

| Time | Channel | Message | Owner | Status |
|---|---|---|---|---|
| T-0 | In-app (migration prompt) | "We've added automatic customer notifications..." | Engineering | on deploy |
| T-0 | In-app (tooltip) | "Changing status? Your customer will be notified." | Engineering | on deploy |
| T+24h | Email (all customers) | "Your customers will now get automatic order updates" | PM | scheduled |
| T+48h | LinkedIn | Variant 1 post | PM | scheduled |

## Post-launch
- [ ] Confirm metrics baseline captured (T+2h)
- [ ] Check Sentry + Resend at T+24h
- [ ] Review notification send rate at T+7d
- [ ] Review support contact volume at T+14d
- [ ] Run `close-feature` retro at T+14d
