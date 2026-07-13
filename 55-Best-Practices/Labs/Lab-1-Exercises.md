> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Thực hành thao tác trên giao diện Keycloak Admin Console để áp dụng các quy tắc bảo mật thiết yếu (Security Checklist) và quy tắc đặt tên (Naming Convention).

## 1. Kịch bản Thực hành (Lab Scenario)

Công ty của bạn vừa cài đặt xong một máy chủ Keycloak cho hệ thống thương mại điện tử. Bạn được giao nhiệm vụ "chuẩn bị cho môi trường Production" (Hardening Keycloak). Nhiệm vụ của bạn là:
1. Tạo một Realm mới và áp dụng đúng chuẩn Naming Convention.
2. Bật cơ chế phòng chống đoán mật khẩu (Brute Force Protection).
3. Thiết lập chính sách mật khẩu chặt chẽ (Password Policy).
4. Kiểm tra các cơ chế phòng thủ web (Security Defenses Headers).

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một máy chủ Keycloak cục bộ đang chạy.
- Tài khoản quyền cao nhất (`admin`).
- Công cụ kiểm tra API như Postman hoặc cURL (tùy chọn, để test brute force).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo Realm theo Naming Convention
1. Đăng nhập vào Admin Console.
2. Tại góc trên bên trái, nhấn vào Dropdown Realm -> **Create Realm**.
3. Trong ô **Realm name**, nhập theo chuẩn quy tắc (kebab-case và chứa tên môi trường): `ecommerce-external-prod`.
4. Nhấn **Create**.
5. Trong mục **Realm settings** -> **General**, chuyển trạng thái `Require SSL` thành **external requests** (đây là thiết lập tối thiểu).

### Bước 3.2: Kích hoạt Brute Force Protection
1. Tại Realm `ecommerce-external-prod`, vào menu **Realm settings**.
2. Chuyển sang tab **Security Defenses** -> **Brute Force Detection**.
3. Bật tùy chọn `Enabled`.
4. Cấu hình các tham số:
   - **Failure Factor:** 5 (Cho phép sai tối đa 5 lần).
   - **Wait Increment:** 1 Minute (Khóa tạm thời 1 phút).
   - **Max Wait:** 15 Minutes (Thời gian khóa tối đa).
   - **Max Login Failures:** 15.
5. Nhấn **Save**.

### Bước 3.3: Thiết lập Password Policy
1. Di chuyển tới menu **Authentication** ở thanh bên trái.
2. Chọn tab **Policies** -> **Password Policy**.
3. Nhấn **Add policy** và thêm các chính sách sau (mỗi lần thêm 1 cái rồi điền thông số):
   - **Length:** Điền `8` (Mật khẩu tối thiểu 8 ký tự).
   - **Digits:** Điền `1` (Ít nhất 1 số).
   - **Special Chars:** Điền `1` (Ít nhất 1 ký tự đặc biệt).
   - **Not Recently Used:** Điền `3` (Không được dùng lại 3 mật khẩu cũ nhất).
4. Nhấn **Save**.

### Bước 3.4: Kiểm tra HTTP Security Headers
1. Trở lại menu **Realm settings** -> **Security Defenses**.
2. Chọn tab **Headers**.
3. Quan sát các giá trị mặc định đã được Keycloak bảo vệ:
   - `X-Frame-Options`: Giá trị là `SAMEORIGIN` (Chống Clickjacking, chặn các trang web khác nhúng giao diện đăng nhập qua thẻ iframe).
   - `Content-Security-Policy`: Giá trị `frame-src 'self'; frame-ancestors 'self'; object-src 'none';`
   - `Strict-Transport-Security` (HSTS): Giá trị `max-age=31536000; includeSubDomains` (Bắt buộc trình duyệt luôn dùng HTTPS trong 1 năm).
4. (Tùy chọn) Thêm `X-XSS-Protection: 1; mode=block` nếu chưa có.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Kiểm tra Password Policy:**
1. Tạo một User mới tên là `testuser`.
2. Chuyển qua tab **Credentials** của User đó, nhấn **Set password**.
3. Thử đặt mật khẩu là `12345` (quá ngắn và không có ký tự đặc biệt).
4. Keycloak sẽ báo lỗi đỏ: `Invalid password: must contain at least one digit and one special character`.
5. Đặt mật khẩu đúng chuẩn (VD: `P@ssw0rd123`) -> Lệnh thành công.

**Kiểm tra Brute Force Protection:**
1. Mở một trình duyệt ẩn danh (Incognito Mode).
2. Lấy đường dẫn Login của ứng dụng hoặc tài khoản `testuser`.
3. Cố tình đăng nhập sai mật khẩu 5 lần liên tiếp.
4. Ở lần thứ 6, dù bạn nhập đúng mật khẩu, hệ thống sẽ báo `Account is temporarily disabled, contact admin or try again later.`.
5. Trong tab **Users** của Admin Console, tìm `testuser`. Bạn sẽ thấy trạng thái `Temporarily Locked` hiện lên. Quản trị viên có thể nhấn **Unlock** để mở khóa lập tức.

**Troubleshooting:**
- **Không tìm thấy mục Password Policy:** Đảm bảo bạn đang ở mục `Authentication` (từ bản 19+) chứ không phải `Realm Settings` như các bản Keycloak cũ (Wildfly).
- **Brute force khóa IP thay vì User:** Bạn cần kiểm tra lại cấu hình Reverse Proxy, nhưng ở mức local lab không dùng Proxy, nó sẽ khóa User theo Username.

> [!TIP]
> Việc cấu hình Security bằng tay có thể dẫn đến sai sót (Human Error). Trong thực tế, các cấu hình này thường được xuất (Export) ra dạng tệp JSON hoặc quản lý bằng Terraform / Keycloak Operator để đồng bộ tự động lên máy chủ.
