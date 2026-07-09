---
title: "Blog 1 — Tầng dữ liệu quan hệ trên Amazon RDS"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Xây dựng tầng dữ liệu quan hệ trên Amazon RDS cho E-Commerce

<!-- TODO: dán link bài đăng thật vào đây -->
**Đăng tại:** _(cập nhật link)_

### Tóm tắt

Bài viết trình bày cách dựng **PostgreSQL trên Amazon RDS** tối ưu chi phí (db.t4g.micro, Single-AZ, gp2 20GB) cho bài toán e-commerce:

* Thiết kế schema ACID cho `users`, `orders`, `payments`, và vì sao transaction quan hệ phù hợp luồng checkout.
* Quyết định về mạng: đặt DB trong VPC với public access cho môi trường dev, security group, bật DNS hostnames.
* Cài đặt đăng ký/đăng nhập với bcrypt và checkout là một transaction SQL duy nhất.
* Mẹo tiết kiệm cho sinh viên: Stop instance khi không dùng, chọn Single-AZ và instance Graviton burstable.

![Ảnh chụp RDS console](/images/3-Blog/blog1-rds.png)
<!-- TODO: chèn ảnh chụp màn hình RDS console -->
