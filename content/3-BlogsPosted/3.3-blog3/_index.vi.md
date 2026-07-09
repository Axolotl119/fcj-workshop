---
title: "Blog 3 — DynamoDB Single-Table & Messaging tin cậy"
date: 2026-07-05
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# DynamoDB Single-Table Design & Messaging tin cậy với SQS và EventBridge

<!-- TODO: dán link bài đăng thật vào đây -->
**Đăng tại:** _(cập nhật link)_

### Tóm tắt

Câu chuyện chuyển backend e-commerce từ thiết kế quan hệ sang **DynamoDB single-table design** kèm pipeline hướng sự kiện:

* Mô hình hóa products, orders, users, ratings, wishlist, coupons trong một bảng với `PK`/`SK` và GSI.
* Tách rời xử lý đơn hàng bằng **SQS + DLQ** (maxReceiveCount = 3) để không bao giờ mất đơn.
* Định tuyến sự kiện nghiệp vụ (`OrderPlaced`, `PaymentSucceeded`, `CampaignRequested`) qua rule **EventBridge**.
* Làm consumer **idempotent** bằng bản ghi claim `EVENT#<id>` với conditional write — và unit test bằng vitest + aws-sdk-client-mock.

![Pipeline hướng sự kiện](/images/3-Blog/blog3-pipeline.png)
<!-- TODO: chèn sơ đồ pipeline -->
