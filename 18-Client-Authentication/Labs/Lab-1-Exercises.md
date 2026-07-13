# Lab 1: Vượt Tường Lửa Mật Khẩu (Client Authentication)

## 1. Mục Tiêu (Objectives)
Trải nghiệm 2 cơ chế chứng thực Client phổ biến nhất từ Dễ đến Cực Khó.
- **Task 1:** Chạy chuẩn `client_secret_basic` bằng Postman.
- **Task 2:** Khởi tạo Cặp Khóa Ngân Hàng Oanh Cáp.
- **Task 3:** Cấu hình và Bắn `Private Key JWT` Xuyên Tường Lửa Keycloak.

---

## 2. Chuẩn Bị (Prerequisites)
Khởi động hệ thống Keycloak bằng docker-compose đã cung cấp.

```bash
cd code
docker-compose up -d
```
Mở Trình Duyệt Truy Cập Admin Console Keycloak tại `http://localhost:8080/` (admin/admin).

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Đánh Bại Cổng Cơ Bản (Client Secret Basic) Oanh Mạng
1. Trong Admin Console Keycloak, Bấm Tạo Client **`api-thuong`**.
2. Bật Cờ **`Client authentication = ON`**. Save Lại Trút Cáp Mạch Máu Cắt.
3. Qua Tab **Credentials**, Thấy Mặc Định Đang Là **`Client Id and Secret`**. Copy Cục Secret Đó.
4. Bạn Lấy 1 Cục Mã Code OIDC Bằng Cách Gọi Lệnh Nhử Mồi Oanh Khung Trên Trình Duyệt:
   `http://localhost:8080/realms/master/protocol/openid-connect/auth?client_id=api-thuong&response_type=code&redirect_uri=https://oauth.pstmn.io/v1/callback`
   Đăng nhập Admin. Copy cái `code=xyz` trên URL.
5. Dùng Postman, Request Đổi Access Token Bằng Chuẩn Mạch Oanh Giao Dịch `Basic`:
   - URL: `POST http://localhost:8080/realms/master/protocol/openid-connect/token`
   - Tab **Authorization**: Chọn Type **`Basic Auth`**. Nhập Username=`api-thuong`, Password=`[Paste Secret Oanh Lụa Vô Đây]`.
   - Tab **Body (x-www-form-urlencoded)**: 
     - `grant_type`: `authorization_code`
     - `code`: `xyz`
     - `redirect_uri`: `https://oauth.pstmn.io/v1/callback`
6. Send Lệnh Đáy DB. Nhận Khối Vàng Thành Công! (Postman Tự Động Gom Base64 Giúp Bạn Ở Lệnh Header Basic Đỉnh Chóp).

### Task 2: Chuyển Sang Lõi Kép Private Key JWT Nhựa Bọc Cắt Chữ Kẽ
1. Tạo 1 Thằng Client Khác Tên Là **`api-ngan-hang`**. Bật `Client authentication = ON`.
2. Qua Tab **Credentials**. Đổi Khung Cắt **Client Authenticator** Sang **`Client Jwt`**. (Ép Dùng Private Key).
3. Qua Tab **Keys** Của `api-ngan-hang`. Bấm Nút **`Generate new keys`**.
4. Định Dạng: JKS Hoặc PKCS12 Đều Được Oanh Tĩnh Lụa Thép. Nhập Password Lõi Trọng Điểm Là `secret`. Bấm Generate Oanh Cáp Trọng Lõi Tự Trị.
5. Trình Duyệt Tự Tải Về File `keystore.jks`. (Đây Chính Là Cục Chứa Private Key Đỉnh Đáy Của Thằng Spring Boot, Còn Keycloak Đã Lưu Khối Public Key Trượt Khung Khớp Lệnh Của Nó Rồi Bọt Cắt Mạch Đứt Kẽ).

### Task 3: Mô Phỏng App Ngân Hàng Ký JWT Bắn Ngược Bọc Mạch
*Lưu Ý Lỗ Bọt Cắt Trắng: Task này giả lập bằng Postman sẽ rất phức tạp vì bạn phải lấy Private Key để tự Ký Đáy Lụa. Trong thực tế Spring Boot Tự Làm Mạch Kẽ!*
1. Tạm Thời Chúng Ta Lên Trang Trang Web Dùng Thử: `https://jwt.io/`.
2. Tạo 1 Cục JWT Bằng Tay Oanh Lệnh (Mô phỏng code Java):
   - Header: `{"alg": "RS256"}`
   - Payload: `{"iss": "api-ngan-hang", "sub": "api-ngan-hang", "aud": "http://localhost:8080/realms/master", "exp": 9999999999, "jti": "random_id_1"}`
3. Để Ký Cục JWT Này, Bạn Phải Dùng Lệnh OpenSSL Bóc Cục Private Key Rỗng Lệnh Từ Trong File `keystore.jks` Ra Dán Vào jwt.io Băng Tần Khung Kẽ. (Nếu Thấy Phức Tạp Lỗ Rò Lệnh, Hãy Xem Thêm Tài Liệu Hướng Dẫn Bóc File KeyStore Của Java Đỉnh Chóp!).
4. Khi Có Được Cục JWT Rác Đã Ký Bằng Private Key. Bạn Qua Lại Postman:
   - URL: `POST /token`
   - Tab **Authorization**: BỎ CHỌN Basic Khúc Tới Ngay Mạch (No Auth).
   - Tab **Body (x-www-form-urlencoded)**:
     - Thêm Oanh Khung Dịch Lụa Mạch Lệnh Mới: `client_assertion_type` = `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`
     - Thêm Lệnh Cũ Rích: `client_assertion` = `[Paste Cục JWT Chữ Ký Ngân Hàng Đó Vào Lệnh Đáy Oanh Mạng Bọc Thép]`
     - Các Biến Còn Lại (Grant_type, Code) Giữ Nguyên.
5. Send Request Bọt Mạch Kéo API Dữ Lụa! Nếu Bắn Token Về Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, Chúc Mừng Bạn Đã Tốt Nghiệp Làm Ngân Hàng Trút Cáp Mạch Máu Cắt!

---

## 4. Dọn Dẹp (Cleanup)
Hủy Mạch Docker Tránh Nặng Máy:
```bash
docker-compose down -v
```
