# 🤖 Ý TƯỞNG VÀ KIẾN TRÚC AI SERVICE (HIRIFY - RAG & AGENTIC AI)

## 🎯 1. TỔNG QUAN TƯ DUY NGHỆ THUẬT & Ý TƯỞNG HỆ THỐNG AI

Dự án **Hirify** không chỉ đơn thuần là một website kết nối tuyển dụng thông thường, mà trọng tâm công nghệ nằm ở **Phân hệ Dịch vụ AI (AI Service)**. Ý tưởng cốt lõi của phân hệ này là giải quyết 3 bài toán lớn bằng Trí tuệ nhân tạo:

1. **Gợi ý & Ghép nối việc làm thông minh (Smart Job Matching):** Dựa trên kỹ năng và nguyện vọng của ứng viên chứ không chỉ tìm kiếm theo từ khóa cứng.
2. **Trợ lý AI đa năng (RAG Conversational Chatbot):** Đóng vai trò chuyên gia tư vấn sự nghiệp cho ứng viên và trợ lý lọc hồ sơ cho nhà tuyển dụng.
3. **Giám sát & Kiểm duyệt tự động (Automated Moderation System):** Thay thế Admin thủ công trong việc phát hiện mã độc, bài đăng rác, CV giả mạo, hành vi spam và lạm dụng chatbot.

---

## 💡 2. PHÂN TÍCH Ý TƯỞNG CÁC TÍNH NĂNG VÀ BÊN LIÊN QUAN

### 2.1 Đối với Ứng viên (Candidate - Khách hàng)
- **Tìm kiếm công việc theo độ phù hợp (Matching Score):** AI sẽ phân tích CV của ứng viên (học vấn, kỹ năng, kinh nghiệm) và so sánh ngữ nghĩa với Mô tả công việc (JD) để đưa ra **điểm số phù hợp (%)**.
- **Chatbot tư vấn sự nghiệp:** Ứng viên trò chuyện với Chatbot để được gợi ý "Công việc nào phù hợp với kỹ năng và mức lương tôi mong muốn?", Chatbot sẽ gọi API nội bộ để kéo dữ liệu việc làm thực tế và trả lời tự nhiên.
- **Tải CV lên hệ thống:** CV khi tải lên sẽ đi qua một màng lọc an toàn trước khi đến tay Nhà tuyển dụng hoặc Admin.

### 2.2 Đối với Nhà tuyển dụng (Recruiter)
- **Quản lý Thông tin & Nhiều Công ty:** Nhà tuyển dụng có thể đại diện hoặc quản lý bài đăng cho nhiều công ty khác nhau trên cùng một tài khoản.
- **Đánh giá & Lọc CV thông minh:** AI tự động trích xuất thông tin CV, chấm điểm độ khớp với JD giúp nhà tuyển dụng tiết kiệm thời gian đọc từng CV thủ công.

### 2.3 Đối với Hệ thống Admin (Kiểm soát Tự động bằng AI/Machine Learning)
Thay vì Admin phải ngồi duyệt từng CV hay bài đăng, AI sẽ làm nhiệm vụ bảo vệ hệ thống hoàn toàn tự động:
- **Kiểm soát bài đăng tuyển dụng:** Mô hình phân loại tự động quét nội dung tin tuyển dụng để chặn các bài đăng lừa đảo, đa cấp, nội dung nhạy cảm hoặc không đúng quy chuẩn (ví dụ: tuyển dụng sức khỏe/thực phẩm chức năng biến tướng).
- **Phân loại Mã độc & CV rác (Malware & Fake CV Detection):** Sử dụng các mô hình học máy phân loại file để phát hiện CV có chèn mã độc, script độc hại, hoặc các CV giả mạo cố tình spam hệ thống.
- **Kiểm soát Lạm dụng Chatbot (Anti-Prompt Injection / Anti-Spam):** Phát hiện ứng viên/người dùng cố tình "jailbreak", hỏi các câu không liên quan đến tuyển dụng, hoặc dùng script tự động gửi hàng loạt truy vấn để làm kiệt quệ tài nguyên AI.

---

## 🛠️ 3. KIẾN TRÚC CHI TIẾT CỦA AI SERVICE (THAM KHẢO TỪ RAG & PYTHON CODE)

Dựa trên việc tham khảo các file mã nguồn Python mẫu (`document_loader`, `text_splitter`, `vector_store`, `llm_service`, `background_tasks`, `router`), kiến trúc của **AI Service** được thiết kế theo luồng xử lý sau:

### 3.1 Luồng 1: Xử lý tài liệu & Tải CV bất đồng bộ (Background Pipeline)
Khi Ứng viên tải CV hoặc Nhà tuyển dụng đăng bài tuyển dụng (JD) mới:
1. **Trích xuất văn bản (Document Loader):** Đọc nội dung thô từ file PDF/Word (`document_loader.py`).
2. **Kiểm tra an toàn & Kiểm duyệt (Pre-moderation):** 
   - Mô hình Machine Learning quét kiểm tra mã độc và tính xác thực của CV.
   - Kiểm tra nội dung bài đăng xem có vi phạm chính sách hay không.
3. **Cắt nhỏ văn bản (Text Splitter):** Chia nhỏ nội dung thành các đoạn văn ngắn (Chunks) từ 500–1000 ký tự để AI dễ tiếp thu (`text_splitter.py`).
4. **Tạo tọa độ không gian (Vector Embedding):** Chuyển đổi các đoạn văn bản thành danh sách số (Vectors 768/3072 chiều) bằng mô hình Google Gemini Embedding (`gemini-embedding`).
5. **Lưu trữ Vector Database (`vector_store.py`):** Lưu các vector vào cơ sở dữ liệu PostgreSQL mở rộng plugin `pgvector`. Việc này chạy ngầm (Background Task) để không làm đơ giao diện người dùng.

### 3.2 Luồng 2: Trợ lý AI Hỏi - Đáp & Gợi ý Việc làm (RAG Chatbot Pipeline)
Khi người dùng đặt câu hỏi trên Chatbot:
1. **Chuyển câu hỏi thành Vector:** AI chuyển câu hỏi của người dùng thành một Vector đại diện.
2. **Tìm kiếm ngữ cảnh (Vector Search / Cosine Similarity):** Hệ thống quét trong `pgvector` để tìm ra các đoạn CV, bài đăng tuyển dụng hoặc quy định có nội dung tương đồng nhất với câu hỏi.
3. **Ghép ngữ cảnh & Gọi LLM (`llm_service.py`):**
   - Đóng gói: *Prompt = Ngữ cảnh tìm được + Thông tin công việc thực tế từ API + Câu hỏi của người dùng*.
   - Gửi đến mô hình ngôn ngữ lớn (Google Gemini Flash) để sinh ra câu trả lời thân thiện, chính xác, không bị "bốc phét" (Hallucination).

---

## 🔄 4. MÔ HÌNH HỌC MÁY & TRANSFORMER ÁP DỤNG TRONG KIỂM DUYỆT

Để đạt được mục tiêu kiểm soát tự động cho Admin, AI Service định hướng sử dụng các mô hình Machine Learning / Transformer thế hệ mới:

1. **Phân loại bài đăng & CV rác (Text Classification):**
   - Sử dụng các mô hình dạng **BERT / RoBERTa (Fine-tuned)** hoặc **Gemini Classification API** để gán nhãn bài đăng: `An toàn`, `Lừa đảo`, `Nội dung nhạy cảm/Sức khỏe`, `Cần Admin xem xét`.
2. **Phát hiện CV chứa Mã độc (File Security & Anomaly Detection):**
   - Trích xuất đặc trưng file (Phần dư thừa, Macro, Script nhúng trong PDF/Word) -> Đưa qua mô hình **Random Forest / XGBoost** hoặc **Deep Neural Network** để dự đoán xác suất độc hại.
3. **Tính điểm Phù hợp (Semantic Similarity Score):**
   - Sử dụng khoảng cách Cosine giữa Vector CV và Vector JD để tính **Độ phù hợp công việc (%)**.

---

## 🚀 5. LỘ TRÌNH TRIỂN KHAI & TÍCH HỢP HỆ THỐNG

1. **Giai đoạn 1: Chuẩn hóa dữ liệu & Môi trường Vector DB**
   - Cấu hình PostgreSQL hỗ trợ extension `pgvector`.
   - Thiết kế bảng lưu trữ `document_chunks`, `cv_embeddings`, `chat_sessions`.

2. **Giai đoạn 2: Xây dựng AI Service (Python FastAPI / Spring AI)**
   - Đóng gói các hàm xử lý RAG (Loader, Splitter, Embedding, LLM Call) thành các RESTful API độc lập.
   - Xây dựng hệ thống Background Worker xử lý quét CV và mã độc khi có lượt upload mới.

3. **Giai đoạn 3: Kết nối Backend chính & Frontend**
   - Backend chính (Java Spring Boot) đóng vai trò trung gian xác thực JWT, gửi request sang AI Service.
   - Frontend (React) hiển thị giao diện Chatbot gợi ý công việc, hiển thị phần trăm độ phù hợp (%) và trạng thái duyệt tự động của bài đăng.