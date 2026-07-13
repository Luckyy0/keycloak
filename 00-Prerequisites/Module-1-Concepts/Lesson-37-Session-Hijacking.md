# Lesson 37: Lỗ hổng Session Hijacking (Cướp Phiên Đăng Nhập)

> [!NOTE]
> **Category:** Theory & Security (Lý thuyết & Bảo mật)
> **Goal:** Kết bài hoàn hảo cho hệ thống Phòng thủ Web. Tổng hợp các kỹ thuật Cướp Phiên (Session Hijacking), tại sao nó là Nỗi Ám Ảnh tột cùng, và Nghệ thuật phòng ngự Chiều Sâu (Defense in Depth) bằng Step-up Authentication.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Session Hijacking là gì?
Nếu Fixation (Bài 36) là việc Hacker ÉP BẠN xài Session của hắn. Thì **Hijacking (Cướp phiên)** là hành động bạo lực nhất: Trình duyệt của bạn đang ôm một cái Cookie Login xịn, Hacker dùng mọi thủ đoạn để BẮT CÓC cái Cookie/JWT đó mang về máy của hắn.
Một khi đã có Token trong tay, Hacker sẽ thay mặt bạn truy cập vào Ngân hàng. Máy chủ Ngân hàng KHÔNG CÓ CÁCH NÀO PHÂN BIỆT ĐƯỢC Hacker và Bạn, bởi vì chuẩn RESTful là Phi trạng thái (Stateless), AI CẦM TOKEN LÀ NGƯỜI ĐÓ CÓ QUYỀN.

### 1.2. 3 Con Đường Bắt Cóc Điển Hình
1. **XSS (Cross-Site Scripting):** Kinh điển nhất. Mã độc JS đọc LocalStorage lấy JWT, hoặc đọc Document.cookie (Nếu lỡ quên bật HttpOnly) và Gửi về Server của Hacker.
2. **Man-in-the-Middle (MitM - Người đứng giữa):** Bạn ngồi quán Cafe Wifi công cộng. Bạn vào web không có HTTPS (Hoặc có nhưng không cài HSTS). Hacker chụp toàn bộ các gói tin bay trong không khí, đọc phần HTTP Header và chép cái Cookie của bạn ra.
3. **Malware/Trojans (Mã độc trên Máy tính/Điện thoại):** Hacker không thèm hack Web. Hắn dụ bạn cài App "Chỉnh sửa ảnh" chứa mã độc. Mã độc này chui vào thư mục cài đặt của Google Chrome trên Ổ cứng (File SQLite Database), Copy nguyên cái File chứa Cookie của bạn gửi về máy chủ. (Đòn này HttpOnly hay HTTPS cũng đều vô dụng).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Tầm quan trọng của Cờ Môi Trường (Cookie Flags) trong việc phòng thủ:

```mermaid
graph TD
    subgraph "Hành trình của một Session Cookie / JWT"
        C[Máy chủ Server trả Cookie]
        
        Flag_Secure{Có cờ Secure?}
        Flag_HttpOnly{Có cờ HttpOnly?}
        Flag_SameSite{Cờ SameSite?}
        
        C --> Flag_Secure
        Flag_Secure -->|Có| HTTPS[Chỉ truyền qua Cáp Mạng HTTPS, Chống MitM]
        Flag_Secure -->|Không| HTTP[Truyền lọt qua HTTP trần, Dễ bị Bắt Cóc ở Quán Cafe]
        
        C --> Flag_HttpOnly
        Flag_HttpOnly -->|Có| JS_Block[Khóa mõm Javascript. Chống XSS chôm Cookie]
        Flag_HttpOnly -->|Không| JS_Read[JS đọc dễ dàng. LocalStorage cũng chung số phận]
        
        C --> Flag_SameSite
        Flag_SameSite -->|Strict/Lax| SOP_Block[Cấm gửi chéo trang. Chống CSRF]
        Flag_SameSite -->|None| Open[Thả cửa cho gọi chéo trang (Phải cẩn thận)]
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!CAUTION]
> **Tuyệt vọng khi Token bị cướp: Thu hồi (Revocation) là chưa đủ**
> Khi Hacker lỡ chôm được Access Token (Sống 15 phút). Bạn không thể "Xóa Session" ở Backend được, vì Access Token là Phi trạng thái (JWT kiểm tra chữ ký tại chỗ). Hacker sẽ tung hoành tự do trong 15 phút đó.
> **Thực hành chuẩn:** Access Token OIDC BẮT BUỘC phải rất ngắn (1-5 phút). Còn Refresh Token (Dùng để đổi Access Token mới) bắt buộc phải cấu hình **Mỗi Lần Đổi Là Chết (Rotation)**. Nếu Hacker cầm Refresh Token đổi lấy Access Token, Keycloak sẽ phát hiện ra "Có 2 người cùng xài 1 cái Refresh Token", nó lập tức chém chết TẤT CẢ các Token của User đó.

> [!IMPORTANT]
> **Tuyệt kỹ Phòng thủ Cuối Cùng: Step-up Authentication (Tái xác thực)**
> Dù bạn cấu hình giỏi cỡ nào, Hacker cài Virus vào Laptop của Sếp bạn là hắn ôm trọn Cookie.
> **Kiến trúc Zero Trust:** Dù đang có Cookie Đăng nhập hợp lệ, khi thực hiện các Hành động Cực kỳ Nhạy cảm (Đổi Mật Khẩu, Chuyển tiền > 50 Triệu, Xóa Database). Máy chủ API BẮT BUỘC phải bắt User **NHẬP LẠI MẬT KHẨU hoặc QUÉT MẶT / VÂN TAY (Biometrics/OTP)** ngay lập tức.
> Điều này đảm bảo: Dù Hacker trộm được Cookie, hắn bấm Nút Xóa DB, hắn sẽ bị hỏi Vân Tay của Sếp bạn (Thứ mà Hacker không thể cướp được từ xa).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Keycloak cung cấp bộ cấu hình Vòng Đời Phiên (Session Lifespan) cực kỳ khắt khe để giảm thiểu thời gian vàng của Hacker.

Vào Keycloak -> `Realm Settings` -> `Sessions`:
- **SSO Session Idle (Mặc định 30 phút):** Nếu User cắm chuột, đi uống Cafe 30 phút không chạm vào Trình duyệt. Session bị Hủy. (Ngăn Hacker lén ngồi vào máy lúc nạn nhân đi WC).
- **SSO Session Max (Mặc định 10 tiếng):** DÙ User có click liên tục, cứ tròn 10 tiếng kể từ lúc Login buổi sáng, Session BẮT BUỘC BỊ HỦY. Họ phải đăng nhập lại vào cuối ngày. (Ngăn Cookie sống dai thành hóa thạch, bị Hacker bắt được và xài mãi mãi).
- **Client Session Idle/Max:** Áp dụng riêng lẻ cho từng App (Ví dụ App Kế Toán Idle 5 phút là văng, App Bán Hàng 30 phút mới văng).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Trói buộc IP (IP Binding):** Nhiều hệ thống cũ phòng thủ Session Hijacking bằng cách: Khi Login (IP là `1.1.1.1`), Server gán cái IP đó vào Token. Lát sau Hacker (IP là `2.2.2.2`) cầm Token đó lên gọi API, Server thấy khác IP nên chặn.
  - **Vỡ trận với Mạng Di Động:** Ngày nay (Thời đại 4G/5G). Người dùng vừa đi đường vừa lướt Web, IP 4G của họ BỊ ĐỔI LIÊN TỤC mỗi khi đi ngang qua một trạm phát sóng (Cell tower) khác. Nếu áp dụng IP Binding, User sẽ bị Văng Đăng Nhập liên tục gây ức chế tột độ. Kỹ thuật này bị loại bỏ hoàn toàn trong kiến trúc IAM hiện đại (Chuyển sang dùng Device Fingerprinting hoặc Token Rotation).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. "Access Token Binding (DPoP - Demonstrating Proof-of-Possession)" là chuẩn OIDC tương lai để giải quyết dứt điểm Session Hijacking. Cơ chế của nó khác gì JWT thông thường?**
- **Junior:** Nó mã hóa JWT mạnh hơn.
- **Senior:** JWT hiện tại là "Bearer Token" (Ai cầm được chìa khóa là mở được cửa).
DPoP nâng cấp nó lên thành Token có chữ ký của THIẾT BỊ. Khi Frontend Login, nó tự sinh ra một Cặp Khóa Public/Private Key (Lưu trên bộ nhớ máy hoặc TPM chip). Keycloak sẽ gắn cái Public Key đó vào ruột JWT.
Mỗi lần Frontend gọi API, nó phải lấy Private Key ĐÓNG DẤU CHỮ KÝ (PoP) lên gói HTTP Request. API Gateway nhận được, sẽ lấy Public Key (Trích từ JWT) ra giải mã chữ ký.
**Lợi ích kinh hoàng:** Hacker chôm được cái JWT mang về máy hắn. Nhưng Hắn KHÔNG THỂ BẮT CÓC ĐƯỢC cái Private Key (Vốn dính chặt vào Chip của Nạn nhân). Do đó hắn gửi JWT mà không có Chữ ký PoP -> API từ chối. Token bị cướp trở thành một mẩu giấy vụn vô dụng.

**2. Kỹ thuật "Session Token Rotation" của Refresh Token hoạt động như thế nào để phát hiện Hacker đã ăn cắp Token?**
- **Junior:** Cứ 1 tiếng đổi token 1 lần cho an toàn.
- **Senior:** Rotation (Xoay vòng) là việc: MỖI MỘT LẦN Frontend mang Refresh Token (RT_1) lên đổi, Keycloak sẽ cấp cho Frontend Access Token mới VÀ MỘT REFRESH TOKEN MỚI (RT_2). Cái RT_1 lập tức bị VÔ HIỆU HÓA.
**Kịch bản phát hiện:** Hacker ăn cắp được cái `RT_1` (Lúc chưa đổi).
Nạn nhân xài web, tự động đổi `RT_1` lấy `RT_2` xài bình thường. Máy chủ ghi chú: "Ok, RT_1 đã đổi thành RT_2".
Ngày hôm sau, Hacker mang cái `RT_1` (Đồ ăn cắp) lên xin Keycloak cấp Access Token.
Keycloak thấy: "Khoan đã! Cái `RT_1` này ĐÃ ĐƯỢC ĐỔI RỒI MÀ (Bởi nạn nhân hôm qua). Tại sao bây giờ lại có kẻ dùng nó lần 2???". Keycloak lập tức phát giác có KẺ TRỘM Token. Nó CHÉM CHẾT cái `RT_1`, và TRUY TÌM CHÉM CHẾT LUÔN cái `RT_2` đang nằm trên máy nạn nhân. Bắt Nạn nhân Đăng nhập lại để xóa sạch tàn dư của Hacker. 

**3. Làm thế nào Hacker có thể cướp được Cookie khi nạn nhân dùng mạng Wifi Công cộng dù Ngân hàng ĐÃ CÀI HTTPS? (SSL Stripping)**
- **Junior:** Bọn hacker bẻ khóa được HTTPS.
- **Senior:** Hacker không bẻ khóa mã hóa. Hắn dùng **SSL Stripping (Lột trần SSL)**.
Ngân hàng có HTTPS. Nhưng Nạn nhân ở Quán Cafe thường lười, gõ thẳng `bank.com` vào Trình duyệt (Tức là mặc định HTTP cổng 80).
Hacker thao túng Router Wifi Quán Cafe (Man in the middle). Hắn BẮT LẤY cái Request cổng 80 đó.
Hacker tự mình MỞ MỘT KẾT NỐI HTTPS BÍ MẬT với Ngân hàng. Lấy toàn bộ trang Web Ngân hàng về.
Sau đó, Hacker XÓA SẠCH toàn bộ chữ `https` trong Source Code thành `http`, và gửi trang web "TRẦN TRUỒNG" đó cho Nạn nhân. 
Nạn nhân điền Password, gửi Form qua HTTP. Hacker đứng giữa BẮT SẠCH Password và Cookie, sau đó hắn mới gửi qua HTTPS xịn lên Ngân hàng.
**Vũ khí chống lại:** Header `HSTS` (Đã học bài 30). Khi web có HSTS, Trình duyệt tự ĐÓNG CHẾT đường HTTP lại, ép gọi HTTPS từ mili-giây đầu tiên, vô hiệu hóa hoàn toàn SSL Stripping.

**4. Khi thiết kế "Step-up Authentication" (Tái xác thực) cho chức năng Chuyển Tiền. Nếu tôi dùng hàm Yêu Cầu Nhập Lại Pass của Keycloak (prompt=login). Điều này có hoàn toàn chặn được Session Hijacking không?**
- **Junior:** Chặn được vì hacker không biết pass.
- **Senior:** Rất An Toàn, nhưng CÓ THỂ LÀ CHƯA ĐỦ.
Nếu Hacker xài mã độc (Keylogger) cài thẳng trên máy Nạn nhân. Hắn cướp Cookie, hắn cướp luôn cả Password (Bằng cách quay lén màn hình lúc nạn nhân gõ pass). Nếu bạn Step-up bằng Password, Hacker chỉ cần lấy Pass đó gõ vào là xong.
Để Step-up đạt chuẩn Zero Trust cấp Ngân Hàng, Phương thức Xác thực Nhập Lại BẮT BUỘC PHẢI KHÁC VỚI (Out-of-band) phương thức Đăng nhập ban đầu.
Ví dụ: Đăng nhập bằng Pass. Nhưng khi Chuyển tiền thì Step-up bằng Gửi tin nhắn SMS OTP vào Điện thoại (Hacker hack máy tính nhưng không hack được Điện thoại di động vật lý), hoặc Yêu cầu cắm USB Token (FIDO2/WebAuthn).

**5. Lỗ hổng "Session Fixation" và "Session Hijacking" đều dẫn đến việc Hacker chui vào tài khoản người dùng. Sự khác biệt về Kiến trúc để vá 2 lỗi này là gì?**
- **Junior:** Fixation thì tạo lại session, Hijacking thì cấm copy.
- **Senior:** Cực kỳ chính xác về tư duy.
- **Session Fixation** là Lỗi của Hệ thống (Thiết kế luồng Nâng quyền - Privilege Escalation bị sai). Cách vá 100% nằm ở Backend: `Regenerate Session ID on Login`. Lỗi này fix Xong Là Dứt Điểm.
- **Session Hijacking** là một Vấn đề Môi trường (Hệ điều hành nạn nhân có mã độc, mạng wifi bị nghe lén). API Backend của bạn không thể cấm hệ điều hành chép file Cookie. Do đó, KHÔNG BAO GIỜ có thể fix dứt điểm 100% Session Hijacking. 
Cách vá Hijacking mang tính chất Giảm thiểu Rủi ro (Mitigation) ở tầng Kiến trúc: Rút ngắn TTL, Xoay vòng Token, Khóa HttpOnly, Đóng đinh DPoP, và Tái xác thực. Đó là sự phòng ngự đa chiều.

---

## 7. Tài liệu tham khảo (References)
- **OWASP:** Session Hijacking Attack.
- **IETF RFC 9449:** OAuth 2.0 Demonstrating Proof-of-Possession (DPoP) at the Application Layer.
- **Keycloak Documentation:** Step-up Authentication and ACR.
