---
title: "Tổng quan Workshop"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Tổng quan Workshop

### 1. Chúng ta sẽ xây gì

Backend của Music Instrument Store là kiến trúc **serverless, hướng sự kiện**. Không có server nào phải vá lỗi hay scale: mọi thành phần đều trả tiền theo mức dùng.

![Sơ đồ kiến trúc](images/2-Proposal/architecture.png)
<!-- TODO: chèn sơ đồ kiến trúc tổng thể -->

**Luồng đơn hàng (trọng tâm của workshop):**

```
Client ──POST /orders──> API Gateway ──> Order API Lambda
                                              │  SendMessage
                                              ▼
                                    SQS OrderQueue ──(lỗi 3 lần)──> Order DLQ ──> CloudWatch Alarm ──> email SNS
                                              │  batch 10
                                              ▼
                                  Order Processing Lambda ──ghi──> DynamoDB (single-table)
                                              │  PutEvents
                                              ▼
                                   EventBridge MusicStoreEventBus
                                              │  rule: OrderPlaced/OrderUpdated/OrderCancelled
                                              ▼
                                    SQS NotificationQueue ──> Notification Lambda ──> Amazon SES
```

### 2. Vì sao thiết kế như vậy?

* **API phản hồi tức thì** — Order API chỉ đẩy message vào queue; việc nặng xử lý bất đồng bộ.
* **Không mất đơn hàng** — processor crash thì SQS gửi lại; lỗi 3 lần thì message được giữ trong **DLQ** để điều tra thay vì biến mất.
* **Ghép nối lỏng** — consumer mới (email, analytics, campaign) chỉ cần đăng ký sự kiện **EventBridge**, không đụng vào code cũ.
* **Hiệu ứng đúng-một-lần** — consumer "claim" bản ghi `EVENT#<id>` bằng conditional write DynamoDB, nên message bị retry không gửi email trùng.

### 3. Dịch vụ AWS sử dụng

| Dịch vụ | Vai trò |
| --- | --- |
| DynamoDB | Kho dữ liệu single-table (sản phẩm, đơn hàng, người dùng, đánh giá, coupon...) |
| S3 | Lưu ảnh sản phẩm |
| SQS | Queue Order / Notification / Campaign + DLQ |
| EventBridge | Event bus tùy chỉnh cho sự kiện nghiệp vụ |
| Lambda | Toàn bộ compute (Node.js 20) |
| API Gateway | Cửa ngõ REST API |
| AWS Backup | Backup DynamoDB định kỳ ngày & tuần |
| CloudWatch + SNS | Cảnh báo DLQ và lỗi |
| CDK (TypeScript) | Infrastructure as Code |

### 4. Thời lượng & chi phí

* **Thời lượng:** 2–3 giờ.
* **Chi phí:** mọi thứ nằm trong Free Tier / giá on-demand; chạy trọn workshop tốn **dưới 1$** nếu dọn dẹp sau khi xong (xem module Dọn dẹp).
