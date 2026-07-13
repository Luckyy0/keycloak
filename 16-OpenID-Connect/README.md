# Chapter 16: OpenID Connect (OIDC)

Chào mừng bạn đến với **Chương 16: OpenID Connect**.
Nếu OAuth2 ở chương trước là để "Cấp quyền cho Ứng dụng B truy cập API của Ứng dụng A", thì OIDC chính là Lớp Vỏ Bọc (Layer) được xây đè lên OAuth2 để giải quyết một bài toán duy nhất: **Xác thực danh tính (Authentication) - Tức là Đăng Nhập**.

OIDC biến Keycloak từ một cái máy bơm Token ủy quyền (AuthZ) trở thành một "Bưu Điện Định Danh" thực thụ (Identity Provider). 

## Mục Tiêu Học Tập (Learning Objectives)
Kết thúc chương này, bạn sẽ hiểu rõ:
1. Tại sao OIDC lại đẻ thêm `ID Token` và nó khác gì `Access Token` của OAuth2.
2. Các tham số bảo mật cực độ của OIDC để chống lại giả mạo: `nonce`, `state`, `prompt`, `max_age`.
3. Sức mạnh của Endpoint `/userinfo` để phân tách giao thức.
4. Cơ chế Đăng Xuất (Logout) đỉnh cao: Front-channel, Back-channel, và Quản lý Session.

## Cấu Trúc Thư Mục (Directory Structure)
- `Module-1-Concepts/`: Lý thuyết chuyên sâu 12 bài về các tham số và luồng OIDC.
- `Labs/`: Thực hành OIDC Login/Logout bằng cURL và Trình duyệt.
- `code/`: File docker-compose khởi tạo môi trường.

## Danh Sách Bài Học (Lesson List)
- Lesson 1: OIDC Discovery (Khám phá OIDC)
- Lesson 2: ID Token (Chứng minh thư điện tử)
- Lesson 3: UserInfo Endpoint (Hồ sơ người dùng)
- Lesson 4: Tham số Nonce (Chống Replay)
- Lesson 5: Tham số State (Chống CSRF)
- Lesson 6: Tham số Prompt (Ép buộc hành vi)
- Lesson 7: Tham số Max_age (Ép thời gian sống)
- Lesson 8: Hybrid Flow (Luồng Lai)
- Lesson 9: Front-Channel Logout (Đăng xuất Tiền đài)
- Lesson 10: Back-Channel Logout (Đăng xuất Hậu đài)
- Lesson 11: Session Management (Quản lý Phiên)
- Lesson 12: OIDC Claims (Khẳng định định danh)

Chúng ta cùng đi vào khám phá Giao Thức Đăng Nhập Tiêu Chuẩn Số 1 Thế Giới Hiện Nay!
