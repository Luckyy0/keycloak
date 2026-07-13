> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Hướng dẫn chi tiết cách cấu hình và thực thi các OpenID Connect (OIDC) flow cốt lõi (Authorization Code Flow, Implicit Flow, Client Credentials Flow) bằng Keycloak và công cụ dòng lệnh (cURL, Postman).

## 1. Kịch bản Thực hành (Lab Scenario)

Trong môi trường doanh nghiệp hiện đại, các ứng dụng (Web App, SPA, Microservices) cần giao tiếp với nhau và với người dùng một cách an toàn. Giao thức OIDC là tiêu chuẩn de facto cho việc xác thực và phân quyền (Authentication & Authorization) hiện nay.
Trong bài Lab này, chúng ta sẽ đóng vai trò là một System Administrator tích hợp ba ứng dụng tiêu biểu vào Keycloak:
- **Ứng dụng Web truyền thống (Traditional Web App)**: Cần sử dụng **Authorization Code Flow** vì có khả năng bảo mật thông tin Client Secret một cách an toàn trên server.
- **Ứng dụng Single Page Application (SPA)**: Giả lập việc lấy token trực tiếp qua trình duyệt với **Implicit Flow** (Mặc dù PKCE được khuyến cáo thay thế hiện nay, việc hiểu Implicit Flow vẫn cần thiết cho hệ thống cũ).
- **Backend Service (Microservice)**: Cần giao tiếp trực tiếp với một API khác mà không có sự tương tác của người dùng, sử dụng **Client Credentials Flow**.

## 2. Chuẩn bị Môi trường (Prerequisites)

- **Keycloak Server**: Đang chạy (ví dụ tại `http://localhost:8080`).
- **Tài khoản Admin Keycloak**: Đã được thiết lập.
- **Realm**: Một Realm mới mang tên `oidc-demo-realm`.
- **User**: Một user test tên `testuser` (password: `testpass`) thuộc `oidc-demo-realm`.
- **Công cụ kiểm thử**: Postman hoặc công cụ dòng lệnh `cURL` + tiện ích `jq` để parse JSON.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Tạo Realm và User cơ bản
1. Đăng nhập vào **Keycloak Admin Console**.
2. Click vào tên Realm hiện tại ở góc trên bên trái, chọn **Create Realm**.
3. Nhập `Realm name`: `oidc-demo-realm` và nhấn **Create**.
4. Chuyển sang mục **Users** ở menu bên trái. Nhấn **Add user**.
   - `Username`: `testuser`
   - Nhấn **Create**.
5. Chuyển sang tab **Credentials**, nhấn **Set password**.
   - `Password`: `testpass`
   - `Password confirmation`: `testpass`
   - Tắt `Temporary` -> Nhấn **Save**.

### Bước 2: Thực hành Authorization Code Flow
**Mục tiêu**: Lấy Authorization Code và đổi lấy ID Token, Access Token.

1. **Tạo Client**:
   - Vào **Clients** -> **Create client**.
   - `Client type`: `OpenID Connect`.
   - `Client ID`: `web-app-client`.
   - Nhấn **Next**.
   - Bật **Client authentication** (để biến nó thành Confidential Client).
   - Chọn **Standard flow** (tương đương Authorization Code Flow).
   - Nhấn **Next**.
   - `Valid redirect URIs`: `http://localhost:8000/*` (hoặc một URL giả định).
   - Nhấn **Save**.
2. **Lấy Client Secret**:
   - Chuyển sang tab **Credentials** của client vừa tạo, copy `Client secret`.
3. **Mô phỏng Request lấy Code**:
   - Mở trình duyệt và truy cập URL sau (thay đổi thông tin nếu cần):
     ```text
     http://localhost:8080/realms/oidc-demo-realm/protocol/openid-connect/auth?client_id=web-app-client&response_type=code&redirect_uri=http://localhost:8000/callback&scope=openid
     ```
   - Trình duyệt sẽ chuyển hướng đến trang đăng nhập của Keycloak. Nhập `testuser` / `testpass`.
   - Sau khi đăng nhập, trình duyệt sẽ redirect về: `http://localhost:8000/callback?code=eyJhbG...`
   - Copy giá trị của tham số `code`.
4. **Đổi Code lấy Token (Exchange Token)**:
   - Mở terminal, sử dụng `cURL`:
     ```bash
     curl -X POST http://localhost:8080/realms/oidc-demo-realm/protocol/openid-connect/token \
       -H "Content-Type: application/x-www-form-urlencoded" \
       -d "grant_type=authorization_code" \
       -d "client_id=web-app-client" \
       -d "client_secret=<YOUR_CLIENT_SECRET>" \
       -d "code=<CODE_TU_BUOC_TREN>" \
       -d "redirect_uri=http://localhost:8000/callback"
     ```
   - Bạn sẽ nhận được JSON response chứa `access_token`, `id_token`, và `refresh_token`.

### Bước 3: Thực hành Client Credentials Flow
**Mục tiêu**: Lấy Access Token cho một Service Account, không qua tương tác người dùng.

1. **Tạo Client**:
   - Vào **Clients** -> **Create client**.
   - `Client ID`: `backend-service-client`.
   - Nhấn **Next**. Bật **Client authentication**.
   - Tắt **Standard flow**, bật **Service accounts roles** (cho phép Client Credentials).
   - Nhấn **Save**.
2. **Lấy Token qua cURL**:
   - Copy `Client secret` từ tab Credentials.
   - Chạy lệnh sau trong terminal:
     ```bash
     curl -X POST http://localhost:8080/realms/oidc-demo-realm/protocol/openid-connect/token \
       -H "Content-Type: application/x-www-form-urlencoded" \
       -d "grant_type=client_credentials" \
       -d "client_id=backend-service-client" \
       -d "client_secret=<YOUR_CLIENT_SECRET>"
     ```
   - Nhận được JSON response chứa `access_token` (không có `id_token` hay `refresh_token` vì đây là uỷ quyền máy-máy).

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

- **Kiểm tra Token (JWT Analysis)**:
  Copy giá trị `access_token` hoặc `id_token` thu được và dán vào [jwt.io](https://jwt.io). Phân tích phần Payload để đảm bảo:
  - `iss`: Phải là `http://localhost:8080/realms/oidc-demo-realm`.
  - `aud`: Tương ứng với Client ID.
  - `sub`: ID của user (`testuser`) hoặc Service Account ID.

> [!WARNING]
> **Troubleshooting: Lỗi `invalid_grant`**
> - Xảy ra khi Authorization Code đã hết hạn (mặc định Keycloak chỉ cho phép mã này sống trong 1 phút) hoặc Code đã được sử dụng một lần.
> - Khắc phục: Yêu cầu lấy Code mới qua trình duyệt và đổi Token ngay lập tức.

> [!WARNING]
> **Troubleshooting: Lỗi `invalid_client` hoặc `unauthorized_client`**
> - Thường do nhập sai `Client Secret` hoặc quên bật **Client authentication** trong cấu hình Client trên Keycloak. Kiểm tra kỹ lại tab Credentials.
