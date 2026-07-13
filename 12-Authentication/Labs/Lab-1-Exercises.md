> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Xây dựng và áp dụng cấu hình Luồng Xác Thực (Authentication Flow) nâng cao trong Keycloak, bao gồm việc thiết lập Xác thực có điều kiện (Conditional MFA) dựa trên Role của người dùng.

### 1. Kịch bản Thực hành (Lab Scenario)
Công ty của bạn yêu cầu triển khai một chính sách bảo mật đa lớp. Người dùng bình thường chỉ cần đăng nhập bằng Username và Password. Tuy nhiên, các nhân viên có quyền quản trị (những người mang Role `admin`) bắt buộc phải trải qua bước xác thực hai yếu tố (OTP / Google Authenticator). 
Thay vì bắt tất cả mọi người dùng OTP, bạn sẽ cấu hình tính năng **Conditional Authentication** trong Keycloak để kiểm tra linh hoạt tại thời điểm đăng nhập.

### 2. Chuẩn bị Môi trường (Prerequisites)
- Đã cài đặt và khởi chạy Keycloak (phiên bản 21+ trở lên) qua Docker hoặc Standalone.
- Có quyền truy cập bằng tài khoản Admin Keycloak.
- Một Realm mới (hoặc sử dụng realm `master`, nhưng khuyến cáo nên tạo realm riêng là `lab-realm`).
- Điện thoại có cài ứng dụng Google Authenticator hoặc Authy để quét mã OTP.

### 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 3.1: Chuẩn bị dữ liệu người dùng và Role**
1. Đăng nhập vào **Keycloak Admin Console**.
2. Chuyển sang Realm `lab-realm`.
3. Vào mục **Realm Roles**, tạo một Role mới tên là `admin`.
4. Vào mục **Users**, tạo 2 người dùng:
   - User 1: Username: `john_doe`, Password: `123` (tắt cờ Temporary). Không gán Role gì thêm.
   - User 2: Username: `boss_admin`, Password: `123` (tắt cờ Temporary).
5. Gán Role `admin` cho `boss_admin`: Nhấn vào User `boss_admin`, chọn tab **Role Mapping**, chọn `admin` và nhấn Assign.

**Bước 3.2: Sao chép Browser Flow Mặc định**
Không thể chỉnh sửa các luồng hệ thống tích hợp sẵn. Bạn phải nhân bản nó:
1. Chuyển đến mục **Authentication** ở menu bên trái.
2. Tại tab **Flows**, chọn luồng `browser` từ danh sách thả xuống.
3. Ở góc trên bên phải của bảng Flow, nhấp vào nút tác vụ (dấu 3 chấm) và chọn **Duplicate**.
4. Đặt tên luồng mới là `Conditional-Browser-Flow`. Nhấn Save.

**Bước 3.3: Thêm Sub-flow có Điều kiện (Conditional MFA)**
1. Mở `Conditional-Browser-Flow` vừa tạo.
2. Xóa các bước liên quan đến WebAuthn hoặc OTP mặc định đang có sẵn trong nhánh Forms (nếu có) để bắt đầu một cấu hình sạch.
3. Tại cấp độ nhánh `Conditional-Browser-Flow forms` (mức thụt lề dưới cùng của Browser Form), nhấn biểu tượng **dấu + (Add Sub-Flow)**.
4. Đặt tên Sub-flow mới là `MFA-Role-Based`.
5. Đổi `Requirement` của `MFA-Role-Based` từ `Alternative` sang `Conditional`.
6. Tại dòng của `MFA-Role-Based`, nhấn biểu tượng **dấu + (Add execution)**. Tìm và chọn **Condition - user role**. Nhấn Add.
7. Đặt `Requirement` của Condition này là `Required`. Nhấn vào biểu tượng **bánh răng (Cài đặt)** của Condition này, gán thuộc tính Role thành `admin`. Nhấn Save.
8. Vẫn tại dòng của `MFA-Role-Based`, tiếp tục nhấn **dấu + (Add execution)**. Tìm và chọn **OTP Form**. Nhấn Add.
9. Đặt `Requirement` của OTP Form là `Required`.

*Cấu trúc cây Flow của bạn lúc này sẽ giống như sau:*
- `Cookie` (Alternative)
- `Identity Provider Redirector` (Alternative)
- `Conditional-Browser-Flow forms` (Alternative)
  - `Username Password Form` (Required)
  - `MFA-Role-Based` (Conditional)
    - `Condition - user role` (Required, alias: admin)
    - `OTP Form` (Required)

**Bước 3.4: Kích hoạt Luồng cho Realm**
1. Vẫn ở trang **Authentication**, trong bảng luồng `Conditional-Browser-Flow`, nhấn vào biểu tượng Tác vụ (3 chấm) góc trên bên phải.
2. Chọn **Bind flow**.
3. Chọn loại Binding là **Browser flow**. Điều này sẽ thay thế luồng đăng nhập mặc định của Realm thành luồng do bạn vừa tạo.

### 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Kiểm tra kịch bản 1: Đăng nhập với người dùng bình thường**
1. Mở một trình duyệt ẩn danh (Incognito Mode).
2. Lấy link test (Account Console của Keycloak): Truy cập `http://localhost:8080/realms/lab-realm/account`.
3. Nhập username `john_doe` và password `123`.
4. **Kết quả kỳ vọng:** Keycloak đăng nhập thành công ngay lập tức và đưa bạn vào giao diện quản lý tài khoản mà không hỏi gì thêm.

**Kiểm tra kịch bản 2: Đăng nhập với người dùng Admin (Yêu cầu MFA)**
1. Đóng trình duyệt ẩn danh, mở lại một tab ẩn danh mới.
2. Truy cập lại đường dẫn `http://localhost:8080/realms/lab-realm/account`.
3. Nhập username `boss_admin` và password `123`.
4. **Kết quả kỳ vọng:** Do Keycloak phát hiện user này có Role `admin` thông qua bộ Evaluator, nó sẽ đưa bạn đến một màn hình yêu cầu cấu hình OTP (hiển thị mã QR Code). Quét bằng điện thoại, nhập mã OTP để tiếp tục. Những lần đăng nhập sau, màn hình chỉ hiện ô nhập số OTP.

**Xử lý sự cố (Troubleshooting):**
- **Lỗi: Không hiện ô nhập mật khẩu mà bị lỗi hệ thống:** Có thể bạn đã vô tình đổi `Username Password Form` từ `Required` sang `Conditional`. Sửa lại thành `Required`.
- **Lỗi: User admin đăng nhập xong báo lỗi 'Invalid Auth Flow':** Nếu thiết lập Condition Role bị sai tên role, Evaluator có thể ném Exception. Hãy kiểm tra lại đúng chữ `admin` (phân biệt hoa thường) trong phần Cài đặt của `Condition - user role`.
- **Lỗi: OTP Form bị bỏ qua dù là Admin:** Kiểm tra xem bạn đã để OTP Form nằm **bên trong** (thụt lề) của Sub-flow `MFA-Role-Based` chưa. Nếu nó nằm ngang hàng với Sub-flow, luồng sẽ bị vỡ.
