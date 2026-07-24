---
title: "Blog 1 — Tầng dữ liệu quan hệ trên Amazon RDS"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Xây dựng tầng dữ liệu quan hệ trên Amazon RDS cho hệ thống E-Commerce

**Đăng tại:** _(cập nhật link bài đăng)_

---

## 1. Giới thiệu

Trong một hệ thống Thương mại điện tử (E-Commerce), tầng dữ liệu (Data Tier) đóng vai trò vô cùng cốt lõi. Khác với các ứng dụng đọc nhiều ghi ít hoặc có thể chấp nhận sự bất đồng bộ tạm thời (Eventual Consistency), luồng thanh toán và quản lý đơn hàng đòi hỏi sự chính xác tuyệt đối. Một sai sót nhỏ như đặt hàng thành công nhưng không trừ kho, hoặc trừ tiền hai lần cho một đơn hàng, sẽ ảnh hưởng trực tiếp đến trải nghiệm người dùng và doanh thu.

Để giải quyết bài toán này, việc lựa chọn một Cơ sở dữ liệu quan hệ (Relational Database) hỗ trợ chuẩn **ACID (Atomicity, Consistency, Isolation, Durability)** là điều bắt buộc. Trong bài viết này, chúng ta sẽ cùng xây dựng tầng dữ liệu cho ứng dụng E-Commerce sử dụng **PostgreSQL trên Amazon RDS**, kết hợp với việc tối ưu hóa chi phí cực kỳ phù hợp cho môi trường Dev/Test hoặc các dự án sinh viên.

---

## 2. Vì sao chọn PostgreSQL trên Amazon RDS?

### 2.1 Tính tuân thủ ACID cho luồng Checkout
Chuẩn ACID đảm bảo tính toàn vẹn của dữ liệu trong các giao dịch phức tạp:

* **Atomicity (Tính nguyên tố):** Khi người dùng nhấn nút "Thanh toán", toàn bộ các thao tác — tạo đơn hàng (`orders`), tạo các mục đơn hàng (`order_items`), trừ tồn kho (`products`), và ghi nhận thanh toán (`payments`) — phải được thực thi thành một khối thống nhất. Nếu một bước thất bại, toàn bộ giao dịch sẽ quay về trạng thái ban đầu (Rollback).
* **Consistency (Tính nhất quán):** Dữ liệu luôn chuyển từ trạng thái hợp lệ này sang trạng thái hợp lệ khác, tuân thủ mọi ràng buộc (Foreign Key, Check constraint, Unique).
* **Isolation (Tính cô lập):** Hai người dùng cùng mua sản phẩm cuối cùng trong kho tại một thời điểm sẽ không thể ghi đè dữ liệu của nhau.
* **Durability (Tính bền vững):** Một khi giao dịch hoàn tất (Commit), dữ liệu sẽ được lưu trữ an toàn ngay cả khi gặp sự cố phần cứng.

### 2.2 Lợi ích của Managed Service (Amazon RDS)
Thay vì tự cài đặt PostgreSQL trên một máy chủ EC2 (Self-hosted), Amazon RDS mang lại nhiều lợi thế quản trị:

* **Tự động hóa sao lưu:** Dễ dàng cấu hình Backup tự động và khôi phục về mốc thời gian bất kỳ (Point-in-Time Recovery).
* **Quản lý bản vá (Patching):** AWS tự động cập nhật hệ điều hành và engine DB.
* **Khả năng mở rộng (Scaling):** Nâng cấp cấu hình (Instance Class) hoặc mở rộng dung lượng đĩa chỉ với vài cú click.

---

## 3. Thiết kế Cơ sở Dữ liệu (Schema Design)

Dưới đây là sơ đồ và cấu trúc DDL cơ bản cho 4 bảng quan trọng nhất trong luồng E-Commerce: `users`, `products`, `orders`, và `payments`.

```sql
-- 1. Bảng người dùng
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 2. Bảng sản phẩm
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(12, 2) NOT NULL CHECK (price >= 0),
    stock_quantity INT NOT NULL CHECK (stock_quantity >= 0),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 3. Bảng đơn hàng
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    total_amount DECIMAL(12, 2) NOT NULL CHECK (total_amount >= 0),
    status VARCHAR(50) NOT NULL DEFAULT 'PENDING', -- PENDING, PAID, CANCELLED
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 4. Bảng chi tiết đơn hàng
CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INT NOT NULL REFERENCES products(product_id),
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(12, 2) NOT NULL
);

-- 5. Bảng thanh toán
CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    order_id INT UNIQUE NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    payment_method VARCHAR(50) NOT NULL, -- CREDIT_CARD, MOMO, BANK_TRANSFER
    amount DECIMAL(12, 2) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'INITIATED',
    transaction_ref VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

## 4. Kiến trúc Mạng & Bảo mật (Networking & Security)

Để kết nối và vận hành RDS an toàn trong môi trường phát triển (Dev), chúng ta cần thiết lập cấu hình mạng chính xác trong AWS VPC:

```text
+-----------------------------------------------------------------------+
| Amazon VPC                                                            |
|                                                                       |
|   +---------------------------------------------------------------+   |
|   | Public Subnet                                                 |   |
|   |                                                               |   |
|   |   +-------------------+             +---------------------+   |   |
|   |   |  Security Group   |  Port 5432  |  Amazon RDS         |   |   |
|   |   |  (Allow IP dev)   | ----------->|  (PostgreSQL Engine)|   |   |
|   |   +-------------------+             +---------------------+   |   |
|   |                                                               |   |
|   +---------------------------------------------------------------+   |
+-----------------------------------------------------------------------+
```

### 4.1 Cấu hình VPC & Subnet
* **VPC:** Sử dụng VPC mặc định hoặc tạo mới với ít nhất hai Subnet nằm ở hai Availability Zone (AZ) khác nhau (đây là yêu cầu tối thiểu khi khởi tạo DB Subnet Group trên RDS).
* **Public Access:** Bật `Publicly Accessible = Yes` (chỉ áp dụng cho môi trường Dev/Học tập để dễ dàng kết nối trực tiếp từ DBeaver, pgAdmin hoặc VS Code ở máy local). 
* **DNS Hostnames:** Đảm bảo hai tùy chọn `Enable DNS hostnames` và `Enable DNS resolution` trong thiết lập VPC đều được bật. Điều này giúp RDS cấp phát một đường dẫn FQDN (Endpoint) hoàn chỉnh để truy cập.

### 4.2 Cấu hình Security Group (Phân quyền truy cập)
Tạo một Security Group riêng dành cho RDS Instance với quy tắc Inbound (Inbound Rules) được thắt chặt:

* **Type:** PostgreSQL (TCP)
* **Port:** `5432`
* **Source:** Chọn `My IP` (Chỉ cho phép duy nhất địa chỉ IP công cộng từ máy tính cá nhân của bạn kết nối vào). **Tuyệt đối không mở `0.0.0.0/0` ra toàn internet để tránh nguy cơ bị quét cổng và tấn công dò mật khẩu.**

---

## 5. Tối ưu Chi phí cho Môi trường Dev/Sinh viên

Chi phí tài nguyên trên AWS có thể tăng nhanh nếu không chọn đúng thông số. Dưới đây là bộ cấu hình tối ưu giúp bạn duy trì hệ thống trong hạn mức **AWS Free Tier** (hoặc chỉ tiêu tốn vài USD/tháng nếu hết hạn Free Tier):

| Thông số | Cấu hình khuyên dùng | Mục đích / Giải thích |
| :--- | :--- | :--- |
| **Engine** | PostgreSQL (Phiên bản mới nhất) | Đầy đủ các tính năng nâng cao, hiệu năng truy vấn tối ưu. |
| **Instance Class** | `db.t4g.micro` | Sử dụng vi xử lý **AWS Graviton2** (kiến trúc ARM). Chi phí rẻ hơn ~10% và hiệu năng vượt trội hơn so với dòng `db.t3.micro`. |
| **Deployment** | Single-AZ | Chỉ chạy 1 node DB duy nhất. Không chọn Multi-AZ cho môi trường Dev để tránh bị nhân đôi chi phí. |
| **Storage Type** | General Purpose SSD (`gp2` hoặc `gp3`) | Mức `gp2` dung lượng 20 GB hoàn toàn nằm trong gói Free Tier. |
| **Allocated Storage** | `20 GB` | Dung lượng đĩa tối thiểu, quá đủ cho nhu cầu học tập và kiểm thử. |
| **Storage Auto-scaling**| Tắt (Disable) | Ngăn chặn việc dữ liệu thử nghiệm tự động phình to làm phát sinh chi phí ngoài dự kiến. |

> **Mẹo tiết kiệm chi phí quan trọng:**
> 1. **Tắt Database khi không sử dụng (Stop Instance):** Khi dừng làm việc (buổi tối hoặc cuối tuần), hãy vào RDS Console và chọn **Stop instance**. Khi ở trạng thái Stop, AWS sẽ không tính phí tính toán (Compute), bạn chỉ tốn vài cent/tháng cho dung lượng lưu trữ đĩa (Storage). *(Lưu ý: RDS sẽ tự động bật lại sau 7 ngày nếu bạn quên).*
> 2. **Tắt Automated Backups:** Trong môi trường test, hãy đặt `Backup retention period` về `0` hoặc `1` ngày để tránh tốn dung lượng lưu trữ các bản sao lưu tự động.

---

## 6. Hiện thực hóa các tính năng cốt lõi

### 6.1 Đăng ký/Đăng nhập (Xử lý Mật khẩu)
Tuyệt đối không lưu mật khẩu ở dạng văn bản thuần (Plaintext). Ở tầng ứng dụng (Backend), bạn cần sử dụng thuật toán băm như **bcrypt** để mã hóa mật khẩu trước khi lưu chuỗi hash vào cột `password_hash`.

```javascript
// Minh họa mã hóa mật khẩu trước khi lưu vào DB trong Node.js
const bcrypt = require('bcrypt');
const saltRounds = 10;

async function registerUser(email, plainPassword, fullName) {
    // 1. Tạo chuỗi hash từ mật khẩu thô
    const passwordHash = await bcrypt.hash(plainPassword, saltRounds);
    
    // 2. Lưu chuỗi hash vào PostgreSQL
    const query = `
        INSERT INTO users (email, password_hash, full_name) 
        VALUES ($1, $2, $3) 
        RETURNING user_id, email;
    `;
    // Thực thi query với các tham số truyền vào [email, passwordHash, fullName]
}
```

### 6.2 Luồng Checkout dạng Single Transaction
Dưới đây là đoạn mã SQL thể hiện trọn vẹn quy trình thanh toán đơn hàng trong **1 Transaction duy nhất**. Nếu xảy ra sự cố ở bất kỳ bước nào (ví dụ: sản phẩm hết hàng), toàn bộ các thao tác trước đó sẽ được tự động Rollback.

```sql
BEGIN;

-- 1. Tạo đơn hàng mới ở trạng thái chờ
INSERT INTO orders (user_id, total_amount, status)
VALUES (1, 150.00, 'PENDING')
RETURNING order_id;

-- Giả định order_id vừa tạo được trả về là 100

-- 2. Thêm danh sách sản phẩm vào bảng chi tiết đơn hàng
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (100, 5, 2, 75.00);

-- 3. Trừ số lượng tồn kho của sản phẩm
-- (Check constraint ở bảng products sẽ tự ngắt & báo lỗi nếu stock_quantity < 0)
UPDATE products
SET stock_quantity = stock_quantity - 2
WHERE product_id = 5 AND stock_quantity >= 2;

-- 4. Ghi nhận thông tin giao dịch thanh toán
INSERT INTO payments (order_id, payment_method, amount, status)
VALUES (100, 'CREDIT_CARD', 150.00, 'SUCCESS');

-- 5. Đánh dấu đơn hàng hoàn tất thanh toán
UPDATE orders
SET status = 'PAID'
WHERE order_id = 100;

COMMIT;
```

---

## 7. Minh họa triển khai trên Amazon RDS

Dưới đây là hình ảnh giao diện quản lý Amazon RDS Instance sau khi khởi tạo thành công với cấu hình `db.t4g.micro` và sẵn sàng nhận kết nối:

![Ảnh chụp RDS console](/images/3-BlogsPosted/blog1-rds.png)
*Hình 1: Cấu hình bảng điều khiển Amazon RDS Instance sẵn sàng kết nối.*

---

## 8. Kết luận

Việc xây dựng tầng dữ liệu trên **Amazon RDS PostgreSQL** mang lại sự an toàn và tin cậy cho luồng xử lý giao dịch E-Commerce nhờ tính tuân thủ nghiêm ngặt chuẩn ACID. Kết hợp với việc tối ưu hệ thống mạng (VPC, Security Group) và lựa chọn đúng dòng instance tiết kiệm (`db.t4g.micro`), bạn hoàn toàn có thể xây dựng một hệ thống vững chắc phục vụ cho công việc nghiên cứu, thử nghiệm với mức chi phí tối ưu nhất.

Trong bài viết tiếp theo, chúng ta sẽ cùng tìm hiểu cách kết nối ứng dụng backend với RDS Database thông qua cơ chế Connection Pooling để nâng cao khả năng chịu tải cho hệ thống.