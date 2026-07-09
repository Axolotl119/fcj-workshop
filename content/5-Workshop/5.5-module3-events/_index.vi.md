---
title: "Module 3: Định tuyến sự kiện"
date: 2026-07-09
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Module 3: Định tuyến sự kiện — EventBridge

DynamoDB lưu trạng thái; SQS chuyển công việc. **EventBridge** trả lời câu hỏi thứ ba: *làm sao các service độc lập phản ứng với sự kiện nghiệp vụ mà không cần biết nhau?*

### 1. Event bus tùy chỉnh

```typescript
const eventBus = new events.EventBus(this, "MusicStoreEventBus", {
  eventBusName: "MusicStoreEventBus",
});
```

Producer phát sự kiện nghiệp vụ với `source` và `detailType`:

```javascript
// services/order-processing/index.js — sau khi lưu đơn hàng
await eventBridge.send(new PutEventsCommand({
  Entries: [{
    EventBusName: process.env.EVENT_BUS_NAME,
    Source: "com.musicstore.order",
    DetailType: "OrderPlaced",
    Detail: JSON.stringify({ orderId, userEmail, total }),
  }],
}));
```

### 2. Rule định tuyến sự kiện tới consumer

**Sự kiện đơn hàng → Notification queue** (có buffer, retry, DLQ bảo vệ):

```typescript
const orderPlacedRule = new events.Rule(this, "OrderPlacedRule", {
  eventBus,
  eventPattern: {
    source: ["com.musicstore.order"],
    detailType: ["OrderPlaced", "OrderUpdated", "OrderCancelled"],
  },
});
orderPlacedRule.addTarget(new targets.SqsQueue(notificationQueue));
```

**Sự kiện thanh toán** theo cùng mẫu (`com.musicstore.payment` / `PaymentSucceeded`) — do Lambda webhook Stripe phát.

**Sự kiện campaign → gọi thẳng Lambda** (fan-out one-shot, không cần buffer):

```typescript
const campaignRequestedRule = new events.Rule(this, "CampaignRequestedRule", {
  eventBus,
  eventPattern: {
    source: ["com.musicstore.campaign"],
    detailType: ["CampaignRequested"],
  },
});
campaignRequestedRule.addTarget(new targets.LambdaFunction(campaignFanOutLambda));
```

### 3. Mẫu fan-out cho campaign

Khi admin phát động chiến dịch marketing:

```
Campaign API ──CampaignRequested──> EventBridge ──> Lambda Fan-out
                                                        │ đọc danh sách khách (DynamoDB)
                                                        │ chia thành batch
                                                        ▼
                                                 SQS CampaignQueue ──> Campaign Sender (SES)
```

Một sự kiện nở thành hàng trăm message trong queue, mỗi message xử lý idempotent. Lambda gửi mail có `reservedConcurrentExecutions: 2` — bóp lưu lượng email hàng loạt để traffic giao dịch luôn thắng.

### 4. Cấp quyền

```typescript
eventBus.grantPutEventsTo(orderProcessingLambda);
eventBus.grantPutEventsTo(paymentWebhookLambda);
eventBus.grantPutEventsTo(campaignApiLambda);
```

![Rule EventBridge](/images/5-Workshop/5.5-eventbridge.png)
<!-- TODO: chèn ảnh 3 rule trên EventBridge console -->

{{% notice info %}}
**Vì sao dùng EventBridge thay vì gọi thẳng Lambda notification?** Ngày mai nhóm có thể thêm consumer analytics hay cảnh báo Slack. Với EventBridge chỉ cần thêm rule — order processor không phải sửa. Đây là *nguyên tắc open/closed* áp vào kiến trúc.
{{% /notice %}}
