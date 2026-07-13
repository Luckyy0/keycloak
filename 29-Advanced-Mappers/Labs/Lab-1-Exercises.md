> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Ứng dụng các kiến thức về Advanced Mappers để tùy biến Token. Cụ thể là triển khai Role Mapper (để chuẩn hóa quyền cho ứng dụng giả định) và Hardcoded Claim Mapper (để gắn cờ Tenant ID cố định) trên môi trường Keycloak cục bộ.

## 1. Kịch bản Thực hành (Lab Scenario)

Công ty bạn đang phát triển một hệ thống API Gateway bằng Spring Cloud Gateway để định tuyến request từ bên ngoài vào. Hệ thống này có hai yêu cầu đặc thù về cấu trúc JWT (Access Token):

1. **Yêu cầu 1 (Ánh xạ Role):** Hệ thống API Gateway sử dụng phân quyền mặc định của Spring Security, yêu cầu danh sách các quyền phải nằm ở claim gốc tên là `authorities` và mỗi quyền phải bắt đầu bằng tiền tố `ROLE_` (ví dụ: `ROLE_api-admin`). Tuy nhiên Keycloak lại lưu ở `realm_access.roles` mà không có tiền tố.
2. **Yêu cầu 2 (Định danh Tenant):** Tất cả các người dùng đăng nhập qua Client (ứng dụng Front-end) tên là `frontend-portal` đều phải được Gateway đánh dấu chung một định danh môi trường. Bạn cần gắn tĩnh (hardcode) claim `tenant_domain: "asia.myapp.com"` vào Token cho ứng dụng này.

Nhiệm vụ của bạn là cấu hình Protocol Mappers trong Keycloak để giải quyết cả hai bài toán này mà không phải viết mã (code) phía Backend.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Đã cài đặt và chạy Keycloak phiên bản >= 22 (hoặc cao hơn qua Docker/Podman).
- Đã có tài khoản Keycloak Admin (`admin` / `admin`).
- Đã tạo sẵn Realm tên là `Lab-Realm`.
- Đã tạo một User tên `test-user` (có password) trong Realm này.
- Công cụ kiểm tra: Trình duyệt web và công cụ [jwt.io](https://jwt.io) để giải mã Token.

*Nếu chưa có Keycloak đang chạy, hãy dùng lệnh sau để khởi động tạm thời:*
```bash
docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:latest start-dev
```

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Thiết lập Client và gán Role

1. Truy cập Keycloak Admin Console: `http://localhost:8080/admin`. Đăng nhập và chuyển sang `Lab-Realm`.
2. Tạo Client:
   - Menu trái -> **Clients** -> **Create client**.
   - **Client ID:** `frontend-portal`.
   - Bấm **Next**. Để mặc định, ở trang cuối đổi **Valid redirect URIs** thành `*` (chỉ dùng cho môi trường dev). Bấm **Save**.
3. Tạo Role cấp Realm:
   - Menu trái -> **Realm roles** -> **Create role**.
   - **Role name:** `api-admin`. Bấm **Save**.
4. Gán Role cho User:
   - Menu trái -> **Users** -> Tìm kiếm và chọn `test-user`.
   - Chuyển sang tab **Role mapping**.
   - Bấm **Assign role** -> Chọn `api-admin` -> Bấm **Assign**.

### Bước 2: Cấu hình Role Mapper để đáp ứng Yêu cầu 1

Chúng ta không cấu hình trực tiếp vào Client mà dùng Client Scopes để tái sử dụng.

1. Tạo Scope mới:
   - Menu trái -> **Client Scopes** -> **Create client scope**.
   - **Name:** `spring-authorities`.
   - **Type:** `Default` (để tự động thêm vào mọi token của client).
   - Bấm **Save**.
2. Cấu hình Mapper trong Scope:
   - Trong giao diện của `spring-authorities`, chọn tab **Mappers**.
   - Bấm **Add mapper** -> **By configuration**.
   - Chọn **User Realm Role**.
   - Cấu hình như sau:
     - **Name:** `roles-to-authorities`
     - **Realm Role prefix:** `ROLE_` (đây là tiền tố sẽ thêm vào).
     - **Multivalued:** `ON` (để tạo mảng JSON, điều này rất quan trọng).
     - **Token Claim Name:** `authorities` (Tên thuộc tính trong Token).
     - **Add to ID token:** `OFF`.
     - **Add to access token:** `ON`.
   - Bấm **Save**.
3. Gán Scope vào Client:
   - Về lại menu **Clients** -> Chọn `frontend-portal` -> Tab **Client scopes**.
   - Bấm **Add client scope** -> Chọn `spring-authorities` -> Bấm **Add** -> Chọn **Default**.

### Bước 3: Cấu hình Hardcoded Claim Mapper đáp ứng Yêu cầu 2

Vì Tenant là đặc thù của Client `frontend-portal`, ta sẽ cấu hình Hardcoded Mapper trực tiếp vào Client này.

1. Menu trái -> **Clients** -> Chọn `frontend-portal`.
2. Chuyển sang tab **Client scopes**.
3. Bấm trực tiếp vào link của client scope tên là `frontend-portal-dedicated` (Đây là scope độc lập của riêng client này).
4. Chuyển sang tab **Mappers**.
5. Bấm **Add mapper** -> **By configuration**.
6. Chọn **Hardcoded claim**.
7. Cấu hình như sau:
   - **Name:** `inject-tenant-domain`
   - **Token Claim Name:** `tenant_domain`
   - **Claim value:** `asia.myapp.com`
   - **Claim JSON Type:** `String`
   - **Add to access token:** `ON`
8. Bấm **Save**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Lấy Token để kiểm tra

Sử dụng Endpoint cấp token của Keycloak để mô phỏng một đăng nhập từ Client `frontend-portal` bằng Resource Owner Password Credentials flow (cho tiện kiểm tra):

Mở Terminal và chạy lệnh `curl`:
```bash
curl -X POST "http://localhost:8080/realms/Lab-Realm/protocol/openid-connect/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "client_id=frontend-portal" \
     -d "username=test-user" \
     -d "password=<mật khẩu của test-user>" \
     -d "grant_type=password"
```

Bạn sẽ nhận được một phản hồi JSON có chứa trường `access_token`.

### 4.2. Giải mã và Nghiệm thu

Copy đoạn mã chuỗi khổng lồ của `access_token` và dán vào phần "Encoded" trên trang web [https://jwt.io](https://jwt.io).

Nhìn vào phần "Payload (Data)" bên phải, bạn PHẢI thấy kết quả tương tự như sau:

```json
{
  "exp": 1699999999,
  "iat": 1699999000,
  "jti": "5a5c...",
  "iss": "http://localhost:8080/realms/Lab-Realm",
  "sub": "b2c3...",
  "typ": "Bearer",
  "azp": "frontend-portal",
  "session_state": "6...",
  "tenant_domain": "asia.myapp.com",
  "authorities": [
    "ROLE_api-admin",
    "ROLE_default-roles-lab-realm"
  ]
}
```

**Tiêu chí thành công:**
- Xuất hiện mảng `"authorities"`.
- Trong mảng đó có chứa chuỗi `"ROLE_api-admin"`.
- Xuất hiện trường phẳng (flat string) `"tenant_domain": "asia.myapp.com"`.

### 4.3. Lỗi thường gặp (Troubleshooting)

- **Lỗi:** Trường `authorities` chỉ là chuỗi text (`"authorities": "ROLE_api-admin"`) thay vì mảng JSON (Array).
  - *Cách sửa:* Bạn quên bật nút gạt **Multivalued** sang `ON` ở Bước 2. Hãy quay lại sửa cấu hình Mapper.
- **Lỗi:** Bắn lệnh `curl` báo lỗi `401 Unauthorized` hoặc `invalid_client`.
  - *Cách sửa:* Nếu Client của bạn cấu hình là "Confidential" (có Client Secret), lệnh curl bắt buộc phải truyền thêm `-d "client_secret=..."`. Trong lab này chúng ta đang dùng Public Client mặc định.
- **Lỗi:** Không thấy `tenant_domain` trong Token.
  - *Cách sửa:* Có thể bạn đã tạo mapper Hardcoded claim nhưng không bật nút **Add to access token**. Kiểm tra lại cấu hình ở bước 3.
