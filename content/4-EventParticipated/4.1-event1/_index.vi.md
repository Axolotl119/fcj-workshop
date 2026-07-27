---
title: "FC Community Day"
date: 22026-06-27
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# FC Community Day

* **Tên sự kiện:** FC Community Day - Ứng dụng AI Agent, Cloud Infrastructure & Voice AI
* **Thời gian:** 04/07/2026
* **Hình thức:** Offline (Tầng 26 & 36) & Online (Livestream YouTube)
* **Video xem lại:**

<iframe 
  width="100%" 
  height="400" 
  src="https://www.youtube.com/embed/G8-WlI7f6dE?start=271" 
  title="FC Community Day" 
  frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" 
  allowfullscreen>
</iframe>

---

### BÁO CÁO TỔNG HỢP NỘI DUNG SỰ KIỆN

#### 1. Hành trình sự nghiệp & Cloud Agentic Platform
* **Diễn giả:** Anh Steve Trần (*Founder & CEO - Cloud Thinker*, cựu SA tại AWS).
* **Bài học & Nội dung chính:**
  * **Hành trình phát triển:** Xuất phát điểm từ vận hành server vật lý tại Contact Center, vượt qua nhiều thất bại để đạt chứng chỉ AWS và trở thành Solution Architect tại AWS Việt Nam chỉ trong 4 năm.
  * **Tác động của AI:** Sự xuất hiện của AI Coding làm thay đổi thị trường tuyển dụng; doanh nghiệp ưu tiên nhân sự Senior biết sử dụng AI hiệu quả.
  * **Giải pháp Cloud Thinker:** Giải quyết bài toán độ phức tạp hạ tầng (Complexity) và nợ công nghệ (Technical Debt) bằng Agentic Platform:
    * *Incident Response:* Xử lý sự cố theo phút thay vì theo giờ.
    * *Code Review & Security:* Pentest và kiểm tra mã nguồn tự động trước khi release lên Production.
    * *FinOps:* Tự động tối ưu chi phí tài nguyên AWS.
  * **Kiến trúc Single vs Multi-Agent:** Multi-Agent giúp giảm cửa sổ ngữ cảnh (Context Window) và quản lý phân quyền (RBAC) tốt hơn nhờ chia nhỏ công việc cho các Specialist Agents.

#### 2. Voice AI cho Doanh nghiệp
* **Diễn giả:** Anh Hiếu Nghị (*Renova Cloud*), Anh Kiệt (*AWS Student Community*), Anh Trung Đỗ (*Founder & CEO - R AI*).
* **Bài học & Nội dung chính:**
  * **Kiến trúc 3 thành phần cho tiếng Việt:** Do tiếng Việt là ngôn ngữ ít tài nguyên (Low Resource Language), mô hình tối ưu nhất bao gồm: 
    Audio Input $\rightarrow$ STT $\rightarrow$ LLM $\rightarrow$ TTS $\rightarrow$ Audio Output.
  * **Thực tế triển khai tại Ngân hàng:**
    * Kiểm soát nội dung và thực thi tác vụ (*Tool Calling*) như khóa thẻ, tra cứu tài khoản.
    * Xử lý ngắt lời tự nhiên (*Barge-in*), nhận diện quãng nghỉ và phân biệt giới tính để xưng hô Anh/Chị phù hợp.
    * Huấn luyện 10-20% dữ liệu giọng vùng miền (Accent) để nhận dạng chính xác.

#### 3. DevOps AI Agent
* **Diễn giả:** Chị Bảo & Anh Nguyên Nguyễn (*Cloud Engineers - Cloud Kinetics*).
* **Bài học & Nội dung chính:**
  * Giải quyết vấn đề dữ liệu phân tán (*Fragmented Telemetry*) gây kéo dài thời gian phục hồi hệ thống (MTTR).
  * **Quy trình 4 bước tự động:** Trigger/Phân loại $\rightarrow$ Điều tra Root Cause $\rightarrow$ Đề xuất phương án khắc phục (Mitigation) $\rightarrow$ Đề xuất cải thiện (Improvement).
  * Ứng dụng *Agent Space* xây dựng sơ đồ hạ tầng (Topology) giúp giảm thời gian xử lý sự cố lên tới 75–77%.

#### 4. Tuyển dụng & Quản trị Nhân sự với Amazon Q (Quick)
* **Diễn giả:** Anh Trường "Wynn" & Chị Minh Anh (*Noventic*).
* **Bài học & Nội dung chính:**
  * Khắc phục hạn chế của việc lọc CV thủ công và rủi ro lộ thông tin khi dùng AI Public.
  * Tạo các *Skill* tùy chỉnh (ví dụ: *HR Talent Review Assistant*) tích hợp dữ liệu từ OneDrive, Sharepoint, S3, Jira...
  * Tự động hóa tạo JD, đọc/trích xuất dữ liệu CV bằng OCR, đánh giá ứng viên theo bộ điểm kỹ thuật (Benchmark) và xuất báo cáo Dashboard trực quan.

#### 5. Thiết lập Kết nối Bảo mật Riêng tư cho Amazon Q qua MCP Server
* **Diễn giả:** Anh Toàn Nguyễn (*AWS Security Builder*) & Anh Hiếu Nghị (*Renova Cloud*).
* **Bài học & Nội dung chính:**
  * Loại bỏ rủi ro lộ thông tin hoặc bị tấn công DoS/MITM khi kết nối ra Internet.
  * Thiết lập mô hình Private Connection: Đặt MCP Server trong Private Subnet của VPC, tạo VPC Connection/Interface Endpoint để toàn bộ luồng dữ liệu truyền tải hoàn toàn trong mạng nội bộ AWS.
  * Kết hợp mã hóa TLS qua ALB, xác thực qua AWS Cognito và phân giải tên miền nội bộ bằng Route 53 Resolver (Chi phí duy trì hạ tầng ước tính khoảng $250 - $350/tháng).

---

![Ảnh sự kiện](/images/4-EventParticipated/4.1-event1/27-6-2026.png)