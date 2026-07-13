> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Hướng dẫn tạo và cấu hình các loại Clients trong Keycloak, bao gồm Public Client, Confidential Client, và thiết lập Service Accounts cho Server-to-Server communication.

## 1. Kịch bản Thực hành (Lab Scenario)

Trong một kiến trúc hệ thống thực tế, Keycloak không chỉ bảo vệ một ứng dụng duy nhất mà thường xuyên quản lý danh tính cho nhiều ứng dụng khác nhau. Mỗi ứng dụng sẽ có tính chất bảo mật riêng, do đó chúng ta cần khai báo chúng như là các `Clients` khác nhau:
- **Frontend App (Angular/React)**: Chạy trên trình duyệt của người dùng, không thể bảo mật mã bí mật (Client Secret). Ứng dụng này sẽ được cấu hình là **Public Client**.
- **Backend API (Spring Boot/Node.js)**: Chạy an toàn trên máy chủ, có khả năng giữ Client Secret. Ứng dụng này sẽ được thiết lập là **Confidential Client**.
- **Cronjob/Worker**: Cần truy cập API định kỳ mà không có sự tham gia của con người. Ứng dụng này sử dụng **Service Accounts**.

Bài Lab này sẽ mô phỏng quá trình tạo và cấu hình chi tiết cho 3 loại ứng dụng trên.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Keycloak Server đang hoạt động ổn định.
- Bạn đã có quyền Admin để thao tác trên **Master Realm** hoặc một Realm tự tạo (ví dụ: `clients-demo-realm`).
- Cài đặt công cụ dòng lệnh `cURL` để giả lập các HTTP Requests.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Khởi tạo Realm cho bài Lab
1. Truy cập Keycloak Admin Console.
2. Tạo mới một Realm bằng cách nhấn `Create Realm` -> Đặt tên: `clients-demo-realm`.

### Bước 2: Tạo Public Client cho Frontend
1. Ở menu bên trái, chọn **Clients** -> Nhấn **Create client**.
2. Thiết lập thông số cơ bản:
   - `Client type`: `OpenID Connect`.
   - `Client ID`: `frontend-spa`.
   - Nhấn **Next**.
3. Cấu hình xác thực và luồng:
   - Đảm bảo **Client authentication** là **OFF** (Đắt tắt). Đây là thiết lập sống còn để biến Client thành Public Client, nó sẽ không yêu cầu Secret.
   - Bật **Standard flow** và **Direct access grants**.
   - Nhấn **Next**.
4. Thiết lập URL liên kết:
   - `Valid redirect URIs`: `http://localhost:3000/*` (URL của Frontend, dùng để nhận token về).
   - `Web origins`: `http://localhost:3000` (Cấu hình CORS để cho phép trình duyệt gọi API Keycloak).
5. Nhấn **Save**.

### Bước 3: Tạo Confidential Client cho Backend API
1. Trở lại mục **Clients** -> Nhấn **Create client**.
2. Thiết lập cơ bản:
   - `Client type`: `OpenID Connect`.
   - `Client ID`: `backend-api`.
   - Nhấn **Next**.
3. Bật **Client authentication** -> **ON**. Tùy chọn này quy định đây là Confidential Client, buộc Keycloak sinh ra một Client Secret.
4. Ở phần Flows, bật **Standard flow**. Nhấn **Next**.
5. Đặt `Valid redirect URIs`: `http://localhost:8080/login/oauth2/code/keycloak` (URL callback của backend).
6. Nhấn **Save**.
7. Kiểm tra Secret: Chuyển sang tab **Credentials**, bạn sẽ thấy trường **Client secret**. Lưu giá trị này lại.

### Bước 4: Tạo Service Account cho Cronjob/Worker
1. Quay lại **Clients** -> **Create client**.
2. Nhập `Client ID`: `cronjob-worker` -> Nhấn **Next**.
3. Bật **Client authentication** -> **ON**.
4. Ở phần Flows:
   - Tắt **Standard flow** (Vì worker không có trình duyệt để thực hiện redirect).
   - Bật **Service accounts roles**.
5. Nhấn **Next** và **Save**.
6. Gán Role cho Service Account (Tuỳ chọn nâng cao):
   - Chuyển sang tab **Service account roles** của client `cronjob-worker`.
   - Nhấn **Assign role** và chọn các role cần thiết để cấp quyền hạn cụ thể cho worker này.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

- **Kiểm tra luồng Public Client (Frontend)**:
  Sử dụng Resource Owner Password Credentials (Direct Access Grant) bằng cURL để giả lập:
  ```bash
  curl -X POST http://localhost:8080/realms/clients-demo-realm/protocol/openid-connect/token \
    -d "client_id=frontend-spa" \
    -d "username=<TEST_USER>" \
    -d "password=<TEST_PASSWORD>" \
    -d "grant_type=password"
  ```
  *(Lưu ý: Bạn phải tạo user trước đó).* Yêu cầu này sẽ thành công và trả về token mà KHÔNG CẦN `client_secret`.

- **Kiểm tra luồng Confidential Client / Service Account**:
  Thử chạy lệnh cURL với `grant_type=client_credentials` đối với `cronjob-worker`:
  ```bash
  curl -X POST http://localhost:8080/realms/clients-demo-realm/protocol/openid-connect/token \
    -d "client_id=cronjob-worker" \
    -d "client_secret=<SECRET_CUA_CRONJOB_WORKER>" \
    -d "grant_type=client_credentials"
  ```
  Hệ thống sẽ trả về Access Token.

> [!WARNING]
> **Lỗi thường gặp: Missing/Invalid Redirect URI**
> Nếu trình duyệt báo lỗi `Invalid redirect uri`, hãy đối chiếu chính xác URL hiện tại của thanh địa chỉ trình duyệt với danh sách cấu hình trong mục `Valid redirect URIs` của Client tương ứng. Ký tự gạch chéo cuối (`/`) cũng được tính là một URL khác.
