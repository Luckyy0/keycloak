> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Luyện tập tương tác trực tiếp với các luồng OAuth 2.0 (Authorization Code, Client Credentials) trên Keycloak thông qua các công cụ dòng lệnh (cURL) và Postman để nắm vững cấu trúc HTTP Request/Response.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là một kỹ sư bảo mật ứng dụng đang trong quá trình tích hợp Microservices của công ty với Keycloak. Trước khi bắt đầu code bằng ngôn ngữ lập trình cụ thể (Java/NodeJS), bạn cần mô phỏng thủ công các luồng lấy Token. 
Trong bài Lab này, chúng ta sẽ:
- Thiết lập một Realm, Users, và hai loại Clients (Public và Confidential).
- Dùng trình duyệt và Postman để mô phỏng từng bước của luồng **Authorization Code**.
- Dùng cURL để mô phỏng luồng **Client Credentials**.
- Dùng Access Token vừa lấy để kiểm tra **Introspection Endpoint**.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một phiên bản **Keycloak Server** đang chạy tại `http://localhost:8080`.
- Có tài khoản Admin (admin/admin).
- Công cụ **Postman** đã cài đặt.
- Terminal có sẵn công cụ **cURL** và **jq** (tùy chọn để format JSON).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Khởi tạo dữ liệu trên Keycloak
1. Truy cập Keycloak Admin Console (`http://localhost:8080/admin`).
2. Tạo một Realm mới tên là `oauth-lab`.
3. Tạo một User mới: 
   - Username: `testuser`
   - Cài đặt Credentials (Password): `password123` (Nhớ tắt `Temporary`).
4. **Tạo Confidential Client (Cho Client Credentials):**
   - Vào `Clients`, tạo mới `backend-service`.
   - Client authentication: `ON`.
   - Authorization: `OFF`.
   - Lưu lại.
   - Tại mục **Capability config**, bật **Service accounts roles**.
   - Tại tab **Credentials**, sao chép giá trị `Client secret`.
5. **Tạo Public Client (Cho Authorization Code):**
   - Tạo Client `spa-app`.
   - Client authentication: `OFF` (Public Client).
   - Valid redirect URIs: `https://oauth.pstmn.io/v1/callback` (Dành cho Postman).
   - Web origins: `*`

### Bước 3.2: Thực hành luồng Client Credentials (M2M) bằng cURL
1. Mở Terminal.
2. Bạn đóng vai trò là `backend-service` muốn xin Token để gọi qua service khác.
3. Chạy lệnh sau (Thay `<secret_cua_ban>` bằng chuỗi copy ở Bước 3.1):
```bash
curl -X POST "http://localhost:8080/realms/oauth-lab/protocol/openid-connect/token" \
  -d "client_id=backend-service" \
  -d "client_secret=<secret_cua_ban>" \
  -d "grant_type=client_credentials" | jq
```
4. Quan sát kết quả. Bạn sẽ nhận được một JSON trả về chứa `access_token` nhưng KHÔNG CÓ `refresh_token`.

### Bước 3.3: Thực hành luồng Authorization Code với Postman
1. Mở Postman, tạo một HTTP Request mới.
2. Tại tab **Authorization**, chọn Type là **OAuth 2.0**.
3. Ở bảng cấu hình bên phải, cấu hình như sau:
   - **Grant Type:** `Authorization Code`
   - **Callback URL:** `https://oauth.pstmn.io/v1/callback` (Bạn nên đánh dấu tích vào "Authorize using browser" nếu muốn).
   - **Auth URL:** `http://localhost:8080/realms/oauth-lab/protocol/openid-connect/auth`
   - **Access Token URL:** `http://localhost:8080/realms/oauth-lab/protocol/openid-connect/token`
   - **Client ID:** `spa-app`
   - **Client Secret:** Bỏ trống (vì đây là Public Client).
   - **Scope:** `openid profile email`
4. Cuộn xuống và click vào nút **"Get New Access Token"**.
5. Postman sẽ bật lên cửa sổ trình duyệt (Keycloak Login Screen).
6. Đăng nhập bằng `testuser` / `password123`.
7. Trình duyệt tự động redirect lại Postman và bạn sẽ thấy Token xuất hiện trong cửa sổ quản lý Token.

### Bước 3.4: Sử dụng Token Introspection Endpoint
1. Copy đoạn `access_token` vừa nhận được từ cURL ở Bước 3.2 (Của backend-service).
2. Tạo request Postman mới.
3. Method: **POST**
4. URL: `http://localhost:8080/realms/oauth-lab/protocol/openid-connect/token/introspect`
5. Tab **Authorization**: Chọn Basic Auth, điền Username là `backend-service` và Password là `<secret_cua_ban>`.
6. Tab **Body**: Chọn `x-www-form-urlencoded`.
   - Key: `token`, Value: Dán đoạn Token vừa copy vào.
7. Click Send và xem JSON trả về (Sẽ có `active: true`).

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

- **Xác thực thành công luồng M2M:** Nếu JSON trả về có trường `access_token` dạng `eyJh...`, bài lab M2M thành công.
- **Xác thực thành công Introspection:** API trả về trạng thái HTTP `200 OK` và thân JSON có thuộc tính `active: true`.
- **Lỗi thường gặp (Troubleshooting):**
  - **Lỗi `invalid_client` trên cURL:** Do bạn nhập sai client_secret hoặc chưa bật `Client authentication` thành `ON`.
  - **Lỗi `Invalid redirect uri` trên Postman:** Xảy ra do URL thiết lập trong Postman không khớp hoàn toàn (từng ký tự) với chuỗi `Valid redirect URIs` mà bạn đã nhập trong bảng điều khiển Keycloak (`https://oauth.pstmn.io/v1/callback`).
  - **Lỗi `HTTP 401` trên API Introspection:** Resource Server thực hiện việc tra cứu (caller) phải cung cấp xác thực của chính nó (Client credentials). Việc bạn quên chọn Basic Auth ở bước 3.4 sẽ dẫn đến lỗi này.
