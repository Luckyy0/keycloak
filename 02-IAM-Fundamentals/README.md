# Chương 02: Nền tảng IAM - Ngôn ngữ của Kiến trúc sư (IAM Fundamentals)

> [!NOTE]
> Chào mừng đến với Chương 2. Ở chương này, chúng ta sẽ tạm gác lại các thao tác Lập trình (Code) hay Triển khai Hạ tầng (Docker) để tập trung xây dựng TƯ DUY. 
> Trong ngành Identity & Access Management (IAM), sự mơ hồ về khái niệm (Ví dụ: Nhầm lẫn giữa Authentication và Authorization) sẽ dẫn đến những Lỗ hổng Bảo mật Kiến trúc (Architectural Flaws) không thể cứu vãn. Ở Tầng Quản trị, ngôn từ là sức mạnh. Hãy học cách gọi ĐÚNG TÊN của Vạn vật.

## Mục tiêu của chương
- Tách bạch và Nhận thức rõ sự khác biệt Sinh tử giữa bộ ba: `Identity` (Định danh) - `Authentication` (Xác thực) - `Authorization` (Phân quyền).
- Hiểu được sự Vĩ đại của `Identity Federation` (Liên minh Danh tính) và sức mạnh thực sự đằng sau thuật ngữ `Single Sign-On (SSO)`.
- Phân biệt vai trò của các Cường quốc trong bàn đàm phán OIDC/SAML: Thế nào là `Identity Provider (IdP)` và Thế nào là `Service Provider (SP)`.
- Cập nhật các triết lý Bảo mật Cực đoan định hình Kiến trúc của tương lai: `MFA`, `Passwordless`, `Zero Trust`, và `Least Privilege`.

## Cấu trúc bài học
- **Nhóm 1: Bộ Ba Vĩnh Cửu**
  - `Lesson-1-Identity.md`: Bản thể và Định danh (Identity vs Account).
  - `Lesson-2-Authentication.md`: Xác thực (Authentication - AuthN).
  - `Lesson-3-Authorization.md`: Phân quyền (Authorization - AuthZ).
- **Nhóm 2: Kỷ nguyên Liên hiệp (Federation)**
  - `Lesson-4-Federation.md`: Liên minh Danh tính (Identity Federation).
  - `Lesson-5-Identity-Provider.md`: Nhà cung cấp Danh tính (IdP).
  - `Lesson-6-Service-Provider.md`: Ứng dụng Khách (Service Provider / Relying Party).
  - `Lesson-7-SSO.md`: Đăng nhập Đơn phương (Single Sign-On).
- **Nhóm 3: Triết lý Bảo mật Cực đoan**
  - `Lesson-8-MFA.md`: Xác thực Đa Yếu Tố (Multi-Factor Authentication).
  - `Lesson-9-Passwordless.md`: Tương lai Không Mật Khẩu (Passwordless / WebAuthn).
  - `Lesson-10-Zero-Trust.md`: Kiến trúc Zero Trust (Không tin ai).
  - `Lesson-11-Least-Privilege.md`: Đặc quyền Tối thiểu (Least Privilege).

## Hướng dẫn thực hành (Labs)
- Cuối chương sẽ có phần Lab Thực chiến để ánh xạ 11 triết lý lý thuyết này vào Giao diện cấu hình (Admin Console) của Máy chủ Keycloak mà bạn vừa khởi chạy ở Lab 1.
