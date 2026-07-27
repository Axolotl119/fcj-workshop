---
title: "Workshop Overview"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Workshop Overview

### 1. What we are building

The Music Instrument Store backend is a **serverless, event-driven architecture**. There are no servers to patch or scale: every component is pay-per-use.

![Architecture Diagram](images/2-Proposal/architecture.png)
<!-- TODO: chèn sơ đồ kiến trúc tổng thể -->

**Order flow (the core of this workshop):**

```
Client ──POST /orders──> API Gateway ──> Order API Lambda
                                              │  SendMessage
                                              ▼
                                    SQS OrderQueue ──(3 failures)──> Order DLQ ──> CloudWatch Alarm ──> SNS email
                                              │  batch 10
                                              ▼
                                  Order Processing Lambda ──write──> DynamoDB (single-table)
                                              │  PutEvents
                                              ▼
                                   EventBridge MusicStoreEventBus
                                              │  rule: OrderPlaced/OrderUpdated/OrderCancelled
                                              ▼
                                    SQS NotificationQueue ──> Notification Lambda ──> Amazon SES
```

### 2. Why this design?

* **API responds instantly** — the Order API only enqueues a message; heavy work happens asynchronously.
* **No lost orders** — if the processor crashes, SQS redelivers; after 3 failures the message is preserved in a **DLQ** for inspection instead of disappearing.
* **Loose coupling** — new consumers (email, analytics, campaigns) subscribe to **EventBridge** events without touching existing code.
* **Exactly-once effects** — consumers claim an `EVENT#<id>` record with a DynamoDB conditional write, so retried messages don't send duplicate emails.

### 3. AWS services used

| Service | Role |
| --- | --- |
| DynamoDB | Single-table datastore (products, orders, users, ratings, coupons...) |
| S3 | Product image storage |
| SQS | Order / Notification / Campaign queues + DLQs |
| EventBridge | Custom event bus for domain events |
| Lambda | All compute (Node.js 20) |
| API Gateway | REST API front door |
| AWS Backup | Daily & weekly scheduled backups of DynamoDB |
| CloudWatch + SNS | DLQ and error alarms |
| CDK (TypeScript) | Infrastructure as Code |

### 4. Duration & cost

* **Duration:** 2–3 hours.
* **Cost:** everything fits AWS Free Tier / on-demand pricing; a full run costs **well under $1** if you clean up afterwards (see the Clean up module).
