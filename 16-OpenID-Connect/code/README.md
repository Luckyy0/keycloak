# Chapter 16 Code & Labs

Thư mục này cung cấp môi trường thực hành OpenID Connect (OIDC) bằng Keycloak.

## Khởi động môi trường

1. Mở terminal tại thư mục này.
2. Chạy lệnh:
```bash
docker-compose up -d
```
3. Đợi vài giây, truy cập `http://localhost:8080` (admin/admin).

## Hướng dẫn thực hành (Labs)
Mở file `../Labs/Lab-1-Exercises.md` để xem hướng dẫn thực hành lấy ID Token, gọi UserInfo và chạy luồng Logout.

## Dọn dẹp môi trường

```bash
docker-compose down -v
```
