---
feature: order-status-notifications
status: approved
date: 2026-03-05
spec: outputs/specs/order-status-notifications/spec.md
---

# GTM Plan: Order Status Notifications

## Positioning
Acme Anvils now automatically notifies your customers when their order is ready - so you spend less time on the phone and more time at the forge.

## Target audience

### Primary: Sam the Shop Owner
Sam is the one answering "is my order ready?" calls. This feature directly removes his biggest daily frustration. He'll notice the reduction in calls within the first week. Lead with the time saved, not the technology.

### Secondary: Priya the Production Manager
Priya will appreciate the delivery tracking in the `order_notifications` table (visible in the order detail view). She'll use it to confirm customers were notified before scheduling pickups. Mention it in the changelog but don't lead with it.

## Value proposition
**Problem:** Shop owners spend 30-60 minutes a day answering "where's my order?" calls.
**Before:** Manual calls and emails, customers left in the dark, missed pickups.
**After:** Customers get automatic emails at key milestones. Fewer calls. Happier customers. More time forging.

## Launch type
- [x] Customer announcement (email + in-app)
- [ ] Quiet rollout
- [ ] Internal announcement only
- [ ] Public launch (blog, social, press)

## Channels & tactics

| Channel | Message | Owner | Timing |
|---|---|---|---|
| In-app (migration prompt) | "We've added automatic customer notifications..." | Engineering | On deploy (T-0) |
| Email to all customers | "Your customers now get automatic updates" | PM | T+24h |
| In-app tooltip on order status field | "Changing status? Your customer will be notified." | Engineering | On deploy (T-0) |
| LinkedIn post | Feature announcement | PM | T+48h |
| Blog post | "How we cut 'where's my order?' calls by 40%" (post-data) | PM | T+60d (after data) |

## Timeline

| Milestone | Date | Owner |
|---|---|---|
| Feature complete | March 28 | Engineering |
| Internal preview + QA | March 29 | PM + Engineering |
| Deploy to production | March 31 | Engineering |
| Customer email send | April 1 | PM |
| LinkedIn post | April 2 | PM |
| Metrics review | April 14 | PM |
| Blog post (if data supports) | May 31 | PM |

## Success metrics
- Inbound "where's my order?" support contacts reduced by 40% within 60 days (baseline: measure current volume before launch)
- 70% of orders with status updates trigger at least one notification within 30 days of launch
- NPS detractor mentions of "communication" reduced by 50% in next quarterly survey (June)

## Risks
- **Low adoption of status updates:** If shop owners don't update order status, notifications won't fire. Mitigation: monitor notification send rate in week 1. If <30% of orders trigger a notification, consider a nudge in the UI.
- **Email deliverability:** Resend is new for transactional email at this volume. Monitor bounce/spam rates in week 1.
- **Customer confusion:** Some customers may not expect emails from their blacksmith. Mitigation: clear sender name ("Acme Anvils via [Shop Name]") and unsubscribe link.

## Open questions
_All resolved_
- Baseline support contact volume: PM to pull from Intercom before launch date.
