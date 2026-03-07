---
feature: order-status-notifications
date: 2026-03-05
spec: outputs/specs/order-status-notifications/spec.md
reviewer: fieldwork:review-spec
---

# Spec Review: Order Status Notifications

## Blockers
_None_

## Important
- [ ] Acceptance criterion "renders correctly in Gmail, Apple Mail, and Outlook" - who owns this verification step? The spec doesn't assign it. Engineering needs to know if this is a PM sign-off or an automated check.
- [ ] The one-time migration prompt for existing orders is mentioned in scope but not in user flows. What does the prompt say? When does it appear? Who sees it (all shop owners, or only those with existing orders)?

## Minor
- [ ] `order_notifications` table - should `resend_message_id` be nullable? Resend may not return an ID on failure. Spec doesn't address this.
- [ ] Toast copy "Customer notified by email" - what if the customer has no email? The spec says skip silently, but the smith gets no feedback. Consider a different toast or no toast in that case.

## Assumption gaps
- The spec correctly loads both riskiest assumptions and includes mitigations. No gaps.

## Opportunity alignment
- All three success metrics from opportunity.md are reflected in acceptance criteria or monitoring plan. ✅

## Summary
Solid spec. The core flow is clear, edge cases are well-covered, and the technical notes are grounded in the actual codebase. Two important issues to resolve before engineering starts: email verification ownership, and the migration prompt user flow. Both are small - this spec is close to ready.

---
## Resolution log

- "Email verification ownership" - resolved 2026-03-05: PM signs off on email rendering via Resend preview tool. Engineering sends preview links before launch.
- "Migration prompt user flow" - resolved 2026-03-05: Prompt appears on first login after deploy for all shop owners with >0 existing orders. Copy: "We've added automatic customer notifications. Your customers will now receive emails when their order status changes. [Got it]". PM to add to launch brief.
