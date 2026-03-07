---
feature: order-status-notifications
date: 2026-03-05
---

# Assumption Map: Order Status Notifications

## High importance / weak evidence (riskiest - validate first)

| Assumption | Dimension | Evidence so far |
|---|---|---|
| Shop owners will consistently update order status in the app | Feasibility / Usability | No data. Current status update rate is unknown. Many shops use the whiteboard as primary. |
| Customers want email notifications (not SMS, not a portal) | Desirability | Assumed from NPS comments. No direct research on preferred channel. |

## High importance / strong evidence (monitor)

| Assumption | Dimension | Evidence so far |
|---|---|---|
| "Where's my order?" is a top pain point | Desirability | 34% of NPS detractors mention it. Strong signal. |
| Shop owners want to reduce inbound calls | Viability | Mentioned in 5/5 customer interviews last quarter. |

## Low importance / weak evidence (low priority)

| Assumption | Dimension | Evidence so far |
|---|---|---|
| Notification content will be clear enough to prevent follow-up calls | Usability | No evidence. Low importance because we can iterate on copy post-launch. |
| Customers will act on pickup notifications within 48h | Usability | No evidence. Affects shop scheduling but not core feature value. |

## Validation ideas

**For "shop owners will update status consistently":**
- Check current status update rate in the DB before building - if <20% of orders have status changes, the feature won't fire
- Consider: does the status update flow need to be improved first?

**For "customers want email (not SMS)":**
- Add a single question to the next customer email: "How would you prefer to be notified when your order is ready?" with 3 options
- Can be done in 1 week before spec is written
