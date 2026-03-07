---
feature: order-status-notifications
status: approved
date: 2026-03-05
---

# Opportunity: Order Status Notifications

## Problem
Blacksmith shop owners and their customers have no visibility into order progress between the initial quote and the pickup call. Shop owners spend 30-60 minutes per day answering "is my order ready?" calls and emails. Customers feel anxious and underserved. This is the #1 support complaint in our NPS responses (mentioned by 34% of detractors).

## Current behaviour
Shop owners manually call or email customers when an order is ready. Some use a whiteboard to track status internally. Customers have no self-service way to check. The result: missed pickups, repeated follow-up calls, and customers who feel like they're bothering the shop.

## Proposed opportunity
Automated order status notifications - triggered when a smith updates an order's status in Acme Anvils. Customers receive an email (and optionally SMS) at key milestones: order confirmed, work started, ready for pickup. Shop owners get fewer inbound calls. Customers feel informed without having to ask.

## Success metrics
- Inbound "where's my order?" support contacts reduced by 40% within 60 days of launch
- 70% of orders with status updates trigger at least one customer notification within 30 days
- NPS detractor mentions of "communication" reduced by 50% in next quarterly survey

## Riskiest assumptions
1. Shop owners will consistently update order status in the app (currently many don't) - **high importance, weak evidence**
2. Customers want email notifications (not SMS, not a portal) - **high importance, weak evidence**
3. The notification content will be clear enough that customers don't call anyway - **medium importance, weak evidence**

## Scope boundary
- **Not in scope:** Customer self-service portal (next quarter), SMS notifications (v2), push notifications, notification preferences UI (v1 uses sensible defaults)
- **Not in scope:** Internal notifications to smiths (separate feature)

## Open questions
- Should notifications be on by default for all existing orders, or opt-in?
- Who controls the notification copy - PM-defined templates, or shop owner customisable?
