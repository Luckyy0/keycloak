> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Nắm vững thao tác quản lý phân cấp người dùng qua mô hình Group, map Role tự động theo cấu trúc cây (Tree) của nhóm, và kiểm tra tính thừa kế quyền trong Keycloak.

## 1. Kịch bản Thực hành (Lab Scenario)
Công ty của bạn có mô trúc tổ chức gồm: Phòng IT (`IT-Dept`) và nhóm quản trị viên hệ thống (`SysAdmins`) trực thuộc phòng IT. Tất cả thành viên phòng IT đều có role `vpn-access`. Riêng các thành viên `SysAdmins` sẽ có thêm role `server-admin`. Bạn cần tạo sơ đồ Group phân cấp này, tự động hóa gán Role thông qua nhóm, và gắn người dùng để kiểm thử.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Keycloak đã khởi động và có thể truy cập trang quản trị.
- Truy cập vào Realm đang hoạt động (ví dụ: `Company-Realm` hoặc `Master`).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo các Roles
1. Ở thanh menu bên trái, nhấp vào **Realm roles**.
2. Nhấn **Create role**. Điền tên `vpn-access` -> **Save**.
3. Tiếp tục nhấn **Create role**. Điền tên `server-admin` -> **Save**.

### Bước 3.2: Xây dựng Cấu trúc Group Phân cấp
1. Điều hướng tới menu **Groups**.
2. Nhấp vào **Create group** để tạo nhóm gốc.
   - Name: `IT-Dept`
   - Bấm **Create**.
3. Mở nhóm `IT-Dept` vừa tạo. Ở góc trên, tìm mục thao tác (hoặc nhấp trực tiếp vào tên nhóm trong giao diện dạng cây/tree).
4. Nhấn **Create child group** (Tạo nhóm con).
   - Name: `SysAdmins`
   - Bấm **Create**.
*Kết quả:* Bạn có cấu trúc cây `IT-Dept` -> `SysAdmins`.

### Bước 3.3: Gán Role mapping cho Group
1. Nhấp mở nhóm **IT-Dept**.
2. Chuyển sang tab **Role mapping**.
3. Nhấp **Assign role**, tick chọn `vpn-access`, rồi bấm **Assign**.
4. Quay lại danh sách Groups, mở nhóm con **SysAdmins**.
5. Chuyển sang tab **Role mapping**.
6. Nhấp **Assign role**, tick chọn `server-admin`, rồi bấm **Assign**.

### Bước 3.4: Thêm User và kiểm chứng Thừa kế Role
1. Đi tới menu **Users** -> Nhấn **Add user**.
2. Tạo User thứ nhất:
   - Username: `it_staff`
   - Nhấn **Create**.
   - Mở User `it_staff`, sang tab **Groups**, chọn **Join Group**, chọn `IT-Dept` -> **Join**.
3. Tạo User thứ hai:
   - Username: `sys_admin`
   - Nhấn **Create**.
   - Mở User `sys_admin`, sang tab **Groups**, chọn **Join Group**, chọn `SysAdmins` nằm trong `IT-Dept` -> **Join**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu:**
- Mở user `it_staff` -> Chuyển sang tab **Role mapping**.
  - Kiểm tra mục *Effective Roles*: Người dùng này phải sở hữu quyền `vpn-access` thông qua nhóm, nhưng **không** sở hữu `server-admin`.
- Mở user `sys_admin` -> Chuyển sang tab **Role mapping**.
  - Kiểm tra mục *Effective Roles*: Người dùng này BẮT BUỘC phải hiển thị cả `server-admin` (do map trực tiếp từ SysAdmins) VÀ `vpn-access` (thừa kế từ nhóm cha là IT-Dept).
  - Điều này chứng minh rằng Keycloak hỗ trợ phân tầng kế thừa Role mapping từ Parent Group xuống Child Group một cách hoàn hảo.

**Troubleshooting (Khắc phục sự cố):**
- Lỗi: `sys_admin` không nhận được quyền `vpn-access` của nhóm gốc.
  - *Kiểm tra:* Xem lại cấu trúc tạo. Nếu `SysAdmins` được tạo ở Root thay vì nằm lồng bên trong `IT-Dept`, sự thừa kế sẽ bị phá vỡ. Bạn có thể kéo thả (Drag & Drop) hoặc xóa đi tạo lại dưới dạng Child Group.
- Lỗi: Quyền Effective hiển thị trống không.
  - *Kiểm tra:* Đảm bảo bạn đang xem ở chế độ "Effective Roles" chứ không phải là trực tiếp gán vào user. (Trong UI Quarkus, mặc định nó có bộ lọc `Hide inherited roles` được kích hoạt ở tab Roles của user. Bạn cần bỏ check bộ lọc này để xem các Role được kế thừa từ Group).
