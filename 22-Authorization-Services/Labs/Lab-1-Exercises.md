> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Thực hành thiết lập một Resource Server cơ bản trên Keycloak, cấu hình Resource, Policy, Permission và kiểm thử quá trình Requesting Party Token (RPT) hoạt động qua API.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là kỹ sư bảo mật của công ty cung cấp dịch vụ chia sẻ tài liệu nội bộ `DocShare`.
Hệ thống có một tài liệu tuyệt mật tên là `Financial_Report_2024`. Yêu cầu kinh doanh đặt ra:
1.  Chỉ những User nào mang chức vụ `Manager` (được gắn qua Role) mới có quyền `view` (xem) tài liệu này.
2.  Bạn không được code logic `if (user.isManager)` vào trong Backend API. Mọi logic phân quyền phải do Keycloak đảm nhận bằng hệ thống Authorization Services (Centralized Policy).
Trong bài Lab này, chúng ta sẽ giả lập API bằng Postman, biến Keycloak thành Policy Decision Point (PDP) hoàn chỉnh.

## 2. Chuẩn bị Môi trường (Prerequisites)

*   **Keycloak Server:** Đang chạy ổn định tại `http://localhost:8080`.
*   **Realm:** Bạn đã tạo sẵn một Realm tên là `docshare-realm`.
*   **User:**
    *   Tạo user `alice` (mật khẩu `123`). Không có Role gì đặc biệt.
    *   Tạo user `bob` (mật khẩu `123`). Gắn Role ở mức Realm là `manager`.
*   **Công cụ:** Postman hoặc cURL để gọi API.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Tạo và cấu hình Authorization Client (Resource Server)
1. Truy cập Admin Console, chọn `docshare-realm`.
2. Chuyển sang menu **Clients**, tạo Client mới:
   * Client ID: `doc-api`
   * Client Authentication: Bật **ON** (đây là Confidential client).
   * Authorization: Bật **ON** (Điều này cực kỳ quan trọng, nó biến Client thành một Resource Server mang tính chất UMA).
3. Lưu lại. Chuyển sang tab **Credentials**, copy lại chuỗi `Client Secret`.

### Bước 2: Tạo Resource và Scope
1. Vẫn ở Client `doc-api`, chuyển sang tab **Authorization**.
2. Chọn tab con **Authorization Scopes**. Nhấn `Create scope` và tạo scope tên là `view`.
3. Chuyển sang tab con **Resources**. Nhấn `Create resource`.
   * Name: `Financial_Report_2024`
   * Type: `urn:docshare:resources:report`
   * Scopes: Chọn scope `view` vừa tạo ở trên.
4. Nhấn Save.

### Bước 3: Tạo Policy (Role-based)
1. Chuyển sang tab con **Policies**. Nhấn `Create policy` và chọn kiểu **Role**.
2. Cấu hình:
   * Name: `Only_Managers_Policy`
   * Realm Roles: Tìm và chọn Role `manager`. Đánh dấu check để yêu cầu bắt buộc (Required).
   * Logic: `Positive`.
3. Nhấn Save.

### Bước 4: Tạo Permission kết nối (Binding)
1. Chuyển sang tab con **Permissions**. Nhấn `Create permission` và chọn kiểu **Resource-based**.
2. Cấu hình:
   * Name: `View_Report_Permission`
   * Resources: Chọn `Financial_Report_2024`
   * Apply Policy: Chọn `Only_Managers_Policy`
   * Decision Strategy: `Unanimous`
3. Nhấn Save. Quá trình cấu hình PAP trên Keycloak đã hoàn tất.

### Bước 5: Kiểm thử phân quyền (Dùng Keycloak Evaluation Tool)
1. Chuyển sang tab con **Evaluate**.
2. Tại mục `Identity Information`:
   * Client: `doc-api`
   * User: Nhập `alice`.
3. Tại mục `Contextual Information`: Chọn Resource là `Financial_Report_2024` và Scope là `view`.
4. Nhấn **Evaluate**. Kết quả sẽ là **DENY** (Bị từ chối) vì alice không có role manager.
5. Thử đổi User thành `bob` và nhấn **Evaluate**. Kết quả sẽ là **PERMIT**.

### Bước 6: Gọi API lấy Requesting Party Token (RPT) qua HTTP
Để thấy rõ quá trình hoạt động dưới góc độ API. Chúng ta dùng Postman giả lập Client.

**Lấy Access Token cho Bob:**
```bash
curl -X POST http://localhost:8080/realms/docshare-realm/protocol/openid-connect/token \
  -d "client_id=doc-api" \
  -d "client_secret=<CLIENT_SECRET_CỦA_BẠN>" \
  -d "username=bob" \
  -d "password=123" \
  -d "grant_type=password"
```
Ghi lại chuỗi `access_token` của Bob.

**Đổi Token thường lấy RPT (Đánh giá ủy quyền):**
```bash
curl -X POST http://localhost:8080/realms/docshare-realm/protocol/openid-connect/token \
  -H "Authorization: Bearer <ACCESS_TOKEN_CỦA_BOB>" \
  -d "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \
  -d "audience=doc-api" \
  -d "permission=Financial_Report_2024#view"
```

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

*   **Nghiệm thu thành công:**
    * Nếu gọi với Token của Bob, Keycloak sẽ trả về HTTP 200 kèm theo một Access Token mới (chính là RPT).
    * Lấy chuỗi Token RPT đó dán vào trang [jwt.io](https://jwt.io). Ở phần Payload, bạn sẽ thấy cấu trúc JSON chứa mảng `authorization.permissions`, chỉ ra rằng Token này mang quyền `view` đối với tài nguyên `Financial_Report_2024`.
    * Nếu lặp lại lệnh RPT API trên nhưng dùng Token của `alice`, Keycloak sẽ trả về HTTP 403 Forbidden kèm dòng lỗi `not_authorized` vì Policy đã chặn lại.

*   **Lỗi thường gặp (Troubleshooting):**
    * *Lỗi `invalid_request` hoặc thiếu `permission`:* Kiểm tra xem cấu trúc lệnh gửi RPT Request có truyền đúng chuỗi định dạng `<Tên_Resource>#<Tên_Scope>` (ví dụ `Financial_Report_2024#view`) hay không.
    * *Lỗi 403 mặc dù Bob là Manager:* Kiểm tra xem Policy Enforcement Mode của Client `doc-api` trong tab Settings đang ở trạng thái nào. Đảm bảo Role được gán đúng cách cho Bob (kiểm tra tab Role Mappings của User bob).
