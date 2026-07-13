> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Làm quen với việc khởi tạo, cấu hình cơ bản và quản lý các luồng thiết lập cho một Realm độc lập trong Keycloak.

## 1. Kịch bản Thực hành (Lab Scenario)
Công ty của bạn có hai bộ phận riêng biệt: "Nhân sự (HR)" và "Kinh doanh (Sales)". Việc sử dụng chung một tập người dùng (Users) và chính sách (Policies) trên `master` realm sẽ gây rủi ro bảo mật và khó khăn trong việc quản lý phân quyền độc lập.
Trong bài Lab này, bạn sẽ tạo một Realm hoàn toàn mới mang tên `Company-HR`. Trong Realm này, bạn sẽ cấu hình các chính sách đăng nhập riêng, cấu hình chủ đề (Theme) và thiết lập tài khoản User cơ bản để sẵn sàng cho các ứng dụng nội bộ của khối Nhân sự.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Đã cài đặt và khởi chạy Keycloak (ví dụ thông qua Docker hoặc Standalone).
- Một trình duyệt web (Chrome/Firefox).
- Có thông tin đăng nhập vào `master` realm qua Keycloak Admin Console (mặc định là `admin`/`admin`).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo mới một Realm
1. Mở trình duyệt và truy cập vào Keycloak Admin Console: `http://localhost:8080/admin/` (hoặc URL tương ứng của bạn).
2. Đăng nhập bằng tài khoản Administrator.
3. Ở menu thả xuống phía trên cùng bên trái (thường đang hiển thị `master`), click chuột và chọn nút **Create Realm**.
4. Điền vào mục **Realm name**: `Company-HR`.
   *(Lưu ý: Tên Realm có phân biệt chữ hoa chữ thường. Không nên sử dụng khoảng trắng trong tên Realm).*
5. Nhấn nút **Create**. Giao diện sẽ tự động chuyển sang môi trường quản lý của Realm `Company-HR`.

### Bước 3.2: Cấu hình chính sách đăng nhập (Login Settings)
1. Trong Admin Console của Realm `Company-HR`, tìm đến menu **Realm Settings** ở thanh bên trái.
2. Chọn tab **Login**.
3. Cấu hình các tuỳ chọn sau để nâng cao trải nghiệm và bảo mật cho nội bộ công ty:
   - **User registration**: Bật thành **ON**. (Cho phép nhân viên mới tự tạo tài khoản).
   - **Forgot password**: Bật thành **ON**. (Cho phép nhân viên tự khôi phục mật khẩu).
   - **Remember me**: Bật thành **ON**.
4. Nhấn **Save** ở cuối trang.

### Bước 3.3: Tinh chỉnh Theme cơ bản
1. Vẫn ở trong phần **Realm Settings**, chuyển sang tab **Themes**.
2. Tại mục **Login theme**, chọn `keycloak` (hoặc theme tuỳ biến nếu đã deploy).
3. Tại mục **Account theme**, chọn `keycloak`.
4. Nhấn **Save**.

### Bước 3.4: Tạo Người dùng (User) trong Realm mới
1. Đi tới menu **Users** ở thanh bên trái (thuộc khối Manage).
2. Nhấn nút **Add user**.
3. Cấu hình thông tin cơ bản:
   - **Username**: `hr-manager`
   - **Email**: `manager@hr.company.com`
   - **First Name**: `Alice`
   - **Last Name**: `Smith`
   - **Email Verified**: **ON** (Bỏ qua bước xác thực email để thuận tiện thực hành).
4. Nhấn **Create** (hoặc Save tuỳ phiên bản).
5. Chuyển sang tab **Credentials** của user `hr-manager` vừa tạo.
6. Nhấn **Set password**. Nhập mật khẩu (ví dụ: `password123`) và tắt tuỳ chọn **Temporary** thành **OFF**. Nhấn **Save**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu kết quả
1. **Kiểm tra màn hình Account Console**:
   Mở một tab ẩn danh (Incognito window) và truy cập URL của Account Console dành riêng cho Realm `Company-HR`:
   `http://localhost:8080/realms/Company-HR/account/`
2. **Kiểm tra tuỳ chọn Login**:
   Ở trang đăng nhập, bạn sẽ thấy các tính năng vừa bật xuất hiện:
   - Nút `Register` để đăng ký tài khoản.
   - Nút `Forgot password?` để khôi phục.
   - Checkbox `Remember me`.
3. **Đăng nhập thử**:
   Sử dụng tài khoản `hr-manager` (với mật khẩu `password123`) để đăng nhập. Nếu đăng nhập thành công và truy cập vào giao diện quản lý tài khoản cá nhân (Account Console), bài Lab đã hoàn thành xuất sắc.

### 4.2. Khắc phục sự cố (Troubleshooting)
- **Lỗi "Invalid username or password"**: Kiểm tra lại việc cấu hình mật khẩu ở tab Credentials. Hãy chắc chắn bạn đã đặt **Temporary** thành OFF, nếu là ON thì Keycloak sẽ bắt buộc người dùng đổi mật khẩu trong lần đăng nhập đầu tiên.
- **Không tìm thấy URL của Realm**: Ghi nhớ cấu trúc URL cho Account Console: `/realms/{realm-name}/account/`. Nếu gõ sai tên Realm (nhớ phân biệt chữ hoa chữ thường), Keycloak sẽ trả về lỗi 404 Not Found.
- **Lỗi liên quan đến SSL/HTTPS**: Nếu Keycloak của bạn đang cấu hình yêu cầu HTTPS (SSL Required = external), việc truy cập qua `http://localhost` ở môi trường ngoài có thể gặp lỗi. Bạn cần tắt tính năng Require SSL trong cấu hình Realm nếu chỉ dùng HTTP, hoặc cấu hình SSL hợp lệ.
