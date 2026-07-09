---
title: "Blog 1 — Relational Data Layer on Amazon RDS"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Building a Relational Data Layer on Amazon RDS for E-Commerce

<!-- TODO: dán link bài đăng thật vào đây -->
**Published at:** _(update link)_

### Summary

This post walks through standing up a cost-optimized **PostgreSQL on Amazon RDS** (db.t4g.micro, Single-AZ, gp2 20GB) for an e-commerce workload:

* Designing an ACID schema for `users`, `orders`, and `payments`, and why relational transactions fit checkout flows.
* Networking decisions: placing the DB in a VPC with public access for development, security groups, and enabling DNS hostnames.
* Implementing registration/login with bcrypt password hashing and checkout as a single SQL transaction.
* Cost tips for students: stopping the instance when idle, choosing Single-AZ and burstable Graviton instances.

![RDS console screenshot](/images/3-Blog/blog1-rds.png)
<!-- TODO: chèn ảnh chụp màn hình RDS console -->
