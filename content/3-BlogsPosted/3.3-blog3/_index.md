---
title: "Blog 3 — Single-Table Design on DynamoDB & Reliable Message Processing with SQS"
date: 2026-07-09
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

### Overview

In this blog series about building the infrastructure for the **Music Instrument Store** system, Blog 1 and Blog 2 covered relational database design (Amazon RDS) and high-performance caching/read architecture. In **Blog 3**, we tackle two major challenges as user scale and transaction volume grow rapidly:

1. **Scalability:** Traditional relational databases start to bottleneck as write and read queries on supporting tables (logs, order history, flexible shopping carts) increase rapidly.
2. **Asynchronous Processing:** When a customer clicks "Place Order," the system needs to perform a series of related tasks: deduct inventory, generate an invoice, send a confirmation email, push a webhook notification, and run analytics. Forcing the user to wait for all these steps to complete within a single HTTP request would result in extremely high latency and a high risk of transaction failure.

To fully address these two problems, we adopted **Amazon DynamoDB** (using a *Single-Table Design* model) and **Amazon SQS (Simple Queue Service)** as the backbone for reliable message delivery.

---

### 1. System Architecture Overview

Below is an overview diagram of the data flow and how the services interact within the music instrument order-processing system:

![DynamoDB and SQS System Architecture Diagram](images/blog3.png)

*Figure 1: Asynchronous processing flow between API Gateway, AWS Lambda, Amazon DynamoDB, and the Amazon SQS Queue.*

---

### 2. Single-Table Design on Amazon DynamoDB

Unlike traditional relational databases, where each entity lives in its own table linked via `JOIN` statements, **DynamoDB** performs best when applying a **Single-Table Design** approach — all application entities (Customers, Orders, Musical Instrument Products, Payment Details) are consolidated into **a single table**.

#### 2.1. Why Choose Single-Table Design?

* **Eliminates JOIN operations:** JOINs in RDBMS consume CPU and increase latency exponentially as data grows. In DynamoDB, retrieving complete order and customer information requires only **a single Query operation**.
* **Low and consistent latency:** Whether the table holds 1,000 records or 100 million records, DynamoDB response times typically remain at **single-digit milliseconds** (under 10ms), assuming keys and access patterns are well designed.
* **Optimized operational cost:** Managing capacity (RCU/WCU or On-Demand) for a single table is simpler and more cost-effective than maintaining dozens of separate tables.

#### 2.2. Primary Key (PK & SK) and Global Secondary Index (GSI) Design

To support the diverse **Access Patterns** required by the Music Instrument Store, we apply a flexible prefixed PK/SK naming strategy:

| Entities | Partition Key (PK) | Sort Key (SK) | GSI1PK | GSI1SK | Additional Attributes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **User Profile** | `USER#<UserId>` | `METADATA` | `EMAIL#<Email>` | `USER#<UserId>` | `FullName`, `Phone`, `Address` |
| **Order Info** | `ORDER#<OrderId>` | `METADATA` | `USER#<UserId>` | `DATE#<CreatedAt>` | `TotalAmount`, `Status`, `PaymentMethod` |
| **Order Line Item**| `ORDER#<OrderId>` | `ITEM#<ProductId>`| `PRODUCT#<ProductId>`| `ORDER#<OrderId>` | `Quantity`, `UnitPrice`, `ProductName` |
| **Product Catalog**| `PRODUCT#<ProductId>`| `METADATA` | `CATEGORY#<Category>`| `PRICE#<Price>` | `Stock`, `Brand`, `Specs` |

![Single Table Design Model on DynamoDB](images/blog3-dynamodb-single-table-schema.png)

*Figure 2: Visualization of multiple entities stored together within a single DynamoDB table.*

#### 2.3. Real-World Query Example Using the AWS SDK (Node.js)

Below is a code example showing how to retrieve **order information along with all instrument line items in that order** using **a single query**:

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, QueryCommand } from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const getOrderDetails = async (orderId) => {
  const params = {
    TableName: "MusicStoreSingleTable",
    KeyConditionExpression: "PK = :pk",
    ExpressionAttributeValues: {
      ":pk": `ORDER#${orderId}`
    }
  };

  try {
    const data = await docClient.send(new QueryCommand(params));

    // The result includes both the general order info (SK=METADATA)
    // and the instrument line items belonging to the order (SK=ITEM#...)
    console.log("Order Data & Line Items:", data.Items);
    return data.Items;
  } catch (err) {
    console.error("Error fetching order:", err);
    throw err;
  }
};
```

### 3. Asynchronous & Reliable Message Processing with Amazon SQS

During instrument order processing, when the number of concurrent buyers spikes (e.g., an Acoustic Guitar Flash Sale), performing all steps synchronously would cause the system to hang or become overloaded. To solve this, **Amazon SQS** acts as a reliable "buffer."

![SQS Message Flow and Dead Letter Queue](images/blog3-sqs-dlq-workflow.png)

*Figure 3: Message processing flow through the SQS Queue, Lambda Worker, and the Dead Letter Queue (DLQ) mechanism.*

#### 3.1. SQS FIFO Queue and Standard Queue Design

The system is split into two types of queues:

* **Order Processing Queue (SQS FIFO Queue):** Guarantees strict processing order (First-In, First-Out) and automatic deduplication (`MessageDeduplicationId`), preventing an order from being processed twice. Note: FIFO queue names must end with the `.fifo` suffix (e.g., `OrderProcessingQueue.fifo`).
* **Notification & Analytics Queue (SQS Standard Queue):** Optimized for speed and high throughput to send notification emails and push logs to the analytics system.

#### 3.2. Dead Letter Queue (DLQ) & Retry Policy

* **Visibility Timeout:** Configured to be 6 times the **configured Lambda Worker timeout** (e.g., Lambda timeout = 30 seconds → Visibility Timeout ≈ 180 seconds).
* **Max Receive Count:** Set to `3`. If a message fails processing 3 times in a row, SQS automatically moves it to the **Dead Letter Queue (DLQ)**.
* **Alerting & Redrive:** Set up an Amazon CloudWatch Alarm to trigger when the DLQ message count > 0, so engineers can intervene promptly and trigger the Redrive process.

#### 3.3. Sample Code for Sending a Message to the SQS Queue

```python
import json
import boto3
import os

sqs = boto3.client('sqs')
# Note: QUEUE_URL must point to a FIFO Queue (ending in .fifo)
# since MessageGroupId and MessageDeduplicationId are only valid for FIFO Queues
QUEUE_URL = os.environ.get('ORDER_SQS_QUEUE_URL')

def send_order_event(order_data):
    try:
        response = sqs.send_message(
            QueueUrl=QUEUE_URL,
            MessageBody=json.dumps(order_data),
            MessageGroupId=f"UserOrder_{order_data['userId']}",
            MessageDeduplicationId=order_data['orderId']
        )
        print(f"Message sent successfully! MessageId: {response['MessageId']}")
        return response
    except Exception as e:
        print(f"Failed to send message to SQS: {str(e)}")
        raise e
```
---

### 4. Lessons Learned & Practical Implementation (Best Practices)

During the real-world implementation of the **Music Instrument Store** system, we gathered the following key lessons:

1. **Carefully analyze Access Patterns before writing code:**
   Unlike relational databases (where you create tables first and write SQL queries later), with DynamoDB Single-Table Design, you must **list 100% of the application's query/access patterns first**, and only then decide on the appropriate Partition Key (PK), Sort Key (SK), and Global Secondary Index (GSI) structure.

2. **Ensure Idempotency on the Worker side:**
   SQS and other event-driven services typically guarantee *At-Least-Once Delivery* (messages may be delivered more than once). Therefore, the Lambda Worker processing messages must always check the order status in DynamoDB before performing critical actions such as deducting inventory or processing payment.

3. **Proactively monitor the system via CloudWatch Metrics:**
   Set up alarms for key metrics: `ThrottledRequests` on DynamoDB, `ApproximateAgeOfOldestMessage` (message delay), and `ApproximateNumberOfMessagesVisible` (backlog size) on the SQS Queue to detect and address performance bottlenecks early.

---

### Conclusion

The combination of **Amazon DynamoDB (Single-Table Design)** and **Amazon SQS** delivered a major infrastructure improvement for the Music Instrument Store project:

* **Improved performance:** Order-placement API response time dropped from **1.2 seconds to under 80ms**.
* **High load resilience:** The system easily handles traffic spikes (Flash Sales) without database bottlenecks.
* **Maximum reliability:** Completely eliminated the risk of lost orders thanks to the asynchronous queuing mechanism and the Dead Letter Queue.
