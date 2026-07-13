# Chapter 13: Authentication Flows

Chào mừng đến với Trái tim Logic của Lãnh chúa OIDC - **Authentication Flows** (Các Luồng Xác Thực).

Nếu Authentication (Chương 12) là các vũ khí bảo vệ (OTP, WebAuthn, Passkeys), thì Authentication Flows chính là Bảng Mạch Điều Khiển (Circuit Board) để bạn quyết định khi nào thì dùng vũ khí nào, cho ai, và trong hoàn cảnh nào.

Trong chương này, chúng ta sẽ mổ xẻ cỗ máy lắp ráp logic của Keycloak. Bạn sẽ học cách thao túng hoàn toàn trải nghiệm đăng nhập của khách hàng, từ lúc họ gõ chữ cái đầu tiên vào form, đến lúc họ nhận được Access Token.

## Mục tiêu học tập
Sau khi hoàn thành chương này, bạn sẽ nắm vững:
- Cấu trúc tĩnh và động của các Luồng Xác thực mặc định (Browser, Registration, Reset Credentials).
- Khái niệm về Flow (Luồng), Sub-flow (Luồng phụ), và Execution (Thực thi).
- Cách sử dụng các chiến lược (Requirements): Required, Alternative, Disabled, Conditional.
- Cách liên kết (Identity Brokering) với các MXH (Google, Facebook) thông qua First Broker Login và Post Broker Login.
- Cách dùng Trí tuệ Nhân tạo (Conditional Flow) để rẽ nhánh luồng đi dựa trên Role, IP, hoặc Device.
- Cách tạo ra một Luồng Xác Thực tùy chỉnh hoàn toàn (Custom Flow) mang đậm dấu ấn thiết kế của riêng công ty bạn.

## Cấu trúc bài học

- **Lesson 1:** Cửa ngõ chính (Browser Flow)
- **Lesson 2:** Lễ tân đón khách (Registration Flow)
- **Lesson 3:** Hỗ trợ mất trí nhớ (Reset Credentials Flow)
- **Lesson 4:** Giao ước Liên minh (First Broker Login)
- **Lesson 5:** Hậu Giao ước (Post Broker Login)
- **Lesson 6:** Trí tuệ Nhân tạo rẽ nhánh (Conditional Flow)
- **Lesson 7:** Hộp Đen lồng Hộp Đen (Sub-Flow)
- **Lesson 8:** Điểm Nổ (Execution)
- **Lesson 9:** Kỷ luật Thép (Requirements)
- **Lesson 10:** Kiến tạo Luồng mới (Custom Flow)

Bắt đầu nhào nặn logic bảo mật thôi!
