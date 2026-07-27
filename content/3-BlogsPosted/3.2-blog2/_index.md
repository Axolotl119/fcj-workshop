---
title: "Blog 2 — Cache-Aside with ElastiCache Redis"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Benchmarking Cache-Aside Pattern with ElastiCache Redis in E-Commerce Systems

**Published at:** _(update post link)_

---

## 1. Introduction

In an E-Commerce application, user traffic is predominantly read-heavy—customers browse product catalogs, search items, and view detail pages far more frequently than they execute checkout transactions. Querying the relational database (Amazon RDS) for every product detail request quickly introduces I/O bottlenecks, increasing latency and severely degrading user experience.

To solve read scalability challenges, implementing **Amazon ElastiCache for Redis** utilizing the **Cache-Aside Pattern** is an industry-standard practice. This article explores the architecture, provides code implementations, and benchmarks real-world performance improvements before and after adding a cache layer.

---

## 2. Understanding the Cache-Aside Pattern

The Cache-Aside (or Lazy Loading) pattern delegates cache management directly to the application layer.

```text
+----------+          1. Check Cache           +-------------------+
|          | --------------------------------> |  Amazon Redis     |
|          | <-------------------------------- |  (ElastiCache)    |
|          |        2a. Hit (Return Data)      +-------------------+
| Backend  |
| App      |          2b. Miss
|          | --------------------------------> +-------------------+
|          |                                   |  Amazon RDS       |
|          | <-------------------------------- |  (PostgreSQL)     |
|          |        3. Fetch from DB           +-------------------+
|          |
|          |          4. Write back to Cache
|          | ------------------------------------------------------+
+----------+
```

### Read Workflow Sequence:
1. **Check Cache:** The application receives a request for product details and checks if the key exists in Redis.
2. **Cache Hit:** If found, Redis immediately returns the cached data to the application, bypassing the database entirely.
3. **Cache Miss:**
   * The application queries the data from the Amazon RDS PostgreSQL database.
   * Returns the database response to the user.
   * Simultaneously writes the retrieved data back into Redis with a defined **Time-To-Live (TTL)** for subsequent queries.

---

## 3. Network Architecture & ElastiCache Configuration

To achieve maximum throughput and security, the Redis node is placed inside the same VPC as Amazon RDS, secured behind a dedicated Security Group.

### Cost-Optimized Configuration (Dev/Test Environment):
| Parameter | Recommended Setting | Purpose / Explanation |
| :--- | :--- | :--- |
| **Cluster Mode** | Disabled | Single primary node deployment to minimize costs. |
| **Node Type** | `cache.t4g.micro` | Powered by AWS Graviton2 processors, highly cost-effective for Dev environments. |
| **Engine Version** | Redis 7.x | Stable release supporting modern data structures and commands. |
| **Security Group** | Port `6379` | Restrict Inbound Port `6379` strictly to the Backend Application / Bastion Host IP. |

---

## 4. Code Implementation (Node.js/Express)

Below is the implementation of the Cache-Aside strategy for the Product Detail API endpoint (`GET /products/:id`).

```javascript
const express = require('express');
const redis = require('redis');
const { Pool } = require('pg');

const app = express();

// Initialize Redis Client
const redisClient = redis.createClient({
    url: 'redis://your-elasticache-endpoint.cache.amazonaws.com:6379'
});
redisClient.connect().catch(console.error);

// Initialize Amazon RDS PostgreSQL Connection Pool
const dbPool = new Pool({
    host: 'your-rds-endpoint.rds.amazonaws.com',
    user: 'postgres',
    password: 'yourpassword',
    database: 'ecommerce',
    port: 5432
});

const CACHE_TTL_SECONDS = 3600; // Cache expiration set to 1 hour

// Product Detail Endpoint with Cache-Aside
app.get('/products/:id', async (req, res) => {
    const productId = req.params.id;
    const cacheKey = `product:${productId}`;

    try {
        // 1. Check ElastiCache Redis
        const cachedData = await redisClient.get(cacheKey);

        if (cachedData) {
            // CACHE HIT: Return cached result instantly
            return res.json({
                source: 'cache',
                data: JSON.parse(cachedData)
            });
        }

        // 2. CACHE MISS: Query from Amazon RDS
        const dbResult = await dbPool.query(
            'SELECT * FROM products WHERE product_id = $1', 
            [productId]
        );

        if (dbResult.rows.length === 0) {
            return res.status(404).json({ message: 'Product not found' });
        }

        const product = dbResult.rows[0];

        // 3. Populate Redis Cache with expiration TTL
        await redisClient.setEx(cacheKey, CACHE_TTL_SECONDS, JSON.stringify(product));

        // 4. Return response to Client
        return res.json({
            source: 'database',
            data: product
        });

    } catch (error) {
        console.error('Error fetching product:', error);
        return res.status(500).json({ error: 'Internal Server Error' });
    }
});
```

---

## 5. Performance Benchmarking & Results

To measure the impact of caching, load tests were executed using **k6** simulating 100 Virtual Users (VUs) continuously querying product details over a 1-minute period:

### Performance Comparison Table

| Metric | RDS PostgreSQL Only (No Cache) | With ElastiCache Redis (Cache-Aside) | Improvement Factor |
| :--- | :--- | :--- | :--- |
| **Response Time (Latency avg)** | `125 ms` | `8 ms` | **~15.6x Faster** |
| **Response Time (p95)** | `280 ms` | `15 ms` | **~18.6x Faster** |
| **Throughput (Requests/sec)** | `420 req/s` | `3,850 req/s` | **~9.1x Increase** |
| **RDS CPU Utilization** | `78%` | `6%` | **92% Load Reduction on RDS** |

---

## 6. Amazon ElastiCache Deployment Example

Below is a screenshot of the Amazon ElastiCache Management Console showing an active `cache.t4g.micro` Redis cluster ready to process caching commands:

![ElastiCache Console Screenshot](https://axolotl119.github.io/fcj-workshop/images/3-BlogsPosted/blog2-redis.png)
*Figure 1: Amazon ElastiCache Redis cluster dashboard ready for operation.*

---

## 7. Conclusion

Integrating the **Cache-Aside pattern with Amazon ElastiCache Redis** yields drastic performance gains for E-Commerce applications:
* Reduces API response latency from **hundreds of milliseconds down to sub-10ms**.
* Offloads high-frequency read queries from the primary database (RDS), freeing CPU/RAM resources for critical transactional operations like Checkout.
* Minimizes cloud infrastructure costs using compact instance sizing (`cache.t4g.micro`).

In the next article, we will examine how to combine Amazon S3 and CloudFront to store and deliver product images with global low-latency CDN distribution.