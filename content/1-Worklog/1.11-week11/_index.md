---
title: "Week 11 Worklog"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 (29/06 — 05/07/2026): Single-Table, Messaging & Payments

### Tasks carried out this week:

| # | Task | Status |
| --- | --- | --- |
| 1 | Migrated the data design to the **DynamoDB single-table model** (PK/SK + GSI1) | Completed |
| 2 | Built the asynchronous processing flow: **SQS + DLQ** for orders, **EventBridge** for payment events | Completed |
| 3 | Wrote the `shared-utils` package handling **idempotency**, with unit tests (Vitest); supported the notification service (Email/SMS) | Completed |
| 4 | Integrated **Stripe** payments and the payment-confirmation webhook | Completed |

### Week 11 Outcomes:

* The event-driven order pipeline is fully operational: order → SQS → processor → EventBridge → notification.
* Consumers are idempotent and covered by unit tests; payments work end-to-end with Stripe test mode.
