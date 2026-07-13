# Lesson 1: Cửa Ngõ Chính (Browser Flow)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Khi một Khách hàng của công ty mở trình duyệt lên và bấm "Đăng nhập", tại sao họ lại thấy màn hình nhập Password? Đó là do Keycloak đang chạy một kịch bản có tên là **Browser Flow**. Đây là luồng xác thực xương sống, chịu trách nhiệm cho 90% lượng traffic của bất kỳ hệ thống SSO nào.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Browser Flow Là Gì?
Trong Hệ Trục Tọa Độ Của OIDC, Browser Flow (Luồng Trình Duyệt) Chính Là "Bản Đồ" Quy Định Xem Khách Hàng Sẽ Bị Keycloak Lôi Đi Qua Những Cửa Ải Nào Trước Khi Được Nhả Access Token.
- Nó Là Một Khối Chứa Các `Execution` (Khối Thực Thi) Được Xếp Theo Thứ Tự Tuyến Tính (Từ Trên Xuống Dưới).
- Bất Kỳ Ứng Dụng Nào Gọi Redirect Lên Keycloak Bằng Trình Duyệt, Mặc Định Máy Chủ Keycloak Sẽ Lấy Luồng `Browser` Này Ra Để Xử Lý Khách Hàng.

### 1.2. Mổ Xẻ Nội Tạng Của Browser Flow Mặc Định
Luồng Có Tên `browser` Bao Gồm Các Bước Logic Sau (Thứ Tự Chạy Từ Dưới Lên Trên Nếu Cùng Bậc):
1. **Cookie (Alternative):** Ngay Bước Đầu Tiên, Keycloak Sẽ Dò Xem Trình Duyệt Của Khách Hàng Có Cầm Theo Cookie `KEYCLOAK_IDENTITY` Hay Không. Nếu Có (Và Còn Hạn) -> Lập Tức **Pass Ngay Cửa Ải Này**, Bỏ Qua Hết Các Form Nhập Pass Đằng Sau (Đây Gọi Là SSO Session). Nếu Không Có Cookie -> Rớt Xuống Nhánh Khác.
2. **Identity Provider Redirector (Alternative):** Nếu Bật Tính Năng Default Identity Provider (Ví Dụ Google Login), Khách Sẽ Bị Đá Thẳng Sang Máy Chủ Của Google. Nếu Không Bật -> Rớt Xuống Khối Cuối Cùng.
3. **Browser Forms (Alternative):** Đây Là Khối Lớn Nhất, Chứa Các Luồng Con Để Hiện Form Đăng Nhập:
    - **Username Password Form (Required):** Bắn Trả Về Giao Diện HTML Chứa Ô Nhập ID Và Mật Khẩu. Bắt Khách Phải Điền Đầy Đủ Để Đăng Nhập.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình Qua Default Browser Flow:

```mermaid
flowchart TD
    A[Khách Login App] --> B[Redirect Tới Keycloak]
    B --> C{Execution 1: Có Cookie KEYCLOAK_IDENTITY Chưa?}
    C -- Có --> D[Bỏ Qua Hết! Cấp Luôn Code/Token]
    C -- Không --> E{Execution 2: Có Bật Default Identity Provider Không?}
    E -- Có --> F[Đá Khách Qua Google/Facebook]
    E -- Không --> G[Execution 3: Sub-flow Browser Forms]
    
    G --> H{Execution 3.1: Username Password Form}
    H --> I[Hiện Giao Diện Nhập Pass]
    I --> J{Mật Khẩu Đúng Không?}
    J -- Đúng --> K[Kiểm Tra Tầng Required Actions (OTP/T&C)]
    K --> D
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Mạng Bọc (Không Bao Giờ Chỉnh Sửa Trực Tiếp Browser Flow Gốc)**
> **Tội Ác Thiết Kế:** Engine Của Keycloak Cố Định Luồng `browser` Cho Các Trình Duyệt. Khối Này Không Cho Xóa. Nếu Dev Sửa Trực Tiếp Và Làm Hỏng Logic (Ví Dụ Disable Form Username Password), Keycloak Sẽ Bắn Lỗi `500 Internal Server Error` Cho TẤT CẢ Khách Hàng, Thậm Chí Cả `admin` Cũng Không Thể Login Vào Lại.
> **Biện Pháp Sống Còn:** LUÔN SỬ DỤNG Nút **`Duplicate`** (Nhân Bản). Khi Cần Chỉnh Sửa, Hãy Nhân Bản Luồng Gốc, Chỉnh Sửa Trên Bản Copy, Rồi Dùng Tính Năng **`Bind Flow`** Để Gán Bản Copy Thành Luồng Mặc Định Cho Browser. Nếu Có Lỗi, Dễ Dàng Bind Ngược Lại Luồng Gốc.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Ánh Xạ Luồng Tùy Chỉnh (Bind Browser Flow):
1. Tại Admin Console -> Menu `Authentication` -> Tab `Flows`.
2. Tại Dòng `browser`, Bấm Nút **Duplicate**. Đặt Tên Là `My-Custom-Browser`.
3. Chỉnh Sửa Logic Của `My-Custom-Browser` Theo Nhu Cầu.
4. Bấm Vào Nút 3 Chấm Của Luồng `My-Custom-Browser` -> Chọn **`Bind Flow`**.
5. Bảng Chọn Hiện Ra, Chọn **`Browser Flow`**.
6. Lúc Này Trong Danh Sách, Luồng Tùy Chỉnh Sẽ Hiển Thị Nhãn Used By `browser`. Mọi Traffic Từ Trình Duyệt Sẽ Chạy Vào Luồng Này.

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Cậu Dev Sắp Xếp Luồng Đăng Nhập, Đặt `Username Password Form` Lên Trên Luồng Kiểm Tra `Cookie` (Cả 2 Đều Alternative). Chuyện Gì Xảy Ra?**
- **Senior:** Phá Hoại Thiết Kế Của Cỗ Máy Flow! Flow Chạy Theo Thứ Tự Tuyến Tính. Nếu Form Nhập Pass Đứng Trên, Nó Sẽ Chặn Khách Lại Và Bắt Nhập Mật Khẩu Ngay Lập Tức, Cho Dù Khách Có Cookie Hợp Lệ SSO Session. Tính Năng Single Sign-On Sẽ Hoàn Toàn Bị Vô Hiệu Hóa, Bắt Người Dùng Đăng Nhập Mọi Lúc Mọi Nơi. Vì Vậy, `Cookie` LUÔN Phải Được Đặt Lên Đầu Trong Browser Flow.

---

## 6. Tài liệu tham khảo (References)
- **OIDC Specifications:** Authorization Endpoint (Browser flow equivalent).
