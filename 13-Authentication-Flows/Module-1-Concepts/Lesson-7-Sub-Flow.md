# Lesson 7: Hộp Đen Lồng Hộp Đen (Sub-Flow)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Khi tạo những kịch bản xác thực phức tạp (vừa đòi WebAuthn, vừa đòi OTP nếu WebAuthn xịt, vừa kết hợp kiểm tra IP mạng), màn hình quản lý Flow của bạn sẽ rối tung như bãi mìn. Để thiết kế gọn gàng và đóng gói logic, Keycloak mang đến khái niệm **Sub-flow (Luồng Phụ)**. Cùng xem cách nhét "Hộp đen lồng Hộp đen" để bẻ khóa mọi bài toán.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Sub-Flow Là Gì?
Luồng Phụ (Sub-Flow) Thực Chất Là Một Khối Flow Con Được Nằm Gọn Bên Trong Lõi Của Một Flow Mẹ (Ví Dụ Browser Flow).
- Giống Chức Năng Group (Nhóm Lại) Của Phần Mềm Đồ Họa Photoshop. Bạn Kéo Nhiều Execution (Khối Chức Năng Rời Rạc) Nhét Cùng Vào Một Sub-Flow Của Nó.
- Bạn Có Quyền Chỉ Cần Thao Tác Trạng Thái (`Required`, `Alternative`, `Conditional`) Lên Chính Lớp Vỏ Sub-Flow Mẹ Đo Đó Thay Vì Chạy Lòng Vòng Sửa Từng Cái Execution Bên Trong Nó.

### 1.2. Phân Loại Type Của Lớp Vỏ Sub-Flow
1 Cái Vỏ Sub-Flow Cung Cấp 3 Lựa Chọn Toán Học Để Dẫn Dắt Dòng API Json Đi Vào Bên Trong:
1. **Generic (Cũ Basic):** Chạy Theo Trục Dọc Tuyến Tính Bình Thường. Các Execution Nằm Bên Trong Sub-flow Này Cứ Thứ Tự Từ Trên Xuống Dưới Mà Vượt Qua Thử Thách Cảnh Sát Giao Thông.
2. **Form Flow (Browser Forms):** Một Lớp Vỏ Cực Kỳ Đặc Biệt Chuyên Trị Dùng Để Gói Những Cái UI Vẽ Tương Tác Của Trình Duyệt Bắt Password Với Người Dùng, Cấp Các Giao Diện Chứa Form. Trình Duyệt Browser Của Browser Flow Bắt Buộc Có Khối Vỏ Bọc Hình Form Này Để Rendering Dữ Liệu Lên DOM.
3. **Client Flow:** Dùng Khối Vỏ Riêng Nhằm Tính Toán Token Để Giải Quyết Bảo Mật Máy Chủ. Không Dùng Cho User.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Dòng Cục Json Qua Mô Hình Các Hộp Đen Gói Sub-Flow:

```mermaid
flowchart TD
    A[Luồng Xác Thực Vỏ Mẹ 'Browser-Flow' Bắt Khách Mở Màn Hình Browser] --> B{Execution: Cookie}
    B -- Pass Thành Công Do Có SSO --> C[Nhả Token Mượt, End Luồng]
    B -- Fail Không Thấy Cookie Đâu Cả --> D[Luồng Rơi Xuống Dòng Cảnh Sát Dưới Tương Lai]
    
    D --> E[Trượt Vào Khối Hộp Vỏ Mẹ: 'Browser Forms' (Sub-Flow)]
    E --> F{Execution Trong Vỏ: Password Form}
    F -- Nhập Đúng Pass --> G[Trượt Tiếp Vào Vỏ Sub-Flow Khác Bọc Bên Trong Nhánh]
    
    G --> H[Vỏ 'MFA-Checks' (Sub-Flow Gắn Lệnh Alternative Gói Rất Cẩn Thận Mảng Conditional)]
    H --> I{Execution Vỏ MFA: WebAuthn Check Passwordless Mắc Tiền}
    I -- Chạy Fail Do Khách Lười Cắm USB Vào --> J{Execution Vỏ MFA Cứu Bồ: OTP Form (Alternative Đón Lõng Ở Dưới Khối Cùng Chung Vỏ Nhau)}
    
    J -- Quét OTP Pass Thành Công Giao Ước Trong Vỏ Gói Hộp Đen -> Trả Tín Hiệu TRUE Lên Mặt Vỏ 'MFA-Checks' --> C
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh An Toàn Mạng Bọc (Luôn Gói Các Cơ Chế Cứu Cánh Dự Phòng Backup MFA Vào Chung 1 Cục Vỏ Sub-flow Thống Nhất!)**
> **Tội Ác Thiết Kế Thẳng Đuột Không Hộp Hóa:** Bạn Cố Cài Đặt Passkeys Sinh Trắc Học Tương Lai (WebAuthn Passwordless) Trộn Chung Cùng Tầng Đáy Trực Diện Với OTP Điện Thoại. Nghĩa Là Nó Nằm Rải Rác Ở Mặt Phẳng Không Gian Luồng Của Browser Flow. Bạn Setup Cục Passkeys: `Alternative`. Xong Gắn OTP Dưới Đáy Cũng: `Alternative`.
> **Hậu Quả:** Engine Của Keycloak Xét Lệnh Random Lộ Liễu Tới Mức: Thay Vì Đẩy UI Của Passkey Chạm Vân Tay Xịn Xò Lên Cho Khách Trải Nghiệm Mượt, Nó Random Hiển Thị Quả Màn Hình Hỏi Mã Chữ Số OTP 6 Số Ra Cho Thằng Chạm Tay Passkey Nhìn Lên Bức Xúc "Máy Hỏng Vân Tay À Bắt Nhập Số Thủ Công Ngứa Mắt!".
> **Biện Pháp Sống Còn Nhét Vào Hộp Đen:** LUÔN Tạo Một Sub-Flow Mang Tên: `MFA-Alternative-Wrapper`. Chỉnh Cái Subflow Này Cấp Độ `Required` Lên Flow Mẹ. Nhét Cục Execution Passkey Vân Tay Và Cục OTP Vô Cùng 1 Vỏ Đó, Đẩy Trạng Thái Tất Cả Tụi Trong Vỏ Đó Thành Cùng Cờ `Alternative`. Keycloak Khi Đụng Phải Cái Vỏ Kén Hộp Đen Wrapper Này, Nó Sẽ Thông Minh Ưu Tiên Ném Thử Vân Tay Chạm Ra Trước, Nếu Khách Đè Cái Chạm Vân Tay Xóa Thì Nó Lấy List Alternative Nằm Chung Vỏ Ra Thả Lên Cứu Cánh Thay Thế Backup Liền Tay Nhẹ Nhàng Bằng Nhập Mã Số 6 Chữ Cũ!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Hệ Thống MFA Nhựa Bọc Kén Sub-flow Lọc Ưu Tiên:
1. Bạn Tạo Một Flow Nhân Bản Mới Hoàn Toàn Tên `My-Browser-Multi-MFA` Kế Thừa Từ Browser Flow Cũ Của Hãng.
2. Tại Dòng Khối Cuối Nhất Đang Chứa Password Thường Cũ, Ấn Nhẹ Khung: `Add Sub-flow`. Đặt Tên: `MFA-Secure-Wrapper`.
3. Để `MFA-Secure-Wrapper` Này Là Lệnh **`Required`** Hoặc **`Conditional`** Tuỳ Doanh Nghiệp Setup Mở Rộng Bắt Buộc Mọi Dân Cày Quét QR MFA Hay Tắt!
4. Ấn Vô Nút `Add execution` Bên Trong Ruột Của Cái `MFA-Secure-Wrapper`.
5. Kiếm Và Chọn Tên Module Execution: **`WebAuthn Passwordless Authenticator`** (Sinh Trắc Học).
6. Tương Tự Tiếp Nhấn Nút Add Ruột Subflow: Chọn Thêm Execution Lấy Tên Là: **`OTP Form`** (Mã OTP Truyền Thống Số Học).
7. Điều Chỉnh Hai Khối Vừa Lọt Vào Ruột Của Nhau Thành Cùng Trạng Thái Khớp Toán Bool Tên Lệnh Là **`Alternative`** (Để Giành Thay Thế Nhau Giữa Hai Sự Lựa Chọn).
8. Nếu Bạn Kéo Luồng Nhựa Của WebAuthn Trượt Nằm Lên Dòng Trực Tràng Phía Trên Dòng Trọng Lượng OTP. Máy Keycloak Sẽ Tự Bật Kính Hiển Vi So Chiếu Ưu Tiên Màn Hình Chạm Sinh Trắc WebAuthn. Khách Nhấn Thử Backup Trả Về Thì Nó Thụt Xuống Dòng Dưới Lấy Cục OTP Lên Cover Lỗ Hổng Hỏng Vân Tay Ảo Vượt Mặt Bot Nhựa.

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Flow Browser, Bạn Chèn Thêm Khối Execution Form Password Đứng Lẻ Loi Trơ Trọi Ở Ngoài Cùng Mặt Phẳng Dưới Cùng Của Khung Logic Trục Chính Của Trình Duyệt. Khi Khởi Động Đăng Nhập Lại Không Vẽ Form Mà Văng Thông Báo Báo Lỗi Trắng Trang HTTP 500 Chạm Lõi Dòng Code Code NullPointerException Từ Form Browser. Lý Do Là Gì Khi Khối Này Được Keycloak Phân Quyền Hợp Cấu Trúc Khung Rỗng?**
- **Senior:** Lỗi Nằm Ở Vỏ Bọc Rendering Forms Browser OIDC Trực Tiếp! Khối Execution `Password Form` Có Chứa Thuộc Tính HTML Tương Tác. Keycloak Có Tính Năng Cứng Nhắc Trong Core Đáy Java, Nó KHÔNG CHO PHÉP Vẽ Trực Tiếp Bất Cứ Form Tương Tác Màn Hình HTML Nhập Dữ Liệu Nào Nếu Khối Lõi Thực Thi Đó Không Được Gói Vô Trọng Một Khối Sub-Flow Bọc Kén Chứa Type Tên Mang Thuộc Tính Mang Nhãn **`Form Flow`** (Hoặc Tên Gốc Default `Browser Forms` Đáy Kẽ Lớn Nguồn Cấp).
Bạn Vứt Trơ Trọi Execution Đòi Nhập Form Ra Tầng Mẹ Nhất Type Generic Chứa Toán Trực Tiếp Không Hiểu Engine Form OIDC Khiến Bắn HTTP Lỗi Tự Hủy Token Rendering Cấp Tốc Lõi Máy Chủ!

---

## 6. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Server Administration Guide - Sub Flows.
