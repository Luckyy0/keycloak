# Code for Authentication Labs

Thư mục này chứa file cấu hình `docker-compose.yml` để chạy Keycloak bản chuẩn phục vụ cho các bài tập thực hành chương Authentication (Bảo vệ Brute-Force, OTP, Passkeys).

## Cách khởi động

Mở terminal tại thư mục này và chạy lệnh:

```bash
docker-compose up -d
```

Keycloak sẽ khởi động tại địa chỉ: `http://localhost:8080`
- **Tài khoản Admin:** `admin`
- **Mật khẩu:** `admin`

## Lưu ý

- Các cấu hình liên quan đến Authentication Flows, Required Actions, và Brute-Force Protection đều được thực hiện thông qua giao diện **Admin Console**, không cần cấu hình file JSON rườm rà.
- Trải nghiệm Passkeys yêu cầu trình duyệt (Chrome/Edge/Safari) hỗ trợ **WebAuthn API** và máy tính của bạn phải có cảm biến Sinh trắc học (Vân tay / Khuôn mặt) hoặc mã PIN Windows Hello.
- HTTPS là bắt buộc với Passkeys trên môi trường Production, tuy nhiên Trình duyệt mặc định cho phép bỏ qua ngoại lệ nếu chạy trên domain `localhost`. Vì vậy bạn vẫn có thể thực hành mượt mà bằng HTTP ở bài Lab này.
