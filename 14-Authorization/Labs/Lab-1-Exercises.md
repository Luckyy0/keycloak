> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Thực hành kích hoạt và cấu hình Authorization Services trên Keycloak. Hiểu cách tạo Resource, Policy, Permission và mô phỏng kiểm tra quyền truy cập (Evaluate) trực tiếp trên Keycloak Admin Console.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là chuyên viên bảo mật được giao nhiệm vụ cấu hình hệ thống phân quyền cho một ứng dụng **Hệ thống Quản lý Tài liệu (Document Management System)**. 

Yêu cầu phân quyền như sau:
1. Có một tài nguyên tên là `Confidential Document`.
2. Định nghĩa hai hành động (scopes): `read` và `delete`.
3. Người dùng thuộc nhóm (hoặc Role) `manager` được phép thực hiện cả `read` và `delete`.
4. Người dùng thuộc nhóm (hoặc Role) `employee` chỉ được phép thực hiện `read`.
5. Bất kỳ ai không thuộc hai Roles trên sẽ bị từ chối mọi quyền truy cập.

Bạn sẽ thiết lập các quy tắc này sử dụng tính năng **Authorization Services** của Keycloak thay vì viết logic if-else cứng nhắc trong mã nguồn của ứng dụng.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một instance Keycloak (phiên bản 20.0 trở lên) đang chạy trên `http://localhost:8080`.
- Tài khoản quản trị `admin` / `admin`.
- Một Realm mới có tên `Company-Realm`. Bạn cần tạo Realm này trước khi làm Lab.
- Tạo sẵn hai User trong `Company-Realm`:
  - `alice` (gán Role: `manager`)
  - `bob` (gán Role: `employee`)
  - (Nhớ tạo Realm Roles `manager` và `employee` trước, sau đó gán cho user).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Tạo Client có hỗ trợ Authorization
1. Truy cập Keycloak Admin Console, chọn Realm `Company-Realm`.
2. Navigate tới menu **Clients**, click **Create client**.
3. **Client ID**: `document-service`, nhấn **Next**.
4. Bật **Client authentication** (bắt buộc để kích hoạt Authorization). Nhấn **Next**.
5. Bật **Authorization**, sau đó nhấn **Save**.
6. Ghi chú lại giá trị của tab **Credentials** (Client Secret) để dùng khi cần tích hợp thật, nhưng ở Lab này chúng ta chủ yếu làm trên giao diện.

### Bước 2: Tạo Authorization Scopes
1. Tại màn hình cấu hình của Client `document-service`, chọn tab **Authorization**.
2. Chọn sub-tab **Scopes** -> Click **Create authorization scope**.
3. Tạo scope thứ nhất: **Name** = `doc:read`, nhấn **Save**.
4. Trở lại Scopes, tạo scope thứ hai: **Name** = `doc:delete`, nhấn **Save**.

### Bước 3: Tạo Resource cần bảo vệ
1. Vẫn trong tab **Authorization**, chọn sub-tab **Resources** -> Click **Create resource**.
2. Cấu hình các thông số:
   - **Name**: `Confidential Document`
   - **Display name**: `Bản cáo bạch tài chính`
   - **Type**: `document`
   - **URIs**: `/api/documents/confidential`
   - **Authorization scopes**: Chọn cả hai scopes `doc:read` và `doc:delete`.
3. Nhấn **Save**.

### Bước 4: Tạo Policies (Chính sách)
Chúng ta sẽ tạo hai Policy dựa trên Role.

**Tạo Manager Policy:**
1. Chuyển sang sub-tab **Policies** -> Click **Create policy** -> Chọn loại **Role**.
2. **Name**: `Require Manager Role`
3. **Realm Roles**: Chọn Role `manager` (đánh dấu chọn Required).
4. **Logic**: `Positive`.
5. Nhấn **Save**.

**Tạo Employee Policy:**
1. Trở lại Policies -> Click **Create policy** -> Chọn loại **Role**.
2. **Name**: `Require Employee Role`
3. **Realm Roles**: Chọn Role `employee` (đánh dấu chọn Required).
4. **Logic**: `Positive`.
5. Nhấn **Save**.

### Bước 5: Tạo Permissions (Quyền hợp nhất)
Bây giờ, chúng ta gắn kết Resource + Scope + Policy lại với nhau.

**Tạo Permission cho chức năng Read (Scope-Based):**
1. Chọn sub-tab **Permissions** -> Click **Create permission** -> Chọn **Scope-based**.
2. **Name**: `Document Read Permission`
3. **Resource**: Chọn `Confidential Document`.
4. **Scopes**: Chọn `doc:read`.
5. **Apply Policy**: Chọn CẢ HAI policies `Require Manager Role` và `Require Employee Role`.
6. **Decision Strategy**: Đổi thành `Affirmative` (Vì CHỈ CẦN user là Manager HOẶC Employee thì đều được phép đọc).
7. Nhấn **Save**.

**Tạo Permission cho chức năng Delete (Scope-Based):**
1. Trở lại Permissions -> Click **Create permission** -> Chọn **Scope-based**.
2. **Name**: `Document Delete Permission`
3. **Resource**: Chọn `Confidential Document`.
4. **Scopes**: Chọn `doc:delete`.
5. **Apply Policy**: CHỈ CHỌN `Require Manager Role`.
6. **Decision Strategy**: `Unanimous`.
7. Nhấn **Save**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

Keycloak cung cấp công cụ giả lập kiểm tra (Evaluate) rất mạnh mẽ để kiểm tra cấu hình mà không cần code Client.

### Kiểm tra bằng công cụ Evaluate
1. Truy cập tab **Authorization** -> sub-tab **Evaluate**.
2. Trong phần **Identity Information**:
   - **User**: Chọn `alice` (Role: manager).
3. Trong phần **Contextual Information** -> **Permissions**:
   - **Resource**: Cấp `Confidential Document`.
   - **Scopes**: Để trống để yêu cầu đánh giá toàn bộ.
4. Click **Evaluate**.
   - **KẾT QUẢ ĐÚNG**: Trạng thái trả về **PERMIT**. Cả quyền `doc:read` và `doc:delete` đều hiện màu xanh (Granted).

5. Lặp lại quá trình, đổi User thành `bob` (Role: employee). Click **Evaluate**.
   - **KẾT QUẢ ĐÚNG**: Trạng thái chung là **PERMIT**. Nhưng trong chi tiết, quyền `doc:read` màu xanh (Granted), còn quyền `doc:delete` màu đỏ (DENY).

6. Lặp lại quá trình với một user không có Role nào.
   - **KẾT QUẢ ĐÚNG**: Trạng thái **DENY** toàn bộ.

### Các lỗi thường gặp (Troubleshooting)
- **Lỗi Evaluate luôn báo DENY cho mọi User:**
  - Kiểm tra xem bạn đã lưu đầy đủ các Scope vào trong mục Resource chưa.
  - Xem lại **Decision Strategy**. Nếu bạn đặt `Unanimous` ở quyền Read, user phải có CẢ hai Role `manager` và `employee` mới được đọc (rất dễ sai ở đây). Bắt buộc phải là `Affirmative`.
- **Không tìm thấy mục Authorization trong Client:**
  - Hãy chắc chắn rằng bạn đã kích hoạt tùy chọn **Client authentication** và gạt công tắc **Authorization** ở cài đặt chung của Client. Chức năng phân quyền chỉ hoạt động với Client bảo mật (Confidential Clients), không hoạt động với Public Clients.
