> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Hướng dẫn chi tiết từng bước (Step-by-Step) cách cấu hình Keycloak hoạt động như một Identity Broker bằng cách tích hợp đăng nhập mạng xã hội (Google Social Login) và cấu hình Identity Provider Mappers.

## 1. Kịch bản Thực hành (Lab Scenario)

Giả sử bạn đang quản lý hệ thống phân quyền cho một ứng dụng thương mại điện tử. Ứng dụng này yêu cầu người dùng có thể đăng nhập nhanh chóng thông qua tài khoản Google của họ.
Bạn sẽ phải:
1. Tạo một ứng dụng trên Google Cloud Console để lấy thông tin kết nối OAuth 2.0 (Client ID & Client Secret).
2. Cấu hình tính năng Identity Provider (Google) trên Keycloak.
3. Cấu hình **Mapper** để tự động gán quyền (Role) `customer` cho bất kỳ ai đăng nhập qua Google.
4. Kiểm thử quá trình Account Linking và đăng nhập First Broker Login.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Keycloak Server đang hoạt động (ví dụ: `http://localhost:8080`).
- Quyền Admin trên Keycloak.
- Tạo một Realm mới trên Keycloak mang tên: `broker-demo-realm`.
- Một tài khoản Google cá nhân (Gmail) hoặc Google Workspace để truy cập [Google Cloud Console](https://console.cloud.google.com).
- Ứng dụng Client (tuỳ chọn) hoặc sử dụng tính năng **Account Console** mặc định của Keycloak để kiểm thử.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Tạo dự án và lấy Credentials từ Google
1. Truy cập [Google Cloud Console](https://console.cloud.google.com).
2. Ở thanh công cụ trên cùng, nhấn vào **Select a project** -> **New Project** -> Đặt tên (VD: `Keycloak Broker Demo`) -> Nhấn **Create**.
3. Chọn dự án vừa tạo.
4. Vào menu điều hướng bên trái, chọn **APIs & Services** -> **OAuth consent screen**.
   - Chọn **External** -> Nhấn **Create**.
   - Điền thông tin bắt buộc: App name (`Keycloak SSO`), User support email, Developer contact information.
   - Nhấn **Save and Continue** qua các bước còn lại (không cần thêm Scope nâng cao).
5. Chuyển sang mục **Credentials** (menu trái) -> Nhấn **Create Credentials** -> Chọn **OAuth client ID**.
   - `Application type`: **Web application**.
   - `Name`: `Keycloak Broker`.
   - `Authorized redirect URIs`: Nhập URL sau (thay đổi localhost/port nếu cần):
     `http://localhost:8080/realms/broker-demo-realm/broker/google/endpoint`
   - Nhấn **Create**.
6. Google sẽ hiển thị một popup chứa **Client ID** và **Client Secret**. Copy 2 chuỗi này vào một file note tạm thời.

### Bước 2: Cấu hình Identity Provider trên Keycloak
1. Truy cập Keycloak Admin Console, chọn realm `broker-demo-realm`.
2. Ở menu bên trái, chọn **Identity Providers**.
3. Chọn **Google** từ danh sách Social providers.
4. Cuộn xuống phần **Config**:
   - `Client ID`: Dán Client ID của Google.
   - `Client Secret`: Dán Client Secret của Google.
   - `Default Scopes`: `openid profile email` (có thể để trống, Keycloak tự động cấp).
5. Nhấn **Add** (hoặc **Save**).

### Bước 3: Cấu hình Mapper tự động gán Role
**Mục tiêu**: Bất kỳ user nào đăng nhập bằng Google sẽ tự động nhận Role `customer`.
1. Trong Realm `broker-demo-realm`, vào **Realm roles** -> Nhấn **Create role** -> Đặt tên `customer` -> Nhấn **Save**.
2. Quay lại **Identity Providers** -> Chọn **Google**.
3. Chuyển sang tab **Mappers** -> Nhấn **Add mapper**.
4. Điền cấu hình:
   - `Name`: `Assign Customer Role`
   - `Sync Mode Override`: `Import` (Chỉ chạy ở lần đăng nhập đầu tiên) hoặc `Force` (Chạy ở mọi lần đăng nhập). Chọn `Import`.
   - `Mapper Type`: `Hardcoded Role`
   - `Role`: Chọn role `customer`.
5. Nhấn **Save**.

### Bước 4: Kiểm thử Đăng nhập
1. Mở một trình duyệt ẩn danh (Incognito) hoặc sử dụng một trình duyệt khác.
2. Truy cập URL cổng tự phục vụ của người dùng (Account Console):
   `http://localhost:8080/realms/broker-demo-realm/account`
3. Nhấn **Sign In**.
4. Bạn sẽ thấy trên màn hình đăng nhập có xuất hiện một nút bổ sung: **Google**.
5. Nhấn vào **Google**, bạn sẽ được chuyển hướng (redirect) sang trang đăng nhập của Google.
6. Đăng nhập bằng tài khoản Gmail của bạn và đồng ý uỷ quyền (nếu được hỏi).
7. Google sẽ redirect bạn trở lại Keycloak.
8. Lần đầu tiên, Keycloak (First Broker Login Flow) có thể yêu cầu bạn nhập/kiểm tra lại thông tin (Update Account Information). Nhấn **Submit**.
9. Bạn sẽ đăng nhập thành công vào trang quản lý Account của Keycloak.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

- **Xác minh người dùng đã được Import**:
  - Trở lại cửa sổ trình duyệt đang chạy quyền Admin.
  - Vào **Users** -> Nhấn tìm kiếm. Bạn sẽ thấy tài khoản Gmail vừa đăng nhập hiển thị trong danh sách.
  - Click vào User đó, chuyển sang tab **Role mapping**.
  - Kiểm tra xem role `customer` đã được gán tự động thành công chưa.
  - Chuyển sang tab **Identity Provider Links**, bạn sẽ thấy tài khoản này đang được liên kết với ID của Google.

> [!WARNING]
> **Troubleshooting: Lỗi `Redirect URI mismatch` từ Google**
> Đây là lỗi cực kỳ phổ biến. Đảm bảo rằng URL trong mục `Authorized redirect URIs` trên Google Cloud Console khớp đến từng ký tự với URL trên Keycloak. Nếu bạn truy cập Keycloak bằng IP (VD: `127.0.0.1`), bạn phải khai báo URI dùng `127.0.0.1` trên Google, không thể dùng `localhost` hoặc ngược lại.

> [!IMPORTANT]
> **Troubleshooting: Không thấy nút Google xuất hiện**
> Kiểm tra lại trong cấu hình Identity Provider của Keycloak, đảm bảo thuộc tính `Enabled` đang được bật thành `ON`. Đồng thời, trong phần Authentication -> Browser Flow, đảm bảo `Identity Provider Redirector` không bị cấu hình sai.
