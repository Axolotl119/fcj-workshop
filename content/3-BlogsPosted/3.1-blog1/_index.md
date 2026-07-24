---
title: "Blog 1 — Relational Data Tier on Amazon RDS"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Building a Relational Data Tier on Amazon RDS for E-Commerce Systems

**Published at:** _(update post link)_

---

## 1. Introduction

In an E-Commerce system, the Data Tier plays a critical role. Unlike read-heavy applications or services that can tolerate temporary inconsistencies (Eventual Consistency), checkout and order management workflows demand absolute precision. A minor issue—such as an order being placed successfully without deducting inventory, or charging a customer twice for a single order—directly impacts customer experience and revenue.

To solve this challenge, adopting a Relational Database that strictly complies with **ACID (Atomicity, Consistency, Isolation, Durability)** standards is mandatory. In this article, we will build a robust relational data tier for an E-Commerce application using **PostgreSQL on Amazon RDS**, while applying cost-optimization strategies ideal for Dev/Test environments and student projects.

---

## 2. Why Choose PostgreSQL on Amazon RDS?

### 2.1 ACID Compliance for the Checkout Flow
ACID properties ensure data integrity across complex transaction scenarios:

* **Atomicity:** When a user clicks "Checkout", all operations—creating the order (`orders`), inserting order line items (`order_items`), deducting stock (`products`), and recording the payment (`payments`)—must execute as a single unit. If any step fails, the entire transaction rolls back to its initial state.
* **Consistency:** Data moves from one valid state to another, adhering strictly to all defined rules, constraints (Foreign Keys, Check Constraints, Unique Keys).
* **Isolation:** Concurrent transactions executed by multiple users purchasing the last available product item will not interfere with or overwrite each other's data.
* **Durability:** Once a transaction is committed, changes are permanently recorded and survive any subsequent hardware or power failure.

### 2.2 Benefits of a Managed Service (Amazon RDS)
Rather than self-hosting PostgreSQL on an EC2 instance, Amazon RDS offers significant management advantages:

* **Automated Backups:** Easily configure automatic daily backups and Point-in-Time Recovery (PITR).
* **Automated Patching:** AWS automatically manages OS and database engine security updates.
* **Seamless Scalability:** Upgrade compute specifications (Instance Classes) or scale storage volume with just a few clicks.

---

## 3. Database Schema Design

Below is the Core DDL schema defining the four key tables required for an E-Commerce checkout workflow: `users`, `products`, `orders`, and `payments`.

```sql
-- 1. Users Table
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 2. Products Table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(12, 2) NOT NULL CHECK (price >= 0),
    stock_quantity INT NOT NULL CHECK (stock_quantity >= 0),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 3. Orders Table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    total_amount DECIMAL(12, 2) NOT NULL CHECK (total_amount >= 0),
    status VARCHAR(50) NOT NULL DEFAULT 'PENDING', -- PENDING, PAID, CANCELLED
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 4. Order Items Table
CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INT NOT NULL REFERENCES products(product_id),
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(12, 2) NOT NULL
);

-- 5. Payments Table
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

## 4. Networking & Security Architecture

To securely connect and operate Amazon RDS in a development environment, proper configuration within AWS VPC is required:

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

### 4.1 VPC & Subnet Setup
* **VPC:** Use the default VPC or build a custom one with at least two subnets spanning across two distinct Availability Zones (AZs) (a mandatory requirement for RDS DB Subnet Groups).
* **Public Access:** Set `Publicly Accessible = Yes` (only recommended for Dev/Learning environments to allow direct connections from local clients like DBeaver, pgAdmin, or VS Code).
* **DNS Hostnames:** Ensure both `Enable DNS hostnames` and `Enable DNS resolution` options are turned on in VPC settings. This allows RDS to assign an accessible FQDN Endpoint.

### 4.2 Security Group Rules (Access Control)
Create a dedicated Security Group for the RDS Instance with strict Inbound Rules:

* **Type:** PostgreSQL (TCP)
* **Port:** `5432`
* **Source:** Select `My IP` (Restricts connection access exclusively to your public workstation IP). **Never open `0.0.0.0/0` to the internet to prevent brute-force attacks and port scanning.**

---

## 5. Cost Optimization for Dev/Student Environments

AWS resource costs can accumulate quickly without proper configuration. Here is the recommended parameter setup to stay within the **AWS Free Tier** limits (or incur minimal charges of a few dollars per month):

| Parameter | Recommended Setting | Purpose / Explanation |
| :--- | :--- | :--- |
| **Engine** | PostgreSQL (Latest Version) | Feature-rich with superior query execution performance. |
| **Instance Class** | `db.t4g.micro` | Powered by **AWS Graviton2** (ARM-based architecture). ~10% cheaper with better performance than `db.t3.micro`. |
| **Deployment** | Single-AZ | Runs a single database instance. Avoid Multi-AZ deployments in Dev to prevent doubling costs. |
| **Storage Type** | General Purpose SSD (`gp2` or `gp3`) | `gp2` up to 20 GB falls entirely within the Free Tier allowance. |
| **Allocated Storage** | `20 GB` | Minimum required capacity, sufficient for testing and dev workloads. |
| **Storage Auto-scaling**| Disabled | Prevents automatic storage expansion from generating unexpected charges. |

> **Key Cost-Saving Tip:**
> 1. **Stop Instance When Idle:** When finishing work (overnight or on weekends), go to the RDS Console and select **Stop instance**. While stopped, compute costs are paused, and you are only charged cents per month for the 20GB EBS storage. *(Note: RDS automatically restarts stopped instances after 7 days).*
> 2. **Disable Automated Backups:** Set `Backup retention period` to `0` or `1` day in testing environments to reduce backup storage usage.

---

## 6. Implementing Core Workflows

### 6.1 Authentication & Password Handling
Never store plain text passwords in the database. On the application backend, use a hashing algorithm like **bcrypt** before storing the hash string in the `password_hash` column.

```javascript
// Example: Password Hashing before DB Insertion in Node.js
const bcrypt = require('bcrypt');
const saltRounds = 10;

async function registerUser(email, plainPassword, fullName) {
    // 1. Generate hash from plain text password
    const passwordHash = await bcrypt.hash(plainPassword, saltRounds);
    
    // 2. Store user record with password hash in PostgreSQL
    const query = `
        INSERT INTO users (email, password_hash, full_name) 
        VALUES ($1, $2, $3) 
        RETURNING user_id, email;
    `;
    // Execute query passing parameterized values [email, passwordHash, fullName]
}
```

### 6.2 Checkout Flow as a Single Transaction
Below is a complete SQL transaction handling the checkout flow as an **Atomic Single Transaction**. If any step encounters an error (e.g., insufficient stock), all previous steps automatically roll back.

```sql
BEGIN;

-- 1. Create a new pending order
INSERT INTO orders (user_id, total_amount, status)
VALUES (1, 150.00, 'PENDING')
RETURNING order_id;

-- Assume the returned order_id is 100

-- 2. Insert line items into order_items table
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (100, 5, 2, 75.00);

-- 3. Deduct product inventory stock
-- (Table check constraint will throw an error if stock_quantity falls below 0)
UPDATE products
SET stock_quantity = stock_quantity - 2
WHERE product_id = 5 AND stock_quantity >= 2;

-- 4. Record payment transaction details
INSERT INTO payments (order_id, payment_method, amount, status)
VALUES (100, 'CREDIT_CARD', 150.00, 'SUCCESS');

-- 5. Update order status to paid
UPDATE orders
SET status = 'PAID'
WHERE order_id = 100;

COMMIT;
```

---

## 7. Amazon RDS Deployment Example

Below is a snapshot of the Amazon RDS Management Console after successfully provisioning a `db.t4g.micro` instance ready for incoming connections:

![RDS Console Screenshot](https://axolotl119.github.io/fcj-workshop/images/3-BlogsPosted/blog1-rds.png)
*Figure 1: Amazon RDS Console showing a running instance ready for database connection.*

---

## 8. Conclusion

Building a relational data tier on **Amazon RDS PostgreSQL** provides the transaction reliability required for E-Commerce applications through strict ACID compliance. Combined with proper network security (VPC, Security Groups) and cost-optimized instance sizing (`db.t4g.micro`), you can operate a production-ready database layer for development and testing at minimal cost.

In the next article, we will explore connecting a backend application to this RDS database using Connection Pooling to optimize query execution and concurrency handling.