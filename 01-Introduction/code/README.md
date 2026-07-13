# Hướng dẫn chạy Lab 1

Đây là kho lưu trữ mã nguồn cho Lab 1 của Chương `01-Introduction`.

## Môi trường yêu cầu
- Docker
- Docker Compose

## Cách khởi chạy

1. Mở Terminal tại thư mục hiện tại (`01-Introduction/code/`).
2. Gõ lệnh khởi động cụm máy chủ:
   ```bash
   docker compose up -d
   ```
3. Xem log để đảm bảo Keycloak đã khởi động thành công:
   ```bash
   docker compose logs -f keycloak
   ```
4. Truy cập vào trang Quản trị:
   - URL: `http://localhost:8080/`
   - Bấm vào **"Administration Console"**.
   - Đăng nhập với tài khoản Admin mặc định:
     - Username: `admin`
     - Password: `admin`

## Dọn dẹp
Để tắt máy chủ và xóa sạch mọi dữ liệu thực hành:
```bash
docker compose down -v
```
