---
title: "Blog 2 — Cache-Aside với ElastiCache Redis"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Đo hiệu năng Cache-Aside với ElastiCache Redis

<!-- TODO: dán link bài đăng thật vào đây -->
**Đăng tại:** _(cập nhật link)_

### Tóm tắt

Bài viết minh họa **mẫu cache-aside** đặt trước CSDL quan hệ bằng **Amazon ElastiCache (Redis)** và định lượng mức tăng tốc:

* Cách cache-aside hoạt động: đọc Redis trước, miss thì đọc DB rồi ghi ngược vào cache kèm TTL.
* Phương pháp benchmark: đo latency p50/p95 cho truy vấn sản phẩm hot khi có và không có cache.
* Kết quả và phân tích: khi nào cache đáng tiền trong catalog e-commerce, và các bẫy khi invalidate cache.
* Cân nhắc chi phí: chọn `cache.t4g.micro` và tắt cluster ngoài giờ làm việc.

![Biểu đồ benchmark](/images/3-Blog/blog2-benchmark.png)
<!-- TODO: chèn biểu đồ đo tốc độ -->
