# Keycloak Group Hierarchy Lab

Thư mục này cấu hình sẵn Môi trường Docker Compose Móng Chạm Trọng để Test Hệ thống Cây Phân Cấp (Group Hierarchy) và Lan Truyền Quyền Lực (Role Mappings).

## Cấu trúc Cụm
1. **Keycloak Server:** SSO Identity Provider.
2. **Postgres DB:** Lưu trữ dữ liệu Nhóm và Gia Phả.

## Hướng dẫn 
Khởi động cụm Server:
```bash
docker-compose up -d
```
Xem log Đợi Khởi Động Kẽ Mạch Kép:
```bash
docker-compose logs -f kc_groups_server
```
Tắt cụm:
```bash
docker-compose down -v
```
