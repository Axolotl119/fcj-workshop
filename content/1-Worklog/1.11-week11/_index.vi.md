---
title: "Nhật ký Tuần 11"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Tuần 11 (29/06 — 05/07/2026): Single-Table, Messaging & Thanh toán

### Công việc thực hiện trong tuần:

| # | Công việc | Trạng thái |
| --- | --- | --- |
| 1 | Chuyển thiết kế dữ liệu sang mô hình **DynamoDB single-table** (PK/SK + GSI1) | Hoàn thành |
| 2 | Xây dựng luồng xử lý bất đồng bộ: **SQS + DLQ** cho đơn hàng, **EventBridge** cho sự kiện thanh toán | Hoàn thành |
| 3 | Viết package `shared-utils` xử lý **idempotency** kèm unit test (Vitest); hỗ trợ service notification (Email/SMS) | Hoàn thành |
| 4 | Tích hợp thanh toán **Stripe** và webhook xác nhận thanh toán | Hoàn thành |

### Kết quả Tuần 11:

* Pipeline đơn hàng event-driven vận hành đầy đủ: order → SQS → processor → EventBridge → notification.
* Consumer idempotent và có unit test bảo vệ; thanh toán chạy end-to-end với Stripe test mode.
