# Hướng Dẫn Chạy Môi Trường Custom User Storage 

Thư mục này chứa cấu hình Docker Compose để khởi chạy một Hệ Sinh Thái thu nhỏ: 1 Máy Chủ Keycloak và 1 Máy Chủ MySQL Đồ Cổ.

## 1. Cấu trúc thư mục

```text
code/
├── docker-compose.yml     # File khởi động Keycloak và MySQL DB.
├── README.md              # File hướng dẫn này
└── my-providers/          # Thư mục hứng file JAR chứa Code Java Provider
```

## 2. Cách Vận Hành

1. Viết Code Java SPI cho `MySqlUserStorageProvider` như hướng dẫn trong phần Lab. Hãy đảm bảo JDBC Driver cho MySQL nằm trong mục dependencies (`pom.xml`).
2. Build Code bằng Maven: `mvn clean package`.
3. Copy file `.jar` vừa Build ném vào thư mục `my-providers`.
4. Mở terminal tại thư mục `code/` và gõ: `docker-compose up -d`.
5. Đợi 1 phút cho MySQL và Keycloak khởi động hoàn toàn.
6. Kết nối vào MySQL bằng công cụ (DBeaver/DataGrip) qua cổng `3306`, user `root`, pass `root_pass`. Mở DataBase `old_system`.
7. Tạo bảng và thêm 1 dòng dữ liệu giả lập:
   ```sql
   CREATE TABLE tbl_khachhang (name VARCHAR(50), pwd VARCHAR(50));
   INSERT INTO tbl_khachhang (name, pwd) VALUES ('teo', '123');
   ```
8. Đăng nhập vào giao diện Admin Keycloak (localhost:8080).
9. Chuyển sang Tab `User Federation`, chọn Add Provider. Tìm `mysql-legacy-db` trong danh sách xổ xuống và bấm Save.
10. Mở Trình duyệt Ẩn Danh, Đăng nhập Keycloak bằng User `teo` và pass `123`. Đón chờ điều kỳ diệu!
