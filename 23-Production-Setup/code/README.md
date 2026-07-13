# Chapter 23 Code & Labs

Thư mục này cung cấp Cỗ Máy Production Hoàn Hảo gồm Load Balancer NGINX, Postgres, và Cluster 2 Node Keycloak.

## Khởi động môi trường

1. Mở terminal tại thư mục này.
2. Chạy lệnh:
```bash
docker-compose up -d
```
3. Truy cập thông qua NGINX: `http://localhost/` (Cổng 80) bằng Admin/admin.
4. Trải nghiệm Tắt Mở Node 1 Xem Cảm Giác Session Không Chết (High Availability).

## Hướng dẫn thực hành (Labs)
Mở file `../Labs/Lab-1-Exercises.md` để xem các bước Trải Nghiệm Kiến Trúc Của Các Chuyên Viên RedHat Đáy Lõi DB Trút Cắt Khung Tương Lai.

## Dọn dẹp môi trường

```bash
docker-compose down -v
```
