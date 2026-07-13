# Lab 1: Thay Áo Vương Quốc và Phép Tái Sinh

> [!NOTE]
> Bài Lab này sẽ biến bạn thành một Nghệ Nhân Theme OIDC. Chúng ta sẽ thay đổi Giao diện trang Đăng Nhập cùi bắp mặc định của Keycloak bằng logo tự chế, sau đó Export Vương Quốc ra File Json để dọn nhà.

## Chuẩn bị
- Máy có Docker và Docker-Compose.

## Bước 1: Ráp Khung Áo Giáp Theme

1. Đi vào thư mục `05-Realms/code`. 
2. Khám phá cấu trúc Theme chúng ta chuẩn bị đưa vào Keycloak:
   - `themes/vingroup-theme/login/theme.properties`: Nơi khai báo Cấu hình Kế thừa (Base on Keycloak theme).
   - `themes/vingroup-theme/login/resources/css/custom.css`: Nơi bạn phá vỡ giao diện CSS.
3. Mở file `docker-compose.yml`. Quan sát Volume Mount:
   `- ./themes/vingroup-theme:/opt/keycloak/themes/vingroup-theme:ro`
   Việc Mount này chính là Quyền Năng "Sửa Cấu Trúc Khung Nhựa Cháy Sóng" Code Front-end không cần Build Lại!

## Bước 2: Bật Cụm Động Cơ Kéo Khách Nhựa

1. Khởi động OIDC bằng lệnh Thép Tĩnh Nền:
```bash
docker-compose up -d
```
2. Đăng Nhập Chỉnh Sửa Tại Admin Console: `http://localhost:8080/admin` (admin/admin).
3. Tạo 1 Lãnh Thổ Realm Mới: `Vingroup`.

## Bước 3: Khoác Áo Theme Cho Khung Rỗng Login

1. Vẫn Trong Realm `Vingroup`, Bấm Mở Menu `Realm Settings` -> Tab `Themes`.
2. Ở Dòng `Login Theme`, Bạn Kéo Chuột Xổ Xuống, Chắc Chắn Sẽ Thấy Tên Lệnh Gắn `vingroup-theme` Của Cậu Dev Vừa Áp Kéo Trọng RAM. (Nếu không thấy tức là Volume Mount Docker đang lỗi đường dẫn hoặc quên Restart).
3. Bấm Lệnh Chặn `Save`.
4. Mở Tab Trình Duyệt Riêng Tư, Trút Thử Đường API Login Form Rỗng OIDC Dọc Mũi: `http://localhost:8080/realms/Vingroup/account`. 
5. BÙM! Logo Đỏ Chót Của Keycloak Đã Biến Mất, Trang Web Có Nền Thay Màu Đẹp Lành Rỗng Nhựa Do Mã CSS Của Bạn Lọc Lệnh Kéo Cắt.

## Bước 4: Sao Lưu Rút Củi Đáy Vương Quốc Ra JSON Nóng

1. Ta Vừa Có Theme Đỉnh Và Realm Đẹp. Trút API Để Nhét Lưu Dữ Ra Ổ Cứng Máy Laptop Kẽ Gãy Cụt.
2. Ép Tắt Nút Docker Để Vào Chế Độ Rút Export An Toàn Không Mạch Kẽ Đứt Ngầm Oanh RAM Đáy:
```bash
docker stop kc_theme_server
```
3. Khai Báo Phóng Lệnh Docker Đáy Bọc Run Container Export Rỗng (Mount Thư Mục Máy Laptop Vô Thùng Lấy Text):
```bash
docker run --rm \
  -v ./export:/opt/keycloak/data/export \
  -v ./themes/vingroup-theme:/opt/keycloak/themes/vingroup-theme \
  -e KC_DB_URL=jdbc:postgresql://postgres_db_theme:5432/keycloak \
  -e KC_DB_USERNAME=keycloak \
  -e KC_DB_PASSWORD=password \
  --network code_theme-network \
  quay.io/keycloak/keycloak:24.0.1 \
  export --dir /opt/keycloak/data/export --users skip
```
*(Nếu Lỗi Mạch Kéo Database DNS Docker Phẳng, Có Thể Trút Nhanh Export Qua Giao Diện Admin Của Cụm Cháy Lúc Sống Khung Đỉnh Tĩnh OIDC Bọc)*

## Bước 5: Dọn Chiến Trường
Bạn Tự Dọn Lệnh Kéo Dọc Mũi Rỗng Kẽ:
```bash
docker-compose down -v
```

> [!TIP]
> Chỉnh Theme Trực Tiếp Volume Này Giúp Làm Tốc Độ Code Web Lên Đỉnh Nhanh Nút API Phẳng Khung Chặn Bọc Không Tốn Mấy Cú Nhấp Chuột Đít Mạch Kép! Hãy Sử Dụng CSS Inspect F12 Chrome Tìm Mã Lệnh Chữ Rỗng Đi Kéo Đít Thằng RedHat Để Đè Class Xé Áp Nhựa Sóng Nút 8080 Kéo Ra Nhất Lõi!
