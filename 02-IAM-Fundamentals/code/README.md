# Hướng dẫn chạy Lab 1 (Chương 02)

Đây là kho lưu trữ mã nguồn cho Lab 1 của Chương `02-IAM-Fundamentals`.
Nó chứa file khởi chạy Hạ tầng y hệt như Chương 1, dùng để làm Môi trường Thực hành Ánh xạ các Triết lý IAM lên Giao diện.

## Cách khởi chạy

1. Mở Terminal tại thư mục hiện tại (`02-IAM-Fundamentals/code/`).
2. Gõ lệnh khởi động cụm máy chủ:
   ```bash
   docker compose up -d
   ```
3. Truy cập vào trang Quản trị:
   - URL: `http://localhost:8080/`
   - Bấm vào **"Administration Console"**.
   - Đăng nhập với tài khoản Admin mặc định (`admin` / `admin`).

## Dọn dẹp
Để tắt máy chủ và xóa sạch mọi dữ liệu thực hành (Volume db):
```bash
docker compose down -v
```
