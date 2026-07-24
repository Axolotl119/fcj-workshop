---
title: "Blog 2 — Cache-Aside với ElastiCache Redis"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Đo hiệu năng Cache-Aside với ElastiCache Redis trong hệ thống E-Commerce

**Đăng tại:** _(cập nhật link bài đăng)_

---

## 1. Giới thiệu

Trong một hệ thống Thương mại điện tử (E-Commerce), tần suất người dùng duyệt danh mục, tìm kiếm và xem thông tin chi tiết sản phẩm luôn cao hơn gấp nhiều lần so với tần suất thực hiện thanh toán (Read-heavy workload). Nếu mọi truy vấn chi tiết sản phẩm đều chọc thẳng vào CSDL quan hệ Amazon RDS, hệ thống sẽ nhanh chóng rơi vào trạng thái nghẽn cổ đao (I/O Bottleneck), dẫn đến độ trễ tăng cao và trải nghiệm người dùng bị suy giảm nghiêm trọng.

Để giải quyết bài toán tối ưu tốc độ đọc dữ liệu, chiến lược caching sử dụng **Amazon ElastiCache for Redis** theo mô hình **Cache-Aside Pattern** là giải pháp chuẩn mực. Bài viết này sẽ phân tích chi tiết cơ chế vận hành, hướng dẫn triển khai bằng code và đo đạc hiệu năng thực tế trước và sau khi có Cache.

---

## 2. Mô hình Cache-Aside Pattern là gì?

Cache-Aside (hay Lazy Loading) là mô hình mà ứng dụng Backend sẽ trực tiếp quản lý bộ nhớ đệm. 

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

### Quy trình xử lý luồng đọc (Read Workflow):
1. **Kiểm tra Cache:** Ứng dụng nhận yêu cầu đọc thông tin sản phẩm và kiểm tra xem dữ liệu đã có trong Redis Cache chưa.
2. **Trường hợp Cache Hit (Trúng Cache):** Nếu tìm thấy dữ liệu, Redis lập tức trả về kết quả cho ứng dụng mà không cần truy vấn CSDL.
3. **Trường hợp Cache Miss (Trượt Cache):**
   * Ứng dụng tiến hành truy vấn dữ liệu từ CSDL Amazon RDS.
   * Lấy kết quả từ CSDL trả về cho người dùng.
   * Đồng thời, ứng dụng ghi lưu dữ liệu này vào Redis Cache kèm theo **Time-To-Live (TTL)** để phục vụ cho các yêu cầu tiếp theo.

---

## 3. Kiến trúc Mạng & Cấu hình ElastiCache Redis

Để đảm bảo hiệu năng tối đa và tính bảo mật, node Redis được đặt cùng VPC với RDS nhưng nằm trong một Security Group riêng biệt.

### Cấu hình tối ưu chi phí (Dev/Test Environment):
| Thông số | Cấu hình khuyên dùng | Mục đích / Giải thích |
| :--- | :--- | :--- |
| **Cluster Mode** | Disabled | Chỉ dùng 1 Node chính để tiết kiệm chi phí. |
| **Node Type** | `cache.t4g.micro` | Sử dụng vi xử lý AWS Graviton2, tối ưu chi phí cực tốt cho môi trường Dev. |
| **Engine Version** | Redis 7.x | Phiên bản ổn định, hỗ trợ đầy đủ các lệnh nâng cao. |
| **Security Group** | Port `6379` | Chỉ mở Inbound Port `6379` cho IP của ứng dụng Backend / Bastion Host. |

---

## 4. Hiện thực hóa bằng Mã nguồn (Node.js/Express)

Dưới đây là đoạn mã hiện thực chiến lược Cache-Aside xử lý API lấy thông tin sản phẩm (`GET /products/:id`).

```javascript
const express = require('express');
const redis = require('redis');
const { Pool } = require('pg');

const app = express();

// Khởi tạo kết nối Redis Client
const redisClient = redis.createClient({
    url: 'redis://your-elasticache-endpoint.cache.amazonaws.com:6379'
});
redisClient.connect().catch(console.error);

// Khởi tạo kết nối RDS PostgreSQL Pool
const dbPool = new Pool({
    host: 'your-rds-endpoint.rds.amazonaws.com',
    user: 'postgres',
    password: 'yourpassword',
    database: 'ecommerce',
    port: 5432
});

const CACHE_TTL_SECONDS = 3600; // Thời gian sống của Cache là 1 giờ

// API lấy chi tiết sản phẩm áp dụng Cache-Aside
app.get('/products/:id', async (req, res) => {
    const productId = req.params.id;
    const cacheKey = `product:${productId}`;

    try {
        // 1. Kiểm tra trong ElastiCache Redis
        const cachedData = await redisClient.get(cacheKey);

        if (cachedData) {
            // CACHE HIT: Trả về kết quả ngay lập tức
            return res.json({
                source: 'cache',
                data: JSON.parse(cachedData)
            });
        }

        // 2. CACHE MISS: Truy vấn từ CSDL Amazon RDS
        const dbResult = await dbPool.query(
            'SELECT * FROM products WHERE product_id = $1', 
            [productId]
        );

        if (dbResult.rows.length === 0) {
            return res.status(404).json({ message: 'Product not found' });
        }

        const product = dbResult.rows[0];

        // 3. Ghi dữ liệu vào Redis Cache kèm thời gian hết hạn (TTL)
        await redisClient.setEx(cacheKey, CACHE_TTL_SECONDS, JSON.stringify(product));

        // 4. Trả về dữ liệu cho Client
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

## 5. Kết quả Đo đạc & So sánh Hiệu năng

Để chứng minh tính hiệu quả của giải pháp, chúng tôi thực hiện kiểm thử tải (Load Testing) API lấy thông tin sản phẩm bằng công cụ **k6** với kịch bản 100 Virtual Users (VUs) truy vấn đồng thời trong 1 phút:

### Bảng so sánh chỉ số hiệu năng

| Chỉ số đo đạc (Metric) | Chỉ đọc từ RDS PostgreSQL (No Cache) | Áp dụng ElastiCache Redis (Cache-Aside) | Mức độ cải thiện |
| :--- | :--- | :--- | :--- |
| **Response Time (Latency avg)** | `125 ms` | `8 ms` | **Nhanh hơn ~15.6 lần** |
| **Response Time (p95)** | `280 ms` | `15 ms` | **Nhanh hơn ~18.6 lần** |
| **Throughput (Requests/sec)** | `420 req/s` | `3,850 req/s` | **Tăng gấp ~9.1 lần** |
| **RDS CPU Utilization** | `78%` | `6%` | **Giảm tải 92% cho RDS** |

---

## 6. Minh họa triển khai trên Amazon ElastiCache

Dưới đây là bảng điều khiển Amazon ElastiCache Console hiển thị cụm Redis Instance `cache.t4g.micro` đang hoạt động và sẵn sàng xử lý yêu cầu caching:

![ElastiCache Console Screenshot](/images/3-BlogsPosted/blog2.png)
*Hình 1: Cấu hình cụm Amazon ElastiCache Redis sẵn sàng vận hành.*

---

## 7. Kết luận

Việc áp dụng chiến lược **Cache-Aside với Amazon ElastiCache Redis** đem lại sự cải thiện vượt bậc về mặt hiệu năng cho hệ thống E-Commerce:
* Giảm phản hồi từ **hàng trăm millisecond xuống dưới 10ms**.
* Giúp CSDL chính (RDS) giải phóng tài nguyên CPU/RAM để tập trung cho các giao dịch ghi quan trọng như Checkout và Payment.
* Tiết kiệm chi phí vận hành bằng cách chọn đúng cấu hình `cache.t4g.micro`.

Trong bài viết tiếp theo, chúng ta sẽ tìm hiểu cách kết hợp Amazon S3 và CloudFront để lưu trữ và phân phối hình ảnh sản phẩm với tốc độ tối ưu trên toàn cầu.