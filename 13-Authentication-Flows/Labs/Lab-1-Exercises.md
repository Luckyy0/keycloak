> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Tự tay cấu hình và tùy biến luồng xác thực (Authentication Flow) trong Keycloak, kết hợp các mức độ Require (Bắt buộc/Thay thế) để thiết lập MFA linh hoạt.

## 1. Kịch bản Thực hành (Lab Scenario)

Công ty của bạn yêu cầu thắt chặt bảo mật cho ứng dụng quản trị nội bộ. Chính sách bảo mật quy định như sau:
- Người dùng bình thường chỉ cần đăng nhập bằng `Username` và `Password`.
- Đối với những người dùng có quyền quản trị (Role `admin`), hệ thống phải bắt buộc yêu cầu thêm bước xác thực thứ 2: **Mã OTP (Time-based One-Time Password)**.
- Người dùng có quyền chọn sử dụng Mật khẩu HOẶC Đăng nhập không mật khẩu thông qua **WebAuthn** (nếu thiết bị hỗ trợ).

Để giải quyết kịch bản này, bạn sẽ cần tạo một **Custom Authentication Flow** sao chép từ luồng Browser mặc định, sau đó sử dụng **Conditional Flow** và kết hợp các toán tử **REQUIRED/ALTERNATIVE**.

## 2. Chuẩn bị Môi trường (Prerequisites)

Để thực hiện bài Lab, bạn cần chuẩn bị:
1. Máy chủ Keycloak đang chạy (phiên bản 21.0.0 trở lên, chạy bằng Docker hoặc Standalone).
2. Tài khoản quản trị cấp cao (Admin) để đăng nhập vào Keycloak Admin Console.
3. Ứng dụng điện thoại để quét mã OTP (Google Authenticator, Microsoft Authenticator hoặc Authy).
4. (Tùy chọn) Máy tính hoặc điện thoại có hỗ trợ WebAuthn (TouchID, Windows Hello, hoặc YubiKey).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo người dùng và gán Role

1. Đăng nhập vào **Keycloak Admin Console**.
2. Chọn Realm (ví dụ: `master` hoặc tạo một Realm mới tên là `SecuredRealm`).
3. Đi đến menu **Realm Roles**, nhấn **Create role** và tạo một role có tên là `admin`.
4. Đi đến menu **Users**, nhấn **Add user**:
   - `Username`: `alice_admin`
   - Nhấn **Create**.
5. Trong tab **Credentials** của người dùng `alice_admin`, thiết lập mật khẩu ban đầu là `password` (Tắt nút "Temporary").
6. Đi đến tab **Role mapping**, nhấn **Assign role**, tìm và chọn role `admin`, sau đó nhấn **Assign**.
7. Tạo thêm một người dùng thứ 2: `bob_user` (Không gán role `admin`, thiết lập chung mật khẩu là `password`).

### Bước 3.2: Sao chép luồng Browser mặc định

Không bao giờ chỉnh sửa trực tiếp luồng Default tích hợp sẵn. Hãy luôn nhân bản (Clone) nó.
1. Đi tới menu **Authentication**.
2. Trong tab **Flows**, tìm luồng có tên `browser`.
3. Bấm vào dấu ba chấm ở góc phải của dòng chứa luồng `browser`, chọn **Duplicate**.
4. Đặt tên luồng mới là: `Browser-Conditional-MFA`. Nhấn **Duplicate**.

### Bước 3.3: Tùy chỉnh Requirement và cấu trúc luồng

1. Mở luồng `Browser-Conditional-MFA` vừa tạo.
2. Tìm nhánh `Browser-Conditional-MFA forms` (Nhánh này đang ở mức ALTERNATIVE).
3. Tại nhánh `Browser-Conditional-MFA forms`, nhấn dấu ba chấm ở bên phải -> Chọn **Add sub-flow**.
   - Tên: `Admin Conditional Sub-flow`
   - Flow Type: `Conditional`
   - Nhấn **Add**.
4. Tìm luồng `Admin Conditional Sub-flow` vừa được tạo (nó sẽ nằm cuối danh sách trong nhánh form). Chỉnh cột Requirement của nó thành **REQUIRED**.
5. Nhấn vào dấu `+` bên cạnh `Admin Conditional Sub-flow` để thêm Execution:
   - Chọn **Add condition**.
   - Tìm và chọn `Condition - user role`, bấm **Add**.
6. Ở dòng `Condition - user role` vừa thêm, bấm vào biểu tượng bánh răng (⚙️ Settings):
   - Đặt **Alias**: `check-admin-role`
   - **Expected role**: `admin`
   - Nhấn **Save**.
7. Tiếp tục nhấn dấu `+` ở nhánh `Admin Conditional Sub-flow`, chọn **Add execution**.
   - Tìm và chọn `OTP Form`, bấm **Add**.
   - Đảm bảo Requirement của `OTP Form` được đặt là **REQUIRED**.

*(Cấu trúc cuối cùng tại nhánh Forms sẽ tương tự thế này:)*
- `Username Password Form` (REQUIRED)
- `Admin Conditional Sub-flow` (REQUIRED)
  - `Condition - user role` (REQUIRED) -> cấu hình check role 'admin'
  - `OTP Form` (REQUIRED)

### Bước 3.4: Gắn luồng vào cấp Realm

Keycloak chưa sử dụng luồng này cho đến khi bạn gán nó làm mặc định.
1. Trong màn hình luồng `Browser-Conditional-MFA`, góc trên bên phải, nhấn nút **Bind flow**.
2. Chọn Binding type là **Browser flow**. Nhấn **Save**.
*(Giờ đây mọi giao dịch đăng nhập qua trình duyệt vào Realm này sẽ dùng luồng mới).*

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### Kịch bản kiểm tra 1: User bình thường (Không có Role admin)

1. Mở trình duyệt ở chế độ Ẩn danh (Incognito/Private window).
2. Truy cập vào Account Console của Keycloak: `http://localhost:8080/realms/SecuredRealm/account/` (Thay URL tùy vào Realm và Port của bạn).
3. Nhấn **Sign In**.
4. Nhập `Username`: `bob_user`, `Password`: `password`.
5. **Kỳ vọng:** Đăng nhập thành công ngay lập tức và đưa vào giao diện Account. Conditional Flow nhận thấy Bob không có role `admin`, nên bỏ qua bước OTP.

### Kịch bản kiểm tra 2: User Admin (Có Role admin)

1. Đóng cửa sổ ẩn danh và mở lại một cửa sổ ẩn danh mới.
2. Truy cập vào URL Account Console tương tự.
3. Nhập `Username`: `alice_admin`, `Password`: `password`.
4. **Kỳ vọng:** Sau khi nhập mật khẩu thành công, Keycloak sẽ đưa Alice đến màn hình **Mobile Authenticator Setup** (Cài đặt OTP).
5. Sử dụng Google Authenticator để quét mã QR và nhập mã 6 số để hoàn tất.
6. Đăng xuất, rồi đăng nhập lại bằng `alice_admin`. Lúc này, Keycloak sẽ hiển thị form yêu cầu nhập mã OTP (không hiện QR code nữa).

### Các lỗi thường gặp (Troubleshooting)

- **Lỗi hiển thị "Invalid Username or Password" khi đã nhập đúng thông tin đối với Admin:** 
  *Nguyên nhân:* Có thể trong Condition - user role, bạn đã gõ sai tên Role (ví dụ: gõ `Admin` thay vì `admin`, Keycloak phân biệt chữ hoa chữ thường).
- **Lỗi Admin không bị yêu cầu OTP:** 
  *Nguyên nhân:* Kiểm tra lại Requirement của `Admin Conditional Sub-flow` có phải là `REQUIRED` không. Nếu bạn vô tình đặt là `ALTERNATIVE`, Keycloak sẽ bỏ qua điều kiện nếu thấy mật khẩu (bước trên) đã thành công.
- **Vòng lặp thiết lập (Infinite Loop) OTP:** 
  *Nguyên nhân:* Lỗi thời gian trên máy chủ và điện thoại không đồng bộ. OTP sử dụng TOTP (Time-based), hãy đảm bảo đồng hồ của server Docker Keycloak và điện thoại quét mã là chính xác (cùng đồng bộ qua NTP). 
