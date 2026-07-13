# Lesson 4: Cuộc chiến Build vs Buy (Why Keycloak?)

> [!NOTE]
> **Category:** Theory & Architecture (Lý thuyết & Kiến trúc)
> **Goal:** Giải quyết cuộc tranh luận khốc liệt nhất trong mọi phòng họp IT khi Start dự án: "Tại sao không tự code cái chức năng Đăng nhập cho nhanh mà phải cài nguyên con Keycloak nặng nề?". Phân tích ưu nhược điểm giữa Tự xây (Build) và Mua công cụ (Buy/Open Source).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Ảo tưởng sức mạnh (The Dunning-Kruger Effect của Auth)
Một Junior Dev nhìn chức năng Login và nghĩ: *"Tôi chỉ cần 1 bảng User, 1 API nhận Username/Password, dùng BCrypt băm nó ra, gọi thư viện JWT.jwt.sign() trả về Token là xong. Tôi code hết 2 ngày"*.

Nhưng hãy nhìn vào sự thật: Sự phát triển của dự án là Tàn Khốc.
- **Tháng 1:** Code xong form Login 2 ngày. Sếp vui vẻ.
- **Tháng 2:** Sếp yêu cầu thêm nút "Quên mật khẩu", bắt bạn code tính năng gửi Email, tạo Token tạm thời, tạo cờ Hết hạn 15 phút, form Nhập pass mới.
- **Tháng 3:** Khách hàng kêu gõ pass mỏi tay, yêu cầu "Đăng nhập bằng Google/Facebook". Bạn hì hục ngồi cày tài liệu OAuth2 của Google, cài chục cái npm packages.
- **Tháng 4:** Công ty lọt vào mắt xanh của Cục An Ninh mạng. Họ ép: BẮT BUỘC có xác thực 2 lớp (MFA/OTP). Bạn lôi cái App ra đập đi xây lại luồng Auth vì kiến trúc cũ không hỗ trợ State.
- **Tháng 6:** Công ty bị HACK. Hacker dùng đòn "Nhồi Password" (Credential Stuffing). Bạn chưa code chức năng "Khóa tài khoản sau 5 lần sai". Trưởng phòng IT bị sa thải.

### 1.2. Giải pháp: Build vs Buy vs Adopt
Thay vì chết chìm trong Đống Rác Bảo mật tự code, Thế giới IT có 3 lựa chọn:
1. **BUILD (Tự code từ đầu):** Chỉ dành cho các ông lớn như Google, Facebook (Họ có cả team 100 chuyên gia mật mã học). Tuyệt đối cấm kỵ với Startups/Công ty nhỏ.
2. **BUY (Mua giải pháp Cloud):** Okta, Auth0, AWS Cognito, Firebase. Quẹt thẻ tín dụng, setup trong 5 phút. Điểm yếu: Vendor Lock-in (Bị trói buộc vào 1 nhà cung cấp), không làm chủ được Dữ liệu (User lưu trên Mỹ, vi phạm Luật An Ninh Mạng VN), giá tiền tăng theo cấp số nhân khi vượt quá 10,000 Users.
3. **ADOPT (Dùng Open Source - KEYCLOAK):** Sự Cân Bằng Hoàn Hảo. Bạn ôm trọn 1 hệ thống IAM siêu to khổng lồ miễn phí, tự host trên Server của bạn (Bảo mật data Việt Nam). Đổi lại, bạn phải có Kỹ sư Hiểu và Vận hành được nó.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Bảng so sánh Mức độ Phức tạp (Complexity) giữa Tự Code và Dùng Keycloak:

| Tính năng bảo mật sinh tử | Tự Code (Backend Spring/Node) | Dùng Keycloak |
| :--- | :--- | :--- |
| **Băm Mật Khẩu (Password Hashing)** | Phải tự chọn thuật toán, tự sinh Salt. Nếu lỡ xài MD5/SHA1 là vào tù. | Tích hợp sẵn chuẩn PBKDF2 siêu việt, 27,500 vòng lặp an toàn tuyệt đối. |
| **Bảo vệ Brute-force** | Phải tự code Cache Redis, đếm số lần sai, khóa tài khoản, báo DB. | Vào Menu, Click 1 Nút: Bật Brute-force Detection. Xong! |
| **Chính sách Mật khẩu (Password Policy)** | Code if-else check độ dài, check số, check ký tự đặc biệt, check ngày hết hạn. | Kéo thả trong giao diện Admin (Đủ 8 chữ, 1 chữ hoa, 90 ngày bắt đổi 1 lần). |
| **Hỗ trợ JWT / OIDC / SAML** | Đọc 500 trang RFC tiếng Anh, tự nhét thư viện, tự vá lỗ hổng JWS. | Hỗ trợ 100% chuẩn quốc tế. Tự động sinh `jwks_uri` chứng chỉ công khai. |

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Đừng phát minh lại cái Bánh xe (Don't Reinvent the Wheel)**
> Bảo mật là một môn học Khắc nghiệt. Nếu bạn viết sai 1 file HTML, hình ảnh bị méo, không ai chết.
> Nhưng nếu bạn viết sai 1 dòng code kiểm tra Token, toàn bộ Dữ liệu Ngân hàng của công ty bốc hơi. Keycloak được hàng chục ngàn Kỹ sư Mật mã trên toàn cầu (và Red Hat) ngày đêm Dòm ngó, Auditing, và Vá Lỗi. Bạn (1 cá nhân) KHÔNG BAO GIỜ có thể code ra một hệ thống Auth an toàn hơn tập thể đó được. Mệnh lệnh tối cao: Uỷ thác bảo mật cho Keycloak.

> [!CAUTION]
> **Khi nào thì KHÔNG NÊN xài Keycloak?**
> Mặc dù Keycloak rất mạnh, nhưng nếu bạn chỉ làm 1 cái Website Quản lý Thư viện cho cái Trường Mẫu Giáo gồm 5 cô giáo xài. Chạy Keycloak (Cần máy chủ Server 1-2GB RAM) là Đập con ruồi bằng súng BAZOOKA.
> Lúc này, tự viết 1 hàm Login nhỏ gọn, hoặc dùng Firebase Authentication bản Free là phương án sáng suốt nhất để tiết kiệm tiền Server.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sức mạnh "Không Code" (No-code Auth) của Keycloak thể hiện rõ nhất khi Bật chức năng OTP (Xác thực 2 bước bằng App Google Authenticator).

- Nếu **Tự Code**: Bạn phải tạo Cột Database chứa SecretKey, Gen ảnh QR Code, Code form Nhập 6 số, Code hàm xác minh TOTP rườm rà.
- Với **Keycloak**: Bạn vào Admin Console -> Menu `Authentication` -> `Browser Flow` -> Tìm mục `OTP Form` -> Sửa từ `Disabled` thành `Required`.
- BÙM! Mọi User khi đăng nhập xong màn hình Pass, Keycloak TỰ ĐỘNG CHẶN HỌ LẠI, hiển thị Màn hình Quét QR Code, tự động lưu SecretKey, tự động Check OTP. Bạn hoàn toàn không cần nhúng tay vào 1 dòng Code nào.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Trói buộc Đám mây (Vendor Lock-in) và Nước mắt của Startup:** Một Startup nổi tiếng Việt Nam xài Auth0 lúc đầu (Vì setup quá nhanh). Khi Startup gọi vốn thành công, có 1 Triệu Users. Auth0 lập tức gửi Email thu phí **Hàng Tỷ Đồng / Tháng** cho gói Enterprise.
  - Startup cuống cuồng muốn bỏ Auth0 sang bên khác. NHƯNG: Auth0 Không bao giờ cho phép bạn Export (Trích xuất) Bảng Mật khẩu gốc đã băm của User ra ngoài (Để giữ chân khách). 
  - Nghĩa là nếu bỏ Auth0, toàn bộ 1 Triệu khách hàng PHẢI BẤM QUÊN MẬT KHẨU LÀM LẠI TỪ ĐẦU. Rất nhiều Startup chết chìm vì bị Cloud "Tống tiền".
  - Với **Keycloak (Mã nguồn mở)**: Máy chủ là của bạn, Database PostgreSQL là của bạn. Bạn nắm Trọn Quyền Sinh Sát. Rẻ, An toàn, và Tự do.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. "Offload Security" (Giảm tải Bảo mật) là khái niệm gì? Keycloak giúp Dev "Offload" điều gì?**
- **Junior:** Giúp Dev làm nhẹ việc đi, không phải code màn hình login.
- **Senior:** "Offload Security" là Kiến trúc Tách biệt Trách nhiệm (Separation of Concerns).
Lập trình viên Backend chỉ nên tập trung code Nghiệp vụ Kinh doanh (Ví dụ: Chuyển tiền, Bán hàng, Giao hàng). Những thứ thuộc về Cửa Khẩu (Xác thực 2 lớp, Quên mật khẩu, Xác minh Token, Khóa tài khoản) không mang lại Giá Trị Kinh Doanh Trực Tiếp.
Bằng việc ném con Keycloak ra đứng trước cửa. Toàn bộ gánh nặng bảo mật được "Offload" (Ủy thác) cho Keycloak. Developer giờ đây ngủ ngon giấc, Code Backend của họ sẽ Mỏng Đi đáng kể, hoàn toàn không dính dấp đến các thư viện mã hóa rườm rà nữa.

**2. Nếu hệ thống Backend (Spring Boot) của tôi bị Hacker lấy trộm Toàn bộ Source Code. Hacker có thể tự tạo ra Token giả mạo để đăng nhập không? Tại sao?**
- **Junior:** Có, vì Hacker coi code sẽ biết cách gen Token.
- **Senior:** Tuyệt đối KHÔNG THỂ. Đây là sự kỳ diệu của Kiến trúc Keycloak.
Trong Spring Boot, bạn CHỈ CẤU HÌNH Public Key (JWKS URI) của Keycloak để Xác Minh (Verify) Token.
Cái **Private Key** (Chìa khóa gốc dùng để Ký và Tạo ra Token) nằm CHẾT CỨNG trong Database của Keycloak (Hoặc trong HSM/Vault). Backend của bạn HOÀN TOÀN KHÔNG BIẾT Private Key là gì. Do đó, dù Hacker cuỗm sạch Source Code Spring Boot, hắn vẫn bất lực vì không có Private Key để Đúc ra Token giả. (Ngoại trừ trường hợp dùng Mã hóa Đối Xứng HS256, nhưng Enterprise bắt buộc dùng Bất Đối Xứng RS256).

**3. Tại sao Bộ Quốc Phòng hoặc các Ngân Hàng Nội Địa từ chối sử dụng AWS Cognito / Auth0 mà bắt buộc phải Setup Keycloak In-house (Tại nhà máy)?**
- **Junior:** Tại vì Keycloak miễn phí đỡ tốn tiền thuế.
- **Senior:** Vấn đề sinh tử là **Chủ Quyền Dữ Liệu Quốc Gia (Data Sovereignty)**.
Theo luật An ninh mạng (Hoặc GDPR ở Châu Âu). Dữ liệu Định danh Nhân sự cấp cao (Mã vân tay, CMND, Lịch sử truy cập mạng) TUYỆT ĐỐI KHÔNG ĐƯỢC PHÉP nằm trên máy chủ đặt tại nước ngoài (AWS ở Mỹ/Singapore).
Hệ thống Auth0/AWS Cognito bắt buộc bạn đẩy thông tin User lên Cloud của họ. Họ có toàn quyền (Hoặc bị NSA/Chính phủ Mỹ ép buộc) soi mói dữ liệu.
Keycloak là giải pháp In-house (On-Premise). Bạn đem cục Source đó về, cài vào Data Center nằm ở Tầng hầm của Ngân hàng tại Hà Nội, xây Tường lửa cắt đứt mạng Internet bên ngoài. Dữ liệu User hoàn toàn được bảo vệ tuyệt mật.

**4. Khuyết điểm ĐỚN ĐAU NHẤT của Keycloak so với các giải pháp Cloud (Auth0) là gì? Tại sao nhiều đội IT vẫn sợ Keycloak?**
- **Junior:** Nó nặng, tốn nhiều RAM, cấu hình khó.
- **Senior:** Khuyết điểm chí mạng là **Chi phí Vận hành Ẩn (Hidden Operational Cost)**.
Dùng Auth0: Server chết, Auth0 đền tiền (SLA 99.99%). Cập nhật phiên bản vá lỗi bảo mật, Auth0 tự làm trong đêm bạn đang ngủ. 
Dùng Keycloak: PHẦN MỀM LÀ MIỄN PHÍ, nhưng người Vận Hành nó thì đắt đỏ. Công ty bạn BẮT BUỘC phải nuôi một đội ngũ Kỹ sư DevOps/SecOps cứng cựa. Bọn họ phải biết cài Đĩa Trống (Storage), Cấu hình High Availability (Cluster 3 Nodes), cài Redis Cache, Backup Database lúc 2h sáng, và Đọc Log Cập nhật (Migration) mỗi khi Keycloak ra bản mới. Nếu đội IT yếu kém làm Sập Keycloak, TOÀN BỘ CÔNG TY (Bao gồm mọi App) sẽ Tê liệt hoàn toàn (Single Point of Failure).

---

## 7. Tài liệu tham khảo (References)
- **OWASP:** Authentication Cheat Sheet.
- **Thoughtworks Technology Radar:** Keycloak (Adopt).
