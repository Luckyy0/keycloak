# Chapter 17: Giao Thức SAML (Security Assertion Markup Language)

Chào mừng bạn đến với **Chương 17: SAML 2.0**.
Nếu OAuth2/OIDC là ngôn ngữ hiện đại, nhẹ nhàng của các công ty Startup và Web App đời mới (dựa trên JSON). Thì SAML lại là **ngôn ngữ "Bố Già" (Godfather)** của giới Tài chính, Ngân hàng, Viễn thông, và các Tập đoàn Doanh nghiệp (Enterprise) lớn.

Toàn bộ thông điệp của SAML không dùng JSON, mà dùng **XML Khổng Lồ**. Việc nắm vững SAML trên Keycloak giúp bạn tích hợp được với mọi hệ thống "Cổ đại nhưng Đầy Tiền" của các tập đoàn lớn.

## Mục Tiêu Học Tập (Learning Objectives)
Chương này sẽ giúp bạn làm chủ:
1. Bản chất của thông điệp XML trong SAML (Khái niệm Assertion).
2. Cách thiết lập Cầu Nối Lòng Tin bằng file Metadata (Bản đồ Tọa độ).
3. Luồng hoạt động Đăng Nhập & Đăng Xuất Đỉnh Cao Bằng SAML.
4. Cách phân biệt "khi nào dùng SAML, khi nào dùng OIDC".

## Cấu Trúc Thư Mục (Directory Structure)
- `Module-1-Concepts/`: Lý thuyết chuyên sâu 6 bài về XML, Metadata, Login/Logout.
- `Labs/`: Thực hành thiết lập kết nối SAML Bằng Keycloak (Đóng vai IdP và SP).
- `code/`: File docker-compose khởi tạo môi trường thực hành.

## Danh Sách Bài Học (Lesson List)
- Lesson 1: SAML Basics (Giao Thức Cơ Bản Bố Già)
- Lesson 2: SAML Assertions (Khẳng Định XML)
- Lesson 3: SAML Metadata (Bản Đồ Nối Cáp)
- Lesson 4: SAML Login Flow (Luồng Đăng Nhập SP-Initiated & IdP-Initiated)
- Lesson 5: SAML Logout Flow (Đăng Xuất Mạch Kẽ)
- Lesson 6: SAML vs OIDC (Trận Chiến Thế Kỷ)

Hãy chuẩn bị tinh thần đương đầu với thế giới XML Đầy Quyền Lực Nhựa Bọc!
