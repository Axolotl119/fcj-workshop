---
title: "Module 2: Messaging"
date: 2026-07-09
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Module 2: Messaging — SQS Queue & Dead Letter Queue

Module này xây trái tim bất đồng bộ của hệ thống trong **BackendStack** (`infrastructure/lib/backend-stack.ts`): ba queue, mỗi queue kèm một DLQ.

### 1. Vì sao cần queue?

Nếu `POST /orders` ghi DynamoDB, trừ tiền thẻ và gửi email một cách đồng bộ, lỗi ở *bất kỳ* bước nào cũng làm hỏng cả request. Có queue ở giữa thì:

* API trả về trong vài mili-giây;
* mỗi bước xử lý tự retry độc lập;
* message hỏng (poison message) bị cách ly vào **DLQ** thay vì chặn cả pipeline.

### 2. Định nghĩa queue và DLQ

```typescript
const orderDLQ = new sqs.Queue(this, "OrderDLQ", {
  queueName: "MusicStoreOrderDLQ",
  retentionPeriod: cdk.Duration.days(14),   // giữ message lỗi đủ lâu để điều tra
});

const orderQueue = new sqs.Queue(this, "OrderQueue", {
  queueName: "MusicStoreOrderQueue",
  visibilityTimeout: cdk.Duration.seconds(30),
  deadLetterQueue: {
    maxReceiveCount: 3,   // nhận lỗi 3 lần → chuyển vào DLQ
    queue: orderDLQ,
  },
});
```

Mẫu này lặp lại cho `NotificationQueue` và `CampaignQueue`.

{{% notice tip %}}
**Vì sao CampaignQueue tách riêng?** Email marketing hàng loạt không được làm chậm thông báo giao dịch (xác nhận đơn, hủy đơn). Queue riêng + **reserved concurrency = 2** trên Lambda gửi campaign giúp traffic marketing không "bóp nghẹt" luồng quan trọng.
{{% /notice %}}

### 3. Producer: Order API gửi vào SQS

`services/order-api/index.js` xác thực đơn hàng rồi đẩy vào queue:

```javascript
await sqs.send(new SendMessageCommand({
  QueueUrl: process.env.ORDER_QUEUE_URL,
  MessageBody: JSON.stringify(order),
}));
return { statusCode: 202, body: JSON.stringify({ orderId }) }; // Accepted
```

Lambda chỉ cần `orderQueue.grantSendMessages(orderApiLambda)` — CDK tự sinh IAM policy tối thiểu.

### 4. Consumer: Order Processing Lambda

Processor tiêu thụ batch tối đa 10 message:

```typescript
orderProcessingLambda.addEventSource(
  new lambdaEventSources.SqsEventSource(orderQueue, { batchSize: 10 })
);
```

`services/order-processing/index.js` ghi đơn hàng vào DynamoDB rồi bắn sự kiện `OrderPlaced` lên EventBridge (module sau).

### 5. Idempotency — an toàn khi bị retry

SQS đảm bảo **at-least-once**: cùng một message có thể đến hai lần. Helper dùng chung (`packages/shared-utils`) "claim" mỗi sự kiện bằng conditional write:

```javascript
// claimEvent: PutItem PK=EVENT#<id> với ConditionExpression attribute_not_exists(PK)
const claimed = await claimEvent(eventId);
if (!claimed) return; // giao trùng — bỏ qua side effect

try {
  await processOrder(order);   // ghi đơn, gửi email, ...
} catch (err) {
  await releaseEvent(eventId); // nhả claim để lần retry chạy được
  throw err;                   // để SQS gửi lại
}
```

Logic này được phủ **unit test vitest** dùng `aws-sdk-client-mock`.

![Các queue SQS trên console](/images/5-Workshop/5.4-sqs.png)
<!-- TODO: chèn ảnh 6 queue (3 queue + 3 DLQ) trên SQS console -->
