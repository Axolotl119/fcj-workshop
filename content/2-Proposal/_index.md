---
title: "Proposal"
date: 2026-07-09
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# SYSTEM ARCHITECTURE PROPOSAL

## PROJECT: MUSIC INSTRUMENT STORE — AWS SERVERLESS E-COMMERCE PLATFORM

---

### 1. INTRODUCTION & PROBLEM STATEMENT

#### 1.1. Cloud-native and Event-Driven Architecture Trends

Entering 2026, e-commerce platforms are shifting away from monolithic, always-on server deployments toward **serverless, event-driven architectures**. This shift is driven by the need to handle unpredictable traffic (flash sales, seasonal promotions) without over-provisioning infrastructure, while keeping operational overhead and cost to a minimum — especially important for a student capstone project running on limited AWS credits.

By decoupling the frontend from backend services, and decoupling backend services from each other through queues and event buses, the system can process each request independently, retry failures safely, and remain available even when a downstream component is temporarily degraded.

#### 1.2. Pain Points of Traditional Architectures

Traditional e-commerce systems deployed on always-on servers or tightly-coupled monoliths commonly suffer from:

- **High idle cost:** servers must be provisioned for peak load, wasting money during low-traffic hours.
- **Poor scalability under spikes:** flash sales or marketing pushes can overwhelm fixed-capacity servers and databases.
- **Fragile checkout flows:** a slow or failing notification/inventory service can block the entire order flow, causing lost sales.
- **Duplicate charges:** unreliable networks and repeated clicks on "Pay" can trigger double payments without idempotency controls.
- **Weak security posture:** manually managed infrastructure is more exposed to bot traffic, credential leakage, and unmonitored API abuse.

---

### 2. PROJECT OBJECTIVES

**Music Instrument Store** is a full-featured e-commerce platform for musical instruments (guitars, pianos, saxophones, …), built by a 5-member team as the FCAJ internship capstone project. The entire backend is **serverless and event-driven**, defined as code with **AWS CDK (TypeScript)**.

GitHub repository: `https://github.com/Thien-132/music-instrument-store`

Project goals:

- **Pay-per-use:** near-zero cost when idle, since every core service (Lambda, DynamoDB on-demand, SQS, EventBridge) bills only for actual usage.
- **Elastic:** absorb traffic spikes automatically without manual scaling.
- **Reliable:** no lost orders or notifications, even when a downstream service fails — enforced with queues, dead-letter queues (DLQ), retries, and idempotency keys.
- **Secure by design:** authentication via Cognito, edge protection via WAF, and continuous monitoring via CloudWatch, X-Ray, GuardDuty, and CloudTrail.
- **AI-assisted shopping:** a chatbot (Amazon Lex) to help customers find products and answer questions.

---

### 3. PROPOSED SYSTEM ARCHITECTURE

#### 3.1. Architecture Diagram

![Architecture Diagram](https://axolotl119.github.io/fcj-workshop/images/2-Proposal/sơ dồ kiến trúc.png)

The system is organized into logical groups of services within a single AWS Region:

| Group                  | Services                                                                                   |
| ---------------------- | ------------------------------------------------------------------------------------------- |
| **Edge & Delivery**     | Route 53, AWS WAF, AWS Amplify (frontend hosting)                                          |
| **Identity**            | Amazon Cognito                                                                              |
| **API Layer**           | Amazon API Gateway                                                                          |
| **Catalog**             | Product Service (Lambda), DynamoDB (Catalog Table), S3 (Product Assets), AWS Backup        |
| **Orders**              | Order Service (Lambda), SQS Order Queue + Order DLQ                                        |
| **Checkout & Payments** | Checkout Service (Lambda), Stripe                                                           |
| **Eventing & Notify**   | EventBridge, SQS Notification Queue + Notification DLQ                                     |
| **AI Chatbot**          | Chatbot Backend (Lambda), Amazon Lex                                                        |
| **Security & Monitoring** | CloudWatch, AWS X-Ray, Amazon GuardDuty, AWS CloudTrail                                   |

#### 3.2. Detailed Data Flow

The numbers below correspond to the flow numbers in the architecture diagram:

1. **User Access:** The user's browser resolves the site domain through **Amazon Route 53**.
2. **Edge Protection:** Route 53 forwards traffic through **AWS WAF**, which filters malicious requests (rate limiting, managed rule groups) before they reach the application.
3. **Static/Frontend Delivery:** Route 53 routes legitimate traffic to **AWS Amplify**, which serves the frontend application.
4. **API Entry:** The frontend calls the backend through **Amazon API Gateway**, the single entry point for all REST APIs.
5. **Authentication:** API Gateway validates the caller's identity by checking the JWT issued by **Amazon Cognito** before routing the request to any backend Lambda.
6. **Catalog & Order Placement:** API Gateway routes catalog requests to the **Product Service**, which reads/writes product data in **DynamoDB (Catalog Table)** — backed up automatically via **AWS Backup** — and stores product images in **S3 (Product Assets)**. Order requests go to the **Order Service**, which places a message on the **SQS Order Queue**; messages that repeatedly fail processing are moved to the **Order DLQ** for investigation.
7. **Checkout & Payment:** API Gateway routes checkout requests to the **Checkout Service**, which calls **Stripe** to process payment. Stripe's response updates the order status back through the Order Service.
8. **Event Publishing:** Once checkout completes, the **Checkout Service** publishes an event to **Amazon EventBridge**.
9. **Async Notification:** API Gateway/Order flow also publishes status-change events to **EventBridge**, which routes them to the **SQS Notification Queue** for delivery; failed notification deliveries land in the **Notification DLQ**.

Separately, the **Chatbot Backend** (Lambda) integrates with **Amazon Lex** to power the AI shopping assistant, and the entire system is continuously observed by the **Security & Monitoring** group — **CloudWatch** (metrics/logs), **AWS X-Ray** (distributed tracing), **Amazon GuardDuty** (threat detection), and **AWS CloudTrail** (API audit logging).

#### 3.3. Security Control Points

- **Edge protection:** AWS WAF filters malicious traffic before it reaches Amplify or API Gateway.
- **Authentication & authorization:** Amazon Cognito issues JWTs; API Gateway rejects unauthenticated requests.
- **Reliability controls:** SQS + DLQ pattern on both the Order and Notification flows ensures no message is silently lost.
- **Data durability:** AWS Backup provides automated, scheduled backups of the DynamoDB catalog table.
- **Observability:** CloudWatch, X-Ray, GuardDuty, and CloudTrail together give full visibility into performance, traces, threats, and API activity.

---

### 4. TEAM & ROLES (5 members)

| Member                                | Role                                       | Key Responsibilities                                                                                     |
| -------------------------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Member 1                               | Frontend Engineer                          | Next.js UI, Amplify deployment, cart/checkout/chatbot UI components.                                     |
| Member 2                               | Auth & API Engineer                        | Cognito setup, API Gateway configuration, Product & Order Service Lambdas.                                |
| **Member 3 (me — Nguyen Trung Hieu)**  | **Data & Messaging Engineer**              | **DynamoDB single-table design, S3 product assets, SQS Order Queue/DLQ, EventBridge routing, AWS Backup, idempotency & tests.** |
| Member 4                               | Payments & Checkout Engineer                | Checkout Service, Stripe integration, webhook signature verification, idempotency keys.                   |
| Member 5                               | Notifications, AI & Monitoring Engineer     | Notification Service, Amazon Lex chatbot, CloudWatch/X-Ray/GuardDuty/CloudTrail setup.                     |

---

### 5. TIMELINE (12 weeks)

- **Weeks 1–6:** AWS foundations (FCJ curriculum).
- **Weeks 7–8:** Requirements gathering, architecture blueprint, data & messaging design.
- **Weeks 9–12:** Implementation, integration, testing, documentation, and final workshop.

---

### 6. COST ESTIMATION

All chosen services rely on generous free tiers or pure pay-per-use pricing (DynamoDB on-demand, Lambda, SQS, EventBridge, Cognito). Estimated monthly cost for demo-level traffic is **under $5/month**, mainly driven by API Gateway requests and CloudWatch logging. Strict cleanup discipline (`cdk destroy` after each demo) keeps total spend within our remaining credits.

---

### 7. RISK ASSESSMENT & MITIGATION

| Risk                                          | Impact | Mitigation                                                                                     |
| ---------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------ |
| Traffic spikes overwhelming backend            | Medium | Serverless auto-scaling (Lambda, DynamoDB on-demand, SQS buffering).                            |
| Lost or duplicated orders/payments             | High   | SQS + DLQ pattern, Stripe idempotency keys, DynamoDB conditional writes.                         |
| Credential leakage                             | High   | Secrets stored in AWS Secrets Manager, never committed to Git; least-privilege IAM roles.        |
| Schedule delay due to architecture complexity  | High   | Clear MVP scope (catalog, checkout, chatbot); short weekly sync meetings to unblock issues.       |