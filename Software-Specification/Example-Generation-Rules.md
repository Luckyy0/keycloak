# Example Generation Rules Specification
# Đặc tả Quy tắc Sinh Ví dụ

## 1. Real-world Scenarios / Kịch bản Thực tế
- Examples must reflect actual enterprise use cases (e.g., multi-tenant B2B SaaS, mobile app BFF).
- Các ví dụ phải phản ánh các trường hợp sử dụng thực tế của doanh nghiệp (ví dụ: SaaS B2B đa người thuê, BFF cho ứng dụng di động).
- Avoid trivial "Hello World" examples without security context.
- Tránh các ví dụ "Hello World" tầm thường không có ngữ cảnh bảo mật.

## 2. Infrastructure as Code / Hạ tầng dưới dạng Mã
- Include `docker-compose.yml` with dependencies like PostgreSQL, Redis, and Keycloak for every major example.
- Bao gồm `docker-compose.yml` với các phụ thuộc như PostgreSQL, Redis và Keycloak cho mọi ví dụ lớn.
- Provide clear setup instructions (`README.md` for each project) covering prerequisites, execution, and teardown.
- Cung cấp hướng dẫn thiết lập rõ ràng (`README.md` cho mỗi dự án) bao gồm các điều kiện tiên quyết, thực thi và gỡ bỏ.

## 3. End-to-End Workflows / Luồng làm việc Từ đầu đến cuối
- Show the complete lifecycle: Keycloak admin configuration -> Application code -> Request/Response cycle.
- Hiển thị toàn bộ vòng đời: Cấu hình quản trị Keycloak -> Mã ứng dụng -> Chu kỳ Yêu cầu/Phản hồi.
