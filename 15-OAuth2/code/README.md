# Chapter 15 Code & Labs

Thư mục này chứa môi trường thực hành cho chương quan trọng nhất: OAuth2.

## Khởi động môi trường

1. Mở terminal tại thư mục này.
2. Chạy lệnh:
```bash
docker-compose up -d
```
3. Đợi khoảng 15 giây, truy cập `http://localhost:8080` để vào Keycloak Admin Console.
4. Thông tin đăng nhập mặc định:
   - Username: `admin`
   - Password: `admin`

## Hướng dẫn thực hành (Labs)
Mở file `../Labs/Lab-1-Exercises.md` để xem hướng dẫn chi tiết từng bước. Bạn sẽ sử dụng Postman (Hoặc cURL) để đóng vai các thành phần trong mạng, gõ từng lệnh API mô phỏng cách OAuth2 Flows vận hành (Client Credentials, Authorization Code, Device Flow) dưới tầng HTTP.

## Dọn dẹp môi trường

Khi hoàn thành bài lab, bạn có thể tắt và xóa hoàn toàn database bằng lệnh:
```bash
docker-compose down -v
```
