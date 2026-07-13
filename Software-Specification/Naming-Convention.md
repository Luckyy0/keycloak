# Naming Convention Specification
# Đặc tả Quy tắc Đặt tên

## 1. Directories and Files / Thư mục và Tệp
- **Format**: `Pascal-Kebab-Case` (capitalize the first letter of every word, separated by hyphens).
- **Định dạng**: `Pascal-Kebab-Case` (viết hoa chữ cái đầu của mỗi từ, cách nhau bằng dấu gạch ngang).
- **Examples**: `04-Authentication`, `Lesson-1-Request-Flow.md`, `Module-2-Advanced-Setup`.
- **Ví dụ**: `04-Authentication`, `Lesson-1-Request-Flow.md`, `Module-2-Advanced-Setup`.
- **Numbering**: Always use two digits for chapters (01, 02) to maintain sorting. Modules and lessons use single digits unless they exceed 9.
- **Đánh số**: Luôn dùng hai chữ số cho các chương (01, 02) để duy trì thứ tự sắp xếp. Mô-đun và bài học dùng một chữ số trừ khi vượt quá 9.

## 2. Images and Assets / Hình ảnh và Tài nguyên
- **Format**: `lowercase-kebab-case.ext`.
- **Định dạng**: `lowercase-kebab-case.ext`.
- **Examples**: `oauth2-auth-code-flow.png`, `keycloak-architecture-diagram.svg`.
- **Ví dụ**: `oauth2-auth-code-flow.png`, `keycloak-architecture-diagram.svg`.

## 3. Source Code / Mã nguồn
- **Java Packages**: `com.enterprise.keycloak.[module]`
- **Java Classes**: `PascalCase` (e.g., `CustomUserStorageProvider`).
- **Spring Properties**: `kebab-case` (e.g., `spring.security.oauth2.client.registration.keycloak`).
- **Docker Containers**: `lowercase-kebab-case` (e.g., `keycloak-postgres-db`).
- Quy tắc mã hóa chuẩn của từng ngôn ngữ phải được tuân thủ nghiêm ngặt.

## 4. Variables in Markdown / Biến trong Markdown
- When referring to placeholders in text, use angle brackets and uppercase: `<REALM_NAME>`, `<CLIENT_ID>`.
- Khi tham chiếu đến các biến giữ chỗ trong văn bản, hãy dùng dấu ngoặc nhọn và viết hoa: `<REALM_NAME>`, `<CLIENT_ID>`.
