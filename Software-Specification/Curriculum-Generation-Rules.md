# Curriculum Generation Rules Specification
# Đặc tả Quy tắc Sinh Giáo trình

## 1. Single Tasking / Đơn nhiệm
- Only one file or module must be generated per AI execution context to ensure maximum quality and depth.
- Chỉ một tệp hoặc mô-đun được phép sinh ra trong mỗi lần thực thi của AI để đảm bảo chất lượng và độ sâu tối đa.
- Stop and wait for the "tiếp tục" command before generating the next part.
- Dừng và chờ lệnh "tiếp tục" trước khi sinh phần tiếp theo.

## 2. Language & Terminology Policy / Chính sách Ngôn ngữ & Thuật ngữ
- The entire content of the generated Markdown lesson files MUST be written in **Vietnamese (Tiếng Việt)**.
- Toàn bộ nội dung của các tệp bài học Markdown PHẢI được viết bằng **Tiếng Việt**.
- **Crucial:** Technical terms MUST be kept in **English** (e.g., Request, Response, 3-way Handshake, Header, Payload, Cipher Suite, Load Balancer). Do not translate them into Vietnamese.
- **Quan trọng:** Thuật ngữ kỹ thuật PHẢI được giữ nguyên bằng **Tiếng Anh**. Tuyệt đối không dịch các thuật ngữ chuyên ngành (không dùng "Bắt tay 3 bước", hãy dùng "TCP 3-way handshake").

## 3. Extreme Depth & Completeness / Độ sâu Tối đa & Tính Toàn diện
- Content must be textbook-level detailed. A lesson cannot be a high-level summary.
- Nội dung phải chi tiết ở cấp độ sách giáo khoa chuyên sâu. Bài học không được phép là một bản tóm tắt chung chung.
- Explicitly cover the internal components and low-level mechanisms.
- Bao phủ rõ ràng các thành phần nội bộ và các cơ chế giao tiếp cấp thấp.

## 4. Depth and Reasoning / Độ sâu và Lập luận
- Always explain *WHY* a feature exists, not just *WHAT* it is.
- Luôn giải thích *TẠI SAO* một tính năng tồn tại, giải quyết bài toán gì trong hệ thống phân tán.
- Compare Keycloak's implementation with alternative solutions when applicable.
- So sánh việc triển khai của Keycloak với các giải pháp thay thế.

## 5. Content Categorization / Phân loại Nội dung
- Each lesson must be explicitly categorized into specific groups: **Theory (Lý thuyết)**, **Practical/Lab (Thực hành)**, **Architecture/Design (Kiến trúc/Thiết kế)**, or **Troubleshooting (Khắc phục sự cố)**.
- Mỗi bài học phải được phân loại rõ ràng thành các nhóm chuyên biệt.

## 6. Factuality & Referencing / Tính xác thực & Tham chiếu
- **Anti-Hallucination:** Absolutely NO hallucination of theory, technical specifications, or workflows. All knowledge must be strictly fact-based and technically accurate.
- **Chống Bịa đặt:** Tuyệt đối KHÔNG ĐƯỢC bịa đặt lý thuyết.
- **Mandatory References:** Every file MUST include a `References` section citing official documents (Keycloak Docs, RFCs, OWASP).
- **Tham chiếu Bắt buộc:** Bắt buộc phải có phần `References` trích dẫn các tài liệu chính thống.

## 7. Mandatory File Structure / Cấu trúc Tệp Bắt buộc
Mọi tệp bài học BẮT BUỘC phải tuân thủ nghiêm ngặt trình tự các phần mang tính khoa học và logic tùy theo loại bài học:

### 7.1. Cấu trúc cho Tệp Lý thuyết (Theory / Concept Files)
- **Meta Block**: Khối GitHub Alert `> [!NOTE]` chứa `Category` và `Goal` (Mục tiêu bài học).
- **1. Lý thuyết chuyên sâu (Detailed Theory)**: Giải thích bản chất, cấu trúc, và vấn đề cốt lõi mà công nghệ này giải quyết.
- **2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)**: Bắt buộc sử dụng Mermaid diagrams và giải thích step-by-step.
- **3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)**: Các tiêu chuẩn Enterprise, cảnh báo bảo mật (`> [!WARNING]`, `> [!IMPORTANT]`).
- **4. Cấu hình minh họa thực tế (Configuration Examples)**: Các đoạn code snippet, cấu hình (Nginx, Spring Boot, Keycloak, v.v.).
- **5. Trường hợp ngoại lệ (Edge Cases)**: Phân tích các lỗi hệ thống, sự cố mạng, lệch thời gian và cách khắc phục.
- **6. Câu hỏi Phỏng vấn (Interview Questions)**: Ít nhất 5 câu hỏi có phân loại đáp án rõ ràng giữa Junior và Senior.
- **7. Tài liệu tham khảo (References)**: Các liên kết chuẩn xác đến RFCs, Official Docs.

### 7.2. Cấu trúc cho Tệp Thực hành (Lab Exercises)
- **Meta Block**: Khối GitHub Alert `> [!NOTE]` chứa `Category` (Practical/Lab) và `Goal` (Mục tiêu bài Lab).
- **1. Kịch bản Thực hành (Lab Scenario)**: Giới thiệu bài toán thực tế cần giải quyết.
- **2. Chuẩn bị Môi trường (Prerequisites)**: Các công cụ, Docker images, hoặc mã nguồn cần chuẩn bị trước.
- **3. Các bước Thực hiện (Step-by-Step Instructions)**: Hướng dẫn chi tiết từng dòng lệnh, từng cú click chuột để hoàn thành bài Lab.
- **4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)**: Các cách kiểm tra xem Lab đã thành công chưa, và các lỗi thường gặp trong lúc làm Lab.
