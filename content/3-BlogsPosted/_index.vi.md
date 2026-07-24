---
title: "Các bài blog đã đăng"
date: 2026-07-09
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Trong thời gian thực tập, tôi đã viết các bài blog kỹ thuật chia sẻ những kiến thức tích lũy được khi xây dựng tầng dữ liệu (data layer) cho dự án Cửa hàng Nhạc cụ (Music Instrument Store).

### [Blog 1 — Xây dựng tầng dữ liệu quan hệ trên Amazon RDS cho hệ thống Thương mại điện tử](3.1-blog1/)

Tổng quan về cách mô hình hóa danh mục sản phẩm (product catalog) và các bảng đơn hàng trên Amazon RDS, bao gồm các quyết định thiết kế schema, chiến lược đánh chỉ mục (indexing), cùng những bài học kinh nghiệm về quản lý connection pooling cho backend serverless.

### [Blog 2 — Đo lường hiệu năng của mô hình Cache-Aside với ElastiCache Redis](3.2-blog2/)

Bài đánh giá hiệu năng (benchmark) thực tế so sánh độ trễ đọc (read latency) khi có và không có lớp cache-aside chạy trên ElastiCache Redis, đề cập đến chiến lược vô hiệu hóa cache (cache invalidation) và các đánh đổi rút ra từ quá trình kiểm thử tải (load testing).

### [Blog 3 — Thiết kế Single-Table trên DynamoDB & Xử lý tin nhắn tin cậy với SQS và EventBridge](3.3-blog3/)

Bài hướng dẫn từng bước về thiết kế bảng đơn (single-table design) trên DynamoDB áp dụng cho dữ liệu đơn hàng và danh mục, kết hợp với mô hình SQS + Dead-Letter Queue và định tuyến EventBridge nhằm đảm bảo quá trình xử lý đơn hàng diễn ra tin cậy ngay cả khi gặp sự cố.