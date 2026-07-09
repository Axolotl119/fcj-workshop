---
title: "Module 1: Tầng dữ liệu"
date: 2026-07-09
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Module 1: Tầng dữ liệu — DynamoDB, S3 & AWS Backup

Trong module này chúng ta định nghĩa **DatabaseStack** (`infrastructure/lib/database-stack.ts`): bảng DynamoDB single-table, S3 bucket ảnh sản phẩm và backup tự động.

### 1. Thiết kế DynamoDB single-table

Thay vì mỗi thực thể một bảng, **mọi thực thể nằm chung một bảng**, phân biệt bằng tiền tố khóa:

| Thực thể | PK | SK |
| --- | --- | --- |
| Sản phẩm | `PRODUCT#<id>` | `METADATA` |
| Đơn hàng | `ORDER#<id>` | `METADATA` |
| Đơn của user | `USER#<id>` | `ORDER#<orderId>` (qua GSI1) |
| Đánh giá | `PRODUCT#<id>` | `RATING#<userId>` |
| Coupon | `COUPON#<code>` | `METADATA` |
| Claim idempotency | `EVENT#<eventId>` | `METADATA` |

Bảng dùng billing on-demand và bật **Point-in-Time Recovery**:

```typescript
this.mainTable = new dynamodb.Table(this, 'MusicStoreMainTable', {
  partitionKey: { name: 'PK', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'SK', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST, // mô hình chi phí serverless
  removalPolicy: cdk.RemovalPolicy.DESTROY,          // chỉ dùng cho dev!
  pointInTimeRecovery: true,                          // PITR: khôi phục về bất kỳ giây nào trong 35 ngày
});
```

**GSI** phục vụ các access pattern bổ sung mà không cần Scan:

```typescript
this.mainTable.addGlobalSecondaryIndex({
  indexName: 'GSI1',
  partitionKey: { name: 'GSI1PK', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'GSI1SK', type: dynamodb.AttributeType.STRING },
  projectionType: dynamodb.ProjectionType.ALL,
});
// GSI2: truy vấn sản phẩm theo category (GSI2PK = TYPE#<type>) thay vì Scan cả bảng
```

### 2. S3 bucket cho ảnh sản phẩm

```typescript
this.productsBucket = new s3.Bucket(this, 'MusicStoreProductsBucket', {
  publicReadAccess: true,          // ảnh sản phẩm là nội dung công khai
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  autoDeleteObjects: true,         // tự xóa file khi destroy stack
  cors: [{
    allowedMethods: [s3.HttpMethods.GET, s3.HttpMethods.POST, s3.HttpMethods.PUT],
    allowedOrigins: ['*'],
    allowedHeaders: ['*'],
  }],
});
```

CORS cho phép admin panel upload ảnh trực tiếp từ trình duyệt bằng **presigned URL**.

### 3. AWS Backup — backup định kỳ

PITR chống ghi nhầm; **AWS Backup** bổ sung điểm khôi phục dài hạn:

```typescript
const backupPlan = new backup.BackupPlan(this, 'MusicStoreBackupPlan', {
  backupPlanName: 'MusicStoreBackupPlan',
});

// Hằng ngày 00:00 UTC, giữ 30 ngày
backupPlan.addRule(new backup.BackupPlanRule({
  ruleName: 'DailyBackup',
  backupVault,
  scheduleExpression: events.Schedule.expression('cron(0 0 * * ? *)'),
  deleteAfter: cdk.Duration.days(30),
}));

// Hằng tuần Chủ Nhật, giữ 90 ngày
backupPlan.addRule(new backup.BackupPlanRule({
  ruleName: 'WeeklyBackup',
  backupVault,
  scheduleExpression: events.Schedule.expression('cron(0 0 ? * SUN *)'),
  deleteAfter: cdk.Duration.days(90),
}));

backupPlan.addSelection('DynamoDbBackupSelection', {
  resources: [backup.BackupResource.fromDynamoDbTable(this.mainTable)],
});
```

### 4. Deploy và kiểm chứng

```bash
npx cdk deploy MusicStoreDatabaseStack-dev
```

Sau đó vào AWS console kiểm tra:

1. **DynamoDB → Tables** — bảng tồn tại với GSI1/GSI2 và PITR **Enabled**.
2. **S3** — bucket sản phẩm tồn tại với CORS đã cấu hình.
3. **AWS Backup → Backup plans** — `MusicStoreBackupPlan` với 2 rule.

![Bảng DynamoDB trên console](/images/5-Workshop/5.3-dynamodb.png)
<!-- TODO: chèn ảnh chụp bảng DynamoDB (tab Indexes + Backups) -->

### 5. Seed catalog

```bash
cd ../scripts
node seed-products.mjs   # nạp 15 nhạc cụ mẫu vào bảng
```

![Dữ liệu sau khi seed](/images/5-Workshop/5.3-seed.png)
<!-- TODO: chèn ảnh Explore items sau khi seed -->
