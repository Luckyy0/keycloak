# Chapter 15: OAuth2

Chào mừng bạn đến với chương quan trọng nhất của toàn bộ khóa học: **OAuth 2.0 (Open Authorization)**.
Toàn bộ hệ sinh thái Keycloak được xây dựng trên nền móng của giao thức này. Nếu không hiểu rõ OAuth2, bạn sẽ không thể bảo vệ API, không biết cách chọn Flow đăng nhập cho Frontend, và không thể gỡ lỗi khi hệ thống báo lỗi Token.

## Mục Tiêu Học Tập (Learning Objectives)
Chương này sẽ biến bạn từ một người "chỉ biết dùng thư viện" thành một "chuyên gia giao thức". Bạn sẽ học:
1. Bản chất sự ra đời của OAuth2 (Giải quyết bài toán "Đưa mật khẩu Facebook cho game").
2. Bốn vai trò (Actors) cấu thành nên vũ trụ OAuth2.
3. Các luồng (Grant Types/Flows) quan trọng: Authorization Code, PKCE, Client Credentials, Device Flow.
4. Cách quản lý Token (Refresh, Revocation, Introspection) và mô hình rủi ro (Threat Model).

## Cấu Trúc Thư Mục (Directory Structure)
- `Module-1-Concepts/`: Lý thuyết chuyên sâu 13 bài từ cơ bản tới Best Practices.
- `Labs/`: Thực hành cấu hình Client và test API bằng Postman/cURL.
- `code/`: File docker-compose khởi tạo môi trường thực hành.

## Danh Sách Bài Học (Lesson List)
- Lesson 1: RFC Overview (Tổng quan RFC)
- Lesson 2: OAuth Actors (Các vai trò)
- Lesson 3: Authorization Code Flow
- Lesson 4: Authorization Code with PKCE (Bảo mật Web/Mobile)
- Lesson 5: Client Credentials Flow (Machine to Machine)
- Lesson 6: Device Flow (Đăng nhập qua Smart TV)
- Lesson 7: Refresh Token (Cấp lại Token)
- Lesson 8: Token Revocation (Thu hồi Token)
- Lesson 9: Token Introspection (Kiểm tra Token)
- Lesson 10: Discovery (Khám phá cấu hình)
- Lesson 11: Dynamic Client Registration
- Lesson 12: Security Best Current Practice (Bảo mật tiêu chuẩn)
- Lesson 13: Threat Model (Mô hình Mối đe dọa)

Hãy bắt đầu bài đầu tiên để xem tại sao thế giới lại cần tới OAuth2!
