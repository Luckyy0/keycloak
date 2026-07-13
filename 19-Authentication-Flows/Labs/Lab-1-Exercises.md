> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Tự tay tạo và tùy chỉnh một Authentication Flow, triển khai cấu hình Conditional MFA (OTP) dành riêng cho các User có quyền Admin.

## 1. Kịch bản Thực hành (Lab Scenario)
Công ty của bạn yêu cầu một chính sách bảo mật khắt khe: Tất cả nhân viên đều có thể đăng nhập bằng Username và Password thông thường. Tuy nhiên, các tài khoản quản trị (được gắn Role `admin`) bắt buộc phải xác thực thêm bước 2 bằng Ứng dụng Authenticator (OTP). Bài Lab này giúp bạn xây dựng và kiểm chứng luồng này trên Keycloak.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Đã cài đặt và chạy Keycloak phiên bản mới nhất (Quarkus-based).
- Có quyền truy cập vào `Master` realm với tài khoản `admin`.
- Đã tạo sẵn một Realm mới tên là `Lab-Auth-Realm`.
- Có một ứng dụng tạo mã OTP trên điện thoại (Google Authenticator, Authy, hoặc FreeOTP).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo User và Role
1. Đăng nhập Keycloak Admin Console.
2. Chuyển sang `Lab-Auth-Realm`.
3. Vào **Realm roles** -> Nhấn **Create role** -> Đặt tên là `require-mfa` -> **Save**.
4. Vào **Users** -> Nhấn **Add user**.
   - Username: `normal_user` -> Bấm **Create**. Chuyển sang tab **Credentials**, set password là `123` (tắt Temporary).
   - Tiếp tục tạo user: `admin_user` -> Bấm **Create**. Set password là `123` (tắt Temporary).
5. Gán Role cho admin_user: Mở user `admin_user`, sang tab **Role mapping**, nhấn **Assign role**, chọn `require-mfa` -> **Assign**.

### Bước 3.2: Sao chép và Tùy chỉnh Browser Flow
1. Điều hướng đến **Authentication** (bên menu trái).
2. Tại tab **Flows**, chọn `browser` (đây là flow mặc định).
3. Nhấp vào nút mũi tên xuống góc phải màn hình, chọn **Duplicate**.
4. Đặt tên là `Conditional-MFA-Browser` -> **Duplicate**.
5. Trong Flow mới tạo, tìm đến dòng `Browser Forms` (Sub-Flow).
6. Ở bên phải của dòng `Browser Forms`, nhấp vào biểu tượng dấu cộng **+ (Add execution)**. (Hoặc Add step/Add condition tùy phiên bản UI).
   - Chọn **Add condition** hoặc thêm Sub-Flow mới tên là `MFA-Condition`. (Ở phiên bản Keycloak 20+, bạn bấm *Add step* tại *Browser Forms*, chọn `Condition - user role`).
7. Để đơn giản cấu trúc:
   - Trong `Conditional-MFA-Browser`, tạo một Sub-flow bên dưới bước *Username Password Form* (hoặc trong cùng khối Browser Forms).
   - Đặt tên Sub-flow là `OTP-Conditional-Subflow`, Requirement là **CONDITIONAL**.
   - Bên trong `OTP-Conditional-Subflow`, add execution `Condition - user role` (Requirement: REQUIRED). Bấm biểu tượng bánh răng cấu hình, điền Alias `Role Check`, Role name là `require-mfa`.
   - Add execution thứ hai vào Sub-flow này: `OTP Form` (Requirement: REQUIRED).

### Bước 3.3: Ràng buộc (Bind) Flow mới vào Realm
1. Ở tab **Flows**, chọn luồng `Conditional-MFA-Browser` bạn vừa làm.
2. Click vào thanh công cụ **Action** -> Chọn **Bind flow**.
3. Chọn `Browser flow` -> **Save**.
*(Lúc này toàn bộ realm sẽ dùng flow mới của bạn để xử lý đăng nhập).*

### Bước 3.4: Kiểm tra Flow
1. Mở cửa sổ ẩn danh (Incognito Window) trên trình duyệt.
2. Truy cập vào trang Account Console của User: `http://localhost:8080/realms/Lab-Auth-Realm/account`.
3. Nhấn **Sign In**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu 1: Kiểm tra tài khoản bình thường**
- Tại trang đăng nhập, nhập `normal_user` / `123`.
- Kết quả mong đợi: Bạn truy cập ngay vào Account Console thành công. Luồng Conditional trả về False và bỏ qua bước OTP.

**Nghiệm thu 2: Kiểm tra tài khoản Admin**
- Đăng xuất khỏi `normal_user`.
- Đăng nhập bằng `admin_user` / `123`.
- Kết quả mong đợi: Sau khi nhập mật khẩu, Keycloak hiển thị màn hình yêu cầu cài đặt Authenticator (Quét mã QR). Bạn dùng app trên điện thoại quét và nhập mã 6 số.
- Đăng nhập thành công và bị ép qua vòng OTP.

**Troubleshooting (Khắc phục sự cố):**
- **Sự cố:** `admin_user` không bị hỏi mã OTP.
  - *Kiểm tra:* Xem lại Requirement của `OTP Form` bên trong Sub-flow. Nó phải là `REQUIRED`. Xem lại Condition Evaluator đã đánh đúng tên role `require-mfa` chưa.
- **Sự cố:** Bị lỗi "Invalid Username or Password" hoặc Server Error 500.
  - *Kiểm tra:* Cấu trúc Flow bị sai. Đảm bảo Condition luôn nằm phía SAU bước `Username Password Form`. Nếu Condition nằm trước, nó không thể tìm thấy thông tin user trong Context.
