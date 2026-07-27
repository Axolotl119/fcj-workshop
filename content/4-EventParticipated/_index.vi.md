---
title: "Sự kiện 3: FCAJ x Agentic AI Build Week 2026"
date: 2026-07-25
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# FCAJ x Agentic AI Build Week: Show Up. Build. Pitch. WIN!

* **Tên sự kiện:** FCAJ x Agentic AI Build Week 2026
* **Thời gian:** 25/07/2026
* **Hình thức:** Offline & Online (Livestream YouTube)
* **Đơn vị tổ chức:** AWS Study Group / First Cloud AI Journey (FCAJ) phối hợp cùng JI Fund
* **Video xem lại:**

<iframe 
  width="100%" 
  height="400" 
  src="https://www.youtube.com/embed/hz32VBrvW7M?start=3" 
  title="FCAJ x Agentic AI Build Week 2026" 
  frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" 
  allowfullscreen>
</iframe>

---

### BÁO CÁO TỔNG HỢP NỘI DUNG CHI TIẾT

#### I. KHAI MẠC & TƯ TƯỞNG ĐỊNH HƯỚNG (KEYNOTE SPEECH)
* **Khách mời đặc biệt:** Mr. Nguyễn Gia Hưng (*Head of Solution Architect - AWS Vietnam*) và Mr. Joseph Marazzota (*Head of Technology - AWS ASEAN*).
* **Thông điệp cốt lõi từ Mr. Joseph Marazzota:**
  * **Sự chuyển dịch Kỷ nguyên Phần mềm (Mental Model Shift):** 20 năm trước, các hệ thống ngân hàng/cốt lõi chỉ cập nhật 1 quý/lần hoặc 2 tuần/lần. Ngày nay, AI Agents cho phép triển khai và cập nhật liên tục tính theo từng phút.
  * **Tư duy Human-in-the-Loop:** Amazon hiện vận hành hơn 1 triệu Robot trong các trung tâm hậu cần nhưng chúng sẽ vô dụng nếu thiếu con người. Con người luôn là nhân tố quyết định cuối cùng trong việc thiết lập, điều phối và xác nhận đề xuất từ AI.
  * **Tinh thần học tập suốt đời (Lifelong Learner):** Thách thức những mô hình cũ, liên tục thử nghiệm thực tế qua các cuộc thi Hackathon để dẫn dắt sự thay đổi của ngành công nghiệp trong 2–3 năm tới.

---

#### II. PHẦN TRÌNH BÀY & DEMO DỰ ÁN CỦA CÁC ĐỘI THI

#### 1. Đội One Team – Dự án KFC AI Voice Order Agent (Giải Nhất Track AWS)
* **Thành viên:** 5 thành viên đa quốc gia (Mỹ, Ấn Độ, Việt Nam) kết nối tại cuộc thi.
* **Bài toán thực tế:** Bài học từ thất bại của McDonald's khi áp dụng AI Drive-thru (AI không hiểu ngữ cảnh hội thoại tự nhiên, gây ra lỗi Hallucination đặt nhầm 100 miếng gà nướng).
* **Giải pháp & Kiến trúc kỹ thuật:**
  * **No App Switching:** Cho phép khách hàng đặt món trực tiếp ngay trên Zalo/WhatsApp thông qua Webhook Adapter mà không cần tải ứng dụng mới hay tạo tài khoản.
  * **AWS Bedrock Agent Core:** Sử dụng tính năng Memory để lưu giữ lịch sử đặt hàng của từng người dùng qua nhiều tuần.
  * **Cào dữ liệu linh hoạt:** Dùng Tiny Fish scraping dữ liệu menu/khuyến mãi thực tế từ website KFC và lưu vào cơ sở dữ liệu AWS.
* **Chi phí & Hiệu năng:** 
  * Chi phí vận hành chỉ **$0.006 / đơn hàng**.
  * Tiết kiệm **75% chi phí Bedrock** và **60% chi phí hạ tầng** (tổng khoảng **$88/tháng** cho 500 đơn/ngày).
  * Độ trễ End-to-End chỉ từ **3 – 5 giây**.

#### 2. Đội Signal Scout – Dự án Multi-Agent Competitive Intelligence Canvas (Giải Nhì Track AWS)
* **Thành viên:** Sinh viên Đại học FPT (Hoàng Hiếu, Quốc Hào, Minh Quân, Công Minh, Di Khiêm, Tuấn Lực).
* **Bài toán thực tế:** Hỗ trợ đội ngũ làm chiến lược thu thập các tín hiệu kinh doanh rải rác (báo cáo tài chính, tài liệu cổ đông...) của đối thủ cạnh tranh để dự báo tỷ lệ hoàn vốn (ROI) khi áp dụng chiến lược mới.
* **Giải pháp & Kiến trúc kỹ thuật:**
  * **Frontend & Bảo mật:** Dashboard React host trên AWS Amplify, bảo mật qua AWS WAF và xác thực bằng Amazon Cognito.
  * **Multi-Agent Flow:** Con *Supervisor Agent* điều phối các Sub-agent:
    * *Crawler Sub-agent:* Dùng **Apify** (cho trang tĩnh) và **Tiny Fish** (cho trang động/bypass login wall).
    * *Analysis Sub-agent:* Xử lý dữ liệu qua **Bedrock Guardrails** và quan sát/chấm điểm chất lượng bằng **Langfuse**.
  * **Cơ chế tự sửa lỗi:** Nếu điểm dữ liệu thấp, hệ thống tự kích hoạt vòng lặp Retrying tối đa 2 lần trước khi gán tag cần con người Review.
* **Chi phí:** Khoảng **$35/tháng** (Mức dùng trung bình) và **$130/tháng** (Mức dùng tối đa).

#### 3. Đội S.A. Plan – Dự án SA Professional AI-Native Assistant
* **Thành viên:** Long, Vĩ, Phát, Ấn, Nghĩa.
* **Bài toán thực tế:** Giải quyết áp lực thời gian của Solution Architect (SA) khi khách hàng yêu cầu gấp sơ đồ kiến trúc và bảng tính chi phí trong đêm.
* **Giải pháp & Luồng vận hành:**
  * Đọc yêu cầu bằng ngôn ngữ tự nhiên (Free text) hoặc file tài liệu Policy/Constraint của doanh nghiệp.
  * Phân tích luồng ETL và tự động vẽ sơ đồ kiến trúc chuẩn trên **Draw.io**.
  * Tự động tính toán chi phí và sinh mã nguồn **Infrastructure as Code (IaC)** chuẩn module **Terraform/CloudFormation**.
  * **Cơ chế Validation:** Sử dụng danh sách Blacklist/Whitelist ngay tại cổng Output để ngăn chặn AI sinh ra các dịch vụ không được phép (như App Runner hay Elastic Beanstalk).

#### 4. Đội Team 3K – Dự án Sheper (Smart Human Flow & Queue Management)
* **Thành viên:** Nguyễn Huy, Huỳnh An Khương, Hoàng Minh Đức, Ngô Khôi, Đặng Nguyễn Phước Lộc.
* **Bài toán thực tế:** Theo dõi và giải tỏa tình trạng ùn tắc đám đông tại các khu vực cổng check-in sân bay, siêu thị hay sự kiện lớn.
* **Giải pháp & Kiến trúc kỹ thuật:**
  * **Video Streaming:** Thu luồng trực tiếp từ Camera/Điện thoại qua **Amazon Kinesis Video Streams**.
  * **Xử lý thị giác máy tính:** Chạy mô hình **YOLOv8 (Small)** kết hợp **ByteTrack** trên **AWS Fargate** để detect người, gán ID theo dõi real-time và đếm số lượng người theo từng Zone thiết lập.
  * **Lưu trữ & AI Agent:** Lưu dữ liệu mật độ vào DynamoDB/S3. Con AI Agent kết nối qua **Amazon Bedrock** (truy vấn Open AI/Claude) để tự động phân tích, cảnh báo khu vực quá tải và đề xuất hướng điều phối nhân sự.

#### 5. Đội Six Pillars – Dự án Adaptive Workflow Engine (Phòng chống rửa tiền - AML)
* **Thành viên:** Anh Việt, Nguyễn Văn Linh, Nguyên, Minh Nhật, Huyền.
* **Bài toán thực tế:** Khắc phục tỷ lệ cảnh báo sai (False Positive lên tới 90–95%) trong các hệ thống phòng chống rửa tiền truyền thống của ngân hàng, vốn gây lãng phí $20–$25 cho mỗi lần review thủ công.
* **Hạ tầng 3 lớp (3-Layer Architecture):**
  * *Layer 1 (Fast Detection):* Dùng **Amazon Kinesis**, **AWS Lambda** và mô hình **XGBoost** trên SageMaker để lọc nhanh 90–95% giao dịch bình thường với chi phí cực rẻ.
  * *Layer 2 (Core Investigation - Multi-Agent):* Sử dụng **AWS Step Functions** điều phối 3 Sub-agents (*KYC Agent*, *Money Flow Agent*, *Sanction Check Agent*) truy vấn dữ liệu từ **OpenSearch Vector DB** để tạo *Evidence File*. Sử dụng thêm một con LLM làm nhiệm vụ Double-check giảm thiểu Hallucination.
  * *Layer 3 (Case Management & Enterprise Security):* Tích hợp Dashboard cho chuyên viên đưa ra quyết định cuối (*Hold, Dismiss, Escalate*). Bảo mật toàn diện với **AWS KMS, Secret Manager, GuardDuty, Security Hub, CloudWatch** và **AWS X-Ray**.

---

![Hình ảnh](/images/4-EventParticipated/4.3-event3/25-7-2026.png)