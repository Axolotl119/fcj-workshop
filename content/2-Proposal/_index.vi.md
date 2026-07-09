---
title: "Đề xuất dự án"
date: 2026-07-09
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Đề xuất dự án: Music Instrument Store — Nền tảng E-Commerce Serverless

### 1. Tổng quan

**Music Instrument Store** là nền tảng thương mại điện tử bán nhạc cụ (saxophone, guitar, piano, ...) do nhóm 5 thành viên xây dựng trong dự án tốt nghiệp của kỳ thực tập FCAJ. Toàn bộ backend chạy trên kiến trúc **serverless, hướng sự kiện (event-driven)** trên AWS, định nghĩa hạ tầng bằng code với **AWS CDK (TypeScript)**.

GitHub repository: `https://github.com/Thien-132/music-instrument-store`

### 2. Bài toán

Triển khai e-commerce truyền thống đòi hỏi vận hành server 24/7 — lãng phí khi vắng khách và quá tải khi cao điểm. Mục tiêu của nhóm:

* **Trả tiền theo mức dùng:** chi phí gần như bằng 0 khi không có traffic — thiết yếu với dự án sinh viên có credit AWS hạn chế.
* **Co giãn:** tự hấp thụ đột biến traffic (flash sale) mà không cần scale thủ công.
* **Tin cậy:** không mất đơn hàng kể cả khi dịch vụ phía sau lỗi — nhờ queue, retry, DLQ và idempotency.

### 3. Kiến trúc

![Sơ đồ kiến trúc](/images/2-Proposal/architecture.png)
<!-- TODO: chèn sơ đồ kiến trúc (vẽ bằng draw.io) -->

Hệ thống chia thành **4 CDK stack**:

| Stack | Tài nguyên |
| --- | --- |
| **SecurityStack** | Secrets Manager (khóa Stripe), GuardDuty, CloudTrail |
| **DatabaseStack** | DynamoDB single-table (PK/SK + GSI1/GSI2, PITR), S3 bucket ảnh sản phẩm, AWS Backup vault & plan |
| **AuthStack** | Cognito User Pool (+ triggers), nhóm người dùng (admin/staff/customer) |
| **BackendStack** | API Gateway REST API, 10+ Lambda service, SQS queue + DLQ, EventBridge bus & rule, CloudWatch alarm + SNS |

**Các luồng chính:**

* **Luồng đơn hàng (bất đồng bộ):** `POST /orders` → Order API → **SQS OrderQueue** → Order Processor → DynamoDB + **EventBridge** `OrderPlaced` → Notification queue → email SES.
* **Luồng thanh toán:** Checkout (Stripe) → Stripe webhook → Payment Webhook Lambda → EventBridge `PaymentSucceeded` → thông báo; giữ/nhả kho nguyên tử.
* **Chiến dịch marketing:** Campaign API → EventBridge `CampaignRequested` → Lambda fan-out → **SQS CampaignQueue** → Campaign Sender (giới hạn reserved concurrency).
* **Chatbot AI:** `/chat` → Lambda → Amazon Lex.

### 4. Nhóm & Vai trò (5 thành viên)

| Thành viên | Vai trò |
| --- | --- |
| Member 1 | Frontend (Next.js), UI/UX |
| Member 2 | Auth & API (Cognito, API Gateway) |
| **Member 3 (tôi — Nguyễn Trung Hiếu)** | **Data & Messaging: thiết kế DynamoDB single-table, S3, SQS/DLQ, EventBridge, AWS Backup, idempotency & test** |
| Member 4 | Thanh toán (Stripe/Momo), checkout service |
| Member 5 | Thông báo (SES), chatbot (Lex), giám sát |

### 5. Tiến độ (12 tuần)

* **Tuần 1–6:** Nền tảng AWS (giáo trình FCJ).
* **Tuần 7–8:** Yêu cầu, blueprint kiến trúc, thiết kế data & messaging.
* **Tuần 9–12:** Triển khai, tích hợp, kiểm thử, tài liệu và workshop.

### 6. Ước tính chi phí

Các dịch vụ được chọn đều có free tier rộng hoặc tính tiền thuần theo mức dùng (DynamoDB on-demand, Lambda, SQS, EventBridge). Chi phí ước tính hằng tháng cho traffic demo: **dưới 5$**, chủ yếu từ API Gateway và CloudWatch. Kỷ luật dọn tài nguyên (`cdk destroy` sau demo) giúp tổng chi phí nằm trong số credit còn lại.
