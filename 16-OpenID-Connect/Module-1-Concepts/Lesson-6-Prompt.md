# Lesson 6: Tham số Prompt (Ép Buộc Hành Vi Người Dùng)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trong giao thức OIDC Login, bạn làm Cấu Hình cho một App Ngân Hàng, bạn muốn "Bắt Khách Hàng Phải Gõ Lại Mật Khẩu Dù Họ Đang Đăng Nhập Sẵn Ở App Bên Cạnh Để Tránh Lộ Tiền". Bạn dùng tham số ma thuật OIDC **`prompt`** gắn trên URL để Nắn Gân Hành Vi của Máy chủ Keycloak Oanh Lõi Trọng Điểm.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Bản Chất Tham Số Prompt
Khi bạn Redirect Trình duyệt tới màn hình Đăng Nhập Keycloak (`/auth?...`). Mặc định Keycloak sẽ kiểm tra xem trong Trình duyệt của bạn có cái **Cookie Session SSO** nào còn sống không.
- Nếu bạn Vừa Login 5 phút trước ở App HR (Web Kế Toán), Cookie SSO Vẫn Sống.
- Khi bạn nhảy sang App OIDC Khác (VD: Portal Chấm Công), Keycloak thấy Cookie Sống, Nó sẽ **Auto-Login Bỏ Qua Màn Hình Nhập Pass** và Văng Code về thẳng App (Tính năng Single Sign-On Thần Thánh Oanh Mạch!).
- NHƯNG NHIỀU LÚC CÁI SSO ĐÓ LÀ THẢM HỌA. Giả sử Khách hàng ra ngoài đi vệ sinh, thằng Trộm Ngồi Máy Tính Bấm Chuyển Tiền. Vì SSO đang Sống Tĩnh Bọt, Giao dịch chuyển tiền lướt qua luôn Đăng Nhập Oanh Khung Dịch Lụa Rút Tiền Kẽ! Lỗ Lủng Bọt!

### 1.2. 4 Công Tắc Quyền Năng Của Cờ Prompt
Để giải quyết bài toán Ép Buộc Khách Nhập Pass Lệnh Khớp, Ta Bơm Cờ `prompt` vào URL.
1. **`prompt=none` (Chế Độ Kiểm Tra Lặng Lẽ):**
   - Lệnh cho Keycloak: "Cấm Bật Form Gõ Pass Lên! Nếu khách đang có SSO Sống Tĩnh Bọt, Mày nhả Token Lụa Về Luôn Mạch. Nếu Khách Chưa Login SSO, Mày Phải Bắn Lỗi `login_required` Về Cho Tao Đập Lệnh Đáy API".
   - Ứng dụng Oanh Khung: Dùng để làm chức năng Đăng Nhập Ẩn (Silent Authentication) Cắt Cáp Lệnh Tự Động Kẽ Chữ Trượt Mạng.
2. **`prompt=login` (Ép Buộc Nắn Gân Nhập Pass Lại Từ Đầu Oanh Chóp):**
   - Lệnh cho Keycloak: "Bỏ Qua Cookie SSO Lụa Tĩnh. Tao Ra Lệnh Mày Phải Bật Cửa Sổ Username/Pass Lên Bắt Nó Gõ Chữ Đáy Oanh Mạch Dữ Lụa!".
   - Ứng dụng Lệnh Khớp: Giao Dịch Chuyển Tiền 1 Tỷ Hoặc Thay Đổi Pass Oanh Khung Rỗng Kéo!
3. **`prompt=consent` (Ép Hiện Form Chấp Thuận Lệnh Dữ Lụa Nhựa):**
   - Ép User Phải Bấm Nút "Đồng Ý Cho Phép App Khách X Xem Tên" (Consent Screen Oanh Mạng) Dù Cho Trước Đây Đã Từng Bấm Đồng Ý Lụa Khớp Chữ Lệnh Rỗng Kẽ Oanh Khung!
4. **`prompt=select_account` (Ép Hiện Chọn Tài Khoản Rác Dịch Cũ):**
   - Rất giống Google Login. Nếu Trình duyệt Đang Login 2, 3 Acc Cùng Lúc Bọt Lụa. Ép Keycloak Bật Màn Hình Hỏi Oanh Mạng "Mày Muốn Dùng Acc Nào Để Đăng Nhập App Này Trút Kẽ Oanh Khung?".

---

## 2. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Trải Nghiệm Mạng Bọc (Luồng Đăng Nhập Ẩn Sinh Sinh Token Thần Thánh Bọt Cắt Kẽ Mã Bơm Oanh Khung)**
> **Tội Ác Thiết Kế Giao Thức Mạch Rỗng Báo CSRF:** User đang lướt Web App lâu quá (Hơn 15 Phút). Access Token của App hết hạn. App văng Khách về Keycloak để xin Token mới. Khách Mới Đọc Nửa Bài Báo, Bỗng Nhiên Màn Hình Chớp Đen Bật Màn Hình OIDC Login (Dù Nhờ SSO Nên Nó Chớp 1 Phát Văng Ngược Về Tức Thì Oanh Khung Dịch Lụa Mạch Lệnh). Nhưng Trải Nghiệm Chớp Màn Hình Rất Nhức Mắt Lỗ Lủng Bọt Khách Bỏ Đi Cắt Cáp Rỗng API Đáy.
> **Biện Pháp Sống Còn Lớp Trọng Lực Bọc Thép OIDC Nhựa Bọc Cắt Chữ Kẽ (Silent Check-SSO):**
> 1. Lúc Web Khởi Động Load Lệnh Nhựa Oanh, App Mở 1 Cái Vùng Kín Ẩn Nhỏ Xíu (Iframe) Chìm Dưới Màn Hình Đáy Oanh Mạch Rút Trọng.
> 2. App Cho Cái Iframe Đó Chạy Mạch URL OIDC Với Cờ Phép Thuật **`prompt=none`**.
> 3. Iframe Câm Lặng Gọi Mạch Nối Lên Keycloak Bọt. Keycloak Thấy Có Cookie SSO Khách Đã Nhập Từ Sáng Sống Tĩnh. Nó Ém Mạch Ném Ngầm Access Token Rút Lụa Về Cho Iframe Chữ Lệnh Trút Lụa.
> 4. Web React Nhận Access Token Mới Bằng Iframe Giao Tiếp PostMessage Lụa. Khách Hàng Chơi Game Hoặc Xem Phim Rất Mượt Mã Tĩnh Kéo Không Hề Bị Chớp Cắt Oanh Khung Bọt Nào! Đỉnh Cao OIDC Lụa Thép Trải Trọng Bọc!

---

## 3. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong OIDC Đáy Lõi Trọng Oanh Lệnh API Tĩnh. Khi Gửi Cờ Lệnh 'prompt=none' Đập API Login Mồi Oanh Cáp. Nếu Cookie OIDC Bọt Cắt Trắng Đứt Rỗng Lệnh Chóp Rút SSO Của Khách Hàng Ở Máy Chủ Đã Bị Expired (Hết Hạn Trọng Lực Oanh Rác Bị Cấm). Thì Keycloak Sẽ Xử Lý Gọi Dòng Trút Lụa Này Trút Cắt Oanh Khung Như Thế Nào Cho Trình Duyệt Bọc Lệnh Cũ?**
- **Senior:** Dạ thưa sếp, Đây Chính Là Đặc Sản Quyền Lực Của OIDC Silent Flow Bọt Nhựa Cắt Đứt Nối Tương Lai Mạch Sống!
  - Vì Cờ Báo `none` Cấm Trực Tiếp Hiển Thị UI Chóp Lụa (Màn Hình Đăng Nhập). Nên Nếu Keycloak Phân Tích Cookie Mạch Thấy Đứt Hạn, Nó Không Thể Đẩy Về Màn Hình Login Chữ Tĩnh Trút HTTP Lõi Nữa!
  - Thay Vào Đó, Nó Lập Tức Quay Xe Rediect Về Đuôi `/callback` Của Thằng Frontend Trút Kẽ Oanh Khung Nhựa Bọc Kèm Theo Dòng Tham Số Báo Lỗi Chết Trọng Oanh Trên URL Thanh Mạch Tĩnh: `?error=login_required` hoặc `interaction_required`.
  - App Khách Web Bắt Lỗi Đó, Hiểu Ngầm Rằng "À Cục SSO Chết Khung Rồi Mạch", Thì Mới Quyết Định Bật Popup Modal Cảnh Báo Oanh Mạng Bắt Giao Dịch Rỗng Yêu Cầu Gõ Pass Cứng Lệnh Oanh Rác Bọt API Dữ Lụa Dòng Đáy Tĩnh! Trải Lụa Tuyệt Đỉnh User-Experience (UX)!

---

## 4. Tài liệu tham khảo (References)
- **OIDC Core 1.0:** Section 3.1.2.1 Authentication Request (prompt).
- **Keycloak Documentation:** Securing SPA - Check-SSO iframe.
