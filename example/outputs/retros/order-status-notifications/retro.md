---
feature: order-status-notifications
shipped-date: 2026-03-31
retro-date: 2026-04-14
---

# Feature Retro: Order Status Notifications

## What shipped vs. what was specced

### Acceptance criteria
- [x] Status change → email sent within 30 seconds
- [x] No customer email → silent skip, no error
- [x] Resend failure → Sentry log + retry
- [x] Duplicate status → no duplicate notification
- [x] `order_notifications` table records correctly
- [x] Toast confirmation appears
- [x] Email templates verified in Gmail, Apple Mail, Outlook
- [ ] Mobile-responsive at 375px - **missed**: Apple Mail on iOS clipped the footer. Fixed in a hotfix 2 days post-launch.

### Scope changes
**Added (not in spec):**
- Unsubscribe link added to all emails after legal flagged it as required. Not in spec. Added 1 day to implementation.

**Cut (was in spec):**
- Nothing cut.

## Surprises
- Resend requires domain verification to be completed 24h before first send - we almost missed this. The pre-launch checklist caught it with 36h to spare.
- The `order.service.ts` god file was worse than expected. The "thin hook" approach worked but took 2x longer than estimated because understanding the existing code took time.
- 3 customers emailed within 24h of launch saying they loved it. Faster positive signal than expected.

## Spec quality
**What the spec got right:**
- Edge cases (no email, duplicate status, Resend failure) were all documented and all needed to be handled
- File paths were accurate - no surprises in the codebase
- The "thin hook only" constraint on order.service.ts was the right call

**What the spec got wrong:**
- No mention of the unsubscribe link requirement - legal should be a standard checklist item for any customer-facing email feature
- Mobile email testing was listed as an acceptance criterion but no specific tool or process was defined - "verified at 375px" is not enough guidance

## GTM execution
**What happened vs. the plan:**
- Customer email sent on schedule (T+24h) - open rate 48% (above average)
- LinkedIn post performed well - 3x normal engagement
- Blog post deferred - waiting for 60-day data

**Early user signal:**
- 3 unsolicited positive emails from customers in week 1
- 1 support ticket: customer confused about who sent the email (sender name was "Acme Anvils" not "Acme Anvils on behalf of [Shop Name]") - fixed in hotfix

## Learnings

1. **Legal review is a standard step for customer-facing email features.** Add "legal/compliance review" to the spec template's dependencies section for any feature that sends external communications.
2. **"Thin hook" in a god file still takes time.** When the spec says "modify X (god file)", add a time buffer. Reading a 900-line file to find the right insertion point is not a 5-minute task.
3. **Email deliverability setup has a lead time.** Domain verification, warming, and DKIM/SPF setup need to be in the pre-launch checklist with a 48h buffer, not a 24h buffer.
4. **Sender name matters.** "Acme Anvils" as the sender confused customers who expected to hear from their specific shop. Spec should have addressed sender name format.
5. **Positive signal came faster than expected.** Don't wait for the 60-day metric to declare success - early qualitative signal is worth capturing and sharing with the team.

## What to carry forward
- Deferred: SMS notifications (v2) - add to next quarter's roadmap
- Deferred: Shop owner notification template customisation (v2)
- Follow-up: Fix sender name format to "Acme Anvils on behalf of [Shop Name]" - small spec, can ship next sprint
- Open question: Should we build a notification preferences UI, or wait for customer demand?
