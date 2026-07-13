# Keycloak HA Cluster Lab

Môi trường này cung cấp một ví dụ hoàn chỉnh về cách dựng Cụm Keycloak Sẵn Sàng Cao (High Availability - HA) trên nền tảng Docker Compose.

## Cấu trúc Cụm
1. **Nginx:** Đóng vai trò Load Balancer, áp dụng thuật toán `ip_hash` (Sticky session).
2. **Keycloak Node 1 & 2:** Chạy chế độ Production (`start`, `KC_PROXY=edge`), kết nối thành cụm Infinispan thông qua giao thức Discovery `JDBC_PING`.
3. **PostgreSQL:** Lưu trữ dữ liệu cấu hình Realm/User chung cho cả 2 Node.

## Hướng dẫn 
Khởi động cụm:
```bash
docker-compose up -d
```
Xem log để theo dõi thời điểm JGroups kết nối cụm Infinispan thành công:
```bash
docker-compose logs -f kc_node_1
```
Tắt cụm:
```bash
docker-compose down -v
```
