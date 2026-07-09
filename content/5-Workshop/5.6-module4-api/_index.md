---
title: "Module 4: API & Monitoring"
date: 2026-07-09
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Module 4: API Gateway, Lambda, Cognito & Monitoring

The final building blocks: exposing the services over REST, protecting admin routes, and alarming on failures.

### 1. Lambda functions

Each service in `services/` becomes a Node.js 20 Lambda, e.g.:

```typescript
const orderApiLambda = new lambda.Function(this, "OrderApiFunction", {
  runtime: lambda.Runtime.NODEJS_20_X,
  handler: "index.handler",
  code: lambda.Code.fromAsset("../services/order-api"),
  environment: {
    ORDER_QUEUE_URL: orderQueue.queueUrl,
    TABLE_NAME: props.productsTable.tableName,
  },
  tracing: lambda.Tracing.ACTIVE,           // X-Ray tracing
  logRetention: logs.RetentionDays.ONE_WEEK, // don't pay for logs forever
});
```

The stack wires 10+ Lambdas: product-api, order-api, order-processing, checkout-service (Stripe), payment-webhook, notification, campaign-api, campaign-fanout, contact-api, chatbot (Lex).

### 2. REST API with API Gateway

```typescript
const api = new apigateway.RestApi(this, "ECommerceApi", {
  restApiName: "Music Store API",
  defaultCorsPreflightOptions: {
    allowHeaders: ["Content-Type", "Authorization"],
    allowMethods: ["OPTIONS", "GET", "POST", "PUT", "DELETE"],
    allowOrigins: apigateway.Cors.ALL_ORIGINS,
  },
  deployOptions: { tracingEnabled: true, loggingLevel: apigateway.MethodLoggingLevel.INFO },
});
```

Key routes: `/products`, `/products/{id}/ratings`, `/users/wishlist`, `/orders`, `/coupons`, `/checkout`, `/campaigns`, `/contact`, `/webhooks/stripe`.

### 3. Protecting routes with Cognito

Public routes (browse products, place order) stay open; admin/user routes require a JWT from the **Cognito User Pool**:

```typescript
const authorizer = new apigateway.CognitoUserPoolsAuthorizer(this, "ECommerceApiAuthorizer", {
  cognitoUserPools: [props.userPool],
});

productsResource.addMethod("POST", productApiIntegration, {
  authorizer,
  authorizationType: apigateway.AuthorizationType.COGNITO,
});
```

### 4. Monitoring: DLQ & error alarms

The most important operational safety net — **if anything lands in a DLQ, we get an email**:

```typescript
const alarmsTopic = new sns.Topic(this, "SystemAlarmsTopic", { topicName: "MusicStoreSystemAlarms" });
alarmsTopic.addSubscription(new sns_subscriptions.EmailSubscription("admin@example.com"));

const orderDlqAlarm = new cw.Alarm(this, "OrderDLQAlarm", {
  metric: orderDLQ.metricApproximateNumberOfMessagesVisible(),
  threshold: 1,
  evaluationPeriods: 1,
  comparisonOperator: cw.ComparisonOperator.GREATER_THAN_OR_EQUAL_TO_THRESHOLD,
});
orderDlqAlarm.addAlarmAction(new cw_actions.SnsAction(alarmsTopic));
```

Similar alarms cover API Gateway 5XX errors and the error metric of every critical Lambda (order processing, checkout, payment webhook).

![CloudWatch alarms](/images/5-Workshop/5.6-alarms.png)
<!-- TODO: chèn ảnh danh sách alarm CloudWatch -->

{{% notice tip %}}
Replace `admin@example.com` with your real email, deploy, and **confirm the SNS subscription** from your inbox — otherwise alarms fire silently.
{{% /notice %}}
