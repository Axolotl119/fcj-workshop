---
title: "Module 4: API & Giám sát"
date: 2026-07-09
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Module 4: API Gateway, Lambda, Cognito & Giám sát

Các khối cuối cùng: mở service qua REST, bảo vệ route admin, và cảnh báo khi có lỗi.

### 1. Lambda function

Mỗi service trong `services/` là một Lambda Node.js 20, ví dụ:

```typescript
const orderApiLambda = new lambda.Function(this, "OrderApiFunction", {
  runtime: lambda.Runtime.NODEJS_20_X,
  handler: "index.handler",
  code: lambda.Code.fromAsset("../services/order-api"),
  environment: {
    ORDER_QUEUE_URL: orderQueue.queueUrl,
    TABLE_NAME: props.productsTable.tableName,
  },
  tracing: lambda.Tracing.ACTIVE,           // bật X-Ray tracing
  logRetention: logs.RetentionDays.ONE_WEEK, // không trả tiền log vĩnh viễn
});
```

Stack nối 10+ Lambda: product-api, order-api, order-processing, checkout-service (Stripe), payment-webhook, notification, campaign-api, campaign-fanout, contact-api, chatbot (Lex).

### 2. REST API với API Gateway

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

Các route chính: `/products`, `/products/{id}/ratings`, `/users/wishlist`, `/orders`, `/coupons`, `/checkout`, `/campaigns`, `/contact`, `/webhooks/stripe`.

### 3. Bảo vệ route bằng Cognito

Route công khai (xem sản phẩm, đặt hàng) để mở; route admin/user yêu cầu JWT từ **Cognito User Pool**:

```typescript
const authorizer = new apigateway.CognitoUserPoolsAuthorizer(this, "ECommerceApiAuthorizer", {
  cognitoUserPools: [props.userPool],
});

productsResource.addMethod("POST", productApiIntegration, {
  authorizer,
  authorizationType: apigateway.AuthorizationType.COGNITO,
});
```

### 4. Giám sát: cảnh báo DLQ & lỗi

Lưới an toàn vận hành quan trọng nhất — **bất cứ thứ gì rơi vào DLQ, chúng ta nhận email ngay**:

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

Các alarm tương tự phủ lỗi 5XX của API Gateway và metric lỗi của mọi Lambda quan trọng (order processing, checkout, payment webhook).


{{% notice tip %}}
Đổi `admin@example.com` thành email thật của bạn, deploy, rồi **xác nhận subscription SNS** trong hộp thư — nếu không alarm sẽ kêu trong im lặng.
{{% /notice %}}
