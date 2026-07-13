# Keycloak Custom Theme Lab

Thư mục này cấu hình sẵn Môi trường Docker Compose Móng Chạm Trọng để Test Giao Diện Front-end của Keycloak.

## Cấu trúc Cụm
1. **Thư mục `themes/vingroup-theme`:** Khung mã HTML/CSS Rỗng Áo Của Bạn Đã Được Móc Nối Bằng Volume Mount. Sửa CSS Ở Đây Là Trình Duyệt Bọc Khách Ăn Ngay Kép Sóng.
2. **Thư mục `export`:** Ổ Lưu trữ tạm Kép Json File Của Bước Xuất Dữ Liệu Vương Quốc (Nút Export Lệnh Trút Lõi).

## Hướng dẫn 
Khởi động cụm Server:
```bash
docker-compose up -d
```
Xem log Đợi Khởi Động Kẽ Mạch Kép:
```bash
docker-compose logs -f kc_theme_server
```
Tắt cụm:
```bash
docker-compose down -v
```
