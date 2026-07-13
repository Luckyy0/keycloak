# Lesson 5: Nhà cung cấp Danh tính (Identity Provider - IdP)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Gọi tên "Đấng Tối Cao" trong Liên minh Danh tính. Hiểu rõ IdP là gì, Quyền lực vô song của nó, và Vì sao Mọi App trên thế giới đều phải Quỳ gối (Delegate) trước nó.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Identity Provider (IdP) là gì?
Trong Kiến trúc Mạng Phân tán (Federation), **Identity Provider (Nhà Cung Cấp Danh Tính - Gọi tắt là IdP)** là Trái tim, là Nguồn Chân Lý (Source of Truth).
Bạn có thể ví nó như **Cục Quản Lý Dân Cư (Bộ Công An)** của một Quốc gia. 
- Nó là Nơi Duy Nhất được quyền Sinh ra, Lưu trữ, và Xóa bỏ Định danh (Identity).
- Nó là Nơi Duy Nhất được quyền Phát Hành "Căn Cước Công Dân Điện Tử" (Token / Assertion).
- Các App/Web khác (Như Ngân hàng, Bệnh viện) TUYỆT ĐỐI KHÔNG có quyền in Căn Cước. Họ chỉ có quyền NHÌN VÀO Căn cước do IdP cấp để cho phép người dùng giao dịch.

### 1.2. Trách nhiệm Sinh Tử của một IdP
Một cỗ máy được gọi là IdP (Ví dụ: Keycloak, Google Workspace, Microsoft Entra ID) phải gánh vác 3 nhiệm vụ:
1. **Authentication (Xác thực):** Nó phải giơ mặt ra để Người dùng đập Password/Vân tay vào. Nó là kẻ phán xử Pass đúng hay sai.
2. **Assertion Generation (Đúc tiền/Phát hành Lời xác nhận):** Khi Pass đúng, nó Tự Động gom các thuộc tính (Tên là A, Tuổi 20, Role là Giám Đốc) nhét vào một cái File, Lấy Khóa Bí Mật KÝ VÀO ĐÓ, tạo thành Token (JWT hoặc SAML).
3. **Session Management (Quản lý Phiên):** Nó giữ "Trạng thái Đăng nhập Toàn cầu" (Global Session). Nếu User Logout tại IdP, nó phải đi báo tin buồn cho toàn bộ các App khác để khóa cửa.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Bên trong Công xưởng Đúc Token (Minting) của IdP:

```mermaid
graph TD
    subgraph "Vương Quốc IdP (Keycloak)"
        DB[(Database: <br/> Hash Passwords)]
        Crypto{Cỗ Máy Mật Mã <br/> (Private Key RSA)}
        
        Input(Nhận Yêu cầu từ App + Username/Pass) --> AuthCheck{Check Pass với DB}
        AuthCheck -->|Đúng| DataGather[Lấy Role, Email, SĐT từ DB]
        DataGather --> Pack[Nhồi Data vào Payload JSON]
        Pack --> Crypto
        Crypto -->|Ký điện tử cực mạnh| Token(Đẻ ra JWT Token xịn)
    end
    
    Note over Token: Token này không thể bị làm giả.<br/>IdP ném Token này ra thế giới bên ngoài cho các App xài.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Quy tắc Quả Trứng Trong Một Giỏ (The Ultimate Target)**
> Vì IdP nắm giữ Chìa Khóa Bí Mật (Private Key) và là Cửa Khẩu duy nhất của Toàn Bộ Công Ty. Nên đối với Hacker, IdP là MỤC TIÊU SỐ 1. Nếu Hack được 1 App Kế toán, chỉ sập 1 App. Nếu Hack được IdP, Hacker Đăng Nhập Được VÀO TẤT CẢ MỌI APP TRONG CÔNG TY.
> **Kiến trúc phòng thủ:** Máy chủ IdP (Keycloak) Bắt Buộc phải nằm sâu trong Private Subnet (Mạng nội bộ cách ly), chỉ phơi đúng Port 443 ra ngoài qua WAF. Database của IdP phải được Mã Hóa Ngay Cả Khi Nằm Lại Ở Ổ Cứng (Encryption at Rest) và có Firewall riêng biệt.

> [!CAUTION]
> **Lỗ hổng Chìa Khóa Mất Cắp (Private Key Compromise)**
> Một khi Private Key của IdP bị Hacker ăn cắp. Hacker mang về máy hắn, tự viết Code sinh ra 1 tỷ cái Token "Đóng dấu chữ ký Xịn" cấp cho hắn quyền Giám Đốc. Toàn bộ App (SP) nhận được Token đều tin sái cổ vì Chữ ký Chuẩn 100%. Đòn đánh này gọi là Đòn Tấn Công "Vương Miện Vàng" (Golden SAML/Golden Ticket).
> **Thực hành chuẩn:** Cấu hình IdP tự động Xoay Vòng Khóa (Key Rotation). Cứ 30 ngày, Keycloak tự sinh Cặp Khóa Public/Private MỚI, vứt khóa cũ đi. Giảm thiểu thời gian Hacker có thể lợi dụng khóa cũ.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Bạn muốn biết Keycloak (Với tư cách là IdP) công bố cho cả Thế giới biết về Năng lực của mình ở đâu?

Bất kỳ hệ thống Keycloak chuẩn mực nào cũng phơi ra 1 đường Link Công Khai (Discovery Document). Gọi là Sách Hướng Dẫn:
`GET https://<your-keycloak>/realms/master/.well-known/openid-configuration`

Mở Link đó ra, bạn sẽ thấy IdP tự hào tuyên bố (Bằng JSON):
- `"issuer": "https://<your-keycloak>/realms/master"` (Tên tao là Master Realm).
- `"jwks_uri": "https://.../certs"` (Đây là Public Key của tao, tụi mày lấy về mà soi chữ ký).
- `"authorization_endpoint": "https://.../auth"` (Gửi Khách Hàng tới đây để nhập Pass).
- `"token_endpoint": "https://.../token"` (Gửi Code tới đây để tao phát Token cho).

Các App (SP) bên ngoài chỉ cần tải Cục JSON này về là TỰ ĐỘNG biết cách Giao Tiếp với IdP mà dev không cần gõ cứng từng cái Link (Zero Configuration).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **IdP Chaining (Chuỗi Môi Giới IdP):**
  - Giả sử Khách hàng bấm Login ở Web A. 
  - Web A đá Khách sang Keycloak Của Công Ty (IdP 1). 
  - Tại Keycloak Của Công Ty, Khách lại bấm nút "Login bằng Microsoft". 
  - Keycloak Của Công Ty LẠI ĐÁ Khách sang Máy Chủ Microsoft (IdP 2). 
  - Microsoft hỏi Pass, Ok xong cấp Token cho Keycloak. Keycloak nhận được, lại bọc 1 lớp Token của nó cấp lại cho Web A.
  - Lúc này: Keycloak đóng 2 vai diễn (Vừa là IdP của Web A, vừa là Kẻ Đi Cầu Cạnh (SP) của Microsoft). Đòn Inception lồng ghép này gọi là IdP Chaining. Giải quyết bài toán Tích hợp Đa tầng Siêu Khủng của Tập đoàn.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong mô hình Federation, IdP lưu thông tin User (Tên, Email). Nhưng Service Provider (Ví dụ một phần mềm Kế toán) LẠI CŨNG CẦN lưu cái Tên và Email đó vào Database của nó để in Hóa Đơn. Làm sao SP đồng bộ thông tin từ IdP mà không copy Database?**
- **Junior:** Nó gọi API sang IdP lấy Data.
- **Senior:** Cơ chế Tiêm Thuộc Tính qua Token (Attribute Injection / Claims).
Khi IdP (Keycloak) xác thực xong, nó không chỉ quăng cái Chữ ký Xác nhận "Thằng này đúng", mà nó Tống Cổ (Inject) TOÀN BỘ DATA cần thiết (Tên: Nguyễn Văn A, Email: a@a.com, Avatar: link.jpg) VÀO TRONG RUỘT CỦA JWT / SAML Assertion.
Khi SP nhận được Token, nó Bổ cái Token đó ra. Lấy Dữ Liệu đó Tự Cập Nhật vào Bảng User Nội bộ của nó (Thao tác này gọi là Cập nhật Dữ liệu Vừa-Đúng-Lúc: Just-in-Time Update). SP Luôn Nhận Được Dữ Liệu Tươi Mới Nhất Mỗi Lần User Đăng Nhập mà không cần Đồng bộ API tốn công.

**2. Nếu Máy Chủ IdP (Keycloak) Của Công Ty Bị Sập Nguồn Chết Tươi 2 Tiếng Đồng Hồ. Toàn Bộ 100 Cái App (SP) Đang Chạy Có Bị Sập Không? Những User ĐANG Ở BÊN TRONG CÁC APP ĐÓ Có Bị Đá Ra Ngoài Không?**
- **Junior:** Sập Keycloak thì chết hết, ai cũng văng ra.
- **Senior:** Đây là Sức Mạnh Vô Song của Kiến trúc Stateless (Phi trạng thái).
- Câu trả lời là: **NGƯỜI Ở TRONG VẪN CHƠI TIẾP, NGƯỜI BÊN NGOÀI KHÔNG THỂ VÀO.**
IdP chỉ đóng vai trò PHÁT VÉ (Token Issuer).
Nếu User ĐÃ ĐĂNG NHẬP (Đã cầm cái vé JWT 15 phút trên tay). Máy chủ IdP sập, kệ nó. Thằng SP (Cái App Kế toán) chỉ check Chữ ký JWT cục bộ (Bằng thuật toán mã hóa) mà KHÔNG CẦN GỌI VỀ IdP. Vậy nên User đó vẫn quẩy tung nóc 15 phút trên App Kế Toán.
Tuy nhiên, Những Kẻ Lạ Mặt đòi Login Mới sẽ văng. Và Khi Vé 15 phút hết hạn, Thằng Kế toán muốn quay lại IdP đổi vé mới (Refresh Token) thì lúc này Mới Gặp Cửa Đóng Then Cài, và chính thức Bị Văng. Sự cách ly này (Fault Isolation) cứu vãn sinh mạng hệ thống rất nhiều.

**3. Khái niệm "Opaque Token" (Token Đục / Bí mật) khác với "Transparent Token" (Token Trong Suốt / JWT) ở Điểm nào? Khi làm IdP, vì sao Ngân Hàng thích dùng Opaque Token hơn?**
- **Junior:** JWT dễ xài hơn.
- **Senior:** JWT là Token Trong Suốt. Cấu trúc nó là Base64. Nghĩa là THẰNG SP (Web Kế toán) NHẬN ĐƯỢC JWT LÀ NÓ ĐỌC ĐƯỢC TẤT CẢ (Biết Tên, Email, Biết Quyền). Đồng thời IdP MẤT QUYỀN KIỂM SOÁT cái JWT đó sau khi nó bay đi.
**Opaque Token (Token Đục):** IdP (Keycloak) chỉ trả ra 1 Chuỗi Vô Nghĩa (VD: `xfh928sdj230`). Thằng SP nhận được chuỗi đó, HOÀN TOÀN MÙ TỊT, KHÔNG THỂ ĐỌC, KHÔNG THỂ CHECK CHỮ KÝ. BẮT BUỘC thằng SP phải cầm Cục Rác đó, CHẠY NGƯỢC VỀ CỔNG API CỦA IdP (Gọi là Introspection Endpoint) để hỏi: *"Anh ơi chuỗi này là của thằng nào, quyền gì?"*.
**Vì sao Ngân hàng thích:** Vì lúc nào SP cũng phải chạy về IdP hỏi, nên MỌI QUYỀN LỰC NẰM GỌN TRONG TAY IdP. Nếu Giám Đốc bấm Xóa User, lúc SP đem Opaque Token lên hỏi, IdP Phán luôn: "Thằng này tao khóa rồi, cút!". Lệnh Hủy Quyền (Revocation) có Tác Dụng LẬP TỨC 100%. Đổi lại: Tốn Băng Thông Cực Lớn vì API gánh tải gấp 10 lần. (Trade-off: An toàn tuyệt đối vs Tốc độ).

---

## 7. Tài liệu tham khảo (References)
- **OAuth 2.0 RFC 6749:** Authorization Server Role.
- **Keycloak Documentation:** Keycloak Server Administration (Realm as Identity Provider).
