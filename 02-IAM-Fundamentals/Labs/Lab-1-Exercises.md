> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Làm quen với các khái niệm IAM cơ bản thông qua việc tạo Realm, Client, User và thực hiện luồng lấy Token (OAuth 2.0 Authorization Code Flow) qua Postman.

## 1. Kịch bản Thực hành (Lab Scenario)

Giả sử bạn đang phát triển một ứng dụng nội bộ có tên là `Employee Portal`. Bạn cần tích hợp ứng dụng này với Keycloak để quản lý việc đăng nhập của nhân viên. Yêu cầu đặt ra là bạn phải tạo một vùng không gian quản lý độc lập (Realm) cho công ty, tạo một Client đại diện cho ứng dụng, tạo một nhân viên thử nghiệm và cuối cùng là đóng vai người dùng để lấy được Access Token thành công bằng quy trình chuẩn Authorization Code Flow.

Bài lab này là bước đệm quan trọng để hiểu rõ toàn bộ vòng đời tương tác của một ứng dụng (Client) đối với hệ thống IAM (Keycloak).

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một máy chủ Keycloak đang chạy (có thể chạy trên Docker tại `http://localhost:8080`).
- Trình duyệt web (Chrome, Firefox, Edge).
- Phần mềm **Postman** (phiên bản Web hoặc Desktop) để thực thi các HTTP Requests lấy Token.
- Đã có tài khoản quản trị tối cao (Admin) của Keycloak.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo Realm mới
1. Đăng nhập vào Keycloak Admin Console (`http://localhost:8080/admin`).
2. Ở menu dropdown góc trên bên trái (đang hiển thị chữ `master`), nhấp vào đó và chọn **Create Realm**.
3. Điền `Realm name`: `Company-Corp`.
4. Nhấn **Create**. 

### Bước 3.2: Tạo User (Nhân viên)
1. Đảm bảo bạn đang ở Realm `Company-Corp`. Ở menu bên trái, chọn **Users**.
2. Nhấn nút **Add user**.
3. Điền `Username`: `alice`.
4. Điền `First name`: `Alice`, `Last name`: `Smith`.
5. Nhấn **Create**.
6. Sau khi tạo, chuyển sang tab **Credentials**.
7. Nhấn **Set password**. Điền mật khẩu (ví dụ: `password123`).
8. Tắt nút gạt **Temporary** (để Keycloak không bắt Alice đổi mật khẩu ở lần đăng nhập đầu tiên).
9. Nhấn **Save** và xác nhận bằng nút **Save password**.

### Bước 3.3: Tạo Client (Ứng dụng)
1. Ở menu bên trái, chọn **Clients**. Nhấn **Create client**.
2. **General Settings:** 
   - `Client type`: `OpenID Connect`
   - `Client ID`: `employee-portal-client`
   - Nhấn **Next**.
3. **Capability config:**
   - Kích hoạt **Client authentication** (Bật ON) -> Hành động này sẽ yêu cầu ứng dụng phải có Client Secret.
   - Standard flow: Đảm bảo đã Bật (ON).
   - Nhấn **Next**.
4. **Login settings:**
   - `Valid redirect URIs`: Nhập `https://oauth.pstmn.io/v1/callback` (Đây là URL callback của Postman để nhận Authorization Code).
   - Nhấn **Save**.
5. Sau khi tạo xong, chuyển sang tab **Credentials**. Sao chép (Copy) giá trị trong ô **Client secret** để dùng cho bước sau.

### Bước 3.4: Cấu hình Postman để lấy Token
1. Mở Postman, tạo một Request mới. Chuyển sang tab **Authorization**.
2. Chọn Type: `OAuth 2.0`.
3. Trong phần **Configure New Token**, điền các thông tin sau:
   - `Token Name`: `My Keycloak Token`
   - `Grant Type`: `Authorization Code`
   - `Callback URL`: Check vào ô `Authorize using browser` (hoặc để mặc định nếu dùng Postman Desktop). Nếu dùng URL tĩnh, nhập đúng giá trị `https://oauth.pstmn.io/v1/callback`.
   - `Auth URL`: `http://localhost:8080/realms/Company-Corp/protocol/openid-connect/auth`
   - `Access Token URL`: `http://localhost:8080/realms/Company-Corp/protocol/openid-connect/token`
   - `Client ID`: `employee-portal-client`
   - `Client Secret`: Nhập Client secret đã copy ở bước 3.3.
   - `Scope`: `openid profile email`
4. Cuộn xuống, nhấn nút **Get New Access Token**.

### Bước 3.5: Đăng nhập giả lập
1. Sau khi nhấn "Get New Access Token", Postman sẽ bật lên một trình duyệt nội bộ hiển thị trang đăng nhập của Keycloak.
2. Nhập `Username`: `alice` và `Password`: `password123`.
3. Nhấn **Sign In**.
4. Postman sẽ thông báo Authentication Complete và hiển thị cấu trúc của Access Token nhận được.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu (Verification):**
1. Copy chuỗi Access Token dài từ Postman.
2. Truy cập trang web giải mã JWT: [jwt.io](https://jwt.io).
3. Dán chuỗi Token vào khung Encoded.
4. Nhìn sang phần **Payload (Data)**. Bạn sẽ phải thấy các trường như:
   - `"iss"`: `http://localhost:8080/realms/Company-Corp` (Nguồn phát hành token)
   - `"azp"`: `employee-portal-client` (Bên ủy quyền/ứng dụng)
   - `"preferred_username"`: `alice` (Người dùng đích)
   Nếu các thông tin này khớp, bạn đã cấu hình chuẩn xác luồng xác thực IAM.

**Các lỗi thường gặp (Troubleshooting):**
- **Lỗi Invalid Redirect URI:** Nếu Keycloak báo lỗi màn hình đỏ "Invalid redirect uri" khi Postman mở trình duyệt, hãy kiểm tra lại cấu hình Client trong Keycloak xem đã nhập chính xác URI callback của Postman hay chưa.
- **Lỗi Client Secret không đúng:** Nếu trang web báo đăng nhập thành công nhưng Postman lại báo "Could not complete OAuth 2.0 login", rất có thể bước Postman dùng Authorization Code để gọi Token Endpoint bị thất bại do sai Client ID hoặc Client Secret. Hãy copy lại Client Secret từ Keycloak.
- **Lỗi User không đăng nhập được:** Đảm bảo rằng bạn đã tắt thuộc tính Temporary lúc đặt mật khẩu, nếu không người dùng bị mắc kẹt ở màn hình yêu cầu đổi mật khẩu.
