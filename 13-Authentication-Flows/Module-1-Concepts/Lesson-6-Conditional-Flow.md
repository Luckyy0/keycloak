# Lesson 6: Trí Tuệ Nhân Tạo Rẽ Nhánh (Conditional Flow)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trong thế giới thực, không phải khách hàng nào cũng cần đối xử giống nhau. Ví dụ: Sếp của bạn đang truy cập từ văn phòng công ty thì được login thẳng; nhưng nếu sếp cầm laptop ra tiệm cafe Wifi công cộng thì phải bắt quét OTP. Để biến Keycloak thành một cỗ máy thông minh biết suy nghĩ theo ngữ cảnh (Context-Aware), ta dùng **Conditional Flow**.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Conditional Flow Là Gì?
Conditional Flow (Luồng Có Điều Kiện) Là Một Khối Block Logic `If...Else` Hoạt Động Trong Ruột Của Các Luồng Xác Thực Lớn Hơn (Browser Flow Chẳng Hạn).
- Thay Vì Chỉ Có 3 Lệnh Chết Cứng `Required` (Bắt Buộc), `Alternative` (Hoặc), `Disabled` (Tắt). Ta Có Trạng Thái Thứ 4: **`Conditional`**. 
- Khi Một Sub-Flow (Luồng Phụ) Hoặc Execution Được Bật Chế Độ `Conditional`, Nó Sẽ Không Tự Động Chạy Mù Quáng Nữa Mà Sẽ Phải Hỏi Ý Kiến Của Các **Condition (Điều Kiện)** Đứng Cạnh Nó Đầu Tiên.

### 1.2. Condition (Điều kiện) Hoạt Động Thế Nào?
Condition Là Những Cục Execution Đóng Vai Trò Giám Khảo (Cảnh Sát Giao Thông). Chúng Được Keycloak Cho Sẵn Những Logic Kiểm Tra Thông Minh:
- **Condition - User Role:** Cảnh Sát Hỏi "Anh Có Role là Admin Không? Nếu Có Mới Cho Đi Vào Khối OTP Này Để Xác Nhận."
- **Condition - Network IP:** Cảnh Sát Hỏi "IP Của Thiết Bị Này Có Thuộc Về Mạng Của Trụ Sở Công Ty (Ví Dụ `192.168.1.x`) Hay Không? Nếu Có Thì Bypass Cho Đi Vào, Không Phải Mở App Điện Thoại Nữa."
- **Condition - Configured OTP:** Cảnh Sát Hỏi "Người Này Trong Database Đã Bật OTP Lần Nào Trong Quá Khứ Chưa? Đã Từng Quét Mã QR App Nào Chưa? Nếu Có Thì Bắt Nhập 6 Số OTP Nhé, Chưa Có Thì Thôi Cho Đi Ngang Bỏ Qua Luôn."

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Dòng Cục Json Qua Một Sub-Flow Gắn Conditional:

```mermaid
flowchart TD
    A[Khách Vừa Đăng Nhập Xong Bằng Mật Khẩu Thành Công] --> B[Chui Vào Sub-Flow 'Security-OTP-Check' (Conditional)]
    
    B --> C{Condition 1: Khách Này Có Role Admin?}
    C -- False --> D[Bỏ Qua Toàn Bộ Sub-flow Chứa OTP]
    
    C -- True --> E{Condition 2: IP Này Là Wifi Ngoài Quán Cafe Đứng Khác Dải IP Trụ Sở?}
    E -- False (Nội Bộ) --> D
    E -- True (Mạng Ngoài) --> F[Kích Hoạt Luồng: Pass Thành Công Rồi]
    
    F --> G{Execution: OTP Form}
    G --> H[Bắt Sếp Mở App Google Authenticator Ra Để Quét Và Nhập 6 Số]
    
    D --> I[Hoàn Thành Browser Flow -> Nhả Token Mượt Mà]
    H --> I
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh An Toàn (Luôn Cặp Đôi Condition Cùng Với Execution)**
> **Tội Ác Thiết Kế:** Tạo 1 Sub-Flow Chỉnh Nó Thành Conditional. Nhét 1 Cục OTP Form Vào Cho Nó Tình Trạng Required. NHƯNG Lại QUÊN Không Chèn Khối Lệnh Điều Kiện Nào Cả (Quên Để Khối Cảnh Sát Vào Cùng).
> **Hậu Quả:** Do Chẳng Có Điều Kiện Nào Ngăn Cản, Khối Sub-Flow Bị Trở Thành Mù Khối Khác Gì Required. Lệnh Của Bạn Đã Biến Thành Trò Hề Bắt Cả Công Ty Dù Đứng Nội Bộ Hay Trong Chăn Đều Bị Quét OTP! Gây Quá Tải Server Và Gây Bực Dọc Trải Nghiệm Lớn.
> **Biện Pháp Sống Còn:** 1 Sub-flow Gắn Cờ `Conditional` Bắt Buộc Phải Chứa Ít Nhất 1 Khối Condition Đi Kèm! Khối Condition Phải Được Thiết Lập Mức Độ Là `Required`! Còn Các Khối Execution Hành Động Yêu Cầu (Ví Dụ Form OTP) Sẽ Được Xếp DƯỚI Khối Điều Kiện Đó Và Gắn Lệnh Cứng `Required` Luôn.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Logic Bắt Buộc Admin Phải Quét OTP, Còn Dân Thường (User) Chỉ Cần Mật Khẩu Trong Browser Flow:
1. Vào Flow `browser` (Hoặc Luồng Clone Của Browser Bạn Đang Dùng `My-Browser-Flow`).
2. Cuộn Xuống Dưới Cùng Của Cây Cấu Trúc, Nhấn `Add Sub-flow`. Đặt Tên: `Admin-OTP-Branch`.
3. Chỉnh Loại Flow Lại Của Cái Nhánh Vừa Tạo Từ Trạng Thái Required Sang **`Conditional`**.
4. Chọn Trạng Thái Type Của Cái Branch Đó Trong Option: Type Basic Flow Mặc Định Rất Ok. Bấm Nút **Save**.
5. Nhấn `Add execution` VÀO TRONG cái nhánh Sub-flow Vừa Tạo. Kiếm Khối Có Chữ **`Condition - User Role`**. Ấn Chọn.
6. Khi Vừa Thêm Khối Điều Kiện Vào, Đừng Quên Click Nút Răng Cưa Settings. Điền Alias Role: Bấm Chọn Alias `admin` Để Máy So Khớp.
7. Đổi Trạng Thái Lệnh Của Cái Cục `Condition - User Role` Trong Bảng Thành **`Required`**.
8. Nhấn `Add execution` Thêm 1 Lần Nữa VÀO TRONG nhánh `Admin-OTP-Branch`. Chọn Khối **`OTP Form`**. 
9. Đổi Lệnh Khối Đó Sang **`Required`**. Cấu Trúc Bây Giờ Nhìn Trực Quan Là: Conditional(Role-Admin) -> Required(Form-OTP).
10. Đăng Nhập Bằng User Có Role Admin Khác Hẳn Trải Nghiệm Tự Động Phân Nhánh Nhảy Tới OTP Khóa Chặt. Còn Acc Khách Cùi Bắp Đăng Nhập Form Cũ Mượt Mà Đi Xuyên Qua Khối Này Một Cách Ảo Diệu!

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Cùng 1 Cái Sub-Flow Lớp Vỏ Có Lệnh `Conditional`. Cậu Chèn Vào Trong Lõi Của Nó 2 Khối Điều Kiện: Cục 1 `Condition - IP Address` (Bắt Phải Từ IP Ngoài Mạng) Đặt Cờ REQUIRED. Cục 2 `Condition - User Role` (Bắt Khách Có Role Vip) Cậu Đặt Cờ Lại Là ALTERNATIVE. Đằng Dưới Đáy Của Tụi Nó Có Khối `OTP Form` Lệnh REQUIRED. Logic Chạy Ở Đây Sẽ Lọc Khách Ra Sao?**
- **Junior:** Bọn Cảnh Sát Cãi Nhau Rối Loạn Mất! Em Sợ Quá Em Chỉ Cài Chơi 1 Condition Chứ Trộn Required Dùng Kèm Alternative Này Dễ Toang Chóp Mất Anh Ơi.
- **Senior:** Keycloak Tính Toán Logic Rất Thông Minh Nhờ Học Toán Bool Rời Rạc! 
  - Khối Điều Kiện Gắn `Required` Đóng Vai Trò Là Phép Tính `AND`.
  - Khối Điều Kiện Gắn `Alternative` Đóng Vai Trò Là Phép Tính `OR`.
  - Nếu Cậu Code Cấu Hình Vậy: `(IP Khác Nội Bộ == TRUE) AND ( (Role == Vip == TRUE) OR (Cảnh Sát Giao Thông Alternative Khác Báo True) )`.
  - Tức Là: Do Cái Cảnh Sát IP Nó Nắm Lệnh Required, Khách Nào Mà Đứng Ở Mạng Wifi Công Ty (FALSE) Là Lập Tức Cái Sub-flow OTP Bị Khóa Sập Cửa Tắt Ngay Lập Tức Kệ Mọi Phép Toán Ở Sau. 
  - Còn Nếu Ra Mạng Ngoài (TRUE), Lệnh Alternative Kế Tiếp Được Đọc, Phát Hiện Có Role Vip Trả Lời "TRUE" Nên Điều Kiện Pass Đủ Hoàn Toàn Subflow, Form OTP Bắn Bùm Ra Trình Duyệt Bắt Sếp Quét Mã!

---

## 6. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Conditional authentication flows.
