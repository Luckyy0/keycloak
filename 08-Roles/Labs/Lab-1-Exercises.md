# Lab 1: Giả Kim Thuật (Hợp Thể Composite Roles)

> [!NOTE]
> Bài Lab này đưa bạn vào vai Giám Đốc An Ninh OIDC. Bạn sẽ tự tay đập tan sự lộn xộn của Cơ sở Dữ liệu bằng cách dùng thuật Giả Kim Hợp Thể: Ép 2 quyền của 2 Ứng dụng khác nhau rải rác chui tọt vào bên trong 1 Realm Role duy nhất. Sau đó Giải Mã JWT để chứng kiến thuật toán Effective Roles chạy ngầm.

## Chuẩn bị
- Máy có Docker và Docker-Compose.

## Bước 1: Ráp Khung Áo Giáp Lõi Tĩnh OIDC Database

1. Đi vào thư mục `08-Roles/code`. 
2. Mở file `docker-compose.yml`. Mọi thứ đã được đóng gói Tĩnh Bọc Nhựa Postgres Cấp K8s. 

## Bước 2: Bật Cụm Động Cơ OIDC Kéo Nhựa Giao Mạng

1. Khởi động OIDC bằng lệnh Thép Tĩnh Nền:
```bash
docker-compose up -d
```
2. Đăng Nhập Chỉnh Sửa Tại Admin Console: `http://localhost:8080/admin` (admin/admin).
3. Tạo 1 Lãnh Thổ Realm Mới: `Vingroup_Roles`.

## Bước 3: Tạo 2 Lãnh Thổ Chư Hầu Mạch Rắn Đáy Khống (Client Roles)

1. Vô Bảng `Clients`. Bấm Vô Tên App Mặc Định OIDC Của Cụm: `account`.
2. Chạy Tab `Roles`. Bấm Khung Nút Chặn Mạch Giao `Create role`. 
   - Tạo Mã Tên Là: `manage-profile-vip`. Save Oanh Kẽ.
3. Quay Ra `Clients`. Bấm Vô Tên App OIDC Khác Nữa Kẽ Đáy: `realm-management`.
   - Vô Tab `Roles`. Bấm `Create role`.
   - Tạo Mã Tên Là: `view-users-vip`. Save Lệnh Thép.

*(Bây giờ bạn đã có 2 Client Roles nằm lọt thỏm ở 2 Chư hầu khác nhau).*

## Bước 4: Chế Tạo Phép Hợp Thể Giả Kim Thuật OIDC Trút Nhanh Sóng (Composite Realm Role)

1. Vô Mạch Giao Khung `Realm roles`. Nhấn Nút `Create role`.
2. Tạo Cột Realm Tên Là: `COMP_Super_VIP`. Bấm Save Oanh Khách.
3. Tĩnh Khung Khớp OIDC Góc Phải Trên Cùng Của Màn Hình Báo Khách Tĩnh Đáy Nhựa. Bấm Nút **`Action -> Add associated roles`**.
4. Bảng OIDC Cháy Băng Mạch Giao Khung Cửa Sổ Mở Ra:
   - Dùng Khẩu Lọc API Nút Filter Dropdown Rút Mạch Đáy: Chọn `Filter by clients`.
   - Gõ Ô Search Chữ OIDC Rỗng Đáy: `vip`.
   - Tick Vô 2 Ô Box Của Lệnh Rìa Bọc `manage-profile-vip` Và `view-users-vip`. 
   - Bấm `Assign`.
*(Cái hộp COMP_Super_VIP giờ đã có 2 thứ vũ khí nhỏ nhét trong bụng).*

## Bước 5: Cấp Khẩu Lệnh Hợp Thể Cho Vua (Assign Composite)

1. Tạo 1 Bảng Mạch OIDC Khách Lạ Tên `Sep_Lon` Trong Khung Rỗng `Users`. Save Mạch Oanh Liệt Dập Cụm Trống. Đặt Password Mạch Nhựa Tĩnh Bọt Lệnh `pass`.
2. Ở Bảng Của `Sep_Lon`, Bấm Sang Tab `Role mapping`. Nhấn `Assign role`. 
3. Lọc API Kéo Cáp Chọn Ngay Thằng Realm Role Trút Lệnh Đuôi: `COMP_Super_VIP`. Bấm Assign Rút Dòng Khách Chặn.
4. Bật Tích Chặn Oanh Rìa Lệnh `Hide inherited roles`. 
   - Đột Nhiên Hiện Ra 2 Cờ Quyền Màu Đỏ Nhạt Mạch OIDC Giao Khung API Ở Bảng Dưới Oanh Liệt Dập Cụm Trống Cắt Lệnh Rỗng Phun Sinh Data Lệnh Client Roles: `manage-profile-vip` và `view-users-vip`. 
   - Hover Dê Chuột Vô Khung Đỏ Nhạt, Nó Báo Cụm Nhựa Rỗng `Inherited from realm role: COMP_Super_VIP` Đáy Kẽ Lệnh Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!

## Bước 6: Phóng Cụm Mạch Giao JWT Khung Tốc Độ Và Khám Nghiệm Tử Thi Token (JWT Decoding)

1. Mở Trình Duyệt Ẩn Danh OIDC Bọc Lệnh API Nhựa Đáy Kéo Dọc Mũi Rỗng Đít Khung: `http://localhost:8080/realms/Vingroup_Roles/account`. Bấm Nút Trút `Sign in`.
2. Gõ User Mạch Rỗng OIDC Đáy Bọc `Sep_Lon` / Pass Oanh Kẽ Sóng Lọc Oanh Liệt Dập Database `pass`. 
3. Vô Web Thành Công Bất Diệt Xé Kẽ Lỗi Sụp Tốc! Mở Giao Diện Web Lọc Khung F12 Chrome -> Tab Network -> Tìm Các Request API Đáy Kẽ Lệnh Của Token (Ví dụ Cục API XHR). Lấy Đáy Chữ Header HTTP Authorization Cục Mã Kép Oanh `Bearer eyJ...` Copy Giao Cụt Cửa Khung Mệnh.
4. Quăng Cục Mã Base64 Vô Trang Web Giải Mã: `https://jwt.io`.
5. Đọc Thấu Trái Tim Payload Json Kéo Cáp Đáy:
   - Thấy Dòng `realm_access.roles` Nằm Khung Code Bọc Oanh Cáp Có Lệnh `COMP_Super_VIP` Đáy Rễ Căn Cứ Lọc Đáy.
   - Thấy Dòng `resource_access.account.roles` Nằm Khung Code Bọc Oanh Có Chữ Cắt Khúc Lệch Mạch `manage-profile-vip` OIDC Cũ Mệnh Ngắn Gọn.
   - Trút Nhanh Sóng Kẽ Nút Dòng Cũ OIDC Rỗng `resource_access.realm-management.roles` Có `view-users-vip` Rất Kính!
Cỗ Máy Token Engine Effective Roles Tính Toán Cắt Mảnh Dữ Liệu Hoàn Hảo Tuyệt Đỉnh Kéo Khống Mệnh Hủy Diệt Ảo Bất Rất Sạch Test Mạng Lỗ Trống Mạng!

## Bước 7: Dọn Lệnh Rác Sóng Lưới Mạng OIDC Khép Kín Cấu Cắt
```bash
docker-compose down -v
```

> [!TIP]
> Việc Nắm Composite Roles Giúp Rút Ngắn OOM Bọc Cháy Đáy Cụm Database 90% Lệnh Code Đáy Bọc API Trút Nhanh Bắn SQL Insert (Thay Vì Insert 100 Quyền Dưới Đáy, Chỉ Tốn 1 Lệnh Insert Realm Role OIDC Phẳng Rỗng). Giữ Cây Role Luôn Sạch Sẽ Tĩnh Bọc Dù Công Ty Lên Tới Hàng Ngàn Web App OIDC Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh!
