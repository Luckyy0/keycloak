> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Áp dụng lý thuyết Protocol Mappers để thực hành cấu hình các loại Mapper phổ biến: Hardcoded Claim, Role Mapper, và User Attribute Mapper nhằm tùy biến dữ liệu trong Access Token và ID Token.

## 1. Kịch bản Thực hành (Lab Scenario)

Công ty của bạn đang phát triển một hệ thống Microservices sử dụng Keycloak làm máy chủ Identity Provider (IdP). Nhóm Backend yêu cầu bạn cấu hình Keycloak sao cho mỗi khi User đăng nhập và nhận được JSON Web Token (JWT), Token đó phải chứa các thông tin sau để Backend có thể xử lý nghiệp vụ:
1. Một trường tĩnh `company_name` luôn mang giá trị `"TechCorp"`.
2. Danh sách các quyền của User trong hệ thống, nằm trong biến `user_authorities`.
3. Phòng ban của User, được lấy từ thuộc tính `department` của User profile.

Bạn cần sử dụng cơ chế Protocol Mappers trong Keycloak để giải quyết yêu cầu này mà không cần viết code phía backend để lấy thêm thông tin.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một máy chủ Keycloak (phiên bản 21+ trở lên) đang chạy cục bộ tại `http://localhost:8080`.
- Tài khoản quản trị `admin` / `admin`.
- Một Realm mới mang tên `Lab-Mappers-Realm`.
- Một Client mang tên `frontend-app` với cấu hình:
  - **Client Authentication**: Off (Public Client)
  - **Standard Flow**: Enabled
  - **Valid Redirect URIs**: `https://oidcdebugger.com/debug` (hoặc `http://localhost:8080/*`).
- Một người dùng (User) mang tên `alice`:
  - Thiết lập mật khẩu cho `alice` (ví dụ: `12345`).
  - Thêm thuộc tính (`Attributes`): Key = `department`, Value = `Engineering`.
  - Có Realm Role là `manager`.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo Hardcoded Claim Mapper cho Company Name
Chúng ta sẽ thêm một Client Scope chuyên biệt cho các Mapper bài Lab này để tiện quản lý.
1. Truy cập Keycloak Admin Console.
2. Chọn menu **Client Scopes** -> Nhấn **Create client scope**.
3. Điền **Name:** `lab-custom-claims`. **Type:** `Default` (hoặc có thể để Optional rồi assign sau). Nhấn **Save**.
4. Vào Client Scope vừa tạo, chuyển sang tab **Mappers**.
5. Nhấn **Configure a new mapper** -> Tìm và chọn **Hardcoded claim**.
6. Điền thông tin cấu hình:
   - **Name:** `company-name-mapper`
   - **Token Claim Name:** `company_name`
   - **Claim value:** `TechCorp`
   - **Claim JSON Type:** `String`
   - **Add to ID token:** `ON`
   - **Add to access token:** `ON`
7. Nhấn **Save**.

### Bước 3.2: Tạo Role Mapper cho Quyền người dùng
1. Vẫn trong tab **Mappers** của Client Scope `lab-custom-claims`.
2. Nhấn **Add mapper** -> **By configuration** -> Chọn **User Realm Role**.
3. Điền thông tin:
   - **Name:** `user-roles-mapper`
   - **Realm Role prefix:** (Bỏ trống)
   - **Token Claim Name:** `user_authorities`
   - **Claim JSON Type:** `String`
   - **Multivalued:** `ON` (Để xuất ra mảng).
4. Nhấn **Save**.

### Bước 3.3: Tạo User Attribute Mapper cho Department
1. Thêm một mapper nữa -> Chọn **User Attribute**.
2. Điền thông tin:
   - **Name:** `department-attribute-mapper`
   - **User Attribute:** `department` (Khớp đúng tên khóa trong tab Attributes của User).
   - **Token Claim Name:** `user_department`
   - **Claim JSON Type:** `String`
3. Nhấn **Save**.

### Bước 3.4: Gán Client Scope vào Client
1. Di chuyển tới menu **Clients** -> Chọn `frontend-app`.
2. Chuyển sang tab **Client Scopes**.
3. Nhấn **Add client scope**, chọn `lab-custom-claims` và ấn **Add** -> chọn **Default** (nếu bạn chưa cấu hình nó là default ở bước 3.1).

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Kiểm tra qua tính năng Evaluate của Keycloak:**
1. Trong màn hình Admin Console của Client `frontend-app`, chuyển sang tab **Client Scopes**.
2. Tìm nút **Evaluate** (Đánh giá).
3. Tại phần **User**, chọn `alice`.
4. Nhấn **Evaluate**.
5. Cuộn xuống và chọn tab **Generated Access Token**.
6. Quan sát mã JSON được sinh ra. Bạn sẽ thấy các cấu trúc tương tự:
```json
{
  "company_name": "TechCorp",
  "user_department": "Engineering",
  "user_authorities": [
    "manager",
    "default-roles-lab-mappers-realm"
  ]
}
```

**Troubleshooting (Xử lý sự cố):**
- **Lỗi không thấy `user_department`:** Hãy kiểm tra kỹ lại tab **Attributes** của user `alice`, đảm bảo bạn đã gõ đúng chữ `department` (phân biệt hoa/thường) và đã nhấn nút "Save" sau khi thêm attribute.
- **Lỗi Token trả về `user_authorities` là chuỗi thay vì mảng:** Bạn đã quên bật tùy chọn `Multivalued` trong Role Mapper. Sửa lại thành ON và sinh lại Token.
- **Mappers không hoạt động trên Client:** Đảm bảo Client Scope chứa các Mapper đã được gán vào tab `Client Scopes` của `frontend-app`. Nếu bạn gán nó dưới dạng `Optional`, Client phải gửi tham số `scope=lab-custom-claims` trong lúc gửi Authorization Request thì Keycloak mới kích hoạt các Mappers này.

> [!TIP]
> Việc sử dụng tính năng **Evaluate** ngay trong giao diện admin giúp các nhà phát triển Backend và IAM Admin kiểm thử các thay đổi Mapper lập tức mà không cần phải gọi API bằng Postman hay cấu hình ứng dụng web phức tạp.
