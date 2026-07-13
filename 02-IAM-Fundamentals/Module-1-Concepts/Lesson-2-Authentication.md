# Lesson 2: Xác thực (Authentication - AuthN)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trả lời cho câu hỏi: *"Bạn là ai?"* (Who are you?). Hiểu được cơ chế Xác thực không chỉ có Mật khẩu, mà nó là sự giao tiếp giữa Lời Tuyên Bố (Claim) và Bằng Chứng (Proof).
> *Thuật ngữ viết tắt chuẩn quốc tế của Authentication là **AuthN**.*

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Bản chất của Authentication (AuthN)
Hãy tưởng tượng bạn ra Cửa khẩu sân bay:
- **Tuyên bố (Claim):** Bạn chỉ tay vào mặt mình và nói với Công an: *"Tôi là Nguyễn Văn A"*. Công an không bao giờ tin bạn. Bất kỳ ai cũng có thể tự xưng là A.
- **Bằng chứng (Proof/Credential):** Công an yêu cầu bạn đưa ra Bằng Chứng. Bạn móc ra cái Hộ Chiếu (Passport). Công an kiểm tra phôi thật, ảnh khớp khuôn mặt.
Quá trình Đưa Bằng Chứng để chứng minh Lời Tuyên Bố chính là **Xác thực (Authentication)**.

### 1.2. Ba Yếu Tố Cốt Lõi (The 3 Factors of Authentication)
Trong thế giới Mật mã, để chứng minh bạn là chính bạn, bạn chỉ có 3 con đường (Được gọi là 3 Yếu tố):

1. **Knowledge Factor (Thứ bạn BIẾT):** 
   - Điển hình: Mật khẩu (Password), Mã PIN, Câu hỏi bảo mật (Con vật nuôi đầu tiên của bạn tên gì?).
   - *Nhược điểm:* Quá yếu, dễ bị Hacker đoán, bị Keylogger ghi lén, hoặc Chủ nhân lười hay đặt `123456`.
2. **Possession Factor (Thứ bạn CÓ):** 
   - Điển hình: Cái Điện thoại của bạn (Để nhận SMS OTP), Thẻ YubiKey NFC cắm vào USB, Thẻ từ quẹt cửa.
   - *Nhược điểm:* Có thể bị Trộm cắp vật lý (Mất điện thoại), bị làm giả (SIM Swapping).
3. **Inherence Factor (Thứ thuộc về CƠ THỂ bạn):** 
   - Điển hình: Sinh trắc học (Biometrics) như Vân tay (TouchID), Khuôn mặt (FaceID), Võng mạc, Giọng nói.
   - *Nhược điểm:* Nếu bị lộ (Ví dụ hacker in 3D vân tay của bạn), bạn không thể "Đổi Vân Tay" như Đổi Mật Khẩu được. Nó là vĩnh viễn. Đòi hỏi thiết bị đọc đắt tiền.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Cơ chế Xác thực Vượt rào (Challenge-Response Mechanism). Đây là cách mọi giao thức (OIDC, Kerberos, SSL) giao tiếp đằng sau hậu trường.

```mermaid
sequenceDiagram
    participant User as Người dùng
    participant AuthServer as Keycloak (AuthN)

    User->>AuthServer: Tuyên Bố (Claim): "Cho tôi vào, tôi là Alice"
    Note over AuthServer: Máy chủ KHÔNG BAO GIỜ chủ động hỏi Pass.<br/>Nó sẽ quăng ra một BÀI TOÁN (Challenge).
    AuthServer-->>User: Bài toán 1 (Challenge): "Nếu mài là Alice, hãy nói cho tao Biết cái Password của Alice là gì?"
    
    User->>AuthServer: Trả lời 1 (Response): "Pass là: i_love_bob"
    AuthServer->>AuthServer: Máy chủ mang vào Hash PBKDF2 khớp với DB.<br/>Đúng Pass!
    
    Note over AuthServer: Nếu hệ thống bật Xác thực 2 bước (MFA).<br/>Máy chủ tung bài toán thứ 2.
    AuthServer-->>User: Bài toán 2 (Challenge): "Tốt. Giờ hãy điền 6 số ngẫu nhiên tao vừa gửi vào CÁI ĐIỆN THOẠI CỦA ALICE"
    
    User->>AuthServer: Trả lời 2 (Response): "Mã là: 556677"
    AuthServer->>AuthServer: So khớp đúng mã thuật toán HOTP/TOTP.
    
    AuthServer-->>User: THÀNH CÔNG (Authenticated). Cấp quyền vào thành (Cấp Token).
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Sự Sụp Đổ của Mật Khẩu (The Death of Passwords)**
> Trong ngành IAM hiện đại, "Thứ bạn BIẾT" (Password) bị coi là rác rưởi. Vì Con người rất ngu ngốc (Hay đặt pass yếu, dùng lại 1 pass cho 10 trang web).
> **Best Practice:** Tuyệt đối không dùng Password làm Yếu tố Xác thực DUY NHẤT. BẮT BUỘC phải ghép nó với 1 yếu tố khác (Knowledge + Possession) -> Cái này gọi là MFA (Multi-Factor Authentication - Xác thực đa yếu tố). Sẽ học sâu ở Lesson 8.

> [!CAUTION]
> **Giới hạn số lần thử (Brute-Force Protection)**
> Một cuộc tấn công Brute-force là Hacker dùng máy tính chạy 1 tỷ cái Mật khẩu/Mã OTP thử liên tục cho đến khi đúng.
> Hệ thống Xác thực Chuẩn mực phải có cờ đếm (Fail Counter): "Chỉ cho phép nhập sai Tối đa 5 lần. Sai quá 5 lần KHÓA TÀI KHOẢN trong 15 phút. Sai quá 15 lần KHÓA VĨNH VIỄN chờ Admin mở". (Keycloak hỗ trợ bật tính năng này trong 1 Click).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Kiến trúc **Authentication Flows (Luồng Xác Thực)** siêu việt của Keycloak:
Thay vì Hard-code (Code cứng) bằng lệnh `if(pass_dung){ if(otp_dung){ login() } }`, Keycloak dùng khái niệm Cây Quyết Định (Decision Tree) gọi là Flows.

Bạn vào Menu `Authentication` của Keycloak, bạn có thể thiết kế một Trải nghiệm Đăng nhập Độc Nhất Vô Nhị bằng cách kéo thả:
- **BƯỚC 1 (REQUIRED):** Nhập Username và Password. (Nếu sai, đá văng ngay lập tức).
- **BƯỚC 2 (ALTERNATIVE):** (Chọn 1 trong 2).
  - Hoặc là Bấm YubiKey (WebAuthn).
  - Hoặc là Nhập OTP qua App Google Authenticator.
- **BƯỚC 3 (CONDITIONAL):** (Điều kiện rẽ nhánh).
  - NẾU IP người dùng từ Mỹ (Nước ngoài) -> BẮT BUỘC Gửi thêm SMS bắt nhập (OTP rủi ro cao).
  - NẾU IP người dùng từ Công ty (Mạng LAN Nội bộ) -> SKIP Bước 3, cho vào luôn.

Bằng Authentication Flows, Keycloak biến việc đổi mới quy trình Xác thực trở nên linh hoạt mà không tốn 1 dòng Code Java nào của Backend.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Cú tát của Hệ Sinh Trắc Học (Biometrics Fallback):**
  - Công ty bạn yêu cầu 100% nhân sự đăng nhập bằng Quét Khuôn Mặt (FaceID).
  - Có một nhân viên nữ làm kỹ thuật viên hóa chất, bị tai nạn nổ phòng thí nghiệm, khuôn mặt băng bó toàn bộ (Hoặc điện thoại bị hỏng Camera). Cô ấy bị Nhốt Vĩnh Viễn bên ngoài hệ thống.
  - **Quy tắc Kiến trúc:** Khi dùng "Inherence Factor" (Cơ thể), BẮT BUỘC phải luôn có **Luồng Dự Phòng (Fallback / Recovery Codes)**. Keycloak cung cấp tính năng "Recovery AuthN Code" (Sinh ra 10 mã ngẫu nhiên cho in ra giấy cất vào két sắt). Khi Vân tay / Khuôn mặt bị phế, người dùng nhập 1 trong 10 mã trên Giấy để cứu hộ tài khoản.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Giải thích phương thức Xác thực "HTTP Basic Auth"? Tại sao nó bị coi là Kẻ Thù của Bảo Mật Hiện Đại và khi nào thì bắt buộc phải dùng nó?**
- **Junior:** Basic auth là cái bảng popup hiện ra bắt gõ username pass. Nó cũ rồi nên không dùng.
- **Senior:** "HTTP Basic Auth" là chuẩn Xác thực tối cổ (RFC 7617). Trình duyệt gom Username và Password ghép lại thành chuỗi `username:password`, sau đó mã hóa Base64 (Chứ không phải Hash), và NÉT vào Header `Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=` vào MỖI MỘT REQUEST GỬI LÊN MÁY CHỦ.
**Tại sao là Kẻ thù:**
1. Base64 là Bản Rõ (Plaintext). Nếu không có HTTPS, Hacker bắt được Request là đọc được Pass lập tức.
2. Nó ĐÍNH KÈM PASS ở MỌI REQUEST. Tăng diện tích tấn công (Attack Surface).
3. Nó Không Hỗ Trợ MFA (Đăng nhập 2 bước) hay Single Sign-On (SSO).
**Khi nào bắt buộc dùng:** Giao tiếp Server-to-Server (M2M) ở các hệ thống Di Sản (Legacy REST API Cũ), hoặc khi Cấu hình Hardware Router (Cisco) không hỗ trợ giao diện Web phức tạp. 

**2. Khái niệm "Risk-based Authentication" (Xác thực Dựa trên Rủi ro) hay "Adaptive Authentication" (Xác thực Thích ứng) là gì? Hệ thống làm sao biết rủi ro mà chặn?**
- **Junior:** Nó phát hiện hacker thì nó chặn.
- **Senior:** Đây là Đỉnh cao của Trải nghiệm Người dùng (UX) và Bảo mật (Security) giao thoa.
Bình thường, hệ thống luôn bắt nhập Pass + SMS OTP (Quá phiền phức).
Với "Risk-based Auth" (Keycloak có hỗ trợ thông qua Conditional Flows), hệ thống sẽ đánh giá Điểm Rủi Ro (Risk Score) ở mỗi lần Đăng nhập, dựa trên:
1. IP Mạng (Có phải từ quán Net không? Có đổi quốc gia đột ngột trong 1 giờ - Impossible Travel không?)
2. Thiết bị (User Agent này có lạ không? Màn hình điện thoại mới đổi độ phân giải à?)
3. Thời gian (Tự nhiên đăng nhập lúc 3h sáng?)
Nếu Score THẤP (Bạn đang ngồi ở Máy tính công ty, mạng Wifi công ty): Hệ thống CHỈ BẮT NHẬP PASS (hoặc tự tin cấp quyền không cần hỏi gì - SSO).
Nếu Score CAO (Bạn dùng Laptop mới ở Châu Phi): Hệ thống Lập tức BẬT CHẾ ĐỘ PHÒNG THỦ, Bắt ép bạn phải Nhập Mã OTP gửi về Email, sau đó hỏi Câu hỏi bảo mật, rồi mới cho vào (Step-up Authentication). 

**3. Tại sao Câu hỏi Bảo mật (Security Questions - Ví dụ: Con vật cưng của bạn tên gì) lại bị Viện Tiêu Chuẩn Công Nghệ Mỹ (NIST) cấm sử dụng trong Môi trường Enterprise?**
- **Junior:** Tại dễ bị đoán ra.
- **Senior:** Nó vi phạm nghiêm trọng tính Bảo Mật Kín (Secrecy).
Mật khẩu là một thứ Vô Nghĩa (Ví dụ: `aB#99x`). Nếu bạn không nói, không ai biết.
Câu hỏi bảo mật (Mẹ bạn sinh ra ở đâu? Tên trường cấp 1 của bạn?) là Dữ Kiện Công Khai (Public Facts). Hacker chỉ cần lướt Facebook của bạn, lướt Instagram 5 phút (Social Engineering) là moi ra được Toàn Bộ câu trả lời. 
Thứ hai, nó tạo ảo giác an toàn. User không nhớ con chó tên Mực hay Muc hay muc, dẫn đến User tức giận và gọi điện réo Support IT khóa cả ngày. NIST SP-800-63 chính thức gạch bỏ nó khỏi danh sách Knowledge Factor hợp lệ ở mức cao.

---

## 7. Tài liệu tham khảo (References)
- **NIST SP 800-63B:** Authentication and Lifecycle Management.
- **Keycloak Documentation:** Authentication Flows and Conditions.
