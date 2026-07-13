# Lesson 2: Lời Khẳng Định Thép Mạch Lụa (SAML Assertions)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trong thế giới OIDC, ta có `ID Token`. Còn trong thế giới SAML, ta có **`Assertion`** (Sự khẳng định/Tuyên bố). Cục Assertion này chính là trái tim của giao thức. Khối lượng của nó to gấp 10 lần một cái ID Token thông thường. Bài học này dạy bạn cách mổ bụng một cục Assertion để xem Cấu Trúc Khung Rỗng XML Của Bố Già Doanh Nghiệp Có Những Bộ Phận Đáy Oanh Lụa Nào.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Cấu Phẫu Thuật Cục Assertion XML
Một khối Lệnh Assertion chuẩn của Keycloak Nhả Về Trút Kéo Nhựa Sẽ Có 4 Bộ Phận Nội Tạng Lệnh Oanh Rút Mạch Máu:

1. **`Issuer` (Tương đương cờ 'iss' bên OIDC):**
   - Nội dung: `https://keycloak.sso.com/realms/master`.
   - Lời Khẳng Định: "Tao, Trạm Cảnh Sát Cục Cấp Khóa Keycloak, Xin Khẳng Định Cục Data Sau Đây Là Sự Thật Bọc Mạch!".
2. **`Subject` (Tương đương cờ 'sub' bên OIDC):**
   - Nội dung: Chứa thẻ `<NameID Format="...emailAddress...">` Ghi rõ Email: `vuduc@gmail.com`.
   - Lời Khẳng Định: "Tên thằng Đang Định Đăng Nhập Ở Mạch Trình Duyệt KIA Chính Là VuDuc".
3. **`Conditions` (Tương đương cờ 'exp' bên OIDC):**
   - Nội dung: Có thẻ `<Audience>` (Đích Nhắm SP) và thẻ `NotBefore` / `NotOnOrAfter` (Thời Hạn Sống Tĩnh Bọt).
   - Lời Khẳng Định: "Tao cấp cho Thằng Kế Toán. Và Lời Khẳng Định Này Chỉ Có Giá Trị Trong Vòng 5 Phút Nữa Oanh Khung Dịch Lụa Mạch Lệnh Cũ!".
4. **`AttributeStatement` (Tương đương cờ Profile 'Claims' bên OIDC):**
   - Nội dung: Chứa một Đống Thẻ Nhựa Bọc Liệt Kê Toàn Bộ User Profile:
     `<Attribute Name="Role"><AttributeValue>Admin</AttributeValue></Attribute>`
   - Lời Khẳng Định: "Tài sản thuộc tính của nó là Tên A, Role B Đáy Oanh Mạng Bắt Lụa".

### 1.2. Chữ Ký Thép Đóng Dấu Rỗng Cắt (XML Signature)
Khác với JSON Web Signature (JWS) Của OIDC Ký Ở Cuối Đuôi Cục Mã Base64.
- Chữ ký của SAML Ký Thẳng Cắt Lụa Vào Giữa Cấu Trúc Bụng Của Khối Dữ Liệu XML.
- Nó Dùng Thẻ **`<ds:Signature>`** Chứa Đầy Đủ Tọa Độ Mã Băm SHA-256 Đáy Lõi DB Của Đúng Chính Xác Cái Đoạn Khung Cắt Nó Đang Ký Nhựa Oanh.
- Nếu Thằng Trộm Dám Đổi 1 Dấu Phẩy Trong Thẻ `NameID` Để Fake Tên Mạch. Thuật Toán XML Nhận Diện Sai Mã Băm Dập Tắt Lỗi Oanh Mạch Rút Trọng Lực Ngay Trút Lệnh Bọt Cắt Kẽ Mã Đáy!

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Đánh Tráo Khối Lệnh Payload JSON So Với Cục Lệnh XML SAML Oanh Cáp Khung Kẽ Bọt Cắt Mạch:

```xml
<!-- MỘT CỤC SAML ASSERTION THU GỌN OANH TĨNH LỤA THÉP -->
<saml:Assertion ID="ID_123456" IssueInstant="2026-01-01T08:00:00Z" Version="2.0">
  
  <saml:Issuer>http://kc.com/realms/master</saml:Issuer>
  
  <!-- Chữ Ký Cắt Khung Nằm Bọc Ngay Giữa -->
  <ds:Signature>
    <ds:SignedInfo>...</ds:SignedInfo>
    <ds:SignatureValue>MÃ BĂM RSA KẺ ĐỊCH BẤT LỰC</ds:SignatureValue>
  </ds:Signature>
  
  <saml:Subject>
    <saml:NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">vuduc_user_1</saml:NameID>
    <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
      <saml:SubjectConfirmationData NotOnOrAfter="2026-01-01T08:05:00Z" Recipient="https://web-ketoan/acs"/>
    </saml:SubjectConfirmation>
  </saml:Subject>
  
  <saml:Conditions NotBefore="2026-01-01T07:55:00Z" NotOnOrAfter="2026-01-01T08:05:00Z">
    <saml:AudienceRestriction>
      <saml:Audience>https://web-ketoan/entity-id</saml:Audience>
    </saml:AudienceRestriction>
  </saml:Conditions>
  
</saml:Assertion>
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Mạng Bọc Thép (Ký Đúp Vành Đai Trút Code Chống Mũi Đục Rò Signature Wrapping Attack)**
> **Tội Ác Thiết Kế Giao Thức Mạch Rỗng Báo CSRF Rác:** 
> Do Kiến Trúc XML Rất Rườm Rà Nhựa Bọc. Có Một Cục To Bên Ngoài Gọi Là `SAMLResponse`, Và Ở Bên Trong Nó Mới Bao Chứa Cái Cục Khẳng Định `Assertion`. 
> Nếu Bạn Cấu Hình Trên Keycloak Lười Biếng, Chỉ Ký Dấu Chữ Ký Đáy Lụa Lên Cục Ngoài Cùng (`Sign Response = ON`), Mà Bỏ Quên Ký Lên Cục Lõi Ở Trong (`Sign Assertion = OFF`).
> **Hậu Quả Mũi Đục Thép XML Signature Wrapping (XML-SW):**
> Thằng Kẻ Trộm Cắt Lấy Toàn Bộ Khối Giao Dịch, Lợi Dụng Lỗ Hổng Parser XML, Chèn Thêm 1 Cục Assertion Giả Mạo Của Nó Lồng Vào Cục Response Ngoại Khung (Nhưng Cục Response Gốc Chưa Bị Phá Vỡ Cấu Trúc Khung Kẽ Bọt Cắt Lệnh Chữ Ký Cũ). App Kế Toán Xài Code XML Thư Viện Lỗi, Decode Mạch Thấy Response Vẫn Có Chữ Ký Sống Tĩnh Bọt, Nhưng Lại Lôi Lộn Mệnh Lệnh Khớp Cục Assertion Giả Mạo Của Kẻ Trộm Lên Parse Tên Tuổi Trút Cắt Khung Lệnh Rỗng! HACK THÀNH CÔNG!
> **Biện Pháp Sống Còn Lớp Trọng Lực:** Lập Tức Bật Cả 2 Công Tắc: **`Sign Documents = ON` VÀ `Sign Assertions = ON`**. Ký Đúp 2 Tầng Oanh Dữ Lụa Xuyên Đáy Mạch Máu Cắt! Thằng Nào Dám Đổi 1 Thẻ Nội Tạng Trút Code Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Sẽ Bị Nổ Sập Máy Chống Giả Mạo!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cấu Hình Ký Đúp Vành Đai XML Chống XML-SW Lõi Mạch Trên Keycloak:
1. Bạn Chọn Client SAML `web-ketoan` Đang Chạy Lệnh Rút Lụa Bọt Mạch Kéo.
2. Di Chuyển Sang Tab **Keys** Của Client Lệnh Đó. 
3. Nếu Công Ty Có Đòi Hỏi Cực Gắt Giao Thức FAPI Oanh Mạng, Bạn Bật Cờ **`Client Signature Required` = ON**.
   - Lúc Này: Bất Cứ Khi Nào Thằng Web Kế Toán (SP) Bắn Lệnh Mồi Đăng Nhập SAMLRequest Lên Keycloak (IdP). Keycloak SẼ TỪ CHỐI Bắn Màn Hình Gõ Pass Ra, NẾU Lệnh Mồi Đó KHÔNG CÓ Chữ Ký Riêng Bằng Private Key Đáy Của Thằng Kế Toán Gửi Lên! 
   - Đảm Bảo Thép: Đã Cấp Giao Dịch Là Phải Xác Minh Rõ Ràng Cả Thằng Xin Lẫn Thằng Cấp Bằng Cặp Public Key Băng Tần Khung Kẽ!

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Giao Thức SAML Khung Cắt Oanh Lụa Mạch Lệnh. Tại Sao Thuộc Tính 'NameID Format' Của Lời Khẳng Định Lại Có Nhiều Chuẩn Nhựa Bọc Như 'EmailAddress', 'Unspecified', 'Persistent', 'Transient'? Sếp Yêu Cầu Code Tính Năng Ẩn Danh User Cắt Bọt Đứt Băng Thì Chọn Định Dạng Nào?**
- **Senior:** Dạ thưa sếp, Đây Chính Là Tính Năng Tuyệt Đỉnh Bảo Vệ Quyền Riêng Tư (Privacy) Của Bố Già SAML Mà OIDC ID Token Bình Thường Khó Làm Mạch Đáy:
  - Nếu Sếp Code Một Hệ Thống Doanh Nghiệp (Bán Hàng Oanh Lụa). Chữ Ký Thẻ Căn Cước NameID Thường Dùng Format **`EmailAddress`** Để Phơi Bày Tên Trút Kẽ Mã Bơm. SP Biết Thằng Đang Login Là "vuduc@gmail.com". Rất Rõ Ràng Khung Tĩnh Oanh Khớp.
  - Nhưng Nếu Sếp Đang Code Hệ Thống Cổng Bầu Cử Bỏ Phiếu (Bầu Sếp Mới Nhựa Bọc). Trạm Khẳng Định IdP Keycloak Khi Bơm Khẳng Định Sang Web Bầu Cử Phải ẨN DANH TRÚT LỤA!
  - Lúc Này Em Cấu Hình Keycloak Chọn Format **`Transient` (Phù Du Mạch Rỗng Cắt Đáy)**.
  - Mỗi Lần Login Khúc Tới Chặt Oanh Tĩnh, Keycloak Sẽ Sinh Ra Một Cái Tên Dịch Kẽ Bọt Ngẫu Nhiên Vô Nghĩa: "User_XYZ_89". Nó Đóng Mộc Lên Assertion Đẩy Về Cho Thằng Bầu Cử. Thằng Bầu Cử Mù Lòa Biết Là "1 Thằng Hợp Lệ Lệnh Đáy" Nhưng KHÔNG BAO GIỜ Dò Ra Được Tên Thật Của Nó! Vượt Mức Bảo Mật Dịch Tễ Oanh Khung Ẩn Danh Quyền Lực Đỉnh Chóp!

---

## 6. Tài liệu tham khảo (References)
- **OASIS SAML V2.0:** Assertions and Protocols.
- **Keycloak Documentation:** SAML Assertions & Signatures.
