# Lab 1: The Final Defense (Bảo vệ Đồ án)

> [!NOTE]
> **Category:** Lab
> **Goal:** Vận dụng tổng hợp toàn bộ kiến thức để chẩn đoán, vá lỗi bảo mật (Troubleshooting & Hardening) và bảo vệ thành công kiến trúc Enterprise IAM.

## 1. Mục tiêu (Objective)

Đồ án này đóng vai trò như một kỳ thi sát hạch cuối cùng. Hệ thống bạn nhận được không phải là một bài Lab hướng dẫn "cầm tay chỉ việc", mà là một hệ thống **Vulnerable IAM Platform** (chứa đầy các lỗi cấu hình ngầm).
Mục tiêu của bạn là:
- Đóng vai "Security Auditor", truy tìm các lỗi chết người trong file cấu hình hạ tầng và cài đặt Keycloak.
- Vá các lỗ hổng (Hardening) để đưa hệ thống đạt chuẩn Production-Ready.
- Chứng minh hệ thống đã an toàn (Verification) qua việc tấn công thử (Red Teaming).

## 2. Chuẩn bị (Prerequisites)

- Kiến thức từ Chương 1 đến Chương 57.
- Cài đặt Docker và Docker Compose.
- Công cụ Postman hoặc cURL để tạo các cuộc tấn công HTTP thô.
- Di chuyển vào thư mục bài tập: `57-Capstone-Project/code`.

## 3. Các bước thực hiện (Step-by-step Instructions)

### Bước 1: Khởi chạy "Hệ thống Lỗi" (Vulnerable System)
Hệ thống này bao gồm Nginx (Load Balancer giả lập Ingress), Keycloak, BFF (Spring Cloud Gateway) và Resource Server. Mở terminal và chạy:

```bash
docker-compose up -d
```
Hệ thống sẽ chạy ở `http://localhost`. Mọi request đều phải đi qua Nginx.

### Bước 2: Nhiệm vụ Truy tìm Lỗ hổng Mã Nguồn (Code Audit)
Bạn hãy mở file cấu hình `docker-compose.yml` và `nginx.conf`, sau đó thực hiện các bước vá lỗi sau:
1. **Lỗi Nginx (WAF Bypass):** Hacker có thể gọi thẳng vào trang quản trị của Keycloak qua đường dẫn `http://localhost/auth/admin`. 
   -> *Hành động:* Sửa file `nginx.conf`, thêm khối lệnh `location /auth/admin` để `deny all` truy cập từ Internet.
2. **Lỗi Database Secret Leak:** File `docker-compose.yml` đang hardcode biến môi trường `KC_DB_PASSWORD=admin123`. 
   -> *Hành động:* Xóa dòng đó đi. Tạo một file `.env` ẩn trong máy, và gọi biến `${DB_PASSWORD}` để Docker Compose đọc ngầm.
3. **Lỗi SSL Offloading:** Nginx đang đứng làm SSL Termination nhưng lại quên báo cho Keycloak biết. Khi bấm nút Login, Keycloak sinh ra các đường dẫn `http://` thay vì `https://`.
   -> *Hành động:* Thêm header `proxy_set_header X-Forwarded-Proto $scheme;` vào cấu hình proxy của Nginx.

### Bước 3: Nhiệm vụ Rà soát Keycloak Console (Console Audit)
Truy cập `http://localhost:8080` (trực tiếp không qua Nginx) để vào Admin Console. Rà soát và vá các lỗ hổng sau:

1. **Realm: `neobank-customer`**
   - **Lỗi Open Redirect:** Vào Client `mobile-app`. Mục *Valid Redirect URIs* đang để dấu sao `*` (Cực kỳ nguy hiểm). Kẻ gian có thể lừa Keycloak trả Authorization Code về máy chủ của chúng. 
   -> *Hành động:* Đổi thành URI chính xác `myapp://oauth2/callback`.
   - **Lỗi Brute-force:** Kẻ gian có thể dùng Tool đoán mật khẩu khách hàng thoải mái. 
   -> *Hành động:* Vào Realm Settings -> Security Defenses -> Bật Brute Force Detection. Cấu hình *Max Login Failures = 5*.

2. **Realm: `neobank-employee`**
   - **Lỗi Đặc quyền (Master Admin Risk):** Bạn đang dùng tài khoản `admin` để cấu hình. 
   -> *Hành động:* Về Realm `master`, tạo một user tên `audit_admin`. Gắn Role `admin` cho user này. Yêu cầu user này bắt buộc cài đặt OTP (Required User Actions: Configure OTP). Đăng xuất, đăng nhập lại bằng `audit_admin` + Google Authenticator. Tắt (Disable) tài khoản `admin` cũ đi.

### Bước 4: Kiểm thử Sự cố (Disaster Testing)
Bạn cần chứng minh rằng cụm Cache Infinispan đang hoạt động:
1. Đăng nhập một tài khoản vào BFF.
2. Mở Terminal, gõ `docker kill keycloak-node-1` (Mô phỏng sập Node số 1).
3. Ra trình duyệt F5 trang web. Nếu BFF không bắt bạn đăng nhập lại (Session không bị mất), xin chúc mừng! Cấu hình JGroups của bạn đã thành công.

## 4. Xác minh - The Final Defense (Verification)

Để chứng minh Đồ án của bạn bảo vệ dữ liệu ở mức Zero Trust tuyệt đối, hãy thực hiện 2 màn trình diễn sau với Ban Giám khảo (hoặc đồng nghiệp):

**Trình diễn 1: Bảo vệ Frontend (BFF Cookie)**
- Đăng nhập vào trang Web ERP của nhân viên (`http://localhost/employee`).
- Bấm F12, chuyển qua tab Network/Cookies.
- Chỉ cho Ban giám khảo thấy rằng trình duyệt CHỈ chứa Cookie `SESSION` được gài cờ `HttpOnly` và `Secure`. Không có dòng code JS nào đụng được vào Cookie này. Hoàn toàn vắng bóng JWT Bearer Token. Nguy cơ XSS bị dập tắt 100%.

**Trình diễn 2: Bảo vệ Backend (Token Audience & Issuer Validation)**
- Dùng Postman, tạo một Request đăng nhập vào Client của Khách hàng (`neobank-customer`) lấy được một chuỗi JWT.
- Lấy JWT của Khách hàng này, gài vào Header `Authorization: Bearer <jwt>`, gọi API trực tiếp vào Backend của Nhân viên (`http://localhost:8082/api/employee-data`).
- **Kết quả mong đợi:** Spring Boot Microservice ném trả lỗi `403 Forbidden` (hoặc `401 Unauthorized`).
- **Giải thích:** Mặc dù JWT có chữ ký hợp lệ (vì cùng do Keycloak sinh ra), nhưng Resource Server của Nhân viên đã kiểm tra Claim `iss` (Issuer) và phát hiện nó đến từ Realm Customer. Kẻ gian không thể lách luật!

---

**Kết luận (Conclusion):**
Nếu bạn vượt qua mọi bước trên, cấu hình được Nginx an toàn, sửa được các lỗ hổng của Keycloak, thiết lập BFF và Multi-Realm Resource Server thành công... thì bạn đã thực sự **chinh phục được Keycloak và kiến trúc Bảo mật Hiện đại**.

Chào mừng bạn bước vào hàng ngũ của những **System & Security Architect** chuyên nghiệp. Cảm ơn bạn đã đồng hành cùng giáo trình này!
