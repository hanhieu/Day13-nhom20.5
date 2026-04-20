# Báo cáo Giải trình: Thiết lập SLO và Alert Rules cho Hệ thống AI/RAG

Báo cáo này trình bày chi tiết về các quyết định thiết kế khi định nghĩa các chỉ số Mục tiêu Chất lượng Dịch vụ (SLO) và Luật Cảnh báo (Alert Rules) cho hệ thống AI Math Tutor (dựa trên RAG và LLM). Đây là tài liệu tham khảo để nhóm bảo vệ đồ án và trả lời các câu hỏi từ giảng viên.

---

## 1. Triết lý thiết kế SLO cho hệ thống GenAI

Khác với các dịch vụ web truyền thống (thường yêu cầu thời gian phản hồi tính bằng mili-giây và độ ổn định 99.99%), các hệ thống ứng dụng LLM/RAG có đặc thù riêng:
- **Xử lý tốn thời gian**: Việc tìm kiếm tài liệu (retrieval) và sinh ngôn ngữ (generation) đòi hỏi nhiều thời gian tính toán.
- **Tính bất định cao**: Phụ thuộc vào chất lượng câu hỏi của người dùng và dịch vụ LLM của bên thứ ba (như OpenAI, Gemini), dẫn đến tỷ lệ lỗi hoặc time-out cao hơn bình thường.
- **Chi phí tỷ lệ thuận với độ dài**: Việc quản lý token và chi phí API là tối quan trọng để tránh "cháy túi".

Vì vậy, các chỉ số SLO của chúng ta được nới lỏng về mặt thời gian nhưng siết chặt về chất lượng và chi phí.

---

## 2. Chi tiết các chỉ số SLO (Service Level Objectives)

File tham chiếu: `config/slo.yaml`

### 2.1. Độ trễ P95 (Latency P95)
- **Mục tiêu (Objective):** `5000ms` (5 giây)
- **Mức cam kết (Target):** `95.0%`
- **Lý do lựa chọn:** 
  - **Tại sao không phải là 500ms hay 1s?** Trong kiến trúc RAG, một request phải đi qua các bước: *Nhúng câu hỏi (Embedding) -> Tìm kiếm Vector (Vector Search) -> Gọi LLM API sinh câu trả lời*. Toàn bộ quá trình này mất trung bình 2-4 giây. Việc đặt P95 ở mức 5 giây là thực tế (realistic) nhất.
  - **Tại sao Target là 95%?** Chúng ta chấp nhận 5% số lượng request (thường là các câu hỏi toán học rất phức tạp yêu cầu LLM suy luận dài) có thể vượt quá 5 giây mà không bị coi là vi phạm SLA chung.

### 2.2. Tỷ lệ lỗi (Error Rate)
- **Mục tiêu (Objective):** `5.0%`
- **Mức cam kết (Target):** `99.0%`
- **Lý do lựa chọn:**
  - LLM thỉnh thoảng có thể trả về định dạng sai (hallucination format) khiến việc parsing JSON bị lỗi, hoặc API bị rate-limit. Đặt ngưỡng lỗi 5% giúp hệ thống không phát tín hiệu báo động giả (false alarm) liên tục với các lỗi thoáng qua của GenAI.

### 2.3. Ngân sách chi phí hàng ngày (Daily Cost)
- **Mục tiêu (Objective):** `5.0` USD/ngày
- **Mức cam kết (Target):** `99.0%`
- **Lý do lựa chọn:**
  - Hệ thống RAG tiêu tốn token cho cả chiều in (context + prompt) và out (câu trả lời). Với ngân sách 5 USD/ngày, nhóm có đủ không gian để thực hiện các bài test tải (load testing) và phục vụ người dùng thử nghiệm mà không sợ vượt quá giới hạn tài chính sinh viên.

### 2.4. Điểm chất lượng trung bình (Quality Score)
- **Mục tiêu (Objective):** `0.80`
- **Mức cam kết (Target):** `95.0%`
- **Lý do lựa chọn:**
  - Trong RAG, chỉ có hệ thống chạy nhanh là chưa đủ, câu trả lời phải chính xác. Điểm 0.80 (đánh giá bằng framework như Ragas) đảm bảo rằng AI Tutor đưa ra lời giải toán đúng đắn, không bị "ảo giác" (hallucination) trong phần lớn các trường hợp.

---

## 3. Chi tiết các Luật cảnh báo (Alert Rules)

File tham chiếu: `config/alert_rules.yaml`

Các luật cảnh báo được xây dựng dựa trên SLO để đảm bảo đội ngũ phát triển (hoặc on-call) được thông báo ngay trước khi SLO bị vi phạm nghiêm trọng.

### 3.1. Cảnh báo Độ trễ (Latency Alerts)
- **`high_latency_p95` (P2):** P95 > 5000ms trong vòng 5 phút. 
  - *Giải thích:* Khớp đúng với SLO. Chỉ cảnh báo nếu tình trạng kéo dài 5 phút để loại trừ các spike (đỉnh nhọn) ngẫu nhiên của mạng lưới.
- **`extreme_latency_p99` (P1):** P99 > 15000ms (15 giây) trong vòng 2 phút.
  - *Giải thích:* Nếu top 1% người dùng phải đợi tới 15 giây, điều đó chứng tỏ có vấn đề nghẽn cổ chai nghiêm trọng (bottleneck) ở Vector DB hoặc LLM Provider. Đây là sự cố mức độ cao (P1) cần xử lý gấp.

### 3.2. Cảnh báo Tỷ lệ lỗi (Error Rate Alerts)
- **`high_error_rate` (P1):** Lỗi > 5.0% trong 3 phút.
  - *Giải thích:* Bắt đầu vi phạm SLO. Thời gian quan sát 3 phút là đủ nhanh để phát hiện LLM API bị sập.
- **`critical_error_rate` (P0):** Lỗi > 10% trong 1 phút.
  - *Giải thích:* Đây là mức độ thảm họa (P0). Rất có thể hết tiền tài khoản OpenAI, sai API Key, hoặc DB bị sập toàn tập.

### 3.3. Cảnh báo Chi phí (Cost Alerts)
- **`daily_cost_budget_exceeded` (P1):** Vượt 5.0 USD trong 1 giờ.
  - *Giải thích:* Nếu tốc độ đốt tiền vượt mốc 5$ (giới hạn một ngày) chỉ trong vòng 1 giờ, khả năng cao hệ thống đang bị tấn công DDoS (spam request) hoặc có bug lặp vô hạn (infinite loop) gọi LLM.

### 3.4. Cảnh báo Chất lượng (Quality Alerts)
- **`low_quality_score` (P2):** Chất lượng < 0.80 trong 10 phút.
  - *Giải thích:* Cảnh báo rằng mô hình LLM đang bị drift hoặc chất lượng bộ dữ liệu RAG đang trả về kết quả rác (ví dụ: mất kết nối tới ElasticSearch/Pinecone dẫn đến context bị rỗng).

---

## 4. Tổng kết (Để chốt với giảng viên)

Việc thiết lập SLO và Alert như trên cho thấy nhóm **hiểu rõ sự khác biệt giữa hệ thống Software Engineering truyền thống và AI/ML Engineering**. Thay vì theo đuổi những con số "đẹp" nhưng phi thực tế như latency < 200ms hay uptime 99.999%, nhóm đã chọn các chỉ số thiết thực, phản ánh đúng trải nghiệm người dùng cuối khi tương tác với một "AI Tutor", đồng thời bảo vệ hệ thống khỏi các rủi ro tài chính (Cost Alerts) đặc thù của LLM.
