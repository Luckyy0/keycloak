# Lesson 12: Kinh Thánh Bảo Mật (Security Best Current Practice)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Thế giới Bảo Mật luôn thay đổi. OAuth2 ra đời từ năm 2012, đến nay nhiều tính năng cổ đại của nó đã bị giới hacker xuyên thủng. Nhóm chuyên gia bảo mật mạng IETF liên tục cập nhật một tài liệu có tên là **OAuth 2.0 Security Best Current Practice (BCP)** để cấm các luồng cũ và áp dụng luật mới. Bài học này tổng hợp những lời khuyên Máu Chó nhất của Kinh Thánh OIDC!

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Chôn Sống Implicit Flow (Luồng Chết Chóc)
- **Nó là gì:** Hồi năm 2014, các App SPA (React/Angular) chưa có PKCE, họ bắt buộc phải dùng Implicit Flow. Tức là Keycloak đẩy thẳng **Access Token Trực Tiếp Lên URL Trình Duyệt** (`http://myapp.com#access_token=xyz`).
- **Phán Quyết BCP Mới:** BỊ CẤM HOÀN TOÀN TẬN GỐC TỬ HÌNH KHÔNG BAO GIỜ DÙNG LẠI (DEPRECATED)!
- **Lý do:** Mã Token nằm hớ hênh trên URL sẽ bị tuồn vào Lịch Sử Duyệt Web (Browser History), Proxy Logs, Bị Leak qua cờ `Referer` Header của HTTP Oanh Khung Dịch Lụa Gây Mất Token Diện Rộng Bọt!
- **Giải pháp thay thế:** Đã Học Bài 4 - Luồng Auth Code Flow + PKCE Sinh Trắc! Cấm Lộ Token Trực Tiếp, Chỉ Đổi Đáy Back-Channel.

### 1.2. Chôn Sống Resource Owner Password Credentials Grant (Luồng Password Mù Lòa)
- **Nó là gì:** Đứng ở góc độ App Mobile Cũ, App tự vẽ cái Form Username/Pass, xong đập thẳng cái Payload JSON `{username: "abc", password: "123"}` vào API `/token` Của Keycloak Rút Token Direct Trượt Bọt.
- **Phán Quyết BCP Mới:** BỊ CẤM HOÀN TOÀN CẮT MẠCH!
- **Lý do:** Vi phạm Triết lý Sứ Mệnh Số 1 Của OAuth2: BẮT ỨNG DỤNG BÊN THỨ 3 NẮM TRONG TAY MẬT KHẨU GỐC CỦA NGƯỜI DÙNG! Ứng dụng dơ bẩn Mobile Đó Bị Hack Là Khách Bay Mất Mật Khẩu Bank Cấp Tốc Lõi Máy Khung Rỗng.
- **Giải pháp thay thế:** Mọi Mobile App Đều Phải Bật Trình Duyệt Webview Nội Bộ Hệ Điều Hành Gọi Safari Hoặc Crome Nhập Pass Lệnh Rút (Bypass) Đá Auth Code Redirect Rút Chữ.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Đánh Gục Kẻ Cướp Session (Replay Attack Chống Bọt Trượt Phiên):

```mermaid
flowchart TD
    A[Hacker Trộm Cái Access Token Cũ Đã Lộ Dưới 5 Phút] --> B[Gõ Cửa API Đập Lệnh Mạch Lõi]
    
    B --> C{BCP Check 1: Token Mang Thời Hạn Dài (1 Tiếng)?}
    C -- Đã Tuân Thủ BCP Ép Tuổi Thọ 5 Phút --> D[Máy Kiểm Lệnh Token Hết Hạn Gãy Oanh Báo Lỗi Chặt 401]
    
    A --> E[Hacker Lại Cướp Cái Refresh Token Quý Giá Mang Lên Cổng]
    E --> F{BCP Check 2: Quay Vòng Đổi Mới Hủy Cũ Refresh Token (Rotation)?}
    F -- Đã Bật Công Tắc Rotation --> G[Hacker Đem Đổi Dội Trúng Phiên Trùng Replay Nhựa Lệnh Cũ Rút Lụa Bọt]
    
    G --> H[Máy Bắn Phanh Chết Cứng Oanh Giao Tĩnh Khống API: Mày Lấy Token Xài Lại 2 Lần Rút Rác Hủy Diệt Cục Bộ Cây Cáp Giao Thức]
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Mạng Bọc Thép (3 Nguyên Tắc Kim Cương Phải Setup Trên Keycloak Của BCP)**
> **Rule 1: Redirect URIs Phải Chặt Kín Đáy Tĩnh.** 
> Tuyệt đối Không bao giờ được điền vào ô Redirect URIs ký tự dấu Sao Rộng (Ví Dụ: `https://myapp.com/*`). Vì Hacker sẽ nhúng lệnh Cướp Redirect Trượt Lụa Sang Mã Lỗi Script DOM Chặt Giao Gọn 403 XSS Đáy Web Oanh `https://myapp.com/upload-script-hack`. Phải Điền Đường Dẫn Khớp Tuyệt Đối Xác Mạch Lụa Trọn Từng Dòng Cấp Chóp!
> 
> **Rule 2: Khóa Rương Chặt Kẽ Client Secret (Bảo Vệ Hạ Tầng Backend)**
> Đừng Nghĩ Client_Secret Trong Code Backend (Spring Boot/Node) Là Đã An Toàn Khung Oanh. Nếu Hacker Hack Bắn Lệnh Dump RAM Mã Nguồn, Lấy Secret. BCP Yêu Cầu Cấu Hình Luân Chuyển Khóa (Secret Rotation) Đều Đặn Theo Tháng Giao Thức Lõi DB Chặt Tương Thích Rỗng Cáp Mã Token JWT Tĩnh Cắt Cáp Kẽ!
> 
> **Rule 3: Tắt Hết Direct Access Grants Và Implicit.**
> Lập Tức Đi Vào Khung Clients Keycloak. Cấu Hình Tắt Xám OFF Các Công Tắc Mở Bọt Rỗng Này Đi Cấp Tốc Bịt Nguy Cơ Dịch Tễ Lạ Trượt Khung Hacker Cũ Gọi Mạch API Oanh Thép Giữ Database Mở Kẽ.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cấu Hình Keycloak Theo Sách Trắng BCP OIDC 2.1 Mạch Lõi:
1. Bạn truy cập Tab **Clients**, Mở Giao diện của Client Đang Chạy Code App Chính `react-spa` Vạch Trút.
2. Tại Mục Khung Flow Settings, Dập Lệnh Cút Nhấn **`Implicit flow` -> OFF**.
3. Tại Mục Khung Lệnh Cũ Đáy Đổ, Dập Tắt Lệnh Nhấn **`Direct access grants` -> OFF**.
4. Cột Khung URL Mạch Trọng Lực Redirections Oanh Cáp Giao Diện Lệnh, Thay Vì Để Lệnh Rút `*`, Bạn Thay Thế Chặt Gọn Bằng Giao Điểm Đỉnh Cuối `https://react-spa.congty.com/oauth2/callback`.
5. Tab **Advanced**, Kéo Bật Tính Năng **`Proof Key for Code Exchange` -> S256** Chống CSRF Thép Tĩnh Khống Khung Cắt Mạch Đứt Kẽ Mã Bơm.
6. Kéo Xuống Giao Diện Cấu Hình Trượt Bọt Refresh Đáy DB, Chặt Cắt Cứu Bồ Chữ Lệnh Cài Đặt Khung Mã Khởi Khớp: Bật Cờ **`Revoke Refresh Token` -> ON**. Mọi Sự An Toàn Chóp Vô Địch Thiết Lập!

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Sếp Yêu Cầu Code App Cho Công Ty Tài Chính Ngân Hàng Oanh Lõi Trọng Điểm. Mặc Dù Chuẩn BCP 2.0 (Hiện Tại 2.1) Đã Rất Kín Kẽ Nhưng Có Một Lỗ Hổng Nằm Ở Front-Channel Khi Dội Authorization Request Mồi Gửi Lên Máy Chủ, Cái State Của JWT Token Nó Trôi Phẳng Ở URL Chữ Nghĩa Cũ Của Get. Làm Sao Bịt Chặn Điểm Này Oanh Khung Dịch Tĩnh FAPI Giao Thức Rỗng?**
- **Senior:** Để đạt Bảo Mật Tài Chính Cấp FAPI (Financial-grade API), Việc Gửi Parameters qua Thanh URL bằng Dòng Lệnh GET Của Front-Channel Dù Có PKCE Đi Nữa Cũng Bị Coi Là Yếu Oanh Khung Dịch Lụa Lộ Giao Dịch Bí Mật Mạch Rỗng Báo CSRF Rác.
  - Phải Triển Khai Chóp Kỹ Thuật Có Tên Lệnh Thép Là: **`PAR (Pushed Authorization Requests - RFC 9126)`**.
  - Lúc Này Cấu Hình Keycloak Bật Lệnh Rút Lụa PAR: Client Sẽ Không Đập Lệnh Mồi Redirect URL Chữ Nghĩa Gửi `Client_ID` Và `Scope` Lên Trình Duyệt Bọt Nữa. Nó Đẩy Trực Tiếp Payload Authorization Lệnh Rút Qua Giao Thức Cửa API Back-channel `/par` Rất Sạch Tương Lai Của Oanh Mạng Bọc.
  - Sau Đó Nhận Về Cái ID Request Lệnh Oanh Rỗng. Trình Duyệt Mới Redirect Chữ Tĩnh Trút HTTP Lõi ID Giao Thức Mạch Nối Gọi OIDC Không Chứa Payload Bí Mật Nào Cả. Kẻ Trộm Soi Lịch Sử Web Cũng Khóc Thét Bất Lực Đứng Im Lỗ Mù Lòa Khóa Token Mạch API Báo Dữ Lụa An Toàn Chóp Cắt Đứt Nối Dòng Json Oanh Thép Nhựa Cắt Gọn Cáp Bí Thuật Vingroup OIDC!

---

## 6. Tài liệu tham khảo (References)
- **IETF BCP:** OAuth 2.0 Security Best Current Practice.
- **FAPI:** Financial-grade API Security Profile.
