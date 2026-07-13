# Lesson 6: Ứng dụng Khách (Service Provider / Relying Party)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Nắm bắt Vai trò của Kẻ Tiêu Thụ Danh Tính. Trong bàn đàm phán IAM, nếu IdP là Cục Quản Lý Dân Cư cấp Hộ chiếu, thì Service Provider (SP) chính là Nhân viên Hải quan sân bay soi Hộ chiếu.
> *Thuật ngữ: Trong SAML nó gọi là Service Provider (SP). Trong OIDC nó gọi là Relying Party (RP).*

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Service Provider (SP) là gì?
**Service Provider (SP)** là bất kỳ Ứng dụng, Phần mềm, Máy chủ API nào mà Khách hàng muốn TRUY CẬP để Lấy Dịch Vụ (Ví dụ: Web Ngân hàng, App Đặt xe, Phần mềm HR).
Tôn chỉ tuyệt đối của SP:
- **Nó Mù Tịt về Mật Khẩu:** Nó KHÔNG BAO GIỜ lưu Mật khẩu, cũng KHÔNG BAO GIỜ được phép hỏi Mật khẩu người dùng.
- **Nó Dựa Dẫm (Relying):** Nó không có khả năng tự phán xét Khách là thật hay giả. Nó ỦY THÁC 100% sinh mạng bảo mật của nó vào tay IdP (Keycloak).
- **Nó Chỉ Biết Đọc Thư:** Nó chỉ tiếp khách khi khách mang tới một Bức Thư (Token/Assertion) ĐÃ ĐƯỢC KÝ BỞI ĐÚNG IDP.

### 1.2. Mối quan hệ "Đẩy và Đỡ"
- Khách vào SP -> SP chặn lại ĐẨY khách về IdP.
- IdP hỏi Pass -> Khách qua ải, IdP ĐẨY Token về lại SP.
- SP ĐỠ lấy Token, Đọc chữ ký, Mở cửa cho khách.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Bên trong Trạm Gác của Service Provider: Làm sao nó biết Token là thật?

```mermaid
graph TD
    subgraph "Bên Trong Service Provider (Spring Boot / NodeJS)"
        Token(Nhận JWT Token từ Khách)
        
        Token --> Check1{Bước 1: Cấu Trúc Header}
        Check1 -->|Có thuật toán RS256| Check2{Bước 2: Giải mã Chữ Ký}
        
        Check2 -->|Lấy Public Key của IdP ra soi| Check3{Bước 3: Còn Hạn Không?}
        Check3 -->|So cột exp với giờ hiện tại| Check4{Bước 4: Đúng Khán Giả Không?}
        
        Check4 -->|So cột aud xem có đúng là gửi cho Tao không?| Success[CẤP QUYỀN VÀO APP]
    end
    
    Note over Token,Success: Toàn bộ quá trình soi JWT này diễn ra OFFLINE ngay tại Máy Chủ SP<br/>Nó diễn ra trong 0.1 Mili-giây.<br/>KHÔNG MỘT GÓI TIN NÀO CHẠY VỀ IDP ĐỂ HỎI. Đó là Stateless.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Xác minh Audience (Khán giả) - Chống Đánh Tráo Token**
> Một Token của Keycloak luôn chứa trường `aud` (Audience - Khán giả mục tiêu).
> Giả sử Hacker đăng nhập vào App Bán Hàng (Lấy được Token xịn, `aud=app-ban-hang`). Sau đó Hacker cầm Token này Bắn vào App Kế Toán.
> Nếu App Kế Toán (SP) code ẩu, chỉ soi Chữ Ký (Chữ ký do Keycloak ký là đúng 100%). Nó sẽ Mở Cửa. Đây là thảm họa.
> **Luật Sinh Tử:** SP Bắt Buộc Phải kiểm tra trường `aud`. App Kế Toán phải kiểm tra `aud == app-ke-toan` thì mới cho vào. Khác là Block lập tức.

> [!CAUTION]
> **Vá Lỗ Hổng Nâng Cấp Kẻ Tấn Công (Downgrade Attack)**
> Header của JWT ghi rõ thuật toán ký: `{"alg": "RS256"}`.
> Hacker lấy JWT của bạn, sửa Header thành `{"alg": "none"}` (Không mã hóa), xóa mẹ Chữ Ký ở đuôi đi, rồi gửi cho SP.
> Rất nhiều Thư viện JWT cùi bắp ngày xưa đọc thấy `alg=none`, nó Vui Vẻ cho qua luôn (Vì không cần check chữ ký).
> **Best Practice:** Mọi SP phải Cấu Hình CỨNG thuật toán duy nhất được chấp nhận (Whitelist `RS256`). Mọi thuật toán khác ném vào thùng rác.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Làm sao để Khai báo một Service Provider với Keycloak?
Trong Keycloak, SP được gọi bằng thuật ngữ: **Client**.
Bạn vào Admin Console -> Menu `Clients` -> `Create Client`.

Có 3 loại Client cấu hình phổ biến nhất:
1. **Confidential Client (Máy Chủ Kín):** Dành cho Spring Boot, NodeJS Backend. Vì nó là Máy chủ Cất dưới gầm bàn, nó có thể Giữ Bí Mật được Mật khẩu riêng của nó. Nó xài `Client ID` và `Client Secret`.
2. **Public Client (Ứng dụng Lộ Thiên):** Dành cho React, Angular, Mobile App (iOS/Android). Vì Mã nguồn phơi trên máy khách hàng, Hacker F12 là lấy được Source. Bọn này TUYỆT ĐỐI KHÔNG CÓ `Client Secret`. Bọn nó phải xài đòn PKCE (Proof Key for Code Exchange) để chống cướp Token.
3. **Bearer-Only (Tàn dư quá khứ):** Chỉ dùng để Validate Token, không có khả năng Redirect Login. (Đã bị loại bỏ trong OIDC chuẩn hiện đại).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Khi SP Bị Hack Toàn Diện:**
  - Nếu Máy chủ Kế Toán (SP) bị Cài Mã Độc (Ransomware), Database Kế toán bị mã hóa.
  - Điều tuyệt vời nhất của Kiến trúc Tách biệt: Mật khẩu User, Thông tin Nhạy cảm của User nằm ở Keycloak (IdP). Hacker đập nát cái SP Kế toán cũng không moi ra được Pass của Giám Đốc. Tầm ảnh hưởng (Blast Radius) bị giới hạn gọn gàng trong 1 App.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong OIDC, tại sao thằng Service Provider (RP) được cấp cái JWT tên là "ID Token", mà nó KHÔNG DÙNG CÁI ĐÓ ĐỂ GỌI API BACKEND?**
- **Junior:** Chắc JWT nào cũng như nhau.
- **Senior:** Phải phân định cực kỳ Rạch Ròi ranh giới của 2 loại Token:
**ID Token:** Là Tấm Thẻ Căn Cước. Nó dùng để chứng minh "Tôi là Alice, email là alice@gmail.com". Thằng Frontend (SP) Cầm nó để ĐỌC, hiện Lời Chào "Hello Alice", đổi Avatar góc phải màn hình. Tuyệt đối không dùng để lấy quyền.
**Access Token:** Là Chiếc Chìa Khóa Phòng. Nó không ghi tên tuổi, nó chỉ ghi `scopes: read_bank_data`. Frontend CẦM NÓ NHƯNG KHÔNG ĐỌC, mà quăng nó vào HTTP Header `Bearer` để gửi sang API Backend. Cầm lộn Căn Cước (ID Token) đi đập cửa (API) là lỗi kiến trúc 401 Unauthorized ngay lập tức.

**2. Nếu SP (Spring Boot) của tôi nằm ở Mạng Nội Bộ (Không có Internet). Làm sao nó soi được Chữ ký Token của Keycloak (Vốn yêu cầu tải Public Key từ URL `jwks_uri`)?**
- **Junior:** Tự tải public key về quăng vào thư mục code.
- **Senior:** Đó chính xác là giải pháp duy nhất. Gọi là **Hardcoded Public Key (Khóa Công Khai Tĩnh)**.
Bạn lên Keycloak, tải cái File Certificate (PEM) chứa Public Key về. Gói nó vào trong thư mục `resources/` của Spring Boot. Spring Boot khi giải mã JWT sẽ không cần kết nối mạng tải Key mà đọc luôn file ở Ổ Cứng.
**Hậu Quả Sinh Tử:** Nếu Keycloak Tự Động Xoay Vòng Khóa (Key Rotation - Đổi khóa mỗi 30 ngày), cái File Cứng của bạn sẽ bị Lỗi Thời (Outdated). Toàn bộ Request bị Đánh rớt vì sai chữ ký. Bắt buộc phải có Quy trình CI/CD tự động bơm File Key mới mỗi khi Keycloak đổi khóa.

**3. Làm sao để Kẻ Xấu không thể tự viết một cái App lậu (Fake SP), sau đó Đá người dùng về Keycloak, lừa người dùng nhập Pass rồi Lấy Token trả về App lậu đó?**
- **Junior:** Bọn nó không có API Key nên không lấy được.
- **Senior:** Bí mật nằm ở Ràng buộc **Valid Redirect URIs (Danh sách URL cho phép)** trên Keycloak.
Trong cấu hình Client (SP) ở Keycloak. Quản trị viên BẮT BUỘC phải điền Cứng 1 cái URL (Ví dụ: `https://app.congty.com/callback`).
Khi App lậu gọi Keycloak yêu cầu Login, nó truyền Parameter `redirect_uri=https://app-cua-hacker.com/callback`.
Keycloak nhận được, nó Lôi cái Parameter đó ra Soi với Danh sách đã cấu hình. Nó thấy chữ "hacker.com" KHÔNG CÓ TRONG DANH SÁCH.
BÙM! Lập tức Keycloak quăng ra màn hình Báo Lỗi: `Invalid Redirect URI`. Hacker không có bất kỳ cơ hội nào cướp Token, dù User có sẵn sàng nhập Pass đi chăng nữa. Đây là phòng tuyến Thép của OAuth2/OIDC.

---

## 7. Tài liệu tham khảo (References)
- **OAuth 2.0 RFC 6749:** Client Registration and Types.
- **IETF RFC 8725:** JSON Web Token Best Current Practices.
