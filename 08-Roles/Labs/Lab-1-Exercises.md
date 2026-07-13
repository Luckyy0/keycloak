> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Thiết lập mô hình phân quyền dựa trên vai trò (RBAC) cơ bản trong Keycloak bằng cách tạo, cấu hình và gán Realm Roles, Client Roles, và Composite Roles cho User.

## 1. Kịch bản Thực hành (Lab Scenario)

Giả sử bạn đang quản trị một hệ thống phân quyền cho ứng dụng "Hệ thống Quản lý Bệnh viện" (`hospital-app`). Bạn cần thiết lập quyền cho hai loại nhân viên: Bác sĩ (Doctor) và Quản trị viên IT (IT Admin).

Yêu cầu thực tiễn:
1. **Realm Role (`doctor-role`)**: Là quyền cấp cao toàn cục dành cho Bác sĩ.
2. **Client Role (`app-manage`)**: Nằm trong Client `hospital-app`, cho phép người dùng quản lý dữ liệu ứng dụng.
3. **Composite Role (`super-admin`)**: Một vai trò tổng hợp tự động kế thừa (inherit) nhiều vai trò khác để dễ dàng cấp phát cho Quản trị viên IT mà không cần phải gán từng quyền lẻ tẻ.

Nhiệm vụ của bạn là sử dụng Keycloak Admin Console để cấu hình cấu trúc Role này và gán cho các tài khoản người dùng tương ứng.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một máy chủ Keycloak đang chạy (đã tạo tài khoản quản trị).
- Có ít nhất một Realm tùy chỉnh (ví dụ: `myrealm`) được tạo thay vì dùng mặc định `master`.
- Đã tạo một Client tên là `hospital-app` trong Realm `myrealm`.
- Đã tạo hai Users: `dr_john` (Bác sĩ) và `it_admin_bob` (Quản trị viên IT).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo Realm Role

1. Truy cập Keycloak Admin Console. Ở góc trên bên trái, chọn Realm `myrealm`.
2. Trên menu bên trái, nhấp vào **Realm roles** (dưới nhóm Manage).
3. Nhấp vào nút **Create role** (Tạo vai trò).
4. Điền các thông tin:
   - **Role name**: `doctor-role`
   - **Description**: Quyền hạn chung dành cho các Bác sĩ trong toàn bộ hệ thống bệnh viện.
5. Nhấp **Save** (Lưu).

### Bước 3.2: Tạo Client Role

Client Roles bị giới hạn trong phạm vi của một Client cụ thể.

1. Trên menu bên trái, chọn **Clients**.
2. Tìm và nhấp vào Client tên `hospital-app`.
3. Chuyển sang tab **Roles** của Client này.
4. Nhấp vào nút **Create role**.
5. Điền các thông tin:
   - **Role name**: `app-manage`
   - **Description**: Quyền thao tác và chỉnh sửa cấu hình ứng dụng hospital-app.
6. Nhấp **Save**.

### Bước 3.3: Tạo Composite Role (Kế thừa Role)

Composite Role là tính năng gộp nhiều roles lại với nhau.

1. Quay lại menu **Realm roles**.
2. Nhấp **Create role**.
3. Tại **Role name**, điền `super-admin` và nhấp **Save**.
4. Lúc này bạn đang ở trang chi tiết của Role `super-admin`. Tại tab **Details**, tìm đến mục **Composite roles** (Các vai trò tổng hợp).
5. Bật công tắc (Toggle) sang trạng thái **ON** ở `Composite roles`. Cửa sổ tìm kiếm role hiện ra bên dưới.
6. Trong thanh tìm kiếm, tìm role `doctor-role` và nhấp **Assign** (hoặc dấu cộng).
7. Đổi bộ lọc từ "Filter by realm roles" sang "Filter by clients" hoặc gõ tên `app-manage`. Chọn `hospital-app app-manage` và nhấp **Assign**.
8. Lúc này, `super-admin` đã chứa cả hai quyền con.

### Bước 3.4: Gán Roles cho Người dùng (Role Mapping)

**Gán quyền cho Bác sĩ John:**
1. Trở về menu **Users**, tìm và nhấp vào user `dr_john`.
2. Chuyển sang tab **Role mapping**.
3. Nhấp vào **Assign role**. Tích chọn hộp kiểm trước `doctor-role`.
4. Nhấp **Assign**.

**Gán quyền tổng hợp cho IT Admin Bob:**
1. Quay lại **Users**, nhấp vào user `it_admin_bob`.
2. Chuyển sang tab **Role mapping**.
3. Nhấp vào **Assign role**. Tích chọn hộp kiểm trước `super-admin`.
4. Nhấp **Assign**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu (Verification)

Để kiểm tra xem Keycloak đã áp dụng cơ chế phân quyền (Role-Based Access Control) chính xác hay chưa:

1. **Kiểm tra Token của Dr. John:**
   - Sử dụng Postman hoặc Terminal gửi một yêu cầu `POST` tới endpoint `/realms/myrealm/protocol/openid-connect/token` sử dụng `grant_type=password` với thông tin đăng nhập của `dr_john`.
   - Copy mã Access Token trả về, dán vào trang web [jwt.io](https://jwt.io).
   - Trong phần payload (màu tím), tìm claim `realm_access.roles`. Bạn phải thấy chuỗi `"doctor-role"`. Nó sẽ không có role nào của client `hospital-app`.

2. **Kiểm tra Kế thừa của Bob:**
   - Lặp lại bước lấy token nhưng với thông tin đăng nhập của `it_admin_bob`.
   - Phân tích Access Token trên jwt.io. Mặc dù bạn chỉ gán trực tiếp quyền `super-admin`, Token của Bob PHẢI hiển thị:
     - Trong claim `realm_access.roles`: có chứa `"super-admin"` và `"doctor-role"`.
     - Trong claim `resource_access.hospital-app.roles`: có chứa `"app-manage"`.
   - Điều này chứng minh quá trình mở rộng quyền (Role Expansion) của Composite Role đã hoạt động thành công.

### 4.2. Khắc phục sự cố (Troubleshooting)

- **Không tìm thấy Client Roles khi tạo Composite Role:**
  - *Nguyên nhân:* Mặc định giao diện tìm kiếm chỉ lọc các Realm Roles.
  - *Khắc phục:* Chú ý thả danh sách xổ xuống (Dropdown filter) ngay cạnh ô tìm kiếm và chọn "Filter by clients" để thấy các role thuộc về client.
- **Access Token không chứa `client_roles` hoặc `realm_roles`:**
  - *Nguyên nhân:* Client Scopes mặc định (`roles`) đã bị xóa khỏi Client hoặc bị vô hiệu hóa cấu hình ánh xạ (Mapper).
  - *Khắc phục:* Vào Client `hospital-app` -> tab **Client scopes** -> Đảm bảo scope `roles` đang ở trạng thái Assigned type là `Default`.
- **Lỗi nhầm lẫn khái niệm Group và Role:**
  - *Sự cố:* Bạn tạo một Group có tên `doctor-role` thay vì Role.
  - *Khắc phục:* Group (Nhóm) dùng để gộp User. Role (Vai trò) dùng để gán Quyền. Hãy xóa group và tạo lại đúng ở mục Realm roles.
