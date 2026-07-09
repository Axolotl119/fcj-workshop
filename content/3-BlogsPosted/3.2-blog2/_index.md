---
title: "Blog 2 — Cache-Aside with ElastiCache Redis"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Measuring Cache-Aside Performance with ElastiCache Redis

<!-- TODO: dán link bài đăng thật vào đây -->
**Published at:** _(update link)_

### Summary

This post demonstrates the **cache-aside pattern** in front of a relational database using **Amazon ElastiCache (Redis)** and quantifies the speedup:

* How cache-aside works: read from Redis first, fall back to the database on miss, then populate the cache with a TTL.
* Benchmark methodology: measuring p50/p95 latency for hot product queries with and without the cache.
* Results and analysis: when caching pays off in an e-commerce catalog, and cache-invalidation pitfalls.
* Cost considerations: choosing `cache.t4g.micro` and turning the cluster off outside working sessions.

![Benchmark chart](/images/3-Blog/blog2-benchmark.png)
<!-- TODO: chèn biểu đồ đo tốc độ -->
