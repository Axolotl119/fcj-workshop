---
title: "Triển khai & Kiểm thử"
date: 2026-07-09
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Triển khai & Kiểm thử

### 1. Deploy toàn bộ stack

```bash
cd infrastructure
npx cdk deploy --all
```

CDK deploy theo thứ tự phụ thuộc: Security → Database → Auth → Backend. Ghi lại các output:

```
MusicStoreBackendStack-dev.ApiUrl = https://xxxxx.execute-api.us-east-1.amazonaws.com/prod/
MusicStoreBackendStack-dev.OrderQueueUrl = https://sqs.us-east-1.amazonaws.com/.../MusicStoreOrderQueue
MusicStoreDatabaseStack-dev.DynamoDBTableName = MusicStoreDatabaseStack-dev-MusicStoreMainTable...
```

### 2. Test 1 — đọc catalog

```bash
curl "$API_URL/products"
```

Kỳ vọng: mảng JSON nhạc cụ đã seed ở Module 1.

### 3. Test 2 — pipeline đơn hàng bất đồng bộ

Đặt một đơn hàng:

```bash
curl -X POST "$API_URL/orders" \
  -H "Content-Type: application/json" \
  -d '{"items":[{"productId":"SAX-001","quantity":1}],"customer":{"email":"test@example.com","name":"Test User"}}'
```

API trả về ngay với mã đơn. Giờ lần theo message qua hệ thống:

1. **SQS console → MusicStoreOrderQueue → Monitoring**: `NumberOfMessagesSent` tăng.
2. **CloudWatch Logs → OrderProcessingFunction**: processor tiêu thụ batch và ghi DynamoDB.
3. **DynamoDB → Explore items**: xuất hiện item `ORDER#<id>` mới.
4. **EventBridge → MusicStoreEventBus → Monitoring**: một sự kiện `OrderPlaced` khớp rule.


### 4. Test 3 — chứng minh DLQ hoạt động (tiêm lỗi)

Gửi thẳng một message hỏng vào queue:

```bash
aws sqs send-message \
  --queue-url "$ORDER_QUEUE_URL" \
  --message-body '{"broken": true'
```

Processor parse lỗi; SQS gửi lại 3 lần; sau đó message xuất hiện trong **MusicStoreOrderDLQ** và **OrderDLQAlarm** chuyển sang `In alarm`, gửi email SNS.


### 5. Test 4 — unit test idempotency

```bash
cd ../packages/shared-utils
npm test    # vitest + aws-sdk-client-mock
```

Toàn bộ test phải pass, chứng minh `claimEvent`/`releaseEvent` xử lý đúng khi trùng lặp và khi lỗi.

### 6. (Tùy chọn) Chạy storefront

```bash
cd ../frontend
npm install && npm run dev
```

Mở `http://localhost:3000` — trang sản phẩm giờ đọc từ bảng DynamoDB của bạn qua API đã deploy.
