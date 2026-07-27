---
title: "Blog 3 — Thiết kế Single-Table trên DynamoDB & Xử lý tin nhắn tin cậy với SQS"
date: 2026-07-09
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

### Giới thiệu tổng quan

Trong chuỗi bài viết chia sẻ về quá trình xây dựng hạ tầng cho hệ thống **Cửa hàng Nhạc cụ (Music Instrument Store)**, Blog 1 và Blog 2 đã giải quyết các bài toán về cơ sở dữ liệu quan hệ (Amazon RDS) cũng như kiến trúc caching/đọc hiệu năng cao. Đến với **Blog 3**, chúng tôi phải đối mặt với hai thách thức lớn khi quy mô người dùng và lượng giao dịch gia tăng đột biến:

1. **Bài toán khả năng mở rộng (Scalability):** Cơ sở dữ liệu quan hệ truyền thống bắt đầu gặp hiện tượng nghẽn cổ chai (bottleneck) khi số lượng truy vấn ghi và đọc các bảng phụ trợ (logs, lịch sử đơn hàng, giỏ hàng linh hoạt) tăng nhanh.
2. **Bài toán xử lý bất đồng bộ (Asynchronous Processing):** Khi khách hàng bấm "Đặt hàng", hệ thống cần thực hiện hàng loạt tác vụ liên quan: trừ kho, tạo hóa đơn, gửi email xác nhận, đẩy thông báo qua webhook, và phân tích dữ liệu. Nếu bắt người dùng chờ tất cả các bước này hoàn tất trong 1 HTTP Request, thời gian phản hồi (latency) sẽ cực kỳ cao và nguy cơ thất bại giao dịch là rất lớn.

Để giải quyết triệt để hai bài toán này, chúng tôi đã đưa vào sử dụng **Amazon DynamoDB** (theo mô hình *Single-Table Design*) và **Amazon SQS (Simple Queue Service)** làm xương sống cho việc truyền nhận tin nhắn tin cậy.

---

### 1. Kiến trúc Tổng quan Hệ thống (System Architecture)

Dưới đây là sơ đồ tổng quan về luồng dữ liệu và cách các dịch vụ tương tác với nhau trong hệ thống xử lý đơn hàng nhạc cụ:

![Sơ đồ Kiến trúc Hệ thống DynamoDB và SQS](images/blog3-architecture-diagram.png)

*Hình 1: Sơ đồ luồng xử lý bất đồng bộ giữa API Gateway, AWS Lambda, Amazon DynamoDB và Amazon SQS Queue.*

---

### 2. Thiết kế Single-Table Design trên Amazon DynamoDB

Khác với mô hình cơ sở dữ liệu quan hệ (Relational Database) truyền thống nơi mỗi thực thể là một bảng riêng biệt và liên kết với nhau bằng các câu lệnh `JOIN`, **DynamoDB** vận hành hiệu quả nhất khi áp dụng tư duy **Single-Table Design** — toàn bộ các thực thể của ứng dụng (Khách hàng, Đơn hàng, Sản phẩm nhạc cụ, Chi tiết thanh toán) được thiết kế tập trung trong đúng **một bảng duy nhất**.

#### 2.1. Tại sao lại chọn Single-Table Design?

* **Tránh hoàn toàn thao tác JOIN:** Thao tác JOIN trong RDBMS tiêu tốn CPU và làm tăng latency theo cấp số nhân khi dữ liệu lớn. Trên DynamoDB, việc truy vấn lấy toàn bộ thông tin đơn hàng và thông tin khách hàng chỉ cần **1 câu lệnh Query duy nhất**.
* **Độ trễ thấp và ổn định (Consistent Latency):** Dù bảng có chứa 1,000 bản ghi hay 100 triệu bản ghi, thời gian phản hồi của DynamoDB vẫn luôn giữ vững ở mức **single-digit millisecond** (dưới 10ms).
* **Tối ưu chi phí vận hành:** Việc quản lý Capacity (RCU/WCU hoặc On-Demand) cho một bảng đơn giản và tiết kiệm hơn nhiều so với việc duy trì hàng chục bảng riêng lẻ.

#### 2.2. Thiết kế Primary Key (PK & SK) & Global Secondary Index (GSI)

Để hỗ trợ đa dạng các **Access Patterns** (mẫu truy vấn) khác nhau cho Cửa hàng Nhạc cụ, chúng tôi áp dụng chiến lược đặt tên PK/SK có tiền tố (Prefix) linh hoạt:

| Entities | Partition Key (PK) | Sort Key (SK) | GSI1PK | GSI1SK | Thuộc tính bổ sung |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **User Profile** | `USER#<UserId>` | `METADATA` | `EMAIL#<Email>` | `USER#<UserId>` | `FullName`, `Phone`, `Address` |
| **Order Info** | `ORDER#<OrderId>` | `METADATA` | `USER#<UserId>` | `DATE#<CreatedAt>` | `TotalAmount`, `Status`, `PaymentMethod` |
| **Order Line Item**| `ORDER#<OrderId>` | `ITEM#<ProductId>`| `PRODUCT#<ProductId>`| `ORDER#<OrderId>` | `Quantity`, `UnitPrice`, `ProductName` |
| **Product Catalog**| `PRODUCT#<ProductId>`| `METADATA` | `CATEGORY#<Category>`| `PRICE#<Price>` | `Stock`, `Brand`, `Specs` |

![Mô hình Single Table Design trên DynamoDB](images/blog3-dynamodb-single-table-schema.png)

*Hình 2: Trực quan hóa dữ liệu đa thực thể được lưu trữ chung trong một bảng DynamoDB duy nhất.*

#### 2.3. Ví dụ truy vấn thực tế bằng AWS SDK (Node.js)

Dưới đây là đoạn code ví dụ thể hiện cách lấy **thông tin đơn hàng cùng tất cả các món nhạc cụ có trong đơn hàng đó** chỉ bằng **một truy vấn duy nhất**:

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
    
    // Kết quả thu được bao gồm cả thông tin chung Đơn hàng (SK=METADATA)
    // và các sản phẩm nhạc cụ thuộc đơn hàng (SK=ITEM#...)
    console.log("Order Data & Line Items:", data.Items);
    return data.Items;
  } catch (err) {
    console.error("Error fetching order:", err);
    throw err;
  }
};

### 3. Xử lý Tin nhắn Bất đồng bộ & Tin cậy với Amazon SQS
Trong quy trình xử lý đơn hàng nhạc cụ, khi số lượng người mua cùng lúc tăng cao (ví dụ: đợt Flash Sale đàn Guitar Acoustic), việc đồng bộ tất cả các bước sẽ khiến hệ thống bị treo hoặc quá tải. Để giải quyết, Amazon SQS đóng vai trò là một "vùng đệm" (buffer) tin cậy.

Hình 3: Quy trình xử lý Message qua SQS Queue, Lambda Worker và cơ chế Dead Letter Queue (DLQ).

#### 3.1. Thiết kế Hàng đợi SQS FIFO và Standard Queue
Hệ thống được chia thành 2 loại hàng đợi:

Order Processing Queue (SQS FIFO Queue): Đảm bảo thứ tự ưu tiên xử lý chính xác tuyệt đối (First-In, First-Out) và tự động lọc trùng lặp (MessageDeduplicationId), tránh việc một đơn hàng bị xử lý 2 lần.

Notification & Analytics Queue (SQS Standard Queue): Tối ưu cho tốc độ và khả năng throughput cao để gửi email thông báo và đẩy log về hệ thống phân tích.

#### 3.2. Cơ chế Dead Letter Queue (DLQ) & Retry Policy
Để đảm bảo không bao giờ bị mất dữ liệu đơn hàng ngay cả khi có sự cố kỹ thuật (ví dụ: Service gửi Email bị sập tạm thời hoặc Third-party Payment Gateway bị timeout):

Visibility Timeout: Được cấu hình gấp 6 lần thời gian thực thi trung bình của Lambda Worker (ví dụ: 30 giây).

Max Receive Count: Cấu hình bằng 3. Nếu một Message bị xử lý thất bại 3 lần liên tiếp, SQS sẽ tự động chuyển Message đó sang Dead Letter Queue (DLQ).

Alerting & Redrive: Đặt cảnh báo Amazon CloudWatch Alarm khi số lượng Message trong DLQ > 0 để đội ngũ Engineer kịp thời can thiệp và kích hoạt quy trình Redrive (đẩy lại Message xử lý sau khi fix bug).

#### 3.3. Đoạn mã minh họa đẩy Message vào SQS Queue

import json
import boto3
import os

sqs = boto3.client('sqs')
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

### 4. Bài học kinh nghiệm & Thực thi thực tế (Best Practices)
Trong quá trình triển khai thực tế cho hệ thống Cửa hàng Nhạc cụ, chúng tôi đã rút ra được những kinh nghiệm xương máu sau:

Phân tích Access Patterns cẩn thận trước khi viết code:
Khác với SQL (nơi bạn tạo bảng trước rồi nghĩ cách query sau), với DynamoDB Single-Table, bạn phải liệt kê 100% các câu hỏi/truy vấn của hệ thống trước, từ đó mới chọn PK, SK, và GSI phù hợp.

Xử lý tính Đậm đà (Idempotency) ở phía Worker:
Dù SQS hay EventBridge có cơ chế At-Least-Once Delivery, Lambda Worker xử lý tin nhắn luôn phải kiểm tra trạng thái trong DynamoDB trước khi thực hiện các hành động tính phí hay trừ kho.

Giám sát chặt chẽ qua CloudWatch Metrics:
Theo dõi sát các chỉ số: ThrottledRequests trên DynamoDB, ApproximateAgeOfOldestMessage và ApproximateNumberOfMessagesVisible trên SQS Queue để phát hiện sớm hiện tượng nghẽn cổ chai.

### Kết luận
Sự kết hợp giữa Amazon DynamoDB (Single-Table Design) và Amazon SQS đã mang lại một bước tiến vượt bậc về mặt hạ tầng cho dự án Cửa hàng Nhạc cụ:

Thời gian phản hồi API đặt hàng giảm từ 1.2 giây xuống chỉ còn dưới 80ms.

Hệ thống có khả năng chịu tải đột biến tốt mà không gặp tình trạng nghẽn cơ sở dữ liệu.

Độ tin cậy đạt mức tối đa, hoàn toàn loại bỏ nguy cơ thất lạc đơn hàng của khách hàng.

Nội dung Blog 3 này đã khép lại phần báo cáo về tầng Dữ liệu và Xử lý bất đồng bộ trong chuỗi hoạt động thực tập của tôi tại chương trình FCJ Workshop.