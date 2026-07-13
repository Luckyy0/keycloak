# Lab 1: Tùy Chỉnh Luồng Xác Thực Keycloak

## 1. Mục Tiêu (Objectives)
Thực hành các kỹ năng đã học trong Chapter 13 bằng cách xây dựng và cấu hình các luồng xác thực (Authentication Flows) nâng cao.
- **Task 1:** Kích hoạt và kiểm tra chức năng tự đăng ký tài khoản (Registration Flow) đi kèm xác minh email.
- **Task 2:** Bảo vệ tính năng quên mật khẩu (Reset Credentials) bằng MFA (OTP).
- **Task 3:** Tạo Custom Browser Flow: User thường chỉ cần mật khẩu, Admin bắt buộc nhập OTP.

---

## 2. Chuẩn Bị (Prerequisites)
Khởi động hệ thống Keycloak bằng docker-compose đã cung cấp.

```bash
cd code
docker-compose up -d
```
Đợi khoảng 10-15 giây để Keycloak khởi động hoàn tất. Đăng nhập vào Admin Console tại `http://localhost:8080/` với tài khoản mặc định (ví dụ `admin` / `admin`).

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Kích Hoạt Đăng Ký Tài Khoản & Bắt Buộc Verify Email
Theo mặc định, Keycloak không cho phép người dùng tự đăng ký. Trong task này, ta sẽ bật Registration và cấu hình kiểm tra Email để chống bot.

1. Đăng nhập Admin Console, chọn Realm đang thao tác (VD: `master` hoặc Realm bạn tạo).
2. Tới Menu **Realm Settings** ở thanh menu trái.
3. Chọn Tab **Login**.
4. Bật công tắc của hai tuỳ chọn lên trạng thái `ON`:
   - **User registration** (Hiện nút đăng ký)
   - **Verify email** (Đòi xác minh qua link email)
5. Mở một trình duyệt ẩn danh (Incognito) và truy cập `http://localhost:8080/realms/master/account/`.
6. Bạn sẽ thấy giao diện Login đã xuất hiện thêm nút **Register**. Bấm vào để thử tạo tài khoản mới. Do tính năng Gửi Mail đang chạy ở local, bạn có thể kiểm tra console log docker để xem link Verify, hoặc sử dụng tool hỗ trợ giả lập Mailtrap.

### Task 2: Bảo Vệ "Quên Mật Khẩu" (Reset Credentials) Với MFA (OTP)
Chức năng khôi phục mật khẩu rất nguy hiểm nếu không bật OTP. Chúng ta sẽ ràng buộc tính năng Reset OTP từ chế độ Tuỳ chọn (Conditional) sang Bắt buộc (Required).

1. Tại Admin Console, truy cập **Authentication** -> Tab **Flows**.
2. Tìm kiếm luồng tên là **`reset credentials`** và bấm nút **Duplicate** (Nhân bản).
3. Đặt tên luồng mới là: `Secure-Reset-Flow` rồi ấn Create/Save.
4. Nhấn vào tên của luồng `Secure-Reset-Flow` để vào giao diện chỉnh sửa cấu trúc bên trong.
5. Tìm cục Execution có tên **`Reset OTP`**. Trạng thái mặc định của nó là `Conditional`.
6. Nhấn vào ô Dropdown của nó và đổi thành **`Required`**.
7. Chuyển tới **Realm Settings** -> Tab **Themes**.
8. Ở mục **Reset Credentials Flow**, đổi từ luồng mặc định sang luồng `Secure-Reset-Flow` bạn vừa tạo. Lưu lại cấu hình.
*(Lưu ý: Sau bước này, bất kỳ ai lấy lại mật khẩu đều phải cấu hình OTP trước khi được nhả quyền truy cập).*

### Task 3: Custom Browser Flow - Rẽ Nhánh Admin Chặn Kép Cấm Cửa (Conditional Flow)
Thiết lập kịch bản thông minh (Context-Aware): Bất kỳ người dùng nào có Role `admin` thì phải Quét OTP mới được login. Người dùng thường chỉ cần mật khẩu.

1. Vào Menu **Authentication** -> Tab **Flows**.
2. Tìm dòng **`browser`** và bấm **Duplicate**. Đặt tên: `Admin-Secure-Browser`.
3. Bấm vào `Admin-Secure-Browser` để chỉnh sửa. 
4. Cuộn xuống cuối cùng của cây cấu trúc, bấm nút **Add Sub-flow** (Thêm luồng phụ). Đặt tên là: `OTP-Role-Check`.
5. Đổi Requirement (Trạng thái) của luồng phụ `OTP-Role-Check` vừa tạo sang dạng **`Conditional`** (Điều kiện rẽ nhánh).
6. Bấm vào nút `Add execution` **bên trong** (dấu cộng nhỏ) của `OTP-Role-Check`. Thêm khối: **`Condition - User Role`**. Đổi thành `Required`.
7. Bấm vào biểu tượng bánh răng ⚙️ của `Condition - User Role`. Cấu hình Alias của role muốn check, điền `admin`. Lưu lại.
8. Lại bấm vào nút `Add execution` **bên trong** `OTP-Role-Check` một lần nữa. Lần này thêm khối: **`OTP Form`**. Đổi thành `Required`.
   - *Cấu trúc lúc này phải là: Nhánh Conditional chứa 2 khối Required là (Condition-User-Role) và (OTP-Form).*
9. Cuối cùng, gán luồng này làm luồng đăng nhập mặc định: 
   - Trở lại danh sách các Flows, tìm dòng `Admin-Secure-Browser`. 
   - Nhấn nút ba chấm (Action) ở cuối dòng -> Bấm **`Bind Flow`**. 
   - Chọn mục **`Browser Flow`** để tráo đổi quyền điều khiển.

Thử nghiệm:
- Login bằng tài khoản admin: Màn hình bắt buộc bạn cài OTP.
- Login bằng tài khoản test user (không có role admin): Giao diện đi thẳng vào app, bỏ qua bước OTP.

---

## 4. Dọn Dẹp (Cleanup)
Sau khi hoàn thành thử nghiệm, nếu lỡ cấu hình sai làm hỏng quyền truy cập Admin của bạn:
- Hãy sử dụng Script truy cập thẳng vào PostgresSQL xoá cache hoặc xoá luôn Container Docker chạy mới bằng lệnh: `docker-compose down -v`. Lệnh này giúp dọn sạch Database trở lại cấu hình gốc.
