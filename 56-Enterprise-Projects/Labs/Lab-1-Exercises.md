# Lab 1: Deploying an Enterprise IAM Platform

> [!NOTE]
> **Category:** Lab
> **Goal:** Tự tay triển khai và cấu hình một nền tảng Enterprise IAM thu nhỏ nhưng đầy đủ các thành phần kiến trúc lõi (Keycloak, OpenLDAP, BFF, Resource Server) ngay trên máy tính cá nhân bằng Docker Compose.

## 1. Mục tiêu (Objective)

Sau khi hoàn thành bài Lab này, bạn sẽ có khả năng:
- Viết file `docker-compose.yml` để dựng cụm hạ tầng: PostgreSQL, OpenLDAP, và Keycloak.
- Tích hợp thành công Keycloak với hệ thống danh bạ OpenLDAP thông qua tính năng User Federation.
- Khởi chạy một Spring Cloud Gateway đóng vai trò là BFF (Backend-For-Frontend) để bảo vệ giao diện Web.
- Xác thực luồng đi của dữ liệu: Trình duyệt gửi Cookie -> BFF nhận Cookie, đổi thành JWT -> BFF gọi Resource Server -> Resource Server kiểm tra JWT và trả về dữ liệu.

## 2. Chuẩn bị (Prerequisites)

- Máy tính đã cài đặt **Docker Desktop** (trên Windows/Mac) hoặc **Docker Engine & Docker Compose** (trên Linux).
- Đảm bảo các cổng `8080`, `8081`, `8082`, `5432` và `389` trên máy tính chưa bị chiếm dụng.
- RAM máy tính tối thiểu còn trống 4GB để chạy cùng lúc nhiều container Java.
- Di chuyển terminal vào thư mục: `56-Enterprise-Projects/code` (nơi chứa sẵn mã nguồn mẫu).

## 3. Các bước thực hiện (Step-by-step Instructions)

### Bước 1: Khởi động Hạ tầng lõi (Core Infrastructure)
Trong thư mục `code`, file `docker-compose.yml` đã được định nghĩa sẵn các dịch vụ. Chúng ta sẽ khởi động Database và LDAP trước.

```bash
# Khởi động PostgreSQL và OpenLDAP trong chế độ ngầm
docker-compose up -d postgres openldap

# Đợi 10 giây để DB khởi động xong, sau đó bật Keycloak
docker-compose up -d keycloak
```

### Bước 2: Cấu hình Liên kết LDAP (User Federation)
1. Mở trình duyệt, truy cập `http://localhost:8080`. Đăng nhập bằng tài khoản `admin` / `admin`.
2. Tạo một Realm mới tên là `enterprise-realm`.
3. Ở menu trái, chọn **User Federation** -> Chọn **Add Ldap providers**.
4. Điền các thông số kết nối tới OpenLDAP container:
   - **Vendor:** `Other`
   - **Connection URL:** `ldap://openldap:389`
   - **Users DN:** `ou=users,dc=mycompany,dc=com`
   - **Bind Type:** `simple`
   - **Bind DN:** `cn=admin,dc=mycompany,dc=com`
   - **Bind Credential:** `admin123`
5. Bấm **Save**. Sau đó bấm sang tab **Action** -> **Sync all users**. Nếu thấy thông báo "Success", bạn đã đồng bộ thành công!

### Bước 3: Cấu hình BFF Client
BFF (Spring Cloud Gateway) cần một định danh để giao tiếp với Keycloak.
1. Tại menu trái, chọn **Clients** -> **Create client**.
2. **Client ID:** `bff-client`
3. **Client authentication:** Bật thành `ON` (Để biến nó thành Confidential Client).
4. **Valid redirect URIs:** `http://localhost:8081/login/oauth2/code/keycloak`
5. Bấm **Save**. Chuyển sang tab **Credentials**, copy chuỗi **Client Secret** và dán vào file cấu hình `application.yml` của BFF.

### Bước 4: Khởi động BFF và Resource Server
Sử dụng Docker Compose để bật 2 Microservices (đã được cấu hình sẵn trong mã nguồn mẫu):

```bash
docker-compose up -d bff-gateway resource-server
```
*Lưu ý:* Cổng `8081` dành cho BFF Gateway, cổng `8082` dành cho Resource Server.

### Bước 5: Gọi API thử nghiệm
1. Truy cập trực tiếp vào Resource Server: Mở trình duyệt vào `http://localhost:8082/api/data`.
   -> Bạn sẽ nhận được lỗi `401 Unauthorized` vì thiếu Bearer Token.
2. Truy cập thông qua BFF: Mở trình duyệt vào `http://localhost:8081/`.
   -> Hệ thống sẽ tự động Redirect bạn sang màn hình đăng nhập của Keycloak.

## 4. Xác minh (Verification)

Để chứng minh kiến trúc Enterprise IAM đã hoạt động hoàn hảo:

1. **Kiểm tra đăng nhập LDAP:** Tại màn hình đăng nhập Keycloak, hãy dùng tài khoản `john.doe` / `password123` (Đây là tài khoản nằm sẵn trong OpenLDAP, không được tạo tay trên Keycloak). Nếu đăng nhập thành công, chứng tỏ User Federation chạy tốt.
2. **Kiểm tra BFF & HttpOnly Cookie:** Sau khi đăng nhập, hệ thống sẽ đẩy bạn về lại giao diện trang chủ của BFF (`http://localhost:8081`). 
   - Nhấn F12 mở Developer Tools, qua tab **Application** -> **Cookies**.
   - Bạn sẽ thấy một Cookie tên là `SESSION` (hoặc `JSESSIONID`).
   - Đánh dấu tick `HttpOnly` phải được bật sáng. Tuyệt đối không thấy bóng dáng của JWT trên trình duyệt.
3. **Kiểm tra Token Relay:** Bấm nút **"Fetch Secure Data"** trên giao diện BFF.
   - Giao diện sẽ hiển thị dòng chữ: `"Hello John Doe! You have successfully accessed the Resource Server via BFF Token Relay!"`.
   - Điều này chứng tỏ BFF đã nhận Cookie của bạn, dịch nó thành Bearer JWT, và chuyển tiếp (Relay) qua cho Resource Server ở cổng `8082` để lấy dữ liệu thành công.

Chúc mừng bạn đã hoàn thành việc triển khai kiến trúc Enterprise IAM bảo mật cấp độ cao!
