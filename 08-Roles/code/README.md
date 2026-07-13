# Keycloak Composite Roles Lab

Thư mục này cấu hình sẵn Môi trường Docker Compose Móng Chạm Trọng để Test Hệ thống Cấp Quyền Hợp Thể (Composite Roles) và Token Engine Effective Roles.

## Cấu trúc Cụm
1. **Keycloak Server:** SSO Identity Provider.
2. **Postgres DB:** Lưu trữ dữ liệu Quyền Hạn và Dòng Chảy.

## Hướng dẫn 
Khởi động cụm Server:
```bash
docker-compose up -d
```
Xem log Đợi Khởi Động Kẽ Mạch Kép:
```bash
docker-compose logs -f kc_roles_server
```
Giải mã Base64 Token OIDC:
- Bạn Có Thể Dùng Trang `https://jwt.io` Của Auth0.
- Hoặc Dùng Mạch Lệnh CLI Code JWT Đáy Nếu Không Muốn Đưa Token Mạch Oanh Liệt Lên Web.

Tắt cụm:
```bash
docker-compose down -v
```
