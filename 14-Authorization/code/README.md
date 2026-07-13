# Chapter 14 Code & Labs

Thư mục này chứa môi trường thực hành cho chương Authorization (Phân Quyền).

## Khởi động môi trường

1. Mở terminal tại thư mục này.
2. Chạy lệnh:
```bash
docker-compose up -d
```
3. Đợi khoảng 15 giây, truy cập `http://localhost:8080` để vào Keycloak Admin Console.
4. Thông tin đăng nhập mặc định:
   - Username: `admin`
   - Password: `admin`

## Hướng dẫn thực hành (Labs)
Mở file `../Labs/Lab-1-Exercises.md` để xem hướng dẫn chi tiết từng bước. Bạn sẽ được đóng vai System Admin để tự tay triển khai tính năng Authorization Services (Phân quyền hạt mịn) lên một kho Tài Liệu bằng cơ chế kết hợp Scopes, Resources, Policies và Permissions.

## Dọn dẹp môi trường

Khi hoàn thành bài lab, bạn có thể tắt và xóa hoàn toàn database (làm mới lại cấu hình) bằng lệnh:
```bash
docker-compose down -v
```
