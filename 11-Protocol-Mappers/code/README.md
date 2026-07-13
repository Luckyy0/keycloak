# Keycloak Protocol Mappers Lab

Thư mục này cấu hình sẵn Môi trường Docker Compose để Test Tính Năng Protocol Mappers (Built-in & Script Mapper).

## Điểm Nổi Bật
- **`KC_FEATURES=scripts`**: Đã được bật mặc định trong Docker Compose để mở khóa giao diện lập trình Javascript Mapper (đã bị Keycloak giấu từ bản 18+).

## Hướng dẫn 
Khởi động cụm Server:
```bash
docker-compose up -d
```
Xem log Đợi Khởi Động Kẽ Mạch Kép:
```bash
docker-compose logs -f kc_mappers_server
```

Tắt cụm:
```bash
docker-compose down -v
```
