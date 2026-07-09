---
title: "Proposal"
date: 2026-07-09
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Project Proposal: Music Instrument Store — Serverless E-Commerce Platform

### 1. Overview

**Music Instrument Store** is a full-featured e-commerce platform for selling musical instruments (saxophones, guitars, pianos, ...), built by a 5-member team as the capstone project of the FCAJ internship. The entire backend runs on a **serverless, event-driven architecture** on AWS, defined as code with **AWS CDK (TypeScript)**.

GitHub repository: `https://github.com/Thien-132/music-instrument-store`

### 2. Problem Statement

Traditional e-commerce deployments require provisioning and operating servers 24/7, which wastes cost at low traffic and struggles at peak traffic. Our goals:

* **Pay-per-use:** near-zero cost when idle — critical for a student project with limited AWS credits.
* **Elastic:** automatically absorb traffic spikes (flash sales) without manual scaling.
* **Reliable:** no lost orders even when downstream services fail — achieved with queues, retries, DLQs, and idempotency.

### 3. Architecture

![Architecture Diagram](/images/2-Proposal/architecture.png)
<!-- TODO: chèn sơ đồ kiến trúc (vẽ bằng draw.io) -->

The system is organized into **4 CDK stacks**:

| Stack | Resources |
| --- | --- |
| **SecurityStack** | Secrets Manager (Stripe keys), GuardDuty, CloudTrail |
| **DatabaseStack** | DynamoDB single-table (PK/SK + GSI1/GSI2, PITR), S3 product-images bucket, AWS Backup vault & plans |
| **AuthStack** | Cognito User Pool (+ triggers), user groups (admin/staff/customer) |
| **BackendStack** | API Gateway REST API, 10+ Lambda services, SQS queues + DLQs, EventBridge bus & rules, CloudWatch alarms + SNS |

**Key flows:**

* **Order flow (async):** `POST /orders` → Order API → **SQS OrderQueue** → Order Processor → DynamoDB + **EventBridge** `OrderPlaced` → Notification queue → SES email.
* **Payment flow:** Checkout (Stripe) → Stripe webhook → Payment Webhook Lambda → EventBridge `PaymentSucceeded` → notification; atomic inventory reservation/release.
* **Marketing campaigns:** Campaign API → EventBridge `CampaignRequested` → Fan-out Lambda → **SQS CampaignQueue** → Campaign Sender (throttled with reserved concurrency).
* **AI Chatbot:** `/chat` → Lambda → Amazon Lex.

### 4. Team & Roles (5 members)

| Member | Role |
| --- | --- |
| Member 1 | Frontend (Next.js), UI/UX |
| Member 2 | Auth & API (Cognito, API Gateway) |
| **Member 3 (me — Nguyen Trung Hieu)** | **Data & Messaging: DynamoDB single-table design, S3, SQS/DLQ, EventBridge, AWS Backup, idempotency & tests** |
| Member 4 | Payments (Stripe/Momo), checkout services |
| Member 5 | Notifications (SES), chatbot (Lex), monitoring |

### 5. Timeline (12 weeks)

* **Weeks 1–6:** AWS foundations (FCJ curriculum).
* **Weeks 7–8:** Requirements, architecture blueprint, data & messaging design.
* **Weeks 9–12:** Implementation, integration, testing, documentation, and workshop.

### 6. Cost Estimation

All chosen services have generous free tiers or pure pay-per-use pricing (DynamoDB on-demand, Lambda, SQS, EventBridge). Estimated monthly cost for demo traffic: **under $5**, dominated by API Gateway and CloudWatch. Cleanup discipline (`cdk destroy` after demos) keeps the total within our remaining credits.
