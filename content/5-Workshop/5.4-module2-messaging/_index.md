---
title: "Module 2: Messaging"
date: 2026-07-09
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Module 2: Messaging — SQS Queues & Dead Letter Queues

This module builds the asynchronous heart of the system inside the **BackendStack** (`infrastructure/lib/backend-stack.ts`): three queues, each paired with a DLQ.

### 1. Why queues?

If `POST /orders` wrote to DynamoDB, charged the card, and sent an email synchronously, a failure in *any* step would fail the whole request. With a queue in the middle:

* the API returns in milliseconds;
* each processing step retries independently;
* poison messages are quarantined in a **DLQ** instead of blocking the pipeline.

### 2. Define the queues and DLQs

```typescript
const orderDLQ = new sqs.Queue(this, "OrderDLQ", {
  queueName: "MusicStoreOrderDLQ",
  retentionPeriod: cdk.Duration.days(14),   // keep failures long enough to investigate
});

const orderQueue = new sqs.Queue(this, "OrderQueue", {
  queueName: "MusicStoreOrderQueue",
  visibilityTimeout: cdk.Duration.seconds(30),
  deadLetterQueue: {
    maxReceiveCount: 3,   // after 3 failed receives → DLQ
    queue: orderDLQ,
  },
});
```

The same pattern is repeated for `NotificationQueue` and `CampaignQueue`.

{{% notice tip %}}
**Why a separate CampaignQueue?** Bulk marketing emails must never delay transactional messages (order confirmations, cancellations). Separate queues + a **reserved concurrency of 2** on the campaign sender Lambda keep marketing traffic from starving the critical path.
{{% /notice %}}

### 3. Producer: Order API sends to SQS

`services/order-api/index.js` validates the order and enqueues it:

```javascript
await sqs.send(new SendMessageCommand({
  QueueUrl: process.env.ORDER_QUEUE_URL,
  MessageBody: JSON.stringify(order),
}));
return { statusCode: 202, body: JSON.stringify({ orderId }) }; // Accepted
```

The Lambda only needs `orderQueue.grantSendMessages(orderApiLambda)` — CDK generates the minimal IAM policy.

### 4. Consumer: Order Processing Lambda

The processor consumes batches of up to 10 messages:

```typescript
orderProcessingLambda.addEventSource(
  new lambdaEventSources.SqsEventSource(orderQueue, { batchSize: 10 })
);
```

`services/order-processing/index.js` writes the order to DynamoDB and emits an `OrderPlaced` event to EventBridge (next module).

### 5. Idempotency — surviving retries safely

SQS guarantees **at-least-once** delivery: the same message can arrive twice. Our shared helper (`packages/shared-utils`) claims each event with a conditional write:

```javascript
// claimEvent: PutItem PK=EVENT#<id> with ConditionExpression attribute_not_exists(PK)
const claimed = await claimEvent(eventId);
if (!claimed) return; // duplicate delivery — skip side effects

try {
  await processOrder(order);   // write order, send email, ...
} catch (err) {
  await releaseEvent(eventId); // free the claim so the retry can run
  throw err;                   // let SQS redeliver
}
```

This logic is covered by **vitest** unit tests using `aws-sdk-client-mock`.

