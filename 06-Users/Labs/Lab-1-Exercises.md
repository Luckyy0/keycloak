> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Cấu hình hệ thống quản lý Người dùng chuyên sâu trong Keycloak. Thực hành bật tính năng Đăng ký tài khoản (User Registration), thiết lập Chính sách Mật khẩu (Password Policies), yêu cầu Xác thực Email (Email Verification), và tùy biến Form đăng ký bằng Declarative User Profile.

# Lab 1: Quản lý Người Dùng và Các Chính sách Bảo mật

## 1. Kịch bản Thực hành (Lab Scenario)
Một doanh nghiệp muốn mở hệ thống cho phép người dùng tự do đăng ký tài khoản mới. Tuy nhiên, để ngăn chặn spam và đảm bảo chất lượng dữ liệu, bộ phận Bảo mật yêu cầu:
1. Người dùng mới tạo tài khoản bắt buộc phải **Xác minh Email** trước khi được phép đăng nhập.
2. Mật khẩu phải tuân thủ kỷ luật thép: Độ dài tối thiểu 8 ký tự, không được phép trùng với tên đăng nhập.
3. Trong Form đăng ký, phải bắt buộc người dùng chọn **Phòng ban** (Department) mà họ trực thuộc. Chỉ cho phép nhập 3 giá trị hợp lệ: `IT`, `Sales`, `HR`. Nếu nhập sai, hệ thống phải báo lỗi ngay trên giao diện mà Frontend developer không cần phải code thêm dòng Javascript nào (nhờ sức mạnh của Declarative User Profile).

Trong bài Lab này, chúng ta sẽ thiết lập toàn bộ các yêu cầu trên và sử dụng **MailHog** như một máy chủ SMTP giả lập để bắt các email xác thực sinh ra từ Keycloak.

## 2. Chuẩn bị Môi trường (Prerequisites)
Bạn cần có Docker và Docker Compose. Hãy chuẩn bị file `docker-compose.yml` với nội dung sau:

```yaml
version: '3.8'
services:
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    command: start-dev
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - 8080:8080
  mailhog:
    image: mailhog/mailhog:latest
    ports:
      - 1025:1025 # Cổng SMTP để Keycloak gửi mail vào
      - 8025:8025 # Cổng Web UI để xem hộp thư đến
```

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Khởi động hệ thống và tạo Realm
1. Mở Terminal, di chuyển đến thư mục chứa file `docker-compose.yml` và chạy lệnh:
   ```bash
   docker-compose up -d
   ```
2. Đăng nhập vào Admin Console của Keycloak tại `http://localhost:8080` (admin/admin).
3. Ở góc trên cùng bên trái, nhấn vào menu dropdown hiển thị chữ `master`, bấm **Create Realm**.
4. Đặt tên Realm là `Vingroup_Users` và nhấn **Create**. Đảm bảo bạn đang thao tác trên Realm mới này ở các bước tiếp theo.

### Bước 3.2: Cấu hình Máy chủ Gửi thư (SMTP Server)
Để tính năng Verify Email hoạt động, Keycloak cần biết cách gửi Email. Chúng ta sẽ trỏ nó sang MailHog.
1. Ở menu bên trái, chọn **Realm settings**.
2. Chuyển sang tab **Email**.
3. Khai báo các thông số SMTP:
   - **Host**: `mailhog`
   - **Port**: `1025`
   - **From**: `admin@vingroup.com`
4. Nhấn **Save**. Bạn có thể nhấn nút **Test connection** để Keycloak thử gửi một email kiểm tra. Báo hiệu màu xanh lá là thành công.

### Bước 3.3: Bật tính năng Đăng ký và Xác minh Email
1. Vẫn trong phần **Realm settings**, chuyển sang tab **Login**.
2. Gạt công tắc bật `User registration` thành **ON**.
3. Gạt công tắc bật `Verify email` thành **ON**.
4. Gạt công tắc bật `Login with email` thành **ON**.
5. Gạt công tắc tắt `Duplicate emails` thành **OFF** (để đảm bảo không có hai người đăng ký cùng 1 email).
6. Nhấn **Save**.

### Bước 3.4: Thiết lập Declarative User Profile
Tính năng Declarative User Profile cho phép định nghĩa các trường dữ liệu bắt buộc ngay tại tầng lõi của Keycloak.
1. Trong menu **Realm settings**, chuyển sang tab **User profile**.
2. Nhấn nút **Add attribute** để tạo thêm trường dữ liệu `department` cho người dùng:
   - **Name**: Nhập `department`.
   - **Display name**: Nhập `Phòng ban`.
   - **Required**: Bật **ON** (để bắt buộc có trong form đăng ký).
3. Tại phần **Permissions**:
   - Ở mục `Who can view?`, chọn **User, Admin**.
   - Ở mục `Who can edit?`, chọn **User, Admin**.
4. Tại phần **Validations** (Xác thực dữ liệu đầu vào):
   - Nhấn nút **Add validator** có biểu tượng dấu cộng.
   - Chọn loại validator là `options`.
   - Trong ô Options, nhập danh sách các giá trị hợp lệ cách nhau bằng dấu phẩy: `IT, Sales, HR`.
5. Nhấn **Save** để lưu Attribute.

### Bước 3.5: Cấu hình Chính sách Mật khẩu (Password Policies)
1. Ở menu bên trái, tìm đến phần **Authentication**.
2. Chuyển sang tab **Policies** -> **Password Policy**.
3. Nhấn **Add policy** và chọn `Length`. Đặt giá trị bằng `8`.
4. Nhấn tiếp **Add policy** và chọn `Not Username` (không được phép giống tên đăng nhập).
5. Nhấn **Save**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Thực hành Nghiệm thu (Test Flow)
1. Mở một trình duyệt Ẩn danh (Incognito), truy cập vào trang quản lý tài khoản của người dùng: 
   `http://localhost:8080/realms/Vingroup_Users/account/`
2. Nhấn vào nút **Sign in**.
3. Bạn sẽ thấy màn hình đăng nhập. Lúc này, dưới nút Sign In đã có thêm link **Register** (Đăng ký). Hãy nhấn vào đó.
4. Giao diện Đăng ký hiện ra, bạn sẽ thấy ô `Phòng ban` (`department`) tự động xuất hiện do chúng ta đã định nghĩa ở User Profile.
5. **Test Validation 1 (User Profile):** Hãy thử điền vào ô Phòng ban chữ `Marketing`. Điền các thông tin khác hợp lệ và bấm Đăng ký. Keycloak sẽ chặn lại và báo lỗi ngay lập tức vì không nằm trong danh sách `IT, Sales, HR`.
6. **Test Validation 2 (Password Policy):** Hãy sửa Phòng ban lại thành `IT`. Nhập Username là `boss`, nhập Password cũng là `boss`. Bấm Đăng ký. Keycloak sẽ báo lỗi đỏ: "Mật khẩu không được giống tên đăng nhập" và "Mật khẩu phải chứa ít nhất 8 ký tự".
7. **Đăng ký thành công:** Nhập Password chuẩn (ví dụ: `P@ssw0rd123`) và bấm Đăng ký.
8. Màn hình Keycloak sẽ bị chặn lại bởi thông báo: **"You need to verify your email address to activate your account."** (Bạn cần xác thực email để kích hoạt). Trải nghiệm đăng nhập bị khóa cứng ở đây.
9. **Kiểm tra MailHog:** Mở thẻ trình duyệt mới và truy cập vào Web UI của MailHog: `http://localhost:8025`.
10. Bạn sẽ thấy có một email nội bộ gửi tới hòm thư. Bấm vào thư đó, bạn sẽ thấy link xác thực do Keycloak sinh ra.
11. Bấm vào đường link trong thư. Bùm! Tài khoản được kích hoạt, màn hình sẽ thông báo thành công và chuyển hướng bạn thẳng vào Account Console. Bài Lab kết thúc thắng lợi.

### 4.2. Khắc phục sự cố (Troubleshooting)
- **Lỗi không gửi được thư Verification:** Nếu bấm Đăng ký xong màn hình quay liên tục hoặc báo lỗi Server Error, kiểm tra lại thông số cổng `1025` và tên host `mailhog` trong bước cấu hình SMTP. Nếu chạy Docker cục bộ, đảm bảo container MailHog không bị crash.
- **Lỗi không hiện ô Department khi Register:** Kiểm tra lại bước 3.4 phần Permissions, bạn phải đảm bảo quyền Edit cho "User" được tích chọn, nếu không trường dữ liệu này chỉ được chỉnh sửa bởi Admin và sẽ bị ẩn khỏi Form đăng ký public của người dùng.

## 5. Dọn dẹp (Cleanup)
Sau khi học xong, hủy bỏ môi trường để giải phóng RAM:
```bash
docker-compose down -v
```
