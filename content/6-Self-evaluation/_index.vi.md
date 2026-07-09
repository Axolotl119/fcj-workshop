---
title: "Tự đánh giá"
date: 2026-07-09
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# Tự đánh giá

### 1. Kiến thức thu được

* **Dịch vụ AWS cốt lõi** qua thực hành: IAM, VPC, EC2, S3, RDS, DynamoDB, Lambda, API Gateway, SQS, SNS, EventBridge, Cognito, CloudWatch, X-Ray, CloudTrail, GuardDuty, AWS Backup.
* **Kiến trúc serverless & event-driven**: thiết kế và triển khai pipeline đơn hàng bất đồng bộ thật với queue, DLQ và sự kiện nghiệp vụ.
* **Mô hình hóa dữ liệu NoSQL**: DynamoDB single-table design với access pattern dựa trên GSI — một lối tư duy thực sự khác so với SQL.
* **Infrastructure as Code**: 4 CDK stack kiểu production bằng TypeScript, deploy qua CI/CD (GitHub Actions + OIDC).

### 2. Kỹ năng phát triển

* **Thói quen kỹ thuật độ tin cậy**: idempotency, retry, dead-letter queue, alarm — coi lỗi là trường hợp mặc định.
* **Ý thức chi phí**: làm việc dưới ngân sách credit cứng dạy tôi ước tính, giám sát và dọn tài nguyên cloud có kỷ luật.
* **Làm việc nhóm**: git workflow 5 người (feature branch, review PR, dev/staging/main), xử lý conflict, làm chủ một workstream rõ phạm vi.
* **Viết kỹ thuật**: tài liệu thiết kế, bài blog và workshop này.

### 3. Đóng góp cá nhân vào dự án nhóm

Với vai trò **Member 3 — Data & Messaging Engineer**, tôi phụ trách:

* Schema DynamoDB single-table (PK/SK + GSI1/GSI2) và cuộc di trú từ thiết kế RDS trước đó.
* Topology SQS/DLQ và rule EventBridge cho luồng đơn hàng, thanh toán, campaign.
* Kế hoạch AWS Backup + PITR cho tầng dữ liệu.
* Package idempotency `shared-utils` (claimEvent/releaseEvent) kèm unit test vitest.

### 4. Điều cần cải thiện

* Viết test hạ tầng (CDK assertions) sớm hơn thay vì chỉ unit test logic nghiệp vụ.
* Dựng load test nhỏ để kiểm chứng pipeline dưới traffic thực tế.
* Ước lượng thời gian tốt hơn — các vòng lặp thiết kế data model tốn hơn dự kiến.

### 5. Tổng kết

Kỳ thực tập FCAJ biến kiến thức cloud của tôi từ lý thuyết thành một hệ thống kiểu production chạy được, có giám sát, có kiểm soát chi phí. Tôi tự tin thiết kế kiến trúc serverless và bảo vệ các trade-off đằng sau nó.
