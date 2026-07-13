# Lesson 9: Kỷ Luật Thép (Requirements)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trong Flow, bạn thường xuyên thấy các lựa chọn `Required`, `Alternative`, `Disabled`, `Conditional`. Chúng được gọi là Requirement (Yêu cầu/Quy tắc). Bài này sẽ mổ xẻ chính xác bản chất của từng quy tắc để bạn không bao giờ ghép sai logic.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Các Trạng Thái Requirement Của Một Execution
Mỗi khi bạn gắn một khối Execution hoặc Sub-flow vào hệ thống, Keycloak sẽ hỏi bạn: "Mức độ ưu tiên và sự ép buộc của cục này là gì?". Bạn có 4 câu trả lời (Requirements):

1. **Required (Bắt Buộc - Toán tử AND):**
   - **Bản chất:** Đã tới dòng này là BẮT BUỘC phải thực thi thành công. Nếu thất bại (VD: Nhập sai mật khẩu), toàn bộ Luồng sẽ dừng lại ngay lập tức, văng lỗi từ chối truy cập.
   - **Đặc điểm:** Không có con đường lùi, không có phương án thay thế.

2. **Alternative (Hoặc - Toán tử OR):**
   - **Bản chất:** "Nếu cục này chạy thành công thì tốt, Pass luôn cả khối. Còn nếu chạy thất bại, thì KHÔNG SAO, cứ bỏ qua nó và trượt xuống cục Alternative tiếp theo nằm dưới để thử vận may."
   - **Đặc điểm:** Cần ít nhất 2 khối Alternative đi liền nhau để tạo ra sự lựa chọn. Chỉ cần 1 trong các khối Alternative báo "Thành công" là toàn bộ chuỗi được thông qua. Nếu tất cả đều báo thất bại, luồng sẽ chết.

3. **Disabled (Vô Hiệu Hóa):**
   - **Bản chất:** Cục này đang bị tắt hoàn toàn. Engine của Keycloak đi ngang qua sẽ coi như nó tàng hình, không chạy, không hỏi, không báo lỗi. 
   - **Đặc điểm:** Thường dùng khi muốn tạm ngưng 1 tính năng (như reCAPTCHA) để test hệ thống mà không muốn phải xóa hẳn nó đi.

4. **Conditional (Điều Kiện):**
   - **Bản chất:** Giống như một công tắc thông minh. Nó chỉ kích hoạt khối chức năng này NẾU các điều kiện (Condition) đi kèm trả về giá trị `TRUE`. Nếu điều kiện trả về `FALSE`, toàn bộ khối sẽ bị bỏ qua (y như Disabled).
   - **Đặc điểm:** LUÔN LUÔN phải có các khối `Condition` nằm ngay dưới nó để làm nhiệm vụ thẩm định.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Minh Họa Logic Chạy Của Các Toán Tử:

```mermaid
flowchart TD
    A[Bắt Đầu Flow] --> B{Khối A: REQUIRED}
    B -- Fail --> C[Chết Luồng, Báo Lỗi Ngay Lập Tức]
    B -- Pass --> D{Khối B1: ALTERNATIVE}
    
    D -- Pass --> E[Thành Công Cả Cụm, Nhảy Xuống Cuối]
    D -- Fail --> F{Khối B2: ALTERNATIVE Dưới Nó}
    
    F -- Pass --> E
    F -- Fail --> C
    
    E --> G{Khối C: CONDITIONAL}
    G -- Condition Pass --> H[Thực Thi Logic Trọng Tâm (VD: Quét OTP)]
    G -- Condition Fail --> I[Bỏ Qua Nhẹ Nhàng Bằng Cửa Trượt Tàng Hình]
    
    H --> J[Hoàn Thành Flow]
    I --> J
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách (Tuyệt Đối Không Để Alternative Đứng Một Mình Đơn Độc)**
> **Tội Ác Thiết Kế:** Bạn Mới Học Xong Luồng, Lôi Một Cục Form Vẽ WebAuthn Passkeys Đặt Vào Flow. Bạn Sợ Nếu Để Required Khách Chạy Lỗi Sẽ Nghẽn Luồng, Nên Bạn Đặt Nó Là `Alternative`. NHƯNG BẠN CHỈ ĐỂ MỖI NÓ LÀ ALTERNATIVE RỒI KẾT THÚC LUỒNG!
> **Hậu Quả:** Engine Của OIDC Quét Tới Dòng Alternative Đó. Khách Trượt Vân Tay Trật. OIDC Hỏi "Thằng Này Fail Rồi, Thằng Nào Trám Chỗ Dự Phòng Tiếp Theo?". Máy Nhìn Xuống Dưới KHÔNG CÓ THẰNG ALTERNATIVE NÀO NỮA ĐỂ CỨU CÁNH! Lập Tức Hệ Thống Văng Lỗi "Internal Error Server 500" Đứt Ngang Xương Form UI, Sập Giao Diện Trắng Bóc Chửi Vô Mặt Khách!
> **Biện Pháp Sống Còn:** Đã Dùng `Alternative` Thì Bắt Buộc Phải Có Thêm 1 Hoặc Nhiều Khối `Alternative` Khác Nằm Ngay Dưới Đáy Để Hứng Lỗi Chạy Chữ Dự Phòng (Fallback Mechanism). Nếu Khối Trên Hỏng Thì Phải Có Khối Dưới Dọn Rác.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Hệ Thống Chọn 1 Trong 2 Phương Thức MFA (Mã OTP Số Hoặc Chạm Vân Tay):
1. Duplicate Browser Flow Ra Và Cấu Hình.
2. Thêm Một Sub-Flow Chứa Tên `MFA-Wrapper`. Set Nó Thành `Required`.
3. Nhét Vào Trong `MFA-Wrapper` 2 Khối Execution:
   - Cục 1: `WebAuthn Passwordless Authenticator`
   - Cục 2: `OTP Form`
4. Chỉnh Requirement Của Cục 1 WebAuthn Sang Bật Cờ **`Alternative`**.
5. Chỉnh Requirement Của Cục 2 OTP Form Nằm Ngay Dưới Sang Trạng Thái **`Alternative`**.
6. Lúc Này, Trình Duyệt Bật Lên Sẽ Ưu Tiên Thử Cục 1 Trước (Nhận Diện Màn Hình Vân Tay Kêu Bíp). Nếu Khách Ấn Nút "Try Another Way", Máy Tự Tụt Xuống Cục Số 2 Để Hứng Sự Thay Thế Bằng Giao Diện Nhập 6 Chữ Số!

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Sub-Flow A Của Browser, Cậu Có Khối 'Username Password Form' Nằm Dưới Dòng 'Cookie'. Tuy Nhiên Cookie Đang Được Đặt `Alternative` Còn Username Password Form Đang Bị Chỉnh Thành Lệnh `REQUIRED`. Khách Hàng Đã Từng Đăng Nhập SSO Bằng Cookie Rất Ngon Ở App 1 Hôm Qua, Sáng Nay Mở App 2 Lên (Cùng Realm Keycloak). Liệu Khách Có Cần Nhập Mật Khẩu Lại Không? Vì Sao?**
- **Senior:** Có, Khách Bắt Buộc Phải Nhập Pass Lại Từ Đầu Đăng Nhập, Trải Nghiệm SSO Đã Hoàn Toàn Bị Phá Hỏng Bởi Cấu Hình Của Bạn.
- Lý Do Nằm Ở Chữ Dòng Form Mật Khẩu Đang Gắn Cờ Toán Logic: `Required`.
  1. Keycloak Chạy Vào Bước 1 Gặp Cục `Cookie` Với Dòng Chữ `Alternative`. Khách Vừa Có Token Hợp Lệ Của Ngày Hôm Qua, Cookie Vui Vẻ Báo Lên Màn Hình: "Thằng Này Đã Đậu Rút Gọn Rồi Sếp Ơi!". Cục Alternative Trả Về Lệnh Thành Công Pass Through.
  2. Tuy Nhiên Oái Oăm, Máy Chạy Trượt Xuống Bước 2 Gặp Khối Form Password Được Dev Trói Chặt Dòng Lệnh Rất Cứng Mạch `REQUIRED` (Bắt Buộc Ràng Dây Đứt Chạm Lõi). Cục Này Cưỡng Chế Render Ra Trình Duyệt Cái Giao Diện Đòi Khách Nhập Pass. Vì Là Required Nên Nó Bỏ Mặc Sự Pass Thông Môn Của Cookie Ở Bước 1 Đã Trả Về. Nó Ép Khách Mù Lòa Điền Pass Trúng Mới Trượt Xong Chuỗi Khối Này. Chấm Hết Trải Nghiệm Đăng Nhập Không Mật Khẩu SSO.

---

## 6. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Authentication Requirements.
