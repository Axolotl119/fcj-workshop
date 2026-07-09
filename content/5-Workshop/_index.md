---
title: "Workshop"
date: 2026-07-09
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building a Serverless Event-Driven E-Commerce Data & Messaging Layer with AWS CDK

Welcome to this hands-on workshop! You will build the **data and messaging backbone** of a real e-commerce platform — the *Music Instrument Store* — using **AWS CDK (TypeScript)**.

By the end of this workshop you will have deployed:

* A **DynamoDB single-table** with GSIs, Point-in-Time Recovery, and scheduled **AWS Backup** plans.
* An **S3 bucket** for product images.
* An asynchronous **order pipeline**: API → **SQS queue** → processor Lambda → DynamoDB, protected by **Dead Letter Queues** and **idempotency**.
* A custom **EventBridge bus** routing domain events (`OrderPlaced`, `PaymentSucceeded`, `CampaignRequested`) to consumers.
* **CloudWatch alarms** notifying you via SNS when messages land in a DLQ.

#### Workshop Contents

1. [Workshop Overview](5.1-overview/)
2. [Prerequisites & Setup](5.2-prerequisites/)
3. [Module 1: Data Layer — DynamoDB, S3 & AWS Backup](5.3-module1-data/)
4. [Module 2: Messaging — SQS Queues & DLQs](5.4-module2-messaging/)
5. [Module 3: Event Routing — EventBridge](5.5-module3-events/)
6. [Module 4: API & Compute — Lambda, API Gateway, Monitoring](5.6-module4-api/)
7. [Deployment & Testing](5.7-deployment-testing/)
8. [Clean up](5.8-cleanup/)
