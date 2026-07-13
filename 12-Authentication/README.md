# Chương 12: Kiến Trúc Xác Thực (Authentication & Security Hardening)

> [!NOTE]
> Bức tường lửa đầu tiên bảo vệ toàn bộ Hệ sinh thái của bạn chính là Lõi Xác Thực (Authentication Engine). Làm thế nào Keycloak biết một Request gọi đến là Khách Hàng thật hay một con Bot Đánh cắp Token? Làm sao để triển khai hệ thống Đăng nhập Sinh trắc học (Vân tay/Khuôn mặt) mà không cần mật khẩu? 
> Tất cả bí mật nằm ở chương cốt lõi này.

## Mục tiêu của chương
- Mổ xẻ 4 Luồng Xác Thực (Authentication Flows) quan trọng nhất thế giới OIDC/OAuth2.
- Phân biệt rạch ròi giữa Xác thực Con Người (Browser Flow) và Xác thực Máy Móc (Service Authentication).
- Bảo mật Client App: Tại sao truyền Client Secret đi qua mạng lại bị coi là tội ác trong ngành Ngân hàng (Chuyển sang dùng Signed JWT).
- Triển khai Xác thực Đa Yếu Tố (MFA/OTP) và Tuyệt đỉnh bảo mật Bypass Mật Khẩu (WebAuthn/Passkeys).
- Cấu hình Conditional Authentication: Tự động đòi quét Khuôn mặt nếu Khách hàng đăng nhập từ một Quốc gia lạ hoặc thiết bị lạ.

## Cấu trúc bài học

- `Lesson-1-Browser-Flow.md`: Luồng cơ bản nhất. Phân tích nguyên lý Redirection và Cookie bảo vệ vòng ngoài.
- `Lesson-2-Direct-Grant.md`: Chống chỉ định! Tại sao mang API truyền Mật khẩu đi xây dựng SPA React/Angular lại là tự sát.
- `Lesson-3-Client-Authentication.md`: Bí kíp ký JWT (Signed JWT Client Auth) chống đánh cắp Secret của Frontend.
- `Lesson-4-Service-Authentication.md`: Rô-bốt giao tiếp. Nguyên lý Machine-To-Machine (Client Credentials Flow).
- `Lesson-5-OTP.md`: Tích hợp Google Authenticator (TOTP) và cấu hình chống Brute-Force.
- `Lesson-6-WebAuthn.md`: Chuẩn FIDO2 bảo mật cao nhất thế giới (Sử dụng USB Yubikey / Windows Hello).
- `Lesson-7-Passkeys.md`: Trải nghiệm hệ thống đăng nhập không mật khẩu bằng FaceID/TouchID trên iOS/Android.
- `Lesson-8-Conditional-Authentication.md`: Trí Tuệ Nhân Tạo của Keycloak. Thiết lập Rule chặn truy cập dựa trên Rủi Ro (Risk-Based Auth).

## Hướng dẫn thực hành (Labs)
- Dựng cụm Keycloak an toàn.
- Cấu hình bắt buộc (Require) OTP cho nhóm Khách Hàng VIP.
- Bật tính năng Passkeys, đăng nhập vào bằng TouchID/FaceID.
- Cấu hình chống Brute-force: Khóa mõm User nếu nhập sai mật khẩu 5 lần.
