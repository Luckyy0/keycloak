# Markdown Convention Specification
# Đặc tả Chuẩn Markdown

## 1. Headers / Tiêu đề
- Document Title: Single `#` at the top of the file. (e.g., `# Lesson 1: OAuth2 Flows`)
- Tiêu đề tài liệu: Một dấu `#` duy nhất ở đầu tệp.
- Main Sections: `## 1. Section Name`
- Các phần chính: `## 1. Tên phần`
- Sub-sections: `### 1.1. Sub-section Name`
- Các phần phụ: `### 1.1. Tên phần phụ`

## 2. Text Formatting / Định dạng Văn bản
- Use **bold** for UI elements, key concepts, and exact term definitions.
- Dùng **in đậm** cho các thành phần giao diện, khái niệm chính và định nghĩa thuật ngữ chính xác.
- Use `inline code` for variable names, technical terms (e.g., `Request`, `Header`), file names, URLs, and small code snippets.
- Dùng `mã nội tuyến` cho tên biến, thuật ngữ kỹ thuật (ví dụ: `Request`, `Header`), tên tệp, URL.

## 3. Code Blocks / Khối Mã
- ALWAYS specify the language in fenced code blocks.
- LUÔN LUÔN chỉ định ngôn ngữ trong các khối mã rào chắn.
  ```json
  { "key": "value" }
  ```
- Do not use plain text blocks unless it's standard console output without a specific format.
- Không dùng các khối văn bản thuần túy trừ khi đó là đầu ra console.

## 4. Alerts and Admonitions / Cảnh báo và Chú ý
Use GitHub Flavored Markdown alerts to emphasize critical enterprise information:
Sử dụng các cảnh báo Markdown của GitHub để nhấn mạnh thông tin doanh nghiệp quan trọng:
- `> [!NOTE]` : Extra context, background info. / Ngữ cảnh bổ sung, thông tin nền.
- `> [!TIP]` : Best practices, performance tweaks. / Thực hành tốt nhất, tinh chỉnh hiệu năng.
- `> [!IMPORTANT]` : Crucial design decisions. / Các quyết định thiết kế cốt lõi.
- `> [!WARNING]` : Potential pitfalls, anti-patterns. / Cạm bẫy tiềm ẩn, anti-pattern.
- `> [!CAUTION]` : Security risks, data loss scenarios. / Rủi ro bảo mật, kịch bản mất dữ liệu.

## 5. Language Policy / Chính sách Ngôn ngữ
- Toàn bộ nội dung văn bản (ngoại trừ tiêu đề tệp chuẩn và file hệ thống) PHẢI được viết bằng Tiếng Việt.
- Thuật ngữ kỹ thuật (Technical terms) tuyệt đối giữ nguyên Tiếng Anh (không dịch sang Tiếng Việt) nhằm đảm bảo tính chuyên nghiệp.
