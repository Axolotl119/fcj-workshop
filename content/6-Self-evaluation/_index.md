---
title: "Self-evaluation"
date: 2026-07-09
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# Self-evaluation

### 1. Knowledge gained

* **AWS core services** hands-on: IAM, VPC, EC2, S3, RDS, DynamoDB, Lambda, API Gateway, SQS, SNS, EventBridge, Cognito, CloudWatch, X-Ray, CloudTrail, GuardDuty, AWS Backup.
* **Serverless & event-driven architecture**: designed and implemented a real asynchronous order pipeline with queues, DLQs, and domain events.
* **NoSQL data modeling**: DynamoDB single-table design with GSI-based access patterns — a genuinely different way of thinking compared to SQL.
* **Infrastructure as Code**: 4 production-style CDK stacks in TypeScript, deployed through CI/CD (GitHub Actions + OIDC).

### 2. Skills developed

* **Reliability engineering habits**: idempotency, retries, dead-letter queues, alarms — thinking about failure as the default case.
* **Cost awareness**: working under a hard credit budget taught me to estimate, monitor, and clean up cloud resources with discipline.
* **Teamwork**: 5-member git workflow (feature branches, PR reviews, dev/staging/main), resolving conflicts, and owning a clearly-scoped workstream.
* **Technical writing**: design documents, blog posts, and this workshop.

### 3. Personal contribution to the team project

As **Member 3 — Data & Messaging Engineer** I owned:

* The DynamoDB single-table schema (PK/SK + GSI1/GSI2) and its migration from the earlier RDS design.
* The SQS/DLQ topology and EventBridge rules for order, payment, and campaign flows.
* AWS Backup plans + PITR for the data layer.
* The `shared-utils` idempotency package (claimEvent/releaseEvent) with vitest unit tests.

### 4. What I would improve

* Write infrastructure tests (CDK assertions) earlier instead of only unit-testing business logic.
* Set up a small load test to validate the pipeline under realistic traffic.
* Improve my time estimation — data modeling iterations took longer than planned.

### 5. Overall

The FCAJ internship transformed my cloud knowledge from theory into a working, monitored, cost-controlled production-style system. I now feel confident designing serverless architectures and defending the trade-offs behind them.
