# Lab 1: Triển Khai Phân Quyền Hạt Mịn (Fine-Grained Authorization) Với Keycloak

## 1. Mục Tiêu (Objectives)
Thực hành cấu trúc phân quyền UMA (User-Managed Access) của Keycloak bao gồm:
- **Task 1:** Bật tính năng Authorization Services trên một Confidential Client.
- **Task 2:** Tạo các Resources và Scopes để mô phỏng một kho tài liệu.
- **Task 3:** Tạo các Policies (Quy tắc kiểm duyệt) và kết dính chúng bằng Permissions.
- **Task 4:** Mô phỏng bài toán "Trọng Tài (Decision Strategy)": Ai được xem Báo Cáo Tài Chính?

---

## 2. Chuẩn Bị (Prerequisites)
Khởi động hệ thống Keycloak bằng docker-compose đã cung cấp.

```bash
cd code
docker-compose up -d
```
Đăng nhập Admin Console tại `http://localhost:8080/` với tài khoản `admin` / `admin`.

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Bật Authorization Services Cho Client
Phân quyền hạt mịn chỉ chạy được trên Confidential Client (Client bảo mật có Client Secret).
1. Truy cập Realm `master` (hoặc Realm tự tạo).
2. Vào **Clients** -> Nhấn **Create client**.
   - Client ID: `document-service`
   - Client authentication: Chuyển sang **ON** (để biến nó thành Confidential Client).
   - Authorization: Chuyển sang **ON** (Kích hoạt bộ máy UMA).
   - Nhấn Save.
3. Sau khi tạo xong, bạn sẽ thấy Client có xuất hiện thêm 1 Tab tên là **Authorization**. Bấm vào đó.

### Task 2: Tạo Tài Nguyên (Resource) Và Hành Động (Scope)
Ta sẽ tạo ra một bức tường bảo vệ cho 1 Tài liệu tên là "Financial-Report-2023" với 2 hành động: Đọc (read) và Xóa (delete).
1. Đứng tại tab **Authorization**, bấm sang sub-tab **Scopes**.
   - Bấm `Create authorization scope`.
   - Đặt Name là: `doc:read`. Nhấn Save.
   - Trở lại Scopes, tạo thêm 1 cái nữa: `doc:delete`.
2. Bấm sang sub-tab **Resources**.
   - Bấm `Create resource`.
   - Đặt Name là: `Financial-Report-2023`.
   - Chỗ ô `Scopes` (dưới cùng), nhấn xổ xuống và chọn MÓC cả 2 scopes `doc:read` và `doc:delete` vào Tài nguyên này.
   - Nhấn Save.

### Task 3: Tạo Cảnh Sát (Policies)
Ta sẽ tạo 2 ông Cảnh Sát: 1 ông đòi Role Kế Toán, 1 ông đòi Role Sếp.
1. Tại tab **Authorization**, bấm sang sub-tab **Policies**.
2. Nhấn `Create policy`, chọn dạng **`Role`**.
   - Tên (Name): `Ke-Toan-Policy`.
   - Chọn Add Roles, gõ tên role `accountant` (Tick ô Required). Save. *(Note: Bạn cần vào menu Realm Roles tạo sẵn role `accountant` và `boss` nếu máy chưa có).*
3. Trở lại Policies, tạo thêm 1 ông Cảnh Sát dạng **`Role`** nữa.
   - Tên (Name): `Sep-Policy`.
   - Chọn Add Roles, gõ tên role `boss`. Save.

### Task 4: Trói Luật Vào Tài Nguyên Bằng Permissions & Chơi Toán Học Quyết Định (Decision Strategies)
Bài toán Sếp yêu cầu:
- **Luật Xem (Read):** Kế toán xem được, Sếp cũng xem được (Lệnh OR - Affirmative).
- **Luật Xóa (Delete):** Chỉ duy nhất Sếp mới được xóa.
Ta sẽ cấu hình Permission để giải bài này.

1. Tại tab **Authorization**, bấm sang sub-tab **Permissions**.
2. Bấm `Create permission` -> Chọn dạng **`Scope-based`**.
   - Name: `Allow-Read-Report-Permission`.
   - Resource: Chọn `Financial-Report-2023`.
   - Scopes: Chọn `doc:read`. (Lệnh này chỉ quản lý hành động Read).
   - Policies: Thêm CẢ 2 ông cảnh sát `Ke-Toan-Policy` và `Sep-Policy` vào.
   - Decision Strategy: Chọn **`Affirmative`** (Quan trọng! Có nghĩa là 1 trong 2 ông pass là được).
   - Save Lại.

3. Trở ra Permissions, tạo Permission thứ 2 (Dạng **`Scope-based`**):
   - Name: `Allow-Delete-Report-Permission`.
   - Resource: Chọn `Financial-Report-2023`.
   - Scopes: Chọn `doc:delete`. (Lệnh này quản lý hành động Xóa).
   - Policies: CHỈ THÊM MỘT MÌNH `Sep-Policy` vào.
   - Decision Strategy: Để Unanimous (Mặc định).
   - Save Lại.

**Kết Quả Logic Phân Quyền Hạt Mịn Của Bạn Vừa Setup:**
- Nếu một nhân viên mang role `accountant` truy cập vào, họ sẽ lọt qua được cổng `doc:read` (nhờ Affirmative), nhưng bị báo 403 Forbidden Access Denied nếu cố gửi lệnh HTTP DELETE gọi cổng `doc:delete` (vì thiếu luật Sếp).
- Tương tự, ông `boss` sẽ đi qua lọt toàn bộ!

---

## 4. Dọn Dẹp (Cleanup)
Sau khi hoàn thành thử nghiệm, nếu muốn làm sạch môi trường:
```bash
docker-compose down -v
```
