# Lab 1: Vận Hành Các Luồng (Flows) Của Oauth2 Bằng Postman

## 1. Mục Tiêu (Objectives)
Thực hành đóng vai Trình duyệt và Ứng dụng để "chạy bằng tay" giao thức OAuth2.
- **Task 1:** Chạy luồng M2M - Client Credentials để sinh Token hệ thống nhanh.
- **Task 2:** Thực hiện luồng đỉnh cao Authorization Code Flow (Có và Không Có PKCE).
- **Task 3:** Trải nghiệm cảm giác nhập Code qua điện thoại của luồng Device Flow.
- **Task 4:** Thử nghiệm gọi Revocation và Introspection API.

---

## 2. Chuẩn Bị (Prerequisites)
Khởi động hệ thống Keycloak bằng docker-compose đã cung cấp.

```bash
cd code
docker-compose up -d
```
Cài đặt phần mềm **Postman** (Hoặc dùng lệnh `curl` qua Terminal) để gửi Request trực tiếp.
Đăng nhập Admin Console Keycloak tại `http://localhost:8080/` với tài khoản `admin` / `admin`.

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Client Credentials Flow (M2M)
1. Trong Admin Console, tạo 1 Client tên `postman-cronjob`.
2. Bật công tắc: **Client authentication** sang **ON**. Tắt **Standard flow**. Bật **Service accounts roles** lên **ON**. Lưu lại.
3. Chuyển sang tab **Credentials**, copy cái chuỗi **Client secret** bí mật cất đi.
4. Mở Postman, tạo một Request `POST`:
   - URL: `http://localhost:8080/realms/master/protocol/openid-connect/token`
   - Tab **Body** (Chọn `x-www-form-urlencoded`):
     - `grant_type`: `client_credentials`
     - `client_id`: `postman-cronjob`
     - `client_secret`: `[Dán Secret lấy ở bước 3 vào đây]`
5. Bấm **Send**. Xem kết quả. Bạn sẽ nhận được cục JSON chứa `access_token` cực lẹ, không cần trình duyệt, không cần Username!

### Task 2: Authorization Code Flow (Luồng 2 Chặng Kinh Điển)
Ta sẽ đóng vai Trình Duyệt để lấy Mã rác, sau đó đóng vai Backend để đổi Token.
1. Tạo 1 Client tên `postman-webapp`.
2. Bật **Client authentication** sang **ON** (Lấy Secret ở Tab Credentials). Bật **Standard flow** sang **ON**.
3. Ô **Valid redirect URIs**: Điền `https://oauth.pstmn.io/v1/callback` (Đây là link ảo của Postman để hứng Code). Lưu lại.
4. Mở Tab Trình Duyệt của máy tính (Chrome/Edge), dán dòng URL "Nhử Mồi" Khổng Lồ Này Vào Trình Duyệt:
   `http://localhost:8080/realms/master/protocol/openid-connect/auth?client_id=postman-webapp&response_type=code&redirect_uri=https://oauth.pstmn.io/v1/callback&state=12345`
5. Nhấn Enter. Trình duyệt tự nhảy sang giao diện Login của Keycloak. Nhập `admin`/`admin`.
6. Ngay lập tức trình duyệt bay sang trang Postman báo lỗi (Kệ nó). Nhìn Lên Thanh Trình Duyệt Lấy Dòng URL Mới Bị Bắn Ra. Copy Cục Chuỗi Dài Loằng Ngoằng Nằm Sau Chữ `?code=xxxxxx`.
7. Trở lại Postman App, Chặn Đuôi Lấy Token Của Bước Auth Code:
   Tạo Lệnh `POST` tới `http://localhost:8080/realms/master/protocol/openid-connect/token`
   - Tab **Body** (`x-www-form-urlencoded`):
     - `grant_type`: `authorization_code`
     - `code`: `[Dán cục mã rác Code copy ở bước 6 vào]`
     - `client_id`: `postman-webapp`
     - `client_secret`: `[Secret lấy ở Tab Credentials]`
     - `redirect_uri`: `https://oauth.pstmn.io/v1/callback`
8. Bấm Send. RÚT LỤA TOKEN THÀNH CÔNG RỰC RỠ TỪ CHẶNG 2 BACKEND!

### Task 3: Trải Nghiệm Device Flow (Cho Smart TV)
1. Tạo Client tên `postman-smart-tv`. (Client auth: OFF, Tắt Standard flow, BẬT **OAuth 2.0 Device Authorization Grant**). Lưu.
2. Mở Postman, gửi Lệnh Tĩnh Của Màn Hình TV Khởi Động Oanh:
   `POST` Tới URL: `http://localhost:8080/realms/master/protocol/openid-connect/auth/device`
   - Body (`x-www-form-urlencoded`): `client_id = postman-smart-tv`. Send Lệnh.
3. Postman Xổ Bụng Nhả Data Tĩnh Về Chứa `device_code`, `user_code` và `verification_uri`.
4. (Đóng Vai Khách Hàng). Lấy Điện Thoại Quét QR Hoặc Mở Link `http://localhost:8080/realms/master/device` Trên Trình Duyệt Gõ Đúng Mã Chữ In Hoa Ngắn Nằm Ở Mục `user_code` Vào Rút Cáp Bấm Login Xác Nhận.
5. Ngay Lập Tức Quay Lại Thằng TV Postman Dội Lệnh Polling Xin Token Mới Đáy Oanh Mạng Chóp Lụa (Lặp Lại Cứ 5 Giây Tới Khi Đậu):
   `POST` Lệnh Về Token Endpoint Trọng OIDC (`/token`) 
   - Body: 
     - `grant_type`: `urn:ietf:params:oauth:grant-type:device_code`
     - `client_id`: `postman-smart-tv`
     - `device_code`: `[Cái chuỗi dài loằng ngoằng nhận ở bước 3]`
6. Bấm Lệnh Chạy Bùm Phát Trả Rút Lụa Access Token Cắt Mạch Đứt Kẽ Oanh Về Giao Diện TV!

---

## 4. Dọn Dẹp (Cleanup)
Sau khi hoàn thành thử nghiệm, nếu muốn làm sạch môi trường test và database rỗng:
```bash
docker-compose down -v
```
