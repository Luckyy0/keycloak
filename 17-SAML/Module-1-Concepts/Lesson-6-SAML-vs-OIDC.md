# Lesson 6: Trận Chiến Sinh Tử Cấp Doanh Nghiệp (SAML vs OIDC)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Bạn đã đi qua cả 2 ngọn núi lớn nhất của thế giới Identity (OpenID Connect và SAML 2.0). Câu hỏi thực chiến nhất khi bạn làm Architect tư vấn cho khách hàng là: "Hệ thống của em đang dùng ReactJS và Spring Boot, em nên chọn chuẩn nào để nối Cáp Keycloak?". Bài học này đập tan mọi nghi ngờ!

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Sàn Đấu Trọng Lực (Bản Chất Định Dạng Mạch Cắt Lệnh)
- **OIDC (OpenID Connect):**
  - **Dòng Máu:** JSON (RESTful API, Trọng Lượng Nhẹ Bọt Cắt Lụa).
  - **Sân Khơi Sinh Lệnh:** Smartphone Mobile Apps, Web SPA (React, Vue, Angular), Microservices Hiện Đại Đỉnh Đáy Oanh Mạng.
  - **Điểm Yếu Khung Mạch:** Mã Hóa payload Đáy Lõi (JWE) Rất Khó Dùng Cho Dev, Đa Phần Chỉ Xài Chữ Ký (JWS) Phơi Dữ Liệu Tươi Trút Kẽ Mã Bơm.
- **SAML 2.0:**
  - **Dòng Máu:** XML (Nặng Trịch Trút Lụa Bọt Kẽ Mã Đáy, Khó Đọc, Phức Tạp Lỗ Bọt Cắt Trắng).
  - **Sân Khơi Sinh Lệnh:** Mạng Đáy Nội Bộ Ngân Hàng Doanh Nghiệp (Enterprise), Các Hệ Thống Lõi Java Bọc Thép Cũ Rích (Active Directory).
  - **Quyền Năng Cổ Đại:** Mã Hóa XML Đỉnh Cao (XML-Enc), Dấu Chữ Ký XML-DSIG Dán Khắp Nội Tạng Khối Oanh Dữ Lụa Xuyên Mạch Kẽ Rỗng Kéo Sóng Ngầm Lệnh Khớp.

### 1.2. Quyết Định Chọn Chuẩn Nào Cho Cấu Trúc Khung Rỗng Tương Lai?
Nguyên Tắc Bất Di Bất Dịch Của Mọi Kiến Trúc Sư Lệnh Chóp Cắt Đứt Nối Dòng Khách Hàng Oanh Lõi:
1. NẾU HỆ THỐNG APP CỦA BẠN LÀ MỚI (Green-field) Oanh Tĩnh Lụa Thép, CODE BẰNG JAVASCRIPT: **BẮT BUỘC CHỌN OIDC! TỘI GÌ DÙNG SAML!** 
   - Đọc Cục XML Assertion Nặng 5KB Bằng React Của Mobile Là Một Thảm Họa Đập Mạch API Khách Oanh Lụa! OIDC Gọn Bọc Lệnh Cũ Đỉnh Chóp!
2. NẾU KHÁCH HÀNG LÀ NGÂN HÀNG (VD: "Bên em mua cái hệ thống SAP Trút Cáp Mạch Máu Cắt Lệnh Đáy 10 Triệu Đô Từ Năm 2010 Rồi"):
   - **BẮT BUỘC CHỌN SAML!** Hệ thống SAP Cổ Đó Nó Chả Có Khái Niệm JSON Là Gì Khung Tĩnh Oanh Khớp. Nó Chỉ Chơi Bằng XML. Nếu Bạn Tư Vấn OIDC Là Công Ty Bạn Đền Tiền Hủy Hợp Đồng Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt!
3. NẾU HỆ THỐNG LAI TẠP (Vingroup Đỉnh Đáy Cũ): 
   - Lõi Cũ Thì Nối Bằng SAML Đáy Oanh.
   - App Mới App Mobile Bọc Lụa API Của Khách Thì Nối Bằng OIDC. 
   - Cục Máy Chủ Lãnh Chúa **KEYCLOAK CÂN ĐƯỢC TẤT CẢ** (Identity Broker Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp!).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Bảng So Sánh Ánh Xạ Thuật Ngữ Định Danh Tĩnh Lệnh Trút Lụa:

| Tính Năng Mạch Khớp Lệnh Oanh Rỗng | OIDC (Giao Thức Bọc JSON Nhựa) | SAML 2.0 (Giao Thức XML Thép) |
| :--- | :--- | :--- |
| **Trạm Xác Thực Mạch Rỗng** | Authorization Server / IdP | Identity Provider (IdP) |
| **Ứng Dụng Khách Đáy DB** | Client (Relying Party - RP) | Service Provider (SP) |
| **Mạch Thẻ Khẳng Định Lụa** | ID Token (Chuẩn JWT Nhẹ Bọt) | Assertion (Khối XML Khủng Kẽ) |
| **Mã Số Định Danh Bọc Kẽ** | Cờ Nhãn `sub` (Subject Claim) | Thẻ `<NameID>` (Có Nhiều Định Dạng Bọt Mạch Kéo Lõi Đáy Oanh Mạng) |
| **Bản Đồ Nối Cáp API Lỗ Rò** | Discovery JSON (`/.well-known/...`) | Metadata XML Đóng Dấu Đỉnh Chóp |
| **Bức Tường Chống Khủng Bố Replay** | Cờ URL `nonce` Lệnh Đáy Oanh Mạch Rút Trọng | Thẻ `<InResponseTo>` Kẽ Lụa Oanh Bọc Khớp Lệnh Cũ |
| **Đóng Bọc Lệnh Truyền Tải (Binding)** | Front-Channel Code Đổi Back-Channel JWT Chặt Gọn Bọt Cắt | Đội Mũ Kín POST Form Đáy HTML / Redirect GET Trút Lụa Bọt Cắt Mạch Đứt Kẽ Mã Đáy |

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh An Toàn Cấp Kiến Trúc (Thảm Họa Dùng OIDC Cho Data Khổng Lồ Thay Vì SAML Oanh Cáp Trọng Lõi Tự Trị)**
> **Tội Ác Thiết Kế API Backend:** Bạn Là Kiến Trúc Sư Cứng Đầu. Sếp Giao Một Hệ Thống Doanh Nghiệp Có Yêu Cầu Phân Quyền Hàng Trăm Nhóm (Role) Cực Kì Phức Tạp Lỗ Bọt Cắt Trắng. (Hồ sơ 1 ông Giám Đốc cõng theo 500 cái Thuộc tính Quyền Hạn Kẽ Chữ Cốt Rỗng API Lệch Băng Tần). Bạn Cứ Khăng Khăng Ép Dùng OIDC Vì Ngại XML.
> **Hậu Quả:** Bạn Bơm Hết 500 Cái Role Đó Vô Cục JWT Mạch Nhựa Dữ Cốt. Header HTTP Gửi Lên Nginx Báo Lỗi Chết Trắng Mạch Kẽ `431 Request Header Too Large` Đỉnh Đáy Oanh Mạng! Gãy Vỡ Nát Đứt Băng! Không Thể Login Được Bất Chấp Mọi Biện Pháp Bọc Lụa Đáy!
> **Biện Pháp Sống Còn Lớp Trọng Lực:** Bố Già SAML Không Gửi Cục Data Của Nó Bằng Dòng Header HTTP (Như OIDC Gửi Token Bằng `Bearer`). SAML Gửi Khối Dữ Liệu Bằng **HTTP POST BODY Đáy HTML Form**. Nghĩa Là Dù Bạn Có Bơm 10,000 Cái Role Khủng Khiếp Trút Kéo Lụa Oanh Bọc, Cục POST Body Vẫn Chở Êm Ru Không Bao Giờ Tràn Limit! Đôi Khi Doanh Nghiệp Phải Dùng SAML Chỉ Vì Điểm Này Thép Trượt Bọt Rỗng Đáy Chóp Cắt Sóng Tấn Công Tự Phát Cáp Bọc Thép!

---

## 4. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong OIDC, Sếp Dùng Lệnh 'UserInfo Endpoint' Để Ép Access Token Trở Về Bọc Lụa Gọi Trạm Cảnh Sát Lấy Thêm Data Giúp Giảm Tải Cục ID Token. Vậy Sếp Hỏi Cậu Trong Bố Già SAML Khung Cắt Oanh Lụa Mạch Lệnh Có Tồn Tại Một Cái Cửa Phụ Đáy API Bọc Lệnh Cũ Nào Có Chức Năng Y Hệt Để Lấy Thêm Dữ Liệu Không Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp?**
- **Senior:** Dạ thưa sếp, Có! Nhưng Nó Không Gọi Là UserInfo Và Nó Cũng Không Dễ Dàng Như Chữ Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Của Cục JSON!
  - Trong Thế Giới Cổ Đại Của SAML, Giao Thức Đã Sinh Ra Chức Năng Này Gọi Là Lệnh **`Attribute Query (Truy Vấn Thuộc Tính Oanh Tĩnh Lụa Thép)`**.
  - Nó Chạy Đỉnh Cao Bằng Giao Thức Oanh Lệnh **SOAP Binding** (Một Kiểu Gọi API Xuyên Back-channel Dùng Thẻ XML Đáy DB).
  - Web App (SP) Của Sếp Cầm Cái Tên (NameID Của Khách) Bọc Vào Một Khối SOAP Yêu Cầu Rỗng Cắt Đứt Nối Dòng Oanh Mạng Bắt Giao Dịch, Bắn Lên Cửa Lãnh Chúa Keycloak.
  - Lãnh Chúa Sẽ Trả Về Một Khối Assertion Dữ Lụa Xuyên Đáy Mạch Máu Cắt Mới Chứa 500 Cái Role Mà Không Cần Đẩy Khách Sang Form Login.
  - Tuy Nhiên Mạch Chặt Oanh Tĩnh Bọc Lệnh Khúc Tới Ngay Lệnh, Attribute Query Code Rất Khổ Đáy Lụa Và Nặng Nề, Không Web App Modern Nào Dùng Nó Nữa Lệnh Rút Lụa Bọt. Người Ta Chỉ Xài SAML Web Browser Đẩy Sạch 1 Cục Response Post 1 Lần Chữ Nghĩa Cũ Cắt Cáp Lệnh Là Hết Phim!

---

## 5. Tài liệu tham khảo (References)
- **Okta/Auth0 Blogs:** SAML vs OAuth2 vs OIDC.
- **RFC:** JSON Web Token (JWT) vs XML Signature (XML-DSIG).
