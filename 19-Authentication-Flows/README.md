# Chapter 19: Authentication Flows (Luồng Xác Thực Tùy Biến)

Chào mừng bạn đến với **Chương 19: Authentication Flows**.
Nếu OIDC và SAML là "Luật Giao Thông", thì Authentication Flows chính là "Cách Keycloak Kiểm Tra Hành Lý Của Khách Trạm Dừng Chân".
Mặc định, Keycloak chỉ yêu cầu Khách hàng nhập `Username / Password`. Nhưng đời không như mơ, Doanh nghiệp của bạn sẽ yêu cầu: "Nếu khách đăng nhập từ Mạng Ngoài, bắt nó quét Vân Tay. Nếu khách đăng nhập từ Mạng Nội Bộ Công Ty, chỉ cần gõ Password". 
Để thiết kế được những kịch bản "Lai ghép - Rẽ nhánh" Khổng Lồ như vậy, bạn phải thuần thục Động Cơ Authentication Flows Của Keycloak.

## Mục Tiêu Học Tập (Learning Objectives)
Kết thúc chương này, bạn sẽ nắm vững:
1. Kiến trúc Cây (Tree Architecture) của một Luồng Xác Thực.
2. Khái niệm `Authenticator` và cách chúng thi hành nhiệm vụ (Execution).
3. 4 Loại Cờ Phép Thuật: `Required`, `Alternative`, `Disabled`, `Conditional`.
4. Cách Nhóm các bước lại bằng Sub-Flows và Lệnh Tự Chạy Có Điều Kiện (Conditional Flows).

## Cấu Trúc Thư Mục (Directory Structure)
- `Module-1-Execution/`: Trọn bộ 5 bài lý thuyết tháo dỡ cỗ máy Authentication.
- `Labs/`: Thực hành thiết kế Luồng: Vừa gõ Pass, Vừa bắt Buộc Khách Nhập OTP Nếu Nằm Ngoài Mạng Tĩnh.
- `code/`: File docker-compose khởi tạo môi trường thực hành.

## Danh Sách Bài Học (Lesson List)
- Lesson 1: Flow Architecture (Kiến Trúc Cây Luồng)
- Lesson 2: Authenticators (Kẻ Chấp Pháp)
- Lesson 3: Execution Requirements (Quyền Lực Của Các Cờ)
- Lesson 4: Sub-Flows (Luồng Phụ Nhánh Mạch)
- Lesson 5: Conditional Flows (Luồng Động Dựa Cờ Môi Trường)

Hãy Chuẩn Bị Tinh Thần Lắp Ráp Lego Cho Cửa Ngõ Của Trạm Gác Vingroup Đỉnh Chóp!
