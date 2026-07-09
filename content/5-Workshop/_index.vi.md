---
title: "Workshop"
date: 2026-07-09
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng tầng Data & Messaging Serverless hướng sự kiện cho E-Commerce với AWS CDK

Chào mừng đến với workshop thực hành! Bạn sẽ xây dựng **xương sống dữ liệu và messaging** của một nền tảng thương mại điện tử thật — *Music Instrument Store* — bằng **AWS CDK (TypeScript)**.

Kết thúc workshop, bạn sẽ deploy được:

* Một bảng **DynamoDB single-table** với GSI, Point-in-Time Recovery và kế hoạch **AWS Backup** định kỳ.
* Một **S3 bucket** chứa ảnh sản phẩm.
* **Pipeline đơn hàng bất đồng bộ**: API → **SQS queue** → Lambda xử lý → DynamoDB, bảo vệ bởi **Dead Letter Queue** và **idempotency**.
* Một **EventBridge bus** tùy chỉnh định tuyến sự kiện nghiệp vụ (`OrderPlaced`, `PaymentSucceeded`, `CampaignRequested`) tới consumer.
* **CloudWatch alarm** báo qua SNS khi có message rơi vào DLQ.

#### Nội dung Workshop

1. [Tổng quan Workshop](5.1-overview/)
2. [Chuẩn bị & Cài đặt](5.2-prerequisites/)
3. [Module 1: Tầng dữ liệu — DynamoDB, S3 & AWS Backup](5.3-module1-data/)
4. [Module 2: Messaging — SQS Queue & DLQ](5.4-module2-messaging/)
5. [Module 3: Định tuyến sự kiện — EventBridge](5.5-module3-events/)
6. [Module 4: API & Compute — Lambda, API Gateway, Giám sát](5.6-module4-api/)
7. [Triển khai & Kiểm thử](5.7-deployment-testing/)
8. [Dọn dẹp tài nguyên](5.8-cleanup/)
