---
title: "Dọn dẹp tài nguyên"
date: 2026-07-09
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

# Dọn dẹp tài nguyên

Hủy toàn bộ tài nguyên để ngừng phát sinh chi phí. Vì cả hệ thống định nghĩa bằng CDK, dọn dẹp gần như chỉ một lệnh.

### 1. Destroy các stack

```bash
cd infrastructure
npx cdk destroy --all
```

Xác nhận `y` cho từng stack. Lưu ý:

* Bảng DynamoDB và S3 bucket được xóa tự động (`RemovalPolicy.DESTROY`, `autoDeleteObjects: true`) — đây là lý do ta đặt các cờ này cho môi trường dev.
* **Log group** CloudWatch do Lambda tạo có thể còn sót — xóa trong *CloudWatch → Log groups* (lọc `/aws/lambda/MusicStore`).

### 2. Xóa các điểm khôi phục backup

Recovery point của AWS Backup cố ý **không** bị xóa cùng stack:

1. Mở **AWS Backup → Backup vaults → MusicStoreBackupVault**.
2. Chọn tất cả recovery point → **Delete**.
3. Vault sau đó được stack destroy gỡ nốt (hoặc xóa thủ công).

### 3. Kiểm tra không còn gì sót

```bash
aws dynamodb list-tables
aws sqs list-queues
aws s3 ls
```

Hôm sau kiểm tra thêm **Billing → Cost Explorer** để xác nhận chi phí ngày về ~0$.

![Tài nguyên đã trống](/images/5-Workshop/5.8-cleanup.png)
<!-- TODO: chèn ảnh xác nhận tài nguyên đã xóa -->

{{% notice warning %}}
Nếu bạn đã bootstrap CDK, stack `CDKToolkit` và S3 bucket staging vẫn còn. Giữ lại nếu còn định deploy tiếp; không thì xóa CloudFormation stack `CDKToolkit` và dọn/xóa bucket `cdk-*-assets`.
{{% /notice %}}

### Chúc mừng! 🎉

Bạn đã dựng và gỡ trọn một tầng data & messaging serverless chuẩn production:

* DynamoDB single-table với PITR + backup định kỳ
* Pipeline SQS với DLQ và consumer idempotent
* Định tuyến sự kiện nghiệp vụ bằng EventBridge
* Alarm CloudWatch nối SNS

Mã nguồn đầy đủ có trong repository của dự án.
