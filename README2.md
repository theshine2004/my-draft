# Hirify — README3.md
## Định hướng nghiên cứu và thiết kế AI cho hệ thống tuyển dụng Hirify

## 1. Giới thiệu

Hirify là hệ thống hỗ trợ tuyển dụng, kết nối ứng viên với nhà tuyển dụng thông qua các chức năng đăng tin tuyển dụng, tìm kiếm việc làm, quản lý hồ sơ ứng viên và ứng tuyển trực tuyến.

Trong hệ thống này, trí tuệ nhân tạo (AI) được định hướng sử dụng để hỗ trợ những công việc tốn nhiều thời gian như phân tích CV, gợi ý việc làm phù hợp, hỗ trợ người dùng tìm kiếm thông tin và phát hiện các nội dung có dấu hiệu bất thường.

Mục tiêu không phải là để AI thay thế hoàn toàn nhà tuyển dụng hoặc quản trị viên, mà là giúp họ xử lý thông tin nhanh hơn, giảm công việc thủ công và có thêm dữ liệu tham khảo khi đưa ra quyết định.

Tài liệu này tập trung vào việc nghiên cứu và đề xuất cách xây dựng các chức năng AI cho Hirify bằng **Java 17 và Spring Boot 3.5.6**, kết hợp các dịch vụ AI hiện có. Đây là tài liệu định hướng thiết kế, không khẳng định các chức năng được đề xuất đã triển khai hoàn chỉnh.

## 2. Công nghệ đang sử dụng và công nghệ dự kiến bổ sung

Một nguyên tắc quan trọng là tận dụng nền tảng hiện tại của Hirify, hạn chế thay đổi kiến trúc chỉ để tích hợp AI.

### 2.1. Công nghệ hiện có

| Thành phần | Công nghệ | Vai trò trong hệ thống |
|---|---|---|
| Ngôn ngữ backend | Java 17 | Xây dựng nghiệp vụ tuyển dụng và tích hợp AI |
| Backend framework | Spring Boot 3.5.6 | Xây dựng REST API và quản lý các dịch vụ |
| Quản lý thư viện | Apache Maven | Quản lý dependency và quá trình build |
| Bảo mật | Spring Security | Kiểm soát xác thực và quyền truy cập |
| Xác thực | JJWT 0.11.5 | Xử lý Access Token và Refresh Token |
| Mã hóa mật khẩu | BCrypt | Bảo vệ mật khẩu trước khi lưu trữ |
| ORM | Spring Data JPA / Hibernate | Truy vấn và quản lý dữ liệu |
| Cơ sở dữ liệu | PostgreSQL | Lưu trữ dữ liệu chính của hệ thống |
| Cơ sở dữ liệu kiểm thử | H2 Database | Hỗ trợ kiểm thử cục bộ |
| Tài liệu API | Springdoc OpenAPI 2.8.5 | Kiểm tra và mô tả REST API |
| Frontend | React 19, JSX | Xây dựng giao diện người dùng |
| Build frontend | Vite 8 | Phát triển và đóng gói frontend |
| Giao diện | Tailwind CSS v4, Lucide React | Thiết kế giao diện và biểu tượng |
| Điều hướng | React Router 8 | Điều hướng giữa các trang và dashboard |
| Gọi API | Axios 1.20.0 | Kết nối frontend với backend |
| Quản lý trạng thái | React Context API | Quản lý thông tin đăng nhập và công ty |
| Dịch vụ AI | OpenAI API, Google Gemini API | Cung cấp khả năng xử lý ngôn ngữ tự nhiên |
| Kiểm tra CV | `SecurityTransformerScannerService` | Module hiện có được mô tả là mô phỏng quá trình kiểm tra CV |

Các thư viện và phiên bản trên được ghi nhận theo cấu hình dự án đã cung cấp. Khi triển khai thực tế, cần kiểm tra dependency, API và khả năng tương thích của từng phiên bản.

### 2.2. Công nghệ đề xuất bổ sung

Những công nghệ sau là phương án có thể xem xét trong quá trình phát triển, không có nghĩa chúng đã được cài đặt hoặc tích hợp.

| Công nghệ / công cụ | Mục đích dự kiến | Mức độ ưu tiên |
|---|---|---|
| Spring `RestClient` hoặc `WebClient` | Gọi OpenAI API, Gemini API và các dịch vụ bên ngoài | Cần thiết khi tích hợp |
| Jackson | Đọc và chuyển đổi JSON từ phản hồi AI | Tận dụng hệ sinh thái Spring Boot |
| Apache PDFBox | Trích xuất văn bản từ CV dạng PDF | Cao |
| Apache Tika | Nhận diện định dạng và trích xuất nội dung tài liệu | Cao |
| Apache POI | Đọc tài liệu Word nếu hệ thống cho phép tải lên DOCX | Theo nhu cầu |
| Apache Tika kết hợp thư viện kiểm tra file | Nhận diện định dạng, kiểm tra các thuộc tính cơ bản của tệp | Cao |
| ClamAV | Quét mã độc trong tệp tải lên | Cao nếu triển khai kiểm tra mã độc |
| OWASP Dependency-Check | Kiểm tra lỗ hổng trong dependency của ứng dụng | Nên có cho bảo mật dự án |
| Flyway | Quản lý phiên bản thay đổi cấu trúc cơ sở dữ liệu | Tùy tình trạng hiện tại |
| pgvector | Lưu trữ và tìm kiếm vector embedding trong PostgreSQL | Khi triển khai RAG hoặc tìm kiếm ngữ nghĩa |

**Lưu ý:** Không cần bổ sung toàn bộ các công nghệ trên ngay từ đầu. Có thể triển khai từng phần để tránh làm hệ thống phức tạp khi chưa có nhu cầu thực tế.

## 3. Kiến trúc AI được đề xuất

AI được tích hợp như một nhóm dịch vụ bên trong backend Spring Boot hiện tại. Cách làm này giúp hệ thống giữ nguyên ngôn ngữ Java, cơ chế xác thực và mô hình phân quyền đã có.

Luồng hoạt động tổng quát:

```text
                    REACT 19 FRONTEND
                           |
                     Axios + JWT
                           |
                           v
                 SPRING BOOT 3.5.6
                           |
            +--------------+--------------+
            |              |              |
            v              v              v
       JobService      ResumeService    ChatService
            |              |              |
            +--------------+--------------+
                           |
                    AI Integration Layer
                           |
              +------------+------------+
              |                         |
              v                         v
         OpenAI API                Gemini API
              |                         |
              +------------+------------+
                           |
                    Kết quả xử lý AI
                           |
                           v
                  PostgreSQL Database
```

Đây là sơ đồ kiến trúc dự kiến. Các lớp dịch vụ, cách gọi API và luồng xử lý cần được đối chiếu với mã nguồn thực tế trước khi triển khai.

### 3.1. Vai trò của từng thành phần

**React 19:** Hiển thị giao diện chatbot, trang tìm kiếm việc làm, kết quả phân tích CV và màn hình kiểm duyệt cho quản trị viên.

**Spring Boot 3.5.6:** Tiếp nhận yêu cầu từ frontend, kiểm tra người dùng, phân quyền, xác thực dữ liệu, gọi dịch vụ AI và lưu kết quả cần thiết.

**AI Integration Layer:** Là lớp Java trung gian chịu trách nhiệm giao tiếp với OpenAI và Gemini. Lớp này quản lý cấu hình, prompt, thời gian chờ, lỗi API và việc chuyển đổi phản hồi thành dữ liệu mà hệ thống có thể sử dụng.

**OpenAI API và Gemini API:** Hỗ trợ các tác vụ như hiểu câu hỏi tự nhiên, tóm tắt CV, trích xuất thông tin, giải thích mức độ phù hợp giữa ứng viên và công việc. Hai dịch vụ có thể được đánh giá để lựa chọn một dịch vụ chính, hoặc dùng kết hợp khi có lý do rõ ràng.

**PostgreSQL:** Tiếp tục lưu trữ người dùng, công ty, tin tuyển dụng, hồ sơ ứng viên và các kết quả AI cần lưu lại. Nếu cần tìm kiếm ngữ nghĩa, có thể nghiên cứu mở rộng bằng pgvector.

## 4. Tổ chức mã nguồn Java cho AI

Thay vì tạo một dịch vụ Python riêng, có thể tổ chức các thành phần AI trong backend Spring Boot theo hướng sau:

```text
src/main/java/.../
├── ai/
│   ├── config/
│   │   └── AiConfig.java
│   ├── client/
│   │   ├── OpenAiClient.java
│   │   └── GeminiClient.java
│   ├── service/
│   │   ├── AiChatService.java
│   │   ├── ResumeAnalysisService.java
│   │   ├── JobRecommendationService.java
│   │   ├── JobMatchingService.java
│   │   └── ContentModerationService.java
│   ├── dto/
│   │   ├── ChatRequest.java
│   │   ├── ChatResponse.java
│   │   ├── ResumeAnalysisResponse.java
│   │   └── JobMatchResponse.java
│   └── controller/
│       └── AiController.java
├── resume/
├── job/
├── company/
├── user/
└── security/
```

Đây là cấu trúc tham khảo, không phải mô tả chính xác cấu trúc thư mục đang có. Tên package và vị trí các lớp cần điều chỉnh cho phù hợp với dự án hiện tại.

### 4.1. Công cụ gọi API AI

Có thể sử dụng `RestClient` của Spring Framework để gọi dịch vụ AI thông qua HTTP. Nếu hệ thống cần xử lý bất đồng bộ hoặc có nhiều yêu cầu I/O đồng thời, có thể cân nhắc `WebClient`.

Nên đóng gói phần giao tiếp API thành các lớp riêng để tránh viết mã gọi AI trực tiếp trong controller.

Các lớp client nên chịu trách nhiệm:

- Đính kèm API key từ cấu hình bảo mật.
- Gửi prompt và dữ liệu đầu vào theo đúng định dạng của nhà cung cấp.
- Đọc phản hồi JSON.
- Xử lý lỗi xác thực, giới hạn tốc độ, timeout và lỗi từ dịch vụ bên ngoài.
- Giới hạn số lần thử lại, tránh tạo quá nhiều yêu cầu hoặc chi phí ngoài dự kiến.
- Ghi log kỹ thuật mà không vô tình lưu API key, CV đầy đủ hoặc dữ liệu cá nhân nhạy cảm.

Không nên gọi cả OpenAI và Gemini cho mọi yêu cầu. Việc lựa chọn nhà cung cấp cần dựa trên chất lượng đầu ra, chi phí, độ trễ, giới hạn sử dụng và yêu cầu bảo vệ dữ liệu.

## 5. Các chức năng AI dự kiến

### 5.1. Trợ lý chatbot tìm kiếm việc làm

Chatbot giúp ứng viên tìm việc bằng ngôn ngữ tự nhiên thay vì phải nhập chính xác từ khóa.

Ví dụ, ứng viên có thể hỏi:

> Tôi mới tốt nghiệp ngành Công nghệ thông tin, biết Java và SQL, muốn tìm việc Backend tại Hà Nội và ưu tiên vị trí thực tập hoặc fresher.

Chatbot có thể thực hiện các bước:

1. Đọc câu hỏi và nhận diện kỹ năng, vị trí mong muốn, địa điểm, cấp bậc và các điều kiện liên quan.
2. Chuyển những thông tin đã nhận diện thành bộ tiêu chí tìm kiếm.
3. Gọi REST API nội bộ của Hirify để lấy các tin tuyển dụng phù hợp.
4. Sắp xếp kết quả theo tiêu chí và mức độ liên quan.
5. Trình bày danh sách việc làm kèm lý do gợi ý.

**Công nghệ đề xuất:**

- Java 17 và Spring Boot 3.5.6 để xử lý nghiệp vụ.
- OpenAI API hoặc Gemini API để hiểu câu hỏi tự nhiên và trích xuất tiêu chí.
- Spring Data JPA để truy vấn tin tuyển dụng.
- PostgreSQL để lưu thông tin công việc.
- React 19 và Axios để hiển thị hội thoại và kết quả.

AI không nên tự tạo ra tin tuyển dụng hoặc mức lương không tồn tại. Danh sách công việc phải lấy từ dữ liệu thật trong hệ thống.

### 5.2. Phân tích CV tự động

Khi ứng viên tải CV lên, hệ thống có thể hỗ trợ trích xuất và chuẩn hóa thông tin để giảm công việc nhập liệu.

Quy trình dự kiến:

1. Kiểm tra quyền tải lên, dung lượng, loại tệp và giới hạn số lượng.
2. Đưa tệp qua bước kiểm tra an toàn trước khi xử lý nội dung.
3. Sử dụng Apache PDFBox hoặc Apache Tika để trích xuất văn bản từ PDF; bổ sung Apache POI nếu cần hỗ trợ DOCX.
4. Gửi phần văn bản cần thiết đến OpenAI hoặc Gemini để trích xuất thông tin có cấu trúc.
5. Chuẩn hóa các trường như kỹ năng, kinh nghiệm, học vấn, chứng chỉ và vị trí công việc mong muốn.
6. Trả kết quả cho ứng viên xác nhận hoặc lưu lại theo chính sách dữ liệu của hệ thống.

Ví dụ, kết quả phân tích có thể chứa:

```json
{
  "skills": ["Java", "Spring Boot", "SQL"],
  "experienceLevel": "Fresher",
  "education": "Information Technology",
  "suggestedRoles": [
    "Java Developer",
    "Backend Developer"
  ]
}
```

Đây chỉ là ví dụ minh họa định dạng dữ liệu. Kết quả thực tế phải được kiểm tra và chuyển đổi từ phản hồi của mô hình AI.

**Giới hạn quan trọng:** AI có thể trích xuất sai hoặc bỏ sót thông tin trong CV. Vì vậy, ứng viên cần được quyền kiểm tra và chỉnh sửa thông tin trước khi sử dụng trong hồ sơ chính thức.

### 5.3. Gợi ý việc làm phù hợp với CV

Sau khi phân tích CV và lấy được thông tin tuyển dụng, hệ thống có thể so sánh hồ sơ với các vị trí đang mở.

Có thể kết hợp hai phương pháp:

**Phương pháp thứ nhất — so khớp tiêu chí:** Sử dụng truy vấn PostgreSQL và Spring Data JPA để lọc theo kỹ năng, địa điểm, cấp bậc, loại hình công việc và các điều kiện bắt buộc.

**Phương pháp thứ hai — so khớp ngữ nghĩa:** Sử dụng embedding để biểu diễn nội dung CV và mô tả công việc thành vector, sau đó tìm các nội dung gần nhau về ngữ nghĩa. Nếu cần lưu vector trực tiếp trong PostgreSQL, có thể xem xét pgvector.

Ví dụ, CV ghi “xây dựng REST API bằng Spring Boot”, còn tin tuyển dụng ghi “phát triển dịch vụ backend Java”. Hai nội dung có thể liên quan dù cách diễn đạt không hoàn toàn giống nhau.

Một công thức điểm phù hợp minh họa:

\[
S = w_1 S_{\text{skills}} +
    w_2 S_{\text{semantic}} +
    w_3 S_{\text{experience}} +
    w_4 S_{\text{preferences}}
\]

Trong đó:

- \(S_{\text{skills}}\): Mức độ khớp kỹ năng.
- \(S_{\text{semantic}}\): Mức độ tương đồng ngữ nghĩa.
- \(S_{\text{experience}}\): Mức độ phù hợp về kinh nghiệm hoặc cấp bậc.
- \(S_{\text{preferences}}\): Mức độ phù hợp với địa điểm và mong muốn của ứng viên.
- \(w_1, w_2, w_3, w_4\): Trọng số cần được xác định và đánh giá bằng dữ liệu thử nghiệm.

Đây là mô hình đề xuất, chưa phải công thức đã được kiểm chứng. Điểm phù hợp chỉ nhằm hỗ trợ tìm kiếm, không phải xác suất ứng viên sẽ được tuyển dụng.

### 5.4. Kiểm duyệt tin tuyển dụng

Nhà tuyển dụng có thể đăng tin cho một hoặc nhiều công ty mà họ được phép quản lý. Tin mới đăng có thể được đưa vào trạng thái chờ duyệt.

AI có thể hỗ trợ phát hiện:

- Nội dung tuyển dụng thiếu thông tin quan trọng.
- Tin trùng lặp hoặc có dấu hiệu đăng hàng loạt.
- Nội dung có dấu hiệu lừa đảo hoặc yêu cầu bất thường.
- Thông tin không nhất quán giữa tiêu đề, mô tả công việc và quyền lợi.
- Nội dung quảng cáo không liên quan đến tuyển dụng.

Quy trình dự kiến:

1. Nhà tuyển dụng gửi tin đăng.
2. Spring Boot kiểm tra dữ liệu và quyền thao tác.
3. `ContentModerationService` gửi phần nội dung phù hợp đến mô hình AI.
4. Hệ thống lưu kết quả phân tích và lý do cảnh báo.
5. Quản trị viên xem xét và quyết định duyệt, từ chối hoặc yêu cầu chỉnh sửa.

AI chỉ nên đưa ra đề xuất và mức độ rủi ro. Những trường hợp có ảnh hưởng lớn đến quyền đăng tuyển cần có cơ chế kiểm duyệt của con người và cho phép xử lý lại khi có khiếu nại.

### 5.5. Kiểm tra CV giả mạo, spam và tệp độc hại

Cần phân biệt ba nhiệm vụ khác nhau: kiểm tra an toàn của tệp, phát hiện nội dung spam và đánh giá dấu hiệu thông tin không nhất quán.

**Kiểm tra tệp độc hại**

Có thể dùng ClamAV hoặc giải pháp quét mã độc tương đương để kiểm tra tệp tải lên. Apache Tika hỗ trợ nhận diện định dạng và trích xuất nội dung, nhưng bản thân Tika không thay thế phần mềm chống mã độc.

Nên bổ sung các biện pháp:

- Giới hạn dung lượng và định dạng được phép.
- Kiểm tra chữ ký tệp thay vì chỉ tin phần mở rộng.
- Lưu tệp ở vị trí không thể thực thi trực tiếp.
- Không mở hoặc thực thi tệp tải lên như chương trình.
- Cách ly tệp đáng ngờ và ngăn việc chuyển tiếp cho nhà tuyển dụng trước khi có kết quả kiểm tra.
- Xem xét sử dụng môi trường cách ly khi cần phân tích tệp có rủi ro cao.

**Phát hiện spam**

AI có thể hỗ trợ nhận diện nội dung lặp lại, nội dung quảng cáo không phù hợp và hành vi gửi yêu cầu bất thường. Spring Boot cũng cần áp dụng giới hạn tần suất, giới hạn số lần gọi chatbot và kiểm soát số lần gửi biểu mẫu.

**Phát hiện dấu hiệu CV giả mạo**

Mô hình AI có thể đánh dấu những dấu hiệu như mốc thời gian mâu thuẫn, thông tin không nhất quán hoặc nội dung được sao chép bất thường. Tuy nhiên, những dấu hiệu này không đủ để kết luận một CV là giả.

Nếu chưa có bộ dữ liệu CV đã được chuyên gia gắn nhãn, không nên tuyên bố rằng hệ thống có khả năng phát hiện chính xác CV giả bằng mô hình Transformer.

### 5.6. Đánh giá CV và hỗ trợ nhà tuyển dụng

Hirify có thể cung cấp thông tin tham khảo giúp nhà tuyển dụng xem xét hồ sơ thuận tiện hơn, chẳng hạn:

- Kỹ năng có liên quan đến công việc.
- Kinh nghiệm được nêu trong CV.
- Những yêu cầu trong mô tả công việc chưa thấy thể hiện trong hồ sơ.
- Các thông tin cần xác minh thêm khi phỏng vấn.

Không nên để AI tự động loại ứng viên chỉ vì điểm số thấp. Cần tránh các tiêu chí suy đoán nhạy cảm, bảo đảm khả năng giải thích và giữ quyền quyết định tuyển dụng ở con người.

## 6. Nghiên cứu mô hình AI: Transformer, LLM và Embedding

Các khái niệm này có liên quan nhưng không đồng nghĩa với nhau.

**Transformer** là một kiến trúc mô hình học sâu được sử dụng trong nhiều hệ thống xử lý ngôn ngữ tự nhiên. **LLM** là nhóm mô hình ngôn ngữ lớn, thường được xây dựng dựa trên kiến trúc Transformer. **Embedding** chuyển đổi văn bản thành vector số để phục vụ so sánh và tìm kiếm ngữ nghĩa.

Trong phạm vi Hirify, có thể nghiên cứu ba nhóm giải pháp:

| Nhóm | Công nghệ dự kiến | Mục đích |
|---|---|---|
| Mô hình ngôn ngữ | OpenAI API hoặc Gemini API | Chatbot, tóm tắt, trích xuất thông tin và giải thích kết quả |
| Embedding | Dịch vụ embedding phù hợp với ngôn ngữ Việt và Anh | Tìm kiếm ngữ nghĩa, so khớp CV với tin tuyển dụng |
| Phát hiện bất thường | Quy tắc nghiệp vụ kết hợp mô hình phân loại nếu có dữ liệu | Đánh dấu spam và nội dung cần kiểm duyệt |

Việc sử dụng một mô hình Transformer không tự động tạo ra khả năng phát hiện mã độc. Quét mã độc cần công cụ bảo mật chuyên dụng. Tương tự, việc sử dụng LLM không tự động bảo đảm khả năng phát hiện CV giả.

Không nhất thiết phải tự huấn luyện một mô hình từ đầu. Với đồ án hoặc giai đoạn xây dựng ban đầu, việc sử dụng mô hình có sẵn qua API thường giúp tiết kiệm thời gian và dễ đánh giá hơn.

Nếu sau này có dữ liệu được gắn nhãn đủ chất lượng, có thể nghiên cứu mô hình phân loại riêng cho spam hoặc dấu hiệu bất thường. Quyết định này cần dựa trên kết quả thử nghiệm, chi phí và yêu cầu bảo vệ dữ liệu.

## 7. RAG và tìm kiếm dựa trên dữ liệu của Hirify

RAG (Retrieval-Augmented Generation) là phương pháp kết hợp tìm kiếm thông tin với mô hình ngôn ngữ. Thay vì chỉ trả lời dựa trên kiến thức có sẵn của mô hình, hệ thống tìm những dữ liệu liên quan rồi cung cấp chúng làm ngữ cảnh cho câu trả lời.

Trong Hirify, RAG có thể hữu ích khi chatbot cần trả lời các câu hỏi như:

- Tin tuyển dụng nào đang phù hợp với hồ sơ của tôi?
- Công việc này yêu cầu những kỹ năng gì?
- Có những vị trí nào dành cho người mới tốt nghiệp?
- Chính sách ứng tuyển hoặc hướng dẫn sử dụng hệ thống được quy định như thế nào?

### 7.1. Công nghệ đề xuất

- **PostgreSQL:** Lưu trữ tin tuyển dụng và dữ liệu nghiệp vụ.
- **Embedding API:** Tạo vector biểu diễn nội dung.
- **pgvector:** Có thể bổ sung để lưu vector và tìm kiếm các nội dung gần nhau.
- **OpenAI API hoặc Gemini API:** Tổng hợp thông tin tìm được thành câu trả lời tự nhiên.
- **Spring Boot:** Điều phối quá trình truy xuất dữ liệu và gọi mô hình.

RAG không bắt buộc phải có ngay trong phiên bản đầu tiên. Nếu số lượng tin tuyển dụng còn ít, truy vấn PostgreSQL kết hợp bộ lọc nghiệp vụ có thể đáp ứng tốt hơn và dễ kiểm thử hơn.

Khi dùng RAG, hệ thống vẫn phải kiểm tra quyền truy cập trước khi truy xuất dữ liệu. Không được đưa CV riêng tư hoặc thông tin thuộc công ty khác vào ngữ cảnh của người dùng không có quyền xem.

## 8. Thiết kế API nội bộ dự kiến

Các API dưới đây là gợi ý để tổ chức chức năng. Chúng không được khẳng định là đã tồn tại trong dự án.

| Endpoint đề xuất | Phương thức | Chức năng |
|---|---|---|
| `/api/ai/chat` | POST | Chatbot hỗ trợ tìm kiếm việc làm |
| `/api/ai/resumes/{id}/analyze` | POST | Phân tích CV đã được xác thực quyền truy cập |
| `/api/ai/jobs/{id}/moderate` | POST | Hỗ trợ kiểm duyệt tin tuyển dụng |
| `/api/ai/jobs/recommendations` | POST | Gợi ý việc làm cho ứng viên |
| `/api/ai/jobs/{id}/match` | POST | Đánh giá mức độ phù hợp với hồ sơ |
| `/api/ai/health` | GET | Kiểm tra trạng thái dịch vụ AI |

Khi triển khai, cần thống nhất các endpoint với API hiện có để tránh trùng chức năng.

Các API phải tuân theo Spring Security và phân quyền theo vai trò:

- `ROLE_CANDIDATE`: Được phân tích CV của chính mình, dùng chatbot và xem gợi ý việc làm.
- `ROLE_RECRUITER`: Được quản lý tin đăng, xem hồ sơ ứng tuyển mà mình có quyền truy cập và sử dụng công cụ hỗ trợ đánh giá.
- `ROLE_ADMIN`: Được sử dụng chức năng kiểm duyệt và xem các cảnh báo thuộc phạm vi quản trị được cấp phép.

Quyền truy cập phải được kiểm tra ở backend. Việc ẩn nút trên React không đủ để bảo vệ dữ liệu.

## 9. Dữ liệu và lưu trữ

PostgreSQL tiếp tục là cơ sở dữ liệu chính. Có thể sử dụng các bảng hiện có cho người dùng, công ty, CV, tin tuyển dụng và đơn ứng tuyển.

Nếu triển khai thêm chức năng AI, có thể nghiên cứu các bảng hoặc thực thể sau:

| Thực thể đề xuất | Thông tin lưu trữ |
|---|---|
| `resume_analysis` | Trạng thái phân tích và thông tin đã trích xuất từ CV |
| `ai_moderation_result` | Kết quả kiểm duyệt, lý do cảnh báo và trạng thái xử lý |
| `job_match_result` | Điểm phù hợp và các tiêu chí liên quan |
| `chat_session` | Thông tin phiên chatbot theo chính sách lưu trữ |
| `chat_message` | Nội dung hội thoại nếu thực sự cần lưu |
| `document_embedding` | Vector embedding nếu triển khai tìm kiếm ngữ nghĩa |

Tên bảng chỉ mang tính minh họa. Trước khi tạo bảng mới, cần kiểm tra mô hình dữ liệu hiện tại để tránh lưu trùng thông tin.

Không nên lưu toàn bộ prompt, CV hoặc lịch sử hội thoại vô thời hạn nếu không có mục đích rõ ràng. Cần xác định thời hạn lưu trữ, quyền truy cập, cơ chế xóa và biện pháp bảo vệ dữ liệu cá nhân.

## 10. Bảo mật và độ tin cậy

Các chức năng AI phải tuân theo nguyên tắc bảo mật chung của Hirify.

### 10.1. Quản lý khóa API

Khóa OpenAI và Gemini phải được lấy từ biến môi trường hoặc hệ thống quản lý bí mật. Không đưa khóa vào mã nguồn frontend, repository công khai hoặc log.

### 10.2. Bảo vệ dữ liệu CV

CV có thể chứa họ tên, thông tin liên hệ, lịch sử làm việc và các dữ liệu cá nhân khác. Trước khi gửi dữ liệu đến dịch vụ AI bên ngoài, cần đánh giá chính sách dữ liệu, điều khoản xử lý và yêu cầu đồng ý phù hợp.

Chỉ gửi những thông tin thực sự cần thiết cho tác vụ. Không nên gửi toàn bộ CV nếu chỉ cần trích xuất một số trường cụ thể.

### 10.3. Phòng chống prompt injection

Nội dung CV và tin tuyển dụng là dữ liệu do người dùng cung cấp, không phải chỉ thị đáng tin cậy cho AI. Hệ thống cần phân tách dữ liệu với chỉ thị của mô hình, giới hạn công cụ mà chatbot được phép gọi và không cho phép nội dung trong CV tự thay đổi quyền truy cập hoặc kích hoạt hành động quản trị.

### 10.4. Kiểm soát lỗi và chi phí

Nên đặt giới hạn kích thước đầu vào, thời gian chờ, số lần gọi API và hạn mức sử dụng theo người dùng. Nếu dịch vụ AI gặp lỗi, hệ thống cần trả thông báo phù hợp thay vì làm gián đoạn toàn bộ chức năng tuyển dụng.

### 10.5. Đánh giá kết quả AI

Cần lưu phiên bản mô hình hoặc cấu hình quan trọng khi thích hợp, đồng thời đánh giá chất lượng đầu ra bằng tập dữ liệu thử nghiệm. Không sử dụng điểm AI làm căn cứ duy nhất để từ chối hồ sơ hoặc kết luận gian lận.

## 11. Kế hoạch triển khai theo giai đoạn

Để phù hợp với nền tảng Java đang có, có thể triển khai theo thứ tự sau.

**Giai đoạn 1 — Tích hợp API AI**

- Tạo lớp client Java để gọi OpenAI hoặc Gemini.
- Quản lý khóa API bằng cấu hình an toàn.
- Xây dựng một API chatbot đơn giản.
- Kiểm tra lỗi, timeout và giới hạn sử dụng.

**Giai đoạn 2 — Phân tích CV**

- Kiểm tra loại tệp và dung lượng.
- Tích hợp Apache PDFBox hoặc Apache Tika.
- Trích xuất văn bản và chuẩn hóa kết quả qua AI.
- Cho phép ứng viên kiểm tra thông tin đã trích xuất.

**Giai đoạn 3 — Gợi ý việc làm**

- Xây dựng bộ lọc bằng Spring Data JPA và PostgreSQL.
- Tính điểm phù hợp theo kỹ năng, cấp bậc và mong muốn.
- Đánh giá kết quả bằng các trường hợp kiểm thử thực tế.

**Giai đoạn 4 — Kiểm duyệt và bảo mật**

- Xây dựng quy tắc kiểm tra tin đăng và spam.
- Bổ sung công cụ quét mã độc cho tệp tải lên.
- Thiết kế quy trình chờ duyệt, cảnh báo và xử lý khiếu nại.
- Kiểm tra phân quyền và nhật ký kiểm duyệt.

**Giai đoạn 5 — Nghiên cứu tìm kiếm ngữ nghĩa và RAG**

- Đánh giá nhu cầu tìm kiếm ngữ nghĩa.
- Thử nghiệm embedding với dữ liệu tuyển dụng.
- Cân nhắc pgvector nếu kết quả thử nghiệm chứng minh có lợi.
- So sánh kết quả với phương pháp tìm kiếm từ khóa hiện tại.

Không cần triển khai toàn bộ các giai đoạn cùng lúc. Nên hoàn thành và đánh giá từng chức năng trước khi mở rộng.

## 12. Tiêu chí đánh giá

Để chứng minh các chức năng AI có ích, cần có tiêu chí đánh giá cụ thể thay vì chỉ trình bày rằng hệ thống sử dụng AI.

| Chức năng | Tiêu chí đánh giá |
|---|---|
| Chatbot | Độ chính xác của câu trả lời, mức độ liên quan và tỷ lệ trả lời không có căn cứ |
| Trích xuất CV | Độ chính xác của các trường như kỹ năng, kinh nghiệm và học vấn |
| Gợi ý việc làm | Mức độ liên quan của danh sách kết quả và tỷ lệ việc làm phù hợp xuất hiện ở vị trí đầu |
| Kiểm duyệt tin đăng | Tỷ lệ cảnh báo đúng và tỷ lệ nội dung hợp lệ bị cảnh báo nhầm |
| Phát hiện spam | Precision, recall và số lượng cảnh báo sai |
| Kiểm tra tệp độc hại | Khả năng phát hiện của công cụ quét chuyên dụng và cách xử lý tệp bị cảnh báo |
| Hiệu năng | Thời gian phản hồi, tỷ lệ lỗi và chi phí API trung bình |

Các chỉ số cần được đo trên dữ liệu thử nghiệm có nhãn hoặc được con người đánh giá. Không nên đưa ra phần trăm chính xác khi chưa thực hiện thử nghiệm.

## 13. Kết luận

Định hướng phù hợp cho AI của Hirify là phát triển trên nền tảng Java 17 và Spring Boot 3.5.6 đang có, sử dụng OpenAI API hoặc Gemini API cho các tác vụ ngôn ngữ tự nhiên, đồng thời tận dụng PostgreSQL và Spring Data JPA cho dữ liệu tuyển dụng.

Các chức năng ưu tiên gồm chatbot tìm kiếm việc làm, trích xuất thông tin CV và gợi ý việc làm. Sau khi các chức năng này hoạt động ổn định, có thể mở rộng sang kiểm duyệt tin đăng, tìm kiếm ngữ nghĩa và RAG.

Đối với bảo mật CV, cần kết hợp kiểm tra tệp, phần mềm quét mã độc và quy tắc nghiệp vụ. Transformer hoặc LLM chỉ đóng vai trò hỗ trợ phân tích nội dung, không thay thế công cụ chống mã độc chuyên dụng và không thể tự mình chứng minh một CV là giả.

Với cách tiếp cận theo từng giai đoạn, Hirify có thể tận dụng công nghệ hiện có, giữ kiến trúc backend thống nhất bằng Java và phát triển các chức năng AI theo hướng dễ kiểm thử, an toàn, có thể giải thích và phù hợp với nhu cầu tuyển dụng thực tế.