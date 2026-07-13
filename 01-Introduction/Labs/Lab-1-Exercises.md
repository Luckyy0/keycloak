> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Tự tay cài đặt, khởi chạy một máy chủ Keycloak hoàn chỉnh bằng Docker, và làm quen với giao diện Admin Console để tạo Realm và User đầu tiên.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là một DevOps/Developer mới bắt đầu tìm hiểu về Keycloak. Công ty giao cho bạn nhiệm vụ dựng một server Keycloak môi trường Development cục bộ trên máy tính cá nhân để các team Frontend và Backend có một hệ thống Identity & Access Management (IAM) kết nối vào.

Trong bài Lab này, bạn sẽ sử dụng Docker để khởi chạy Keycloak cực kỳ nhanh chóng mà không cần cài đặt Java trực tiếp lên máy. Sau đó, bạn sẽ thiết lập cấu hình nền tảng đầu tiên.

## 2. Chuẩn bị Môi trường (Prerequisites)

- **Docker Desktop** (trên Windows/Mac) hoặc Docker Engine (trên Linux) đã được cài đặt và đang chạy.
- Một trình duyệt web (Chrome, Firefox, Edge).
- (Tùy chọn) Windows PowerShell hoặc Linux Bash Terminal.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1. Khởi chạy Keycloak Container

Mở Terminal của bạn và chạy lệnh sau để kéo image mới nhất của Keycloak và khởi chạy nó ở chế độ phát triển (`start-dev`):

```bash
docker run --name my-keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:latest \
  start-dev
```

**Giải thích lệnh:**
- `--name my-keycloak`: Đặt tên cho container.
- `-p 8080:8080`: Map cổng 8080 của máy tính với cổng 8080 của Keycloak.
- `-e KEYCLOAK_ADMIN=admin`: Tạo tự động tài khoản quản trị viên với username là `admin`.
- `-e KEYCLOAK_ADMIN_PASSWORD=admin`: Đặt mật khẩu quản trị viên là `admin`.
- `quay.io/keycloak/keycloak:latest`: Hình ảnh Docker chính thức của Keycloak.
- `start-dev`: Lệnh bắt Keycloak chạy ở chế độ Development (không yêu cầu HTTPS bắt buộc và dùng database H2 in-memory).

Đợi vài giây cho đến khi Terminal hiển thị dòng: `Keycloak x.x.x (Quarkus x.x.x) started in X ms.`

### Bước 3.2. Truy cập Admin Console

1. Mở trình duyệt web, truy cập địa chỉ: `http://localhost:8080`
2. Bạn sẽ thấy trang chào mừng của Keycloak. Nhấp vào nút **Administration Console**.
3. Tại trang đăng nhập, nhập tài khoản vừa cấu hình: Username: `admin`, Password: `admin`. Nhấp **Sign in**.
4. Chào mừng bạn đến với Master Realm của Keycloak!

### Bước 3.3. Tạo một Realm mới (Môi trường cách ly)

Tuyệt đối không nên lưu thông tin người dùng của ứng dụng kinh doanh vào `Master` realm. Bạn cần tạo một realm riêng biệt.

1. Ở góc trên cùng bên trái màn hình, di chuột hoặc nhấp vào chữ **master** (kế bên logo Keycloak). Một menu xổ xuống sẽ xuất hiện, nhấp vào nút **Create Realm**.
2. Trong ô **Realm name**, nhập `MyFirstRealm`.
3. Bấm **Create**. Giao diện sẽ tự động chuyển sang Realm mới tạo này.

### Bước 3.4. Tạo người dùng đầu tiên

1. Đảm bảo thanh menu bên trái đang hiển thị cấu hình của `MyFirstRealm`.
2. Nhấp vào menu **Users** ở cột bên trái.
3. Nhấp nút **Add user**.
4. Điền thông tin:
   - **Username:** `user01`
   - **Email:** `user01@example.com`
   - **First name:** `Nguyen`
   - **Last name:** `Van A`
5. Nhấp **Create**.
6. Sau khi user được tạo, chuyển sang tab **Credentials** của user đó.
7. Nhấp vào nút **Set password**.
8. Nhập mật khẩu: `password123` (cho cả 2 ô).
9. Tắt cờ **Temporary** (chuyển sang OFF). Nếu để ON, hệ thống sẽ ép user phải đổi mật khẩu ngay lần đăng nhập đầu tiên.
10. Nhấp **Save** và xác nhận bằng cách nhấp **Save password**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu
Người dùng `user01` có thể đăng nhập vào cổng thông tin quản lý tài khoản cá nhân của họ.
1. Mở một cửa sổ trình duyệt ẩn danh (Incognito/Private window).
2. Truy cập vào URL Account Console của Realm bạn vừa tạo: 
   `http://localhost:8080/realms/MyFirstRealm/account/`
3. Nhấp vào nút **Sign in**.
4. Nhập tài khoản: `user01` và mật khẩu `password123`.
5. Đăng nhập thành công! Bạn sẽ thấy trang quản lý thông tin cá nhân của `user01` nơi họ có thể tự cập nhật email, mật khẩu hoặc cấu hình bảo mật 2 lớp (2FA).

### 4.2. Troubleshooting (Khắc phục sự cố)
- **Lỗi cổng bị chiếm dụng (Port is already allocated):** Khi chạy docker, nếu báo lỗi `Bind for 0.0.0.0:8080 failed`, nghĩa là trên máy bạn đã có phần mềm khác dùng cổng 8080 (vd: Tomcat, Jenkins). **Khắc phục:** Đổi cổng mapping trong lệnh docker thành `-p 9090:8080` và truy cập qua `http://localhost:9090`.
- **Lỗi không đăng nhập được bằng admin:** Chắc chắn rằng bạn đã thêm đúng hai biến môi trường `-e KEYCLOAK_ADMIN...` ở bước chạy lệnh. Nếu quên, tài khoản admin sẽ không được tạo. Bạn phải xóa container cũ (`docker rm -f my-keycloak`) và chạy lại lệnh.
- **Trình duyệt tự động chuyển hướng sang HTTPS:** Trên môi trường Production, Keycloak bắt buộc dùng HTTPS. Nhưng ở chế độ `start-dev`, HTTP được cho phép. Nếu trình duyệt tự ép sang HTTPS (Lỗi SSL_ERROR), hãy kiểm tra tiện ích chặn quảng cáo hoặc xóa bộ nhớ cache HSTS của trình duyệt.
