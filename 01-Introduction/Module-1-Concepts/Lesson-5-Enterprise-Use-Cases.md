# Lesson 5: Bài toán Doanh nghiệp (Enterprise Use Cases)

> [!NOTE]
> **Category:** Theory & Architecture (Lý thuyết & Kiến trúc)
> **Goal:** Áp dụng vũ khí Keycloak vào chiến trường kinh doanh nghìn tỷ. Phân tích 3 bài toán (Use Cases) kinh điển mà Kiến trúc sư Phần mềm (Software Architect) thường xuyên phải thiết kế và giải quyết.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Keycloak không sinh ra để làm màu cho dự án sinh viên. Nó tỏa sáng rực rỡ nhất khi đối mặt với sự Dày vò Khủng khiếp của Kiến trúc Enterprise, nơi một công ty có Thập cẩm các loại Công nghệ (C\#, Java, Python, Go) và Hàng chục Ứng dụng khác nhau.

Dưới đây là 3 Use Cases định hình nên Ngôi vương của Keycloak:
1. **SSO Tập Đoàn:** Đăng nhập một lần cho toàn bộ Hệ sinh thái.
2. **SaaS Multi-Tenancy (Phần mềm Dịch vụ Đa Tiền sảnh):** Chia vách ngăn dữ liệu độc lập cho nhiều Khách hàng Doanh nghiệp trên cùng 1 hệ thống.
3. **Identity Brokering B2B:** Làm Cò Mồi Đứng Giữa Sát Nhập Công Ty.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

### Use Case 1: Tòa Lâu Đài SSO Nội Bộ (Corporate SSO)
Tập đoàn VinGroup có: Hệ thống Kế Toán (SAP), Mạng nội bộ HR (Java), App Đặt cơm trưa (NodeJS), và Diễn đàn Công ty (WordPress). TẤT CẢ phải chung 1 tài khoản.

```mermaid
graph TD
    subgraph "Hệ Sinh Thái VinGroup"
        User(Nhân viên Vin)
        
        subgraph "Các Ứng dụng Phân mảnh"
            App1[SAP Kế Toán]
            App2[HR Portal]
            App3[App Cơm Trưa]
        end
        
        KC{Keycloak <br/> Cửa Khẩu Duy Nhất}
        AD[(Microsoft AD <br/> CSDL Nhân sự)]
        
        User -->|Truy cập App nào cũng bị đá về đây| KC
        KC -->|Xác minh 1 Lần Duy Nhất| AD
        
        KC -->|Cấp JWT| App1
        KC -->|Cấp SAML| App2
        KC -->|Cấp OIDC| App3
    end
    Note over User,App3: User Đăng nhập ở App 1. Lát sau qua App 2, App 3 KHÔNG CẦN NHẬP LẠI PASS.<br/>Trải nghiệm Liền mạch (Seamless Experience).
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Use Case 2: Multi-Tenancy SaaS (Vách Ngăn Dữ Liệu)**
> Bạn viết 1 phần mềm Quản lý Bán Hàng (SaaS) cực ngon. Công ty A đăng ký xài, Công ty B cũng đăng ký xài. 
> **Điều Tối Kỵ:** Tài khoản Giám đốc của Công ty A (Thực ra là Giám đốc Fake) vô tình nhảy sang đọc được Doanh thu của Công ty B. 
> **Giải pháp Keycloak:** Tính năng **REALMS (Vương quốc)**. Keycloak cho phép bạn tạo ra Hàng Trăm Vương Quốc Ảo trên Cùng 1 Máy chủ. Mỗi Vương quốc (Realm) có Data User ĐỘC LẬP HOÀN TOÀN, không dính dáng gì nhau. 
> - `Realm_A` có Admin tên là Alice. 
> - `Realm_B` cũng có Admin tên là Alice.
> Hai người này là 2 Identity song song ở 2 Vũ trụ khác nhau, không bao giờ Đụng độ (Collision) hay truy cập chéo dữ liệu của nhau được.

> [!CAUTION]
> **Giới hạn của Realms (Kiến trúc)**
> Mặc dù Keycloak quảng cáo là chạy được 500 Realms trên 1 Máy chủ. Nhưng các Chuyên gia cảnh báo: Nếu bạn có 10,000 khách hàng (Tenants) nhỏ lẻ, ĐỪNG TẠO 10,000 Realms. Keycloak sẽ bị Nổ Cấu Hình (Boot mất hàng tiếng đồng hồ do Tải Memory Data).
> **Thực hành chuẩn:** Với SaaS siêu lớn, chỉ tạo 1 Realm Duy nhất. Sau đó dùng **Groups** hoặc nhét thuộc tính `tenant_id=companyA` vào Payload của JWT Token. API Backend của bạn sẽ đọc `tenant_id` đó để chia vách ngăn CSDL (Database sharding).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

### Use Case 3: Kẻ Môi Giới B2B (Identity Brokering)
Đây là Cảnh giới tối thượng của Hợp nhất Tập đoàn (M&A).
Công ty VNG của bạn (Chạy Keycloak) vừa mua lại một Startup ZaloPay. Đội ngũ ZaloPay đang xài một cái Server Auth0 hoặc PingIdentity cũ rích. Bạn KHÔNG THỂ ép 1000 nhân sự ZaloPay lập tức bỏ hết Tool cũ chuyển sang hệ thống mới.

**Tuyệt chiêu Brokering:**
Trong Admin Console Keycloak của VNG, bạn vào Menu `Identity Providers` -> Bấm Thêm `SAML 2.0` hoặc `OpenID Connect`.
Bạn điền Thông tin Máy chủ Auth0 của ZaloPay vào đó.
**Kết quả:** Khi nhân viên ZaloPay vào web của VNG, họ sẽ thấy Nút "Đăng nhập bằng ZaloPay". Bấm vào nút đó, Keycloak đóng vai trò Cò Mồi, đẩy họ sang trang Login cũ của ZaloPay. Họ nhập Pass cũ, ZaloPay cấp Token cho Keycloak, Keycloak tự động bọc 1 lớp Token của VNG ra ngoài và cho họ đi vào. Quá trình M&A Mượt Mà Không Rớt 1 Nhịp.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Bài toán Xác minh Không mật khẩu (Passwordless / WebAuthn):**
  Một Bệnh viện yêu cầu Bác sĩ không được gõ Password (Do tay đeo găng y tế hoặc bẩn, dính máu). Họ muốn Đăng nhập bằng cách Áp Thẻ Từ có chip NFC vào máy vi tính.
  - Keycloak cực kỳ mạnh trong bài toán này. Nó hỗ trợ chuẩn **WebAuthn (FIDO2)** mặc định. Bác sĩ chỉ cần vào máy tính, hệ thống tự kích hoạt Cảm biến Vân tay của Windows Hello / Mac TouchID, hoặc Thẻ YubiKey NFC để xác thực danh tính mà không cần bất kỳ giọt Mật khẩu nào lọt ra ngoài mạng Internet. Trải nghiệm Tương lai (Future-proof).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong một Hệ sinh thái Microservices, tại sao Keycloak được coi là "Trạm Kiểm Soát Biên Giới" nhưng lại Không bao giờ đứng cản đường lưu lượng mạng (Traffic) giữa các Microservices nội bộ?**
- **Junior:** Nó đứng ngoài để cấp vé, còn mấy cái kia tự đưa vé cho nhau.
- **Senior:** Đây là sức mạnh của Kiến trúc Phi trạng thái (Stateless Authentication).
Keycloak là Máy chủ CẤP PHÁT Chứng minh nhân dân (Token Issuer). Người dùng chỉ gặp Keycloak đúng 1 lần khi nhập Mật khẩu. Khi lấy được Token, người dùng cầm Token đập vào API Gateway (Microservice 1). Gateway kiểm tra chữ ký (Bằng Public Key). Hợp lệ, nó cho qua. Nó lại quăng Token cho Microservice 2, 3... Toàn bộ các Microservices nói chuyện trực tiếp với nhau hàng triệu lần/giây mà KHÔNG CẦN CHỌC VỀ KEYCLOAK để hỏi xem Token đó thật hay giả. (Trừ luồng Opaque Token / Introspection). Do đó Keycloak không bao giờ trở thành Nút thắt cổ chai (Bottleneck) của Tốc độ băng thông.

**2. Nếu tôi thiết kế một App Bán Hàng (SaaS). Khi nào tôi nên dùng 1 Realm chung cho mọi Khách hàng, và khi nào bắt buộc phải tách mỗi Khách hàng 1 Realm riêng biệt trong Keycloak?**
- **Junior:** Tách ra cho dễ quản lý đỡ nhầm.
- **Senior:** Quyết định dựa trên **Nhu cầu Cô lập Bảo mật (Isolation Requirements)** và **Tùy biến**.
**Dùng 1 Realm Chung (Logical Isolation):** Khi khách hàng chỉ là các Shop nhỏ lẻ. Họ chấp nhận dùng chung 1 Nút Đăng nhập Facebook/Google do nền tảng cung cấp. Quy mô này có thể scale lên hàng triệu Shop. Dữ liệu chia cách bằng Biến (Claim) trong JWT.
**Tách Realm Riêng (Physical/Tenant Isolation):** Khi khách hàng là Tập đoàn Lớn (B2B). Công ty A yêu cầu: "Cái màn hình Login phải có Logo Công ty tôi", "Nhân viên công ty tôi phải tích hợp Login qua Máy chủ LDAP của RIÊNG Công ty tôi". Lúc này, chỉ có Tách Realm riêng biệt mới đáp ứng được khả năng Cấu hình Custom Giao diện và Identity Providers độc lập cho từng khách. Nhưng đánh đổi là chi phí Memory của Keycloak tăng phi mã.

**3. Làm thế nào để Keycloak giải quyết bài toán "Tích hợp Di sản" (Legacy Integration) với một hệ thống Kế toán cũ kỹ chỉ chấp nhận Xác thực bằng IP hoặc Mật khẩu tĩnh trong Database mà không biết JWT là gì?**
- **Junior:** Chắc không được, phải sửa code hệ thống cũ.
- **Senior:** Tuyệt đối không chạm vào code hệ thống Di sản rủi ro cao. Ta dùng **Mẫu Thiết Kế Mật Sứ (Ambassador/Sidecar Pattern)** kết hợp API Gateway.
Sơ đồ: Người dùng -> [Login Keycloak lấy JWT] -> API Gateway.
Tại API Gateway, ta thiết lập một Plugin. Khi Request từ Frontend mang theo JWT đi vào, API Gateway kiểm tra chữ ký JWT xịn. Sau đó, nó LỘT BỎ cái JWT đi. Nó lấy Username trong JWT đó, tra cứu trong một Bảng Mapping, Tự Động chèn cái "Mật khẩu tĩnh cũ rích" của hệ thống Kế toán vào HTTP Basic Auth (Hoặc Fake IP trong Header), và ném cái Request Trần Trụi đó vào hệ thống Kế toán nội bộ.
Hệ thống Di sản ngỡ mình đang chạy với môi trường cũ, vẫn hoạt động hoàn hảo. Nhưng Lớp Vỏ Bọc bên ngoài đã được hiện đại hóa lên OIDC 100% nhờ bàn tay của Keycloak + Gateway.

**4. Khái niệm "Social Login" (Đăng nhập bằng Google/FB) là B2B hay B2C Use Case? Thuộc tính gì của Keycloak biến Social Login trở thành trò đùa trẻ con?**
- **Junior:** B2C, do Keycloak có nút thêm Google.
- **Senior:** Nó là Đặc trưng của hệ thống **CIAM (Customer IAM - B2C)**.
Keycloak biến nó thành trò đùa nhờ Khối Kiến trúc **Identity Brokering (Môi giới danh tính)**. Bạn chỉ cần lên Google Cloud, tạo Oauth Credentials lấy Client ID & Secret. Quăng 2 mã đó vào giao diện Keycloak.
Đỉnh cao nằm ở chỗ: Khi User bấm Login Google, Keycloak nhận thông tin từ Google, sau đó Keycloak TỰ ĐỘNG MAP (Ánh xạ) cái Email Google đó tạo thành 1 User Nội Bộ trong Bảng Database User của Keycloak (Luồng First-broker-login). Hệ thống API Backend của bạn không cần quan tâm User đó từ Google hay Facebook tới, vì Keycloak đã Nấu chín Data thành chuẩn JWT nội bộ duy nhất.

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Server Administration Guide (Realms & Identity Brokering).
- **Microsoft Azure:** SaaS and Multi-tenant architecture patterns.
- **NIST:** Zero Trust Architecture.
