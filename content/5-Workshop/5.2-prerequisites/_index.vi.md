---
title: "Chuẩn bị & Cài đặt"
date: 2026-07-09
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Chuẩn bị & Cài đặt

### 1. Bạn cần gì

* Một **tài khoản AWS** có quyền administrator (tài khoản cá nhân/sandbox là đủ).
* **Node.js 20+** và **npm**.
* **AWS CLI v2** đã cấu hình credentials (`aws configure`).
* **AWS CDK CLI**: `npm install -g aws-cdk`
* **Git**.

{{% notice warning %}}
Hãy tạo **AWS Budget** kèm cảnh báo email (vd: 5$) trước khi bắt đầu. Chỉ mất 2 phút và tránh được hóa đơn bất ngờ.
{{% /notice %}}

### 2. Clone dự án

```bash
git clone https://github.com/Thien-132/music-instrument-store.git
cd music-instrument-store
npm install
```

Repository là một monorepo:

```
music-instrument-store/
├── frontend/          # Storefront Next.js
├── services/          # Lambda function (order-api, order-processing, notification, ...)
├── packages/          # Code dùng chung (shared-utils: helper idempotency)
├── infrastructure/    # CDK app: 4 stack
└── docs/              # Tài liệu thiết kế
```

### 3. Bootstrap CDK

CDK cần bootstrap một lần cho mỗi account/region (tạo S3 bucket staging và IAM role):

```bash
cd infrastructure
npm install
npx cdk bootstrap
```

![Kết quả cdk bootstrap](/images/5-Workshop/5.2-bootstrap.png)
<!-- TODO: chèn ảnh chụp kết quả cdk bootstrap -->

### 4. Kiểm tra

```bash
npx cdk list
```

Bạn sẽ thấy 4 stack:

```
MusicStoreSecurityStack-dev
MusicStoreDatabaseStack-dev
MusicStoreAuthStack-dev
MusicStoreBackendStack-dev
```

{{% notice info %}}
Hậu tố `-dev` đến từ biến context `env` của CDK (mặc định `dev`). Deploy môi trường riêng bằng `npx cdk deploy --context env=staging ...`.
{{% /notice %}}
