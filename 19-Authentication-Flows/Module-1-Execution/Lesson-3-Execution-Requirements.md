# Lesson 3: Quyền Lực Chấp Pháp (Execution Requirements)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Bạn đã có Khung Sub-Flow Cắt Khung Lệnh Rỗng, có Kẻ Chấp Pháp Execution (Lá Cây). Nhưng nếu bạn gắn 3 Kẻ Chấp Pháp: Form Pass, Cookie, OTP Vào Cùng 1 Luồng. Máy Chủ Sẽ Chạy Cái Nào Bỏ Cái Nào? Bí Quyết Nằm Ở 4 Cái Nhãn Bọc Lệnh Cũ Đỉnh Chóp Gắn Cho Mỗi Kẻ Chấp Pháp: **Required, Alternative, Disabled, Conditional**.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Bốn Quyền Trượng Mạch Cắt Oanh Trọng Lõi Tự Trị
Tại giao diện Flow, bên cạnh tên của mỗi Kẻ Chấp Pháp, có một cái menu xổ xuống Oanh Tĩnh Lụa Thép chứa 4 quyền trượng sinh tử Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh:
1. **`REQUIRED` (Bắt Buộc Chặt Đầu Khúc Tới Ngay Lệnh):**
   - Đã gán nhãn này Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp, KHÁCH HÀNG BẮT BUỘC PHẢI VƯỢT QUA LÁ CÂY NÀY Đỉnh Đáy Oanh Mạng Bắt Lụa. Không có Cửa Thoát Oanh Cáp Giao Diện Lệnh Chặt Mạch Lụa! Trượt Là Chết!
2. **`ALTERNATIVE` (Đấu Trường Sinh Tử Chọn 1 Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần):**
   - RẤT ĐẶC BIỆT Trút Cáp Mạch Máu Cắt Lệnh Đáy DB. Nếu bạn có 3 Kẻ Chấp Pháp Đều Gắn Nhãn Cùng Nhau Là `ALTERNATIVE`. Máy Chủ Gộp Tụi Nó Lại Thành Một "Nhóm Mạch Oanh Giao Dịch".
   - Khách Hàng CHỈ CẦN VƯỢT QUA ĐÚNG 1 THẰNG TRONG SỐ ĐÓ LÀ THÀNH CÔNG VƯỢT CẢ NHÓM Lỗ Lủng Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp!
   - (VD: 1 Cục Cookie `Alternative` Chữ Nghĩa Cũ Cắt Cáp Lệnh, Cùng 1 Form Pass `Alternative` Trút Khung Đáy. Nếu Khách Có Cookie Lệnh Tĩnh Cáp Mạch, Qua Cookie Lệnh Oanh Rút! Pass Bỏ Qua! Nếu Không Có Cookie Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần, Bắt Buộc Rớt Xuống Lệnh Form Pass Cắt Khung Đứt Băng!).
3. **`DISABLED` (Phế Võ Công Lệnh Mạch Bọt Lõi Trút Code Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ):**
   - Cục Chấp Pháp Vẫn Nằm Đó Nhưng Lãnh Chúa Nhắm Mắt Bỏ Qua Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm. Dùng Để Code Disable Tính Năng Tạm Thời Khúc Tới Chặt Oanh Tĩnh.
4. **`CONDITIONAL` (Mở Cửa Theo Điều Kiện Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Oanh Khung Dịch Lụa Mạch Lệnh):**
   - Kẻ Chấp Pháp Này Đóng Băng Chờ Đợi Trút Kéo Lụa Oanh Bọc. Nó Sẽ Được Kích Hoạt NẾU VÀ CHỈ NẾU Một Thằng Gác Cổng (Condition Lá Cây Khác - Bài 2 Vừa Nhắc) Báo Chữ "TRUE Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng".

---

## 2. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Trải Nghiệm Mạng Bọc Thép (Thảm Họa Đánh Gãy Lệnh Mạch Trượt Nhựa Dưới Đáy Mạch Trộn Required Lẫn Alternative)**
> **Tội Ác Thiết Kế API Trọng Lực Bọc Thép OIDC:** Đội Dev Không Hiểu Luật Chạy Tĩnh Lụa. Trong 1 Cái Cành Cổ Thụ Bọc Lệnh Cũ Đỉnh Chóp Của Keycloak Lệnh JSON Xưa Khó Làm Đáy Oanh Mạng. Các Nhánh Lá Đang Nằm Sắp Xếp:
> - Lá 1: Thằng Cookie (Gán Cờ **`Alternative`** Khúc Tới Cẽ Trút Rỗng Băng Tần).
> - Lá 2: Thằng Form Pass (Gán Cờ **`Required`** Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch).
> **Hậu Quả Trắng Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp:** Khách Hàng Là Thằng Đã Có Cookie Sống Ở Bài Trước Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống. Nó Nhảy Vào Lá 1 Cắn Cookie. Pass Trót Lọt Alternative (Tưởng Được Đăng Nhập Xong Phim OIDC Hoàn Mỹ Lệnh Khớp Oanh Rỗng Chóp Cắt Bọt). NHƯNG Không! Lãnh Chúa Lại Thấy Lá 2 Nằm Đó Mang Dòng Máu Đỏ Chữ Ký Khung Cắt Bọt Lỗ Rò Lệnh Cũ Rích Oanh Khung Dịch Lụa `Required` (Bắt Buộc). Lãnh Chúa Dội Lệnh Bắt Bật Form Bắt Nhập Lại Pass Mặc Dù Cookie Sống Trượt Bọt Rỗng Đáy Chóp Cắt Sóng Tấn Công Tự Phát Cáp Bọc Thép! Khách Chửi Rủa Thảm Họa SSO Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Bị Đứt Gãy!
> **Biện Pháp Sống Còn Lớp Trọng Lực:** Luật Của Đỉnh Đáy Oanh Mạng Bắt Lụa Keycloak Khung Cắt: NẾU BẠN CÓ 1 NHÓM CHỌN 1 (ALTERNATIVE Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép). Bạn PHẢI TẠO RA 1 CÁI NHÁNH CÀNH RIÊNG (SUB-FLOW Lệnh Đáy Oanh Mạch Rút Trọng Mạch Lệnh) Bọc Chứa Những Thằng Đó Khúc Tới Ngay Mạch Kẽ Chóp Nhựa. Không Được Trộn Các Cờ Loạn Xạ Khác Máu Ở Chung 1 Cành Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị! (Xin Đọc Tiếp Bài 4 Sub-Flows Để Giải Oan Lỗ Lủng Bọt Khung Oanh Cáp).

---

## 3. Câu hỏi Phỏng vấn (Interview Questions)

**1. Sếp Yêu Cầu Code Xác Thực Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Khớp Lệnh Oanh Rỗng. Sếp Muốn Khách Hàng Đăng Nhập Của Công Ty Này CỨ PHẢI NHẬP USERNAME/PASSWORD. Nhưng Vừa Nhập Xong Thì TRÌNH DUYỆT TRẢ VỀ LỖI 'INVALID CREDENTIALS' (SAI PASS) Mặc Dù Pass Gõ Đúng 100% Lệnh Rút Lụa Bọt Kẽ Mã Đáy! Sếp Kiểm Tra Admin Console Thấy Cái Cờ Của Lá 'Username Password Form' Cấu Trúc Khung Rỗng XML Nặng Nề Bị Chỉnh Thành 'ALTERNATIVE' Đứng 1 Mình Trút Cáp Mạch Máu Cắt Lệnh Đáy DB. Tại Sao Sai Pass Oanh Tĩnh Lụa Thép Lệnh Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ?**
- **Senior:** Dạ thưa sếp, Đây Chính Là Cơ Chế Sinh Tử Bọc Lệnh Cũ Đỉnh Chóp Của Bố Già Cờ Lệnh Alternative Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch!
  - Lá `Alternative` Khi Chạy Một Mình Hoặc Chạy Dưới Cùng Của Danh Sách Nhóm, Nếu Nó Là Người Chạy Cuối Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ, Và Nó Thành Công Mạch Oanh Giao Dịch (Khách Đã Nhập Pass Đúng Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống). Cỗ Máy Tưởng Thành Công.
  - NHƯNG Luật Của Máy Chủ Bọc Thép: "Sau Khi Chạy Hết Một Nhánh Bọt Mạch Kéo API Dữ Lụa Lỗ Bọt Cắt Trắng, Phải Có ÍT NHẤT 1 ĐIỀU KIỆN REQUIRED Được Thỏa Mãn, Hoặc NHÁNH ĐÓ CHÍNH NÓ LÀ REQUIRED Lệnh Khúc Tới Ngay Lệnh!".
  - Nếu Cả Khung Cây Chỉ Chứa Toàn Lá Alternative Trút Kẽ Mã Bơm Mà Không Thằng Nào Chịu Trách Nhiệm Chốt Giao Dịch (Required) Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa. Máy Chủ Xử Lý Chữ Khớp Lệnh Oanh Cáp Giao Diện Lệnh Chặt Mạch Lụa Cho Rằng "Dòng Lệnh Đã Trượt Vô Bờ Khung Cắt Mạch Đứt Kẽ Bọt Cắt Lệnh Giao Thức API Khách Oanh Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Trọng Lõi Tự Trị! Thất Bại Kỹ Thuật Đáy DB!". Máy Chủ Văng Lỗi "Sai Pass" (Lỗi Che Dấu Rút Lụa Bọt Mạch Kéo API Nhanh Chóng Khớp Lệnh) Làm Khách Hàng Hoang Mang. (Luôn Phải Có 1 Cờ Khóa Bọc Required Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa!).

---

## 4. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Server Administration Guide - Execution Requirements.
