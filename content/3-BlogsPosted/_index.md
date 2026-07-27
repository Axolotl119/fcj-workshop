---
title: "Blogs Posted"
date: 2026-07-09
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

During the internship I wrote technical blog posts sharing what I learned while building the data layer of the Music Instrument Store project.

### [Blog 1 — Building a Relational Data Layer on Amazon RDS for E-Commerce](3.1-blog1/)

An overview of how the product catalog and order tables were modeled on Amazon RDS, including schema design decisions, indexing strategy, and lessons learned around connection pooling for a serverless backend.

### [Blog 2 — Measuring Cache-Aside Performance with ElastiCache Redis](3.2-blog2/)

A hands-on benchmark comparing read latency with and without a cache-aside layer backed by ElastiCache Redis, covering cache invalidation strategy and the trade-offs discovered during load testing.

### [Blog 3 — DynamoDB Single-Table Design & Reliable Messaging with SQS and EventBridge](3.3-blog3/)

A walkthrough of the single-table DynamoDB design used for the order/catalog data, paired with an SQS + dead-letter-queue pattern and EventBridge routing to keep order processing reliable under failure.
