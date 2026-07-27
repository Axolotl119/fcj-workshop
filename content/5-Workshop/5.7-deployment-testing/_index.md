---
title: "Deployment & Testing"
date: 2026-07-09
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Deployment & Testing

### 1. Deploy all stacks

```bash
cd infrastructure
npx cdk deploy --all
```

CDK deploys in dependency order: Security → Database → Auth → Backend. Note the outputs:

```
MusicStoreBackendStack-dev.ApiUrl = https://xxxxx.execute-api.us-east-1.amazonaws.com/prod/
MusicStoreBackendStack-dev.OrderQueueUrl = https://sqs.us-east-1.amazonaws.com/.../MusicStoreOrderQueue
MusicStoreDatabaseStack-dev.DynamoDBTableName = MusicStoreDatabaseStack-dev-MusicStoreMainTable...
```

### 2. Test 1 — read the catalog

```bash
curl "$API_URL/products"
```

Expected: JSON array of instruments seeded in Module 1.

### 3. Test 2 — the async order pipeline

Place an order:

```bash
curl -X POST "$API_URL/orders" \
  -H "Content-Type: application/json" \
  -d '{"items":[{"productId":"SAX-001","quantity":1}],"customer":{"email":"test@example.com","name":"Test User"}}'
```

The API answers immediately with an order id. Now trace the message through the system:

1. **SQS console → MusicStoreOrderQueue → Monitoring**: `NumberOfMessagesSent` ticks up.
2. **CloudWatch Logs → OrderProcessingFunction**: the processor consumed the batch and wrote to DynamoDB.
3. **DynamoDB → Explore items**: a new `ORDER#<id>` item exists.
4. **EventBridge → MusicStoreEventBus → Monitoring**: one `OrderPlaced` event matched the rule.


### 4. Test 3 — prove the DLQ works (fault injection)

Send a malformed message directly to the queue:

```bash
aws sqs send-message \
  --queue-url "$ORDER_QUEUE_URL" \
  --message-body '{"broken": true'
```

The processor fails to parse it; SQS redelivers 3 times; then the message appears in **MusicStoreOrderDLQ** and the **OrderDLQAlarm** goes to `In alarm`, sending an SNS email.


### 5. Test 4 — idempotency unit tests

```bash
cd ../packages/shared-utils
npm test    # vitest + aws-sdk-client-mock
```

All tests should pass, proving `claimEvent`/`releaseEvent` behave correctly on duplicates and failures.

### 6. (Optional) Run the storefront

```bash
cd ../frontend
npm install && npm run dev
```

Open `http://localhost:3000` — the products page now reads from your DynamoDB table through the deployed API.
