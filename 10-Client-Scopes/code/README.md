# Keycloak Client Scopes Lab

Thư mục này cấu hình sẵn Môi trường Docker Compose Móng Chạm Trọng để Test Hệ thống Bộ Lọc Client Scopes (Scope Evaluation).

## Cấu trúc Cụm
1. **Keycloak Server:** SSO Identity Provider.
2. **Postgres DB:** Lưu trữ dữ liệu Clients Scopes.

## Hướng dẫn 
Khởi động cụm Server:
```bash
docker-compose up -d
```
Xem log Đợi Khởi Động Kẽ Mạch Kép:
```bash
docker-compose logs -f kc_scopes_server
```

Tắt cụm:
```bash
docker-compose down -v
```
