# Lab 1: Giải Phẫu JWT & Cỗ Máy Lọc Quyền (Client Scope Evaluation)

> [!NOTE]
> Bài Lab này đưa bạn vào vai Kiến Trúc Sư Hệ Thống chuyên chống thảm họa Token Bloat (Token phình to vỡ Headers). Bạn sẽ thiết kế 1 Optional Scope chứa quyền Nhạy cảm. Dùng Postman đóng vai React App, tự tay gắn tham số `scope=` để xem Token có được Keycloak bơm quyền vào hay không, và check chức năng Evaluate.

## Chuẩn bị
- Máy có Docker và Docker-Compose.
- Có cài đặt Postman hoặc Terminal.
- JWT Decoder (trang `jwt.io`).

## Bước 1: Ráp Khung Áo Giáp Lõi Tĩnh OIDC Database

1. Đi vào thư mục `10-Client-Scopes/code`. 
2. Mở file `docker-compose.yml`. Mọi thứ đã có đủ bản KC 24+. 

## Bước 2: Bật Cụm Động Cơ OIDC Kéo Nhựa Giao Mạng

1. Khởi động OIDC bằng lệnh Thép Tĩnh Nền:
```bash
docker-compose up -d
```
2. Đăng Nhập Chỉnh Sửa Tại Admin Console: `http://localhost:8080/admin` (admin/admin).
3. Tạo 1 Lãnh Thổ Realm Mới: `Vingroup_Scopes`.

## Bước 3: Tạo Khách Hàng Và Cờ Quyền (Users & Roles)

1. Vô Bảng `Realm roles`. Bấm `Create role`. 
2. Tên Quyền: `doc-bao-cao-vip`. Bấm Save.
3. Vô Bảng `Users`. Nhấn `Add user`.
   - Username: `sep_lon`. Save lại.
   - Ở tab `Credentials`, đặt mật khẩu `pass`.
   - Ở tab `Role mapping`, Assign cái role `doc-bao-cao-vip` cho `sep_lon`.

## Bước 4: Tạo Client Và Đóng Van Tự Sát OOM

1. Vô Bảng `Clients`. Tạo 1 Client tên là `app-bao-cao-react`.
2. Capabilities Config: `Client authentication = OFF` (Public Client).
3. Valid redirect URIs: Điền `http://localhost:3000/*`. Bấm Save.
4. **THAO TÁC SỐNG CÒN:** Vô Tab `Client scopes` của `app-bao-cao-react`.
   - Chạy Xuống Cuối Bảng, Ở Khúc Các Scopes Mặc Định.
   - Tìm dòng `roles`. Bấm `Action -> Remove` để vứt cờ Roles ra khỏi Default!

## Bước 5: Tạo Gói Optional Scope Đổ Khuôn Quyền Mạch Oanh Liệt

1. Đứng Ở Thanh Menu Trái (Realm Menu), Bấm `Client scopes`.
2. Tạo Mới Scope tên là: `scope-bao-cao-mat`.
   - Type: Chọn `Optional`.
   - Display on consent screen: `ON` (để Khách thấy khi bấm Login).
   - Bấm Save.
3. Vô Tab `Scope` Của Thằng Mới Tạo Này, Nhấn `Assign role`.
   - Chọn cờ `doc-bao-cao-vip` ghim vô bụng nó.

## Bước 6: Ghim Kén Optional Vào Client React

1. Vô Menu `Clients` -> Mở thằng `app-bao-cao-react`.
2. Tab `Client scopes` -> Bấm nút `Add client scope`.
3. Tích chọn `scope-bao-cao-mat`. Bấm Nút **`Add -> Optional`**.

## Bước 7: Thực Chiến Bắn Lệnh Gọi API Mở Khóa Động

Bây giờ bạn sẽ gọi Direct Access Grants (User/Pass API) để test, thay vì Mở Trình Duyệt cho nhanh (để kiểm tra JWT Token sinh ra sao). Tạm Vô Bảng App `app-bao-cao-react` Bật `Direct access grants = ON` Lên Lại Lệnh Để Test Cục Oanh Kẽ (Thực Tế Enterprise Không Bật Cho React).

**CÚ BẮN SỐ 1: Bắn Bình Thường (Không Kèm Optional Scope)**
```bash
curl -X POST \
  http://localhost:8080/realms/Vingroup_Scopes/protocol/openid-connect/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password' \
  -d 'client_id=app-bao-cao-react' \
  -d 'username=sep_lon' \
  -d 'password=pass'
```
-> Dán Token lên `jwt.io`. BÙM! Dòng `realm_access.roles` trống trơn không có cờ `doc-bao-cao-vip`. Dù Khách Hàng thật sự nắm quyền này ở Keycloak Database! (Vì ta đã gỡ Default Scope và chưa gọi Optional).

**CÚ BẮN SỐ 2: Ép Cửa Bơm Data Động Bằng Thẻ Phím Mạch Oanh Kẽ Khung Mã Code**
```bash
curl -X POST \
  http://localhost:8080/realms/Vingroup_Scopes/protocol/openid-connect/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password' \
  -d 'client_id=app-bao-cao-react' \
  -d 'username=sep_lon' \
  -d 'password=pass' \
  -d 'scope=openid scope-bao-cao-mat'
```
-> Dán Token Lần Này Lên `jwt.io`. BÙM BÙM! Phép thuật của Lõi Đánh Giá Token (Token Evaluation Engine). Cờ `doc-bao-cao-vip` Đã Nằm Chình Ình Trong JSON Payload.

## Bước 8: Dọn Lệnh Rác Sóng Lưới Mạng OIDC Khép Kín Cấu Cắt
```bash
docker-compose down -v
```

> [!TIP]
> Just-in-Time Token Scope Mạch Này Giúp Bạn Điều Tiết Toàn Bộ Hệ Thống Enterprise Trăm Ứng Dụng Giữ Mức Token Tối Thiểu Nhanh Chóng Cắt Giao Lỗi RAM Đứt 431 Nginx! Ngủ Ngon Khỏi Canh Trực Lệnh OOM Database Lệnh Kéo Bơm Đáy Lên Rìa Đáy Cục Nhựa Dữ Mạch!
