# Keycloak Architecture - Code Lab

Thư mục này chứa hạ tầng Docker Compose để phục vụ Bài Lab Giải Phẫu Hệ Quản Trị Cơ Sở Dữ Liệu Quan Hệ (PostgreSQL) của Hệ Thống Nhúng Tách Biệt Keycloak.

## Hướng dẫn chạy

1. Chạy cụm Container ngầm:
```bash
docker-compose up -d
```

2. Đợi khoảng 10-15s cho Liquibase Engine tạo bảng. Trạng thái Keycloak sẽ in ra trên logs. Kiểm tra bằng lệnh:
```bash
docker-compose logs -f keycloak
```

3. Dùng trình quản lý (DBeaver) kết nối vào DB theo thông số:
- Host: `localhost`
- Port: `5432`
- User/Pass: `keycloak` / `password`

4. Để dừng và xóa sạch data sau khi Lab xong:
```bash
docker-compose down -v
```
