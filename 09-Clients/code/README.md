# Keycloak Client Types Lab

Thư mục này cấu hình sẵn Môi trường Docker Compose Móng Chạm Trọng để Test Hệ thống Phân Biệt Các Loại OIDC Client Type (Public vs Confidential).

## Cấu trúc Cụm
1. **Keycloak Server:** SSO Identity Provider.
2. **Postgres DB:** Lưu trữ dữ liệu Clients và Token Sessions.

## Hướng dẫn 
Khởi động cụm Server:
```bash
docker-compose up -d
```
Xem log Đợi Khởi Động Kẽ Mạch Kép:
```bash
docker-compose logs -f kc_clients_server
```

Tắt cụm:
```bash
docker-compose down -v
```
