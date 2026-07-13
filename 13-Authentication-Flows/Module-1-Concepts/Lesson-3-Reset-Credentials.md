# Lesson 3: Hỗ Trợ Mất Trí Nhớ (Reset Credentials Flow)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** "Quên mật khẩu" là nút bấm được dùng nhiều thứ 2 trên thế giới sau "Đăng nhập". Khi bấm vào, Keycloak đưa khách hàng vào **Reset Credentials Flow**. Cùng xem Lãnh chúa làm cách nào cấp lại mật khẩu mà không để Hacker trục lợi.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Reset Credentials Flow Là Gì?
Khi Một Khách Hàng Nhấn Nút "Forgot Password?", Họ Sẽ Bị Ngắt Khỏi Luồng Browser Flow Và Chuyển Vào Luồng `Reset Credentials Flow`.
- Luồng Này Nhằm Mục Đích Xác Minh Danh Tính Của Người Muốn Đổi Mật Khẩu Qua Các Kênh Liên Lạc Ngoại Tuyến (Out-of-band) Như Email Hoặc SMS.
- Mặc Định Tính Năng Này Bị Tắt. Để Bật Nó, Cần Vào `Realm Settings` -> Tab `Login` -> Bật Công Tắc **`Forgot password`**.

### 1.2. Mổ Xẻ Nội Tạng Của Reset Credentials Flow Mặc Định
Luồng Reset Pass Mặc Định Gồm Các Khối Thực Thi Sau:
1. **Choose User (Required):** Buộc Khách Nhập Username Hoặc Email Để Database Có Thể Tìm Ra Tài Khoản Cần Đổi Pass.
2. **Send Reset Email (Required):** Máy Chủ Sinh Ra 1 Mã Action Token Dùng Một Lần Và Bắn Email Chứa Link Phục Hồi Vào Hộp Thư Của Người Dùng. Trình Duyệt Bị Chặn Và Chờ Đợi.
3. **Reset Password (Required):** Khi Khách Bấm Link Trong Email, Họ Trở Lại Keycloak Và Được Vẽ Ra Form Yêu Cầu Nhập Mật Khẩu Mới.
4. **Reset OTP (Conditional):** Nếu Người Dùng Có Cài Đặt 2FA (Ví Dụ Google Authenticator), Khối Này Sẽ Bật Lên Bắt Yêu Cầu Nhập OTP Để Hoàn Tất Quá Trình Đổi Pass. Nếu Bỏ Qua, Kẻ Cắp Email Sẽ Chiếm Luôn Tài Khoản.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình Reset Credentials Flow:

```mermaid
flowchart TD
    A[Khách Bấm Forgot Password] --> B[Keycloak Chuyển Sang Luồng Reset]
    B --> C{Execution 1: Choose User}
    C --> D[Hiển Thị Form Nhập Email/Username]
    D --> E[Tìm Thấy Khách Hàng]
    
    E --> F{Execution 2: Send Reset Email}
    F --> G[Gửi Token Hồi Sinh Vô Hộp Thư Khách]
    
    G --> H((Tạm Dừng Luồng Chờ Check Mail))
    
    H -- Bấm Link Trong Email --> I{Execution 3: Reset Password}
    I --> J[Hiện Màn Hình Nhập Pass Mới]
    J --> K[Lưu Mật Khẩu Tạm Thời]
    
    K --> L{Execution 4: Khách Đã Cài OTP Chưa? (Conditional)}
    L -- Đã Cài --> M[Đòi Quét Mã OTP / Nhập Mã 6 Số]
    L -- Chưa Cài --> N[Hoàn Thành Đổi Mật Khẩu]
    M --> N
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Bảo Mật (Luôn Giữ Chặt Cục Reset OTP Cho Quá Trình Đổi Mật Khẩu)**
> **Tội Ác Thiết Kế:** Vô Hiệu Hóa Bước Reset OTP Trong Reset Credentials Flow Đi Để Đơn Giản Hóa Quá Trình Lấy Lại Mật Khẩu.
> **Hậu Quả:** Nếu Hacker Đánh Cắp Được Tài Khoản Gmail Của Khách Hàng, Hắn Chỉ Cần Bấm Quên Mật Khẩu Và Bấm Link Xác Nhận. Nếu Không Có Khối Xác Minh OTP Đứng Sau Bảo Vệ, Hacker Sẽ Dễ Dàng Chiếm Luôn Tài Khoản Chứa MFA Của Người Dùng Đang Bị Lộ Email Này.
> **Biện Pháp Sống Còn:** LUÔN ĐỂ `Reset OTP` Ở Trạng Thái **`CONDITIONAL`** Hoặc **`REQUIRED`**. Đảm Bảo Bypass Mật Khẩu Luôn Phải Trải Qua Bước Bypass Của Đa Yếu Tố (MFA).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Luồng Reset Tùy Chỉnh (Thêm Yêu Cầu Cảnh Báo Khác Nếu Cần):
1. Đứng Ở Admin Console -> `Authentication` -> `Flows`.
2. Duplicate Dòng `reset credentials` Và Đặt Tên Mới, Ví Dụ `My-Reset-Secure`.
3. Chỉnh Sửa Luồng Bằng Cách Bấm Nút **Add execution** Nếu Bạn Đã Có Các Plugin Bổ Sung (Như Gửi Mã Xác Minh Bằng SMS). Thêm Nó Vào Luồng.
4. Đổi Trạng Thái Thành `Required` Nếu Thật Sự Cần Thiết Ép Mọi Trường Hợp.
5. Vô `Realm Settings` -> Tab `Themes` -> Mục `Reset Credentials Flow`, Chọn Sang Luồng `My-Reset-Secure` Bạn Vừa Tạo.
6. Khi Người Dùng Yêu Cầu Đổi Mật Khẩu, Hệ Thống Sẽ Áp Dụng Luồng Thực Thi Mới Theo Bảo Mật Công Ty Của Bạn Thay Cho Mặc Định Của Hãng.

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Nếu `Reset OTP` Đang Bật `Conditional`. Giả Sử Hacker Cướp Được Email Của Một Tài Khoản Vừa Mới Lập (Chưa Kịp Đăng Ký MFA/OTP Bao Giờ). Hắn Có Thể Đổi Mật Khẩu Và Chiếm Luôn Tài Khoản Đó Không? Lỗ Hổng Đây Là Gì?**
- **Senior:** Hoàn Toàn Có Thể Chiếm Được! Vì Tính Năng Đang Bật Trạng Thái "Conditional" (Tức Là Có Thì Bắt Quét, Chưa Có OTP Thì Bỏ Qua). Do Tài Khoản Chưa Từng Cài OTP, Máy Chủ Keycloak Sẽ Bỏ Qua Bước Này Và Cho Phép Đổi Mật Khẩu Thành Công Chỉ Dựa Vào Email.
- **Cách Chống Lại:** Để Chống Lỗ Hổng 100%, Thay Vì Đặt Conditional, Hãy Đổi Lại Khối `Reset OTP` Thành **`REQUIRED`**. Khi Đó, Ngay Cả Khi Khách Hàng Chưa Có OTP, Keycloak Sẽ Dừng Quá Trình Lại Và Buộc Khách Hàng Phải Cài Đặt Trọn Vẹn 1 Bộ OTP Ngay Trước Khi Lệnh Đổi Mật Khẩu Bắt Đầu Hiệu Lực. Hacker Sẽ Bị Kẹt Ở Bước "Thiết Lập Mã QR Mới" Do Thiếu Thiết Bị Cá Nhân Của Khách.

---

## 6. Tài liệu tham khảo (References)
- **NIST 800-63B:** Digital Identity Guidelines (Authentication and Lifecycle Management).
