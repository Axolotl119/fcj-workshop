---
title: "Blog 3 — DynamoDB Single-Table & Reliable Messaging"
date: 2026-07-05
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# DynamoDB Single-Table Design & Reliable Messaging with SQS and EventBridge

<!-- TODO: dán link bài đăng thật vào đây -->
**Published at:** _(update link)_

### Summary

The story of migrating our e-commerce backend from a relational design to a **DynamoDB single-table design** with an event-driven pipeline:

* Modeling products, orders, users, ratings, wishlist, and coupons in one table with `PK`/`SK` and GSIs.
* Decoupling order processing with **SQS + DLQ** (maxReceiveCount = 3) so no order is ever lost.
* Routing domain events (`OrderPlaced`, `PaymentSucceeded`, `CampaignRequested`) through **EventBridge** rules.
* Making consumers **idempotent** with conditional-write `EVENT#<id>` claim records — and unit-testing it with vitest + aws-sdk-client-mock.

![Event-driven pipeline](/images/3-Blog/blog3-pipeline.png)
<!-- TODO: chèn sơ đồ pipeline -->
