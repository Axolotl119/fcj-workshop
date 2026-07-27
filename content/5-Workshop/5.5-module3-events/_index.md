---
title: "Module 3: Event Routing"
date: 2026-07-09
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Module 3: Event Routing — EventBridge

DynamoDB stores state; SQS moves work. **EventBridge** answers a third question: *how do independent services react to business events without knowing about each other?*

### 1. A custom event bus

```typescript
const eventBus = new events.EventBus(this, "MusicStoreEventBus", {
  eventBusName: "MusicStoreEventBus",
});
```

Producers publish domain events with a `source` and `detailType`:

```javascript
// services/order-processing/index.js — after saving the order
await eventBridge.send(new PutEventsCommand({
  Entries: [{
    EventBusName: process.env.EVENT_BUS_NAME,
    Source: "com.musicstore.order",
    DetailType: "OrderPlaced",
    Detail: JSON.stringify({ orderId, userEmail, total }),
  }],
}));
```

### 2. Rules route events to consumers

**Order events → Notification queue** (buffered, retried, DLQ-protected):

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

**Payment events** follow the same pattern (`com.musicstore.payment` / `PaymentSucceeded`) — published by the Stripe webhook Lambda.

**Campaign events → Lambda directly** (one-shot fan-out, no buffering needed):

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

### 3. The campaign fan-out pattern

When an admin launches a marketing campaign:

```
Campaign API ──CampaignRequested──> EventBridge ──> Fan-out Lambda
                                                        │ reads customers (DynamoDB)
                                                        │ splits into batches
                                                        ▼
                                                 SQS CampaignQueue ──> Campaign Sender (SES)
```

One event explodes into hundreds of queue messages, each processed idempotently. The sender Lambda has `reservedConcurrentExecutions: 2`, throttling bulk email so transactional traffic always wins.

### 4. Granting permissions

```typescript
eventBus.grantPutEventsTo(orderProcessingLambda);
eventBus.grantPutEventsTo(paymentWebhookLambda);
eventBus.grantPutEventsTo(campaignApiLambda);
```


{{% notice info %}}
**Why EventBridge instead of calling the notification Lambda directly?** Tomorrow we might add an analytics consumer or a Slack alert. With EventBridge we just add a rule — the order processor never changes. This is the *open/closed principle* applied to architecture.
{{% /notice %}}
