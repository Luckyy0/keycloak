# Lab 1: Mê Cung Gia Phả (Group Tree & Kế Thừa Quyền Lực)

> [!NOTE]
> Bài Lab này đưa bạn vào Trung Tâm Cấp Quyền Hàng Loạt. Bạn sẽ Tự Tay Dựng Lên Cây Gia Phả Của 1 Tập Đoàn. Cấp Quyền Từ Nút Cha Cho Thác Nước Đổ Tràn Xuống Nút Con, Sau Đó Cắm Nút Mặc Định Đón Lõng Người Mới.

## Chuẩn bị
- Máy có Docker và Docker-Compose.

## Bước 1: Ráp Khung Áo Giáp Lõi Tĩnh OIDC

1. Đi vào thư mục `07-Groups/code`. 
2. Mở file `docker-compose.yml`. Mọi thứ đã được đóng gói Tĩnh Bọc Nhựa Postgres Cấp K8s. (Giống Bài Lab Khung Users Ở Trương Khúc Sóng Trầm).

## Bước 2: Bật Cụm Động Cơ OIDC Kéo Nhựa Vingroup Cấp

1. Khởi động OIDC bằng lệnh Thép Tĩnh Nền:
```bash
docker-compose up -d
```
2. Đăng Nhập Chỉnh Sửa Tại Admin Console: `http://localhost:8080/admin` (admin/admin).
3. Tạo 1 Lãnh Thổ Realm Mới: `Vingroup_Corp`.

## Bước 3: Đẽo Cây Phả Hệ OIDC Trút Nhanh Sóng (Group Tree)

1. Vô Bảng `Groups`. Nhấn `Create group`. 
2. Tên Nhóm Gốc Đáy: `Vingroup`. Bấm Save.
3. Bấm Vô Tên Group `Vingroup` Mạch Nhựa Kéo Sát Vừa Cháy. Nhấn Nút `Create child group` (Sinh Nút Con Tĩnh).
4. Tên Đáy: `Vinmec`. Bấm Save Khung Lệnh Rỗng.
5. Vô Lại `Groups` -> Bấm Trút Nhanh Vô Cây `Vingroup` -> `Vinmec`. Tiếp Tục Tạo `Create child group` Tên Là `IT_Vinmec`. Save Mạch.

## Bước 4: Chế Tạo Quyền Lực Cắt Khúc Lệch Mạch OIDC Cũ Mệnh (Realm Roles)

1. Vô Mạch Giao Khung `Realm roles`. Nhấn Nút `Create role`.
2. Tạo Mã Cắt Mạch Đáy: `view-company-secrets`. Save Oanh Kẽ.
3. Tạo Trút Thêm Mã Nữa Lệnh Đáy: `manage-it-servers`. Save Oanh Khách.

## Bước 5: Cắm Cờ Quyền Lực Đáy Thép (Group Role Mappings)

1. Quay Lại Bảng Lệnh Mạch `Groups`.
2. Nhấn Vô Đỉnh Cụm Cây `Vingroup`. Chạy Tab `Role mapping`. Nhấn `Assign role`. 
   - Check Dấu Tích Lệnh `view-company-secrets`. Bấm Assign Lọc Oanh Liệt Dập Database Thủng Căng.
   *(Quyền này sẽ Thác Nước Xuống Tận Đáy Của Tạp Vụ Các Công Ty Con Bọc Kẽ Lệnh).*
3. Nhấn Mạch Vô Bảng Lưới Cây Con Tận Cùng `/Vingroup/Vinmec/IT_Vinmec`.
   - Vô Tab OIDC Kéo Nhựa `Role mapping`. Bấm `Assign role`.
   - Lọc API Kéo Cáp Chọn `manage-it-servers`. Bấm Assign Rút Dòng Khách Chặn OOM Vỡ Lỗ.

## Bước 6: Phóng Cụm Mạch Giao JWT Khung Tốc Độ (Kiểm Kiểm OIDC Thác Kế Thừa Mạch Khách Vô Group)

1. Tạo 1 Bảng Mạch OIDC Khách Lạ Tên `NguyenA` Trong Khung Rỗng `Users`. Save Mạch Oanh Liệt Dập Cụm Trống. Đặt Password Mạch Nhựa Tĩnh Bọt Lệnh `pass`.
2. Ở Bảng Của `NguyenA`, Bấm Sang Tab `Groups`. Nhấn `Join group`. Chọn Kéo Cáp OIDC Kẽ Nút Áp Lá Đáy: `Vingroup > Vinmec > IT_Vinmec`. Bấm Join.
3. Sang Mạch Giao Tab `Role mapping` Của Cái Cậu Khách Này.
4. Tích Vô Ô Box Mệnh Lệnh Khống `Hide inherited roles`. 
   - Đột Nhiên Hiện Ra 2 Cờ Quyền Màu Đỏ Nhạt Lệnh Database UUID Không Gãy Chỗ: `view-company-secrets` và `manage-it-servers`. 
   - Dê Chuột OIDC Phẳng Nhựa Bọc Kép Mạng Đáy Cột Nhựa Hover Lên Quyền `view-company-secrets`. Bảng Tooltip Đáy Thép Kẽ Lệnh Sẽ Báo Chữ: Khách Cũ Kẽ Khung Được Cấp Nhờ Group Gốc Cha `/Vingroup`. Mọi Tính Toán Thác Đổ Của Keycloak Đã Xác Nhận Mã Thành Công 100% Cắt Lệnh Sạch Sẽ Trút Bọc Nhựa Bất Sát Giao OIDC Thép Nhựa Cáp Sóng Đáy Kẽ Lệnh Database!

## Bước 7: Dọn Lệnh Rác Sóng Lưới Mạng OIDC Khép Kín Cấu Cắt
```bash
docker-compose down -v
```

> [!TIP]
> Việc Nắm Cây Gia Phả Của Keycloak (Group Tree) Sẽ Giúp Cho Bảng Web Admin OIDC Của Bạn Luôn Sạch Sẽ Tĩnh Bọc Dù Công Ty Tăng Trưởng Khách Lên Tới 1 Triệu Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh. Không Có Cây Phả Hệ Tĩnh Nền Đáy Bọc Khách (Flat Network), IAM System Coi Như Chết Đứt Nhanh Cụm Cháy Băng Thép Dây Cáp Mạng!
