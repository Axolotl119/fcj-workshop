---
title: "Đề xuất"
date: 2026-07-09
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# ĐỀ XUẤT KIẾN TRÚC HỆ THỐNG

## DỰ ÁN: MUSIC INSTRUMENT STORE — NỀN TẢNG THƯƠNG MẠI ĐIỆN TỬ SERVERLESS TRÊN AWS

---

### 1. GIỚI THIỆU & VẤN ĐỀ CẦN GIẢI QUYẾT

#### 1.1. Xu hướng kiến trúc Cloud-native và Event-driven

Bước vào năm 2026, các nền tảng thương mại điện tử đang dần chuyển từ mô hình triển khai máy chủ nguyên khối, chạy 24/7 sang **kiến trúc serverless, hướng sự kiện (event-driven)**. Sự chuyển dịch này xuất phát từ nhu cầu xử lý lưu lượng truy cập khó dự đoán (flash sale, khuyến mãi theo mùa) mà không cần cấp phát dư thừa hạ tầng, đồng thời giảm thiểu chi phí vận hành — điều đặc biệt quan trọng với một dự án capstone của sinh viên chạy trên nguồn credit AWS có hạn.

Bằng cách tách rời frontend khỏi backend, và tách các backend service khỏi nhau thông qua hàng đợi (queue) và event bus, hệ thống có thể xử lý từng request một cách độc lập, retry an toàn khi có lỗi, và vẫn duy trì hoạt động ngay cả khi một thành phần downstream bị gián đoạn tạm thời.

#### 1.2. Hạn chế của kiến trúc truyền thống

Các hệ thống thương mại điện tử truyền thống triển khai trên máy chủ chạy liên tục hoặc kiến trúc monolith thường gặp phải:

- **Chi phí nhàn rỗi cao:** máy chủ phải được cấp phát theo tải cao điểm, gây lãng phí trong giờ ít traffic.
- **Khả năng mở rộng kém khi tải tăng đột biến:** flash sale hoặc chiến dịch marketing có thể làm quá tải máy chủ và database có công suất cố định.
- **Luồng checkout dễ gãy:** một service thông báo/tồn kho chậm hoặc lỗi có thể chặn toàn bộ luồng đặt hàng, gây mất doanh thu.
- **Tính phí trùng lặp:** mạng không ổn định hoặc bấm "Thanh toán" nhiều lần có thể gây tính phí kép nếu thiếu cơ chế idempotency.
- **Bảo mật yếu:** hạ tầng quản lý thủ công dễ bị tấn công bot, rò rỉ credential, và lạm dụng API mà không được giám sát.

---

### 2. MỤC TIÊU DỰ ÁN

**Music Instrument Store** là nền tảng thương mại điện tử đầy đủ tính năng để bán nhạc cụ (guitar, piano, saxophone, …), được xây dựng bởi nhóm 5 thành viên như dự án capstone của kỳ thực tập FCAJ. Toàn bộ backend chạy trên kiến trúc **serverless, hướng sự kiện**, được định nghĩa dưới dạng code bằng **AWS CDK (TypeScript)**.

Repository GitHub: `https://github.com/Thien-132/music-instrument-store`

Mục tiêu dự án:

- **Trả tiền theo mức sử dụng:** chi phí gần như bằng 0 khi hệ thống nhàn rỗi, vì mọi service cốt lõi (Lambda, DynamoDB on-demand, SQS, EventBridge) chỉ tính phí theo mức sử dụng thực tế.
- **Co giãn linh hoạt:** tự động hấp thụ lưu lượng tăng đột biến mà không cần can thiệp thủ công.
- **Đáng tin cậy:** không mất đơn hàng hay thông báo nào, ngay cả khi một service downstream gặp sự cố — được đảm bảo bằng queue, dead-letter queue (DLQ), cơ chế retry và idempotency key.
- **Bảo mật theo thiết kế:** xác thực qua Cognito, bảo vệ tại tầng edge bằng WAF, và giám sát liên tục qua CloudWatch, X-Ray, GuardDuty, CloudTrail.
- **Hỗ trợ mua sắm bằng AI:** chatbot (Amazon Lex) giúp khách hàng tìm sản phẩm và giải đáp thắc mắc.

---

### 3. KIẾN TRÚC HỆ THỐNG ĐỀ XUẤT

#### 3.1. Sơ đồ kiến trúc

![Sơ đồ kiến trúc](https://axolotl119.github.io/fcj-workshop/images/2-Proposal/sơ dồ kiến trúc.png)

Hệ thống được tổ chức thành các nhóm dịch vụ logic trong cùng một AWS Region:

| Nhóm                      | Dịch vụ                                                                                     |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| **Edge & Delivery**        | Route 53, AWS WAF, AWS Amplify (lưu trữ frontend)                                          |
| **Định danh (Identity)**   | Amazon Cognito                                                                              |
| **Tầng API**               | Amazon API Gateway                                                                          |
| **Danh mục sản phẩm**      | Product Service (Lambda), DynamoDB (Catalog Table), S3 (Product Assets), AWS Backup        |
| **Đơn hàng**               | Order Service (Lambda), SQS Order Queue + Order DLQ                                        |
| **Thanh toán**             | Checkout Service (Lambda), Stripe                                                           |
| **Sự kiện & Thông báo**    | EventBridge, SQS Notification Queue + Notification DLQ                                     |
| **Chatbot AI**             | Chatbot Backend (Lambda), Amazon Lex                                                        |
| **Bảo mật & Giám sát**     | CloudWatch, AWS X-Ray, Amazon GuardDuty, AWS CloudTrail                                    |

#### 3.2. Luồng dữ liệu chi tiết

Các số bên dưới tương ứng với số thứ tự luồng trong sơ đồ kiến trúc:

1. **Truy cập của người dùng:** Trình duyệt của người dùng phân giải tên miền của trang web thông qua **Amazon Route 53**.
2. **Bảo vệ tại tầng Edge:** Route 53 chuyển traffic qua **AWS WAF**, lọc các request độc hại (giới hạn tốc độ, managed rule group) trước khi đến ứng dụng.
3. **Phân phối Frontend:** Route 53 định tuyến traffic hợp lệ đến **AWS Amplify**, nơi phục vụ ứng dụng frontend.
4. **Cổng vào API:** Frontend gọi backend thông qua **Amazon API Gateway** — điểm vào duy nhất cho toàn bộ REST API.
5. **Xác thực:** API Gateway kiểm tra danh tính người gọi bằng cách xác thực JWT do **Amazon Cognito** cấp, trước khi định tuyến request đến Lambda backend.
6. **Danh mục & Đặt hàng:** API Gateway định tuyến request danh mục đến **Product Service**, service này đọc/ghi dữ liệu sản phẩm trong **DynamoDB (Catalog Table)** — được sao lưu tự động qua **AWS Backup** — và lưu ảnh sản phẩm trong **S3 (Product Assets)**. Request đặt hàng được chuyển đến **Order Service**, service này đẩy message vào **SQS Order Queue**; các message xử lý thất bại nhiều lần sẽ được chuyển sang **Order DLQ** để xử lý sau.
7. **Thanh toán:** API Gateway định tuyến request thanh toán đến **Checkout Service**, service này gọi **Stripe** để xử lý giao dịch. Kết quả từ Stripe được dùng để cập nhật trạng thái đơn hàng thông qua Order Service.
8. **Phát sự kiện:** Sau khi hoàn tất thanh toán, **Checkout Service** phát một sự kiện đến **Amazon EventBridge**.
9. **Thông báo bất đồng bộ:** Luồng API Gateway/Order cũng phát các sự kiện thay đổi trạng thái đến **EventBridge**, sự kiện này được định tuyến đến **SQS Notification Queue** để gửi thông báo; các lần gửi thất bại sẽ rơi vào **Notification DLQ**.

Ngoài ra, **Chatbot Backend** (Lambda) tích hợp với **Amazon Lex** để vận hành trợ lý mua sắm AI, và toàn bộ hệ thống được giám sát liên tục bởi nhóm **Bảo mật & Giám sát** — **CloudWatch** (metric/log), **AWS X-Ray** (truy vết phân tán), **Amazon GuardDuty** (phát hiện mối đe dọa), và **AWS CloudTrail** (ghi log hoạt động API).

#### 3.3. Các điểm kiểm soát bảo mật

- **Bảo vệ tại Edge:** AWS WAF lọc traffic độc hại trước khi đến Amplify hoặc API Gateway.
- **Xác thực & phân quyền:** Amazon Cognito cấp JWT; API Gateway từ chối các request chưa xác thực.
- **Cơ chế đảm bảo tin cậy:** Mô hình SQS + DLQ trên cả luồng Đơn hàng và Thông báo đảm bảo không message nào bị mất âm thầm.
- **Độ bền dữ liệu:** AWS Backup cung cấp cơ chế sao lưu tự động, theo lịch cho bảng DynamoDB catalog.
- **Khả năng quan sát:** CloudWatch, X-Ray, GuardDuty và CloudTrail cùng nhau cung cấp khả năng giám sát toàn diện về hiệu năng, truy vết, mối đe dọa và hoạt động API.

---

### 4. NHÓM & PHÂN CÔNG (5 thành viên)

| Thành viên                              | Vai trò                                    | Nhiệm vụ chính                                                                                            |
| ---------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Thành viên 1                             | Kỹ sư Frontend                             | Giao diện Next.js, triển khai Amplify, các component giỏ hàng/thanh toán/chatbot.                        |
| Thành viên 2                             | Kỹ sư Auth & API                           | Cấu hình Cognito, cấu hình API Gateway, Lambda Product & Order Service.                                   |
| **Thành viên 3 (tôi — Nguyễn Trung Hiếu)** | **Kỹ sư Dữ liệu & Messaging**              | **Thiết kế DynamoDB single-table, tài nguyên S3, SQS Order Queue/DLQ, định tuyến EventBridge, AWS Backup, idempotency & test.** |
| Thành viên 4                             | Kỹ sư Thanh toán & Checkout                 | Checkout Service, tích hợp Stripe, xác thực chữ ký webhook, idempotency key.                               |
| Thành viên 5                             | Kỹ sư Thông báo, AI & Giám sát              | Notification Service, chatbot Amazon Lex, cấu hình CloudWatch/X-Ray/GuardDuty/CloudTrail.                  |

---

### 5. TIẾN ĐỘ (12 tuần)

- **Tuần 1–6:** Nền tảng AWS (theo chương trình FCJ).
- **Tuần 7–8:** Thu thập yêu cầu, xây dựng bản vẽ kiến trúc, thiết kế dữ liệu & messaging.
- **Tuần 9–12:** Triển khai, tích hợp, kiểm thử, viết tài liệu, và workshop tổng kết.

---

### 6. ƯỚC TÍNH CHI PHÍ

Tất cả các dịch vụ được lựa chọn đều có free tier hào phóng hoặc tính phí thuần theo mức sử dụng (DynamoDB on-demand, Lambda, SQS, EventBridge, Cognito). Chi phí ước tính hàng tháng cho lưu lượng demo là **dưới $5/tháng**, chủ yếu đến từ request API Gateway và log CloudWatch. Việc dọn dẹp nghiêm ngặt (`cdk destroy` sau mỗi lần demo) giúp giữ tổng chi phí trong phạm vi credit còn lại.

---

### 7. ĐÁNH GIÁ RỦI RO & BIỆN PHÁP GIẢM THIỂU

| Rủi ro                                          | Mức độ ảnh hưởng | Biện pháp giảm thiểu                                                                             |
| ------------------------------------------------- | ------------------ | -------------------------------------------------------------------------------------------------- |
| Lưu lượng tăng đột biến làm quá tải backend        | Trung bình          | Tự động mở rộng serverless (Lambda, DynamoDB on-demand, SQS đệm message).                          |
| Mất hoặc trùng lặp đơn hàng/thanh toán              | Cao                 | Mô hình SQS + DLQ, Stripe idempotency key, ghi có điều kiện (conditional writes) trong DynamoDB.    |
| Rò rỉ credential                                    | Cao                 | Lưu secret trong AWS Secrets Manager, không commit lên Git; IAM role theo nguyên tắc quyền tối thiểu. |
| Chậm tiến độ do độ phức tạp kiến trúc               | Cao                 | Xác định rõ phạm vi MVP (danh mục, thanh toán, chatbot); họp sync ngắn hàng tuần để gỡ vướng mắc.    |
