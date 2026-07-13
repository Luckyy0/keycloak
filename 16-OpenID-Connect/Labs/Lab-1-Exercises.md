# Lab 1: Giải Mã Bí Ẩn OpenID Connect (OIDC)

## 1. Mục Tiêu (Objectives)
Thực hành đóng vai Hacker và Trình duyệt để phân tích Giao Thức Đăng Nhập Tiêu Chuẩn.
- **Task 1:** Đọc và hiểu file OIDC Discovery.
- **Task 2:** Chạy luồng OIDC bằng Postman để lấy ID Token, Bóc tách ID Token xem `nonce`.
- **Task 3:** Gọi Cửa Phụ `/userinfo` bằng Access Token.
- **Task 4:** Vận hành luồng OIDC Front-channel Logout.

---

## 2. Chuẩn Bị (Prerequisites)
Khởi động hệ thống Keycloak bằng docker-compose đã cung cấp.

```bash
cd code
docker-compose up -d
```
Mở **Postman**. Truy cập Admin Console Keycloak tại `http://localhost:8080/` với tài khoản `admin` / `admin`.

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Khám Phá OIDC Discovery Tĩnh Oanh
1. Mở Trình duyệt, không cần đăng nhập, dán thẳng URL Này Vào:
   `http://localhost:8080/realms/master/.well-known/openid-configuration`
2. Đọc cục JSON. Tìm các dòng chứa chữ `userinfo_endpoint`, `end_session_endpoint`.
3. Nhìn xem Keycloak của bạn đang hỗ trợ `scopes_supported` gồm những chữ gì? Có chữ `openid` không? Đỉnh Chóp!

### Task 2: Chạy Luồng Auth Code Sinh ID Token Rút Lụa Bọt
1. Trên Keycloak, tạo 1 Client tên `oidc-lab-app`. Bật **Client authentication** sang **ON**. Lưu Lại Cấp Tốc.
2. Tại Tab Settings của `oidc-lab-app`, mục `Valid redirect URIs` điền `https://oauth.pstmn.io/v1/callback`.
3. Giờ ta đóng vai Frontend gọi Nhử Mồi Bọt Cắt Lụa. Dán Dòng Này Vào Trình Duyệt:
   `http://localhost:8080/realms/master/protocol/openid-connect/auth?client_id=oidc-lab-app&response_type=code&redirect_uri=https://oauth.pstmn.io/v1/callback&scope=openid profile email&nonce=N99_OanhMang&state=S88_TrutKhoe`
4. Login `admin`/`admin`. Bị đá sang Postman. Nhìn URL có `code=...`. Copy Cái Code Đó Lại!
5. Mở Postman App Đập Mạch Đổi Code Lấy JWT Lụa:
   - Request `POST` Tới: `http://localhost:8080/realms/master/protocol/openid-connect/token`
   - Body (`x-www-form-urlencoded`):
     - `grant_type`: `authorization_code`
     - `code`: `[Paste Code Vào Đáy]`
     - `client_id`: `oidc-lab-app`
     - `client_secret`: `[Lấy Secret Từ Tab Credentials Của Keycloak]`
     - `redirect_uri`: `https://oauth.pstmn.io/v1/callback`
6. Send Lệnh. Bùm! Khác Với Bài OAuth2. Lần Này Ngoài Access Token, Bạn Nhận Được Thêm 1 Cục JSON Là **`id_token`**.
7. Copy Cục `id_token` Đó, Mở Trang Web `https://jwt.io` Bằng Trình Duyệt Dán Vô Bóc Đáy! Nhìn Xem Có Cái Chữ Khớp Lệnh `nonce: N99_OanhMang` Ở Trong Đó Chưa? Tuyệt Vời!

### Task 3: Chọc API UserInfo Rút Dữ Liệu Hồ Sơ Khách Oanh Lụa
1. Bạn Cần Bốc Khối Chữ **`access_token`** Vừa Lấy Ở Bước Postman Bên Trên.
2. Tạo 1 Request `GET` Mới Bằng Postman Khung Cắt:
   - URL: `http://localhost:8080/realms/master/protocol/openid-connect/userinfo`
   - Tab **Authorization**: Chọn Type Là `Bearer Token`. Paste cái Access Token Vào Ô Tĩnh.
3. Bấm Send Giao Dịch Oanh. Bạn Sẽ Nhận Về Nguyên Cục JSON Chứa Tên, Preferred_username Trút Lệnh Đáy Oanh Mạng!

### Task 4: Chạy Đăng Xuất (End_Session_Endpoint) Trượt Nhựa
1. Trong Request Mới Trên Postman, Cấu Hình Lệnh `GET` Tĩnh Bọt:
   - URL: `http://localhost:8080/realms/master/protocol/openid-connect/logout?id_token_hint=[Paste Cục ID_Token Vào Đây]`
2. Send Lệnh. Nó Sẽ Báo `200 OK` HTML. 
3. Giờ Bạn Thử Quay Lại Task 3, Đập Access Token Lại Bằng Postman Lên UserInfo Xem. Nó Báo Lỗi `401 Unauthorized` Oanh Khung Dịch Lụa Ngay Tức Khắc Vì Cơn Bão Logout Đã Hủy Session Oanh Rỗng Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng!

---

## 4. Dọn Dẹp (Cleanup)
Sau khi hoàn thành OIDC Lab, hủy Docker tránh lãng phí RAM Máy:
```bash
docker-compose down -v
```
