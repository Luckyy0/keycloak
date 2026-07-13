# Chapter 21 Code & Labs

Thư mục này cung cấp môi trường thực hành User Federation (LDAP) Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt.

## Khởi động môi trường

1. Mở terminal tại thư mục này.
2. Chạy lệnh:
```bash
docker-compose up -d
```
3. Lưu ý: Compose file này cắm lên Máy Chủ Keycloak và MÁY CHỦ OPENLDAP chứa sẵn 1 Khách Hàng ảo `nguyen-van-b` (Pass: `admin`) Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm.
   - Keycloak: `http://localhost:8080`

## Hướng dẫn thực hành (Labs)
Mở file `../Labs/Lab-1-Exercises.md` để xem hướng dẫn thực hành Đấu Nối Ống Nước Bơm Dữ Liệu LDAP Về Trái Tim Keycloak Oanh Cáp Giao Diện Lệnh Chặt Mạch Lụa.

## Dọn dẹp môi trường

```bash
docker-compose down -v
```
