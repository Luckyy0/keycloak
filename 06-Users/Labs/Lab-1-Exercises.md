# Lab 1: Cuộc Chiến Bảo Vệ Thành Trì (Users & Policies)

> [!NOTE]
> Bài Lab này sẽ đẩy bạn vào vị trí của một Quản trị viên An ninh. Bạn sẽ tự tay dựng lên hệ thống Bắt Buộc Xác Thực Email, Đóng Băng Dữ Liệu bằng User Profile, và Hành hạ Hacker bằng Kỷ Luật Mật Khẩu Thép.

## Chuẩn bị
- Máy có Docker và Docker-Compose.

## Bước 1: Ráp Khung Áo Giáp Email Giả Lập

1. Đi vào thư mục `06-Users/code`. 
2. Mở file `docker-compose.yml`. Quan sát Khung Ống Tiêm Ảo Nước Mới Tinh:
   - Cụm `mailhog`: Đây là Máy chủ Bắt Thư Tạm Thời Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng (Giúp bạn Test Tính Năng Email Verification mà không tốn tiền API gửi thật).

## Bước 2: Bật Cụm Động Cơ OIDC Kéo Nhựa

1. Khởi động OIDC bằng lệnh Thép Tĩnh Nền:
```bash
docker-compose up -d
```
2. Đăng Nhập Chỉnh Sửa Tại Admin Console: `http://localhost:8080/admin` (admin/admin).
3. Tạo 1 Lãnh Thổ Realm Mới: `Vingroup_Users`.

## Bước 3: Khoác Áo Lưới SMTP Cho Khung Rỗng Email OIDC

1. Vào `Realm Settings` -> `Email`.
2. Khai Báo Phóng Lệnh SMTP MailHog:
   - Host: `mailhog`
   - Port: `1025`
   - From: `admin@vingroup.com`
   - Nhấn Save. (Nút Test Connection Sẽ Báo Xanh Đáy Rực Rỡ Kéo Khống Mệnh Rút Lệnh).
3. Qua Tab `Login` Đỉnh Oanh Kẽ Sóng. Gạt Nút Bật Bão Mạng Sạch `User registration` Và Khung `Verify email`. Bật Cả Lệnh Đáy `Login with email` và TẮT Khung Rỗng Kéo Kẻ Khách Nằm Trữ Bọc `Duplicate emails` Cắt Lệch Mạch OIDC Khung Rác Mạng.
4. Save Lại Oanh Khống Mạch.

## Bước 4: Thiết Quân Luật Database Bằng Declarative User Profile OIDC

1. Vẫn Trong `Realm Settings` -> Tab `User Profile`. Bật Khung (Enabled) Đáy Nếu Đang Bị Tắt Lệnh.
2. Bấm Khung Nút `Add Attribute`:
   - Name: `department`. 
   - Permissions: Mở Tích Lệnh Rỗng `User (View/Edit)` Của Cột Này Cho Phép Đăng Ký.
   - Validations: Nút Add Lệnh Rỗng OIDC `options`. Nhập OIDC String Nhựa Giá Trị Cấm Đáy Kẽ Lệnh Tĩnh: `IT, Sales, HR`. (Khách Gõ Dữ Cột Khác Bị Chặn Lỗi Văng Form).

## Bước 5: Sao Lưu Trút Củi Đáy OIDC Mật Khẩu (Password Policies Đỉnh Cao Cháy Nhất)

1. Vô Bảng Lệnh Mạch `Authentication` -> `Policies` -> `Password Policy`.
2. Bấm Trút Nhanh Khúc `Add Policy`. Thêm Đáy `Length` -> Số Rỗng OIDC `8`.
3. Thêm Chữ Khung Khách Bọc `Not Username`.

## Bước 6: Trận Đánh Cuối Cùng Cắt Khung Tĩnh OIDC Bọc Oanh Cáp Sóng Token (Test Lệnh)

1. Mở Trình Duyệt Ẩn Danh OIDC Bọc Lệnh API Nhựa Đáy Kéo Dọc Mũi Rỗng Đít Khung: `http://localhost:8080/realms/Vingroup_Users/account`. Bấm Nút Trút `Sign in`.
2. Khung Tĩnh OIDC Nhanh Trút Bảng Khách Nắm Sẽ Có Nút `Register` Rìa Lệnh OIDC. Bấm Đăng Ký.
3. Form HTML OIDC Rỗng Đã Tự Động Kéo Sinh Ô Nhập `department`. (Hãy Thử Nhập `Marketing` Sẽ Bị Lỗi Dội OOM Vỡ Lỗ Nhựa. Phải Nhập Đúng Chữ Cấu Cắt Khách Kép `IT`).
4. Nhập Tên Đăng Nhập Là `boss` Đáy Bọc Khống Gãy Kẽ Đáy. Nhập Mật Khẩu Là Khúc OIDC Bất Kỳ Chữ Kéo Cáp Chữ Oanh Phẳng `boss` (Sẽ Bị Chính Sách Not Username Cắt Đứt Khách Văng Gãy Cụt Form). Hãy Chỉnh Mật Khẩu OIDC API Đáy Nhanh Thành Lệnh Khống Ép Bức `pass12345`.
5. Đăng Ký Thành Công Bất Diệt Xé Kẽ Lỗi Sụp Tốc! Màn Hình Keycloak Đáy Rễ Căn Cứ Chặn Báo Khách Tĩnh Khung Oanh Lệnh Yêu Cầu Email Verification Bọc Oanh Cáp Sóng.
6. Sang Trình Duyệt Mở Giao Diện MailHog: `http://localhost:8025`. 
7. Bạn Sẽ Thấy Tờ Thư Báo Lệnh Chữ Rỗng Đi Kéo Bằng Văng OIDC Ngang. Bấm Vô Email Lệnh Khống Gãy Khung Tốc Độ Và Click Link Nằm Kéo Trút! BÙM! Khách Hàng OIDC Nhựa Bọc Kép Đã Sinh Vào Tĩnh Nền Đáy Gắn Gốc Rút Chữ OIDC Rỗng!

## Bước 7: Dọn Lệnh Rác Sóng Lưới Mạng OIDC Khép Kín Cấu Cắt
```bash
docker-compose down -v
```

> [!TIP]
> Việc Dựng Form OIDC Đăng Ký Ở Bài Này Dùng Lại Hoàn Toàn Tĩnh Cơ Chế Tự Rendering Khung Sinh Bọc Của Keycloak. Khung Lệnh Frontend Developer Web Không Cần Chạm Form HTML 1 Dòng Code JS OIDC Nào Nhưng Sức Bật Data Lõi Bảo Mật Vẫn Khớp Rễ Giao Nhựa 100% Sạch Lưới Mạng!
