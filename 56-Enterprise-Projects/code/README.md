# Lab 1 Code: Enterprise IAM Platform

Thư mục này chứa mã nguồn mẫu để khởi chạy kiến trúc Enterprise IAM thu nhỏ cho Bài Lab số 1 của Chương 56.

## Thành phần hệ thống

1. **PostgreSQL (`postgres`)**: Database lưu trữ cấu hình và Users của Keycloak. Cổng: `5432`.
2. **OpenLDAP (`openldap`)**: Máy chủ danh bạ giả lập Active Directory của doanh nghiệp. Cổng: `389`.
3. **Keycloak (`keycloak`)**: Động cơ xác thực trung tâm. Cổng: `8080`.
4. **BFF Gateway (`bff-gateway`)**: Microservice đóng vai trò mặt tiền (Backend-For-Frontend) xử lý Token Relay (hiện tại dùng Nginx giả lập). Cổng: `8081`.
5. **Resource Server (`resource-server`)**: Microservice cung cấp API nội bộ (hiện tại dùng Nginx giả lập). Cổng: `8082`.

## Hướng dẫn chạy

1. Mở Terminal tại thư mục chứa file `docker-compose.yml`.
2. Chạy lệnh:
   ```bash
   docker-compose up -d
   ```
3. Truy cập Keycloak Admin Console tại: `http://localhost:8080` (Tài khoản: `admin` / Mật khẩu: `admin`).
4. Theo dõi bài Lab để tiến hành kết nối LDAP và cấu hình luồng BFF.
