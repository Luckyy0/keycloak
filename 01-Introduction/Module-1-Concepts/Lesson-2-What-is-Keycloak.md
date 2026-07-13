# Lesson 2: Tượng đài Keycloak (What is Keycloak?)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Lột xác khỏi định kiến thiển cận "Keycloak chỉ là cái Form đăng nhập". Khám phá định nghĩa hàn lâm của Keycloak dưới góc độ một Máy chủ Ủy quyền (Authorization Server) và Nhà cung cấp Danh tính (Identity Provider) đẳng cấp thế giới.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Định nghĩa chuẩn Quốc tế
**Keycloak** là một sản phẩm phần mềm Mã Nguồn Mở (Open Source Identity and Access Management) được bảo trợ tài chính và phát triển bởi gã khổng lồ **Red Hat (Hiện thuộc IBM)**.
Định nghĩa chính thức của hãng: *Keycloak is an open source identity and access management solution for modern applications and services.*

Tuy nhiên, định nghĩa dưới góc độ Kiến trúc (Architecture) của Hệ thống mạng:
- Đối với giao thức OIDC: Keycloak đóng vai trò là một **OpenID Provider (OP)** hay **Authorization Server (AS)**.
- Đối với giao thức SAML: Keycloak đóng vai trò là một **Identity Provider (IdP)**.
- Đối với quản lý người dùng: Nó là một **User Directory (Thư mục người dùng)** hoặc **Broker (Môi giới danh tính)**.

### 1.2. Keycloak KHÔNG PHẢI LÀ GÌ?
Để hiểu đúng, chúng ta phải biết nó KHÔNG phải là gì:
1. **Nó không phải là Database nghiệp vụ:** Đừng cố nhét các trường dữ liệu `Chieu_Cao`, `Can_Nang`, `So_Du_Tai_Khoan` vào bảng User của Keycloak. Keycloak chỉ quản lý "Danh tính cốt lõi", dữ liệu Nghiệp vụ (Business Data) phải nằm ở Máy chủ API của bạn.
2. **Nó không phải là WAF (Tường lửa Web):** Nó cấp Token, nhưng nhiệm vụ "Chặn Request rác, chặn IP Hacker" phải là của Nginx / API Gateway đứng trước nó.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Tứ giác quyền lực: Vị trí của Keycloak khi đứng giữa vũ trụ Công nghệ:

```mermaid
graph TD
    subgraph "Các Ứng dụng Khách (Clients)"
        SPA[ReactJS / Angular / Vue]
        Mobile[iOS / Android App]
        Backend[Spring Boot / NodeJS API]
    end
    
    subgraph "Trái Tim - Bộ Não (Keycloak)"
        KC((Keycloak Server))
    end
    
    subgraph "Hệ thống Danh tính Ngoại lai (External IdPs)"
        AD[Microsoft Active Directory / LDAP]
        Social[Google / Facebook / Github]
        Partner[Máy chủ SAML của Đối tác]
    end
    
    SPA -->|Bấm Nút Login| KC
    Mobile -->|Lưu Token, Gọi API| KC
    Backend -->|Nhờ KC giải mã/Validate Token| KC
    
    KC -->|Lấy Password từ LDAP công ty| AD
    KC -->|Ủy thác Login qua Google| Social
    KC -->|Sát nhập Công ty B| Partner
    
    Note over KC: Keycloak ĐỨNG CHÍNH GIỮA (Middleman). <br/>Nó che giấu toàn bộ sự phức tạp của 10 loại phương thức Đăng nhập Khác nhau.<br/>Bất chấp người dùng Đăng nhập bằng Google hay bằng Thẻ từ, Keycloak đều<br/>gom lại và ném ra 1 cục JWT tiêu chuẩn duy nhất cho Client xử lý.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Sức mạnh Môi giới Danh tính (Identity Brokering)**
> Đây là tính năng Sát thủ (Killer Feature) đưa Keycloak lên Ngôi vương. 
> Công ty bạn có 10 ứng dụng (App Bán hàng, App Chấm công...). Khách hàng muốn Đăng nhập bằng Google.
> **Cách Làm Dở:** Bạn phải viết Code kết nối Google API cho cả 10 App. Tuần sau Khách muốn thêm Facebook, bạn lại vào sửa Code của cả 10 App.
> **Cách Của Keycloak:** 10 App đó chỉ kết nối DUY NHẤT với Keycloak (Bằng giao thức OIDC chuẩn). Trên giao diện Admin của Keycloak, bạn nhập 1 cái API Key của Google/Facebook vào. BÙM! Toàn bộ 10 App ngay lập tức có thêm Nút "Đăng nhập bằng Google/Facebook" mà KHÔNG CẦN VIẾT THÊM 1 DÒNG CODE NÀO Ở CÁC APP ĐÓ. Keycloak đóng vai trò Cò Mồi (Broker) trung gian hoàn hảo.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sự thống trị của Keycloak được đo lường bằng Khả năng "Nói được đa ngôn ngữ Giao thức" (Multi-Protocol Support).
Một Máy chủ Keycloak khi cài xong, Mặc định nó sẽ tự động Bật Lên tất cả các Endpoints (Cổng giao tiếp) phục vụ cho Cả thế giới cũ và thế giới mới:

- **Dành cho OIDC (Cổng hiện đại):**
  `https://auth.com/realms/myrealm/.well-known/openid-configuration` (Sách hướng dẫn tự động của OIDC).
- **Dành cho SAML (Cổng Doanh nghiệp Cổ điển):**
  `https://auth.com/realms/myrealm/protocol/saml/descriptor` (Sách hướng dẫn tự động của SAML).
- **Dành cho Quản trị viên (Admin REST API):**
  `https://auth.com/admin/realms/myrealm/users` (Dùng để viết Tool tự động thay con người).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Air-Gapped Environments (Môi trường Cô lập Không Internet):** Rất nhiều công ty hỏi: "Tại sao không xài Auth0, AWS Cognito, hay Firebase cho sướng, bọn đấy Setup có 5 phút?".
  - **Lý do Tối thượng:** Chính Phủ, Quân Đội, Ngân Hàng có những Server nằm ở dướt tầng hầm, bị CẮT ĐỨT TOÀN BỘ DÂY CÁP MẠNG INTERNET (Air-gapped). Bạn không thể mang Auth0 (Nằm trên Cloud của AWS) xuống tầng hầm đó được.
  - Keycloak là mã nguồn mở, đóng gói bằng file `.jar` hoặc Docker. Bạn có thể mang USB copy Keycloak xuống tầng hầm, nhét vào Server và chạy ngon lành không cần Internet. Lợi thế Tuyệt đối về **Data Sovereignty (Chủ quyền Dữ liệu)**.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong thế giới Enterprise, khái niệm "Single Sign-On (SSO)" do Keycloak cung cấp có phải là "Dùng chung 1 Tài khoản / Mật khẩu cho 10 trang Web" không?**
- **Junior:** Đúng, nó là xài chung 1 cái pass cho tiện đỡ phải nhớ nhiều.
- **Senior:** Nhầm Lẫn Nghiêm Trọng. Xài chung 1 cái Password được gọi là **Same Sign-On (Đăng nhập cùng 1 mật khẩu)** - Ví dụ: Bạn dùng chung Password cho Zalo, Facebook, Ngân hàng. Mỗi lần vào Web, bạn vẫn PHẢI GÕ mật khẩu đó lại.
**Single Sign-On (Đăng Nhập Đơn - SSO)** là một Đẳng Cấp khác: Bạn mở Web A, hệ thống bắt bạn gõ Pass. Nửa tiếng sau, bạn mở Web B. HỆ THỐNG KHÔNG HỀ HIỆN RA CÁI Ô NHẬP PASS NỮA. Bạn vào thẳng Web B (Bằng tài khoản của bạn) trong sự ngỡ ngàng. Đăng nhập 1 Lần, Trải thảm đỏ đi vào mọi Căn phòng khác. Đó mới là SSO đích thực do Keycloak mang lại.

**2. Nếu tôi xài Keycloak, có phải là App ReactJS của tôi sẽ gọi thẳng vào Database MySQL (Nơi lưu User của Keycloak) để tự kiểm tra User không?**
- **Junior:** Đúng, React sẽ chọc vào DB để tìm cái tên user đó xem có thật không.
- **Senior:** Một trong những Tội ác Kiến trúc khủng khiếp nhất.
Tuyệt đối KHÔNG MỘT AI (Kể cả App Backend của công ty bạn) được phép kết nối trực tiếp vào Database của Keycloak (PostgreSQL/MySQL).
Keycloak là một **Black Box (Hộp Đen)** bảo mật. Cách duy nhất để nói chuyện với nó là qua các chuẩn Giao thức (OIDC/SAML) hoặc REST API do nó cấp. Nếu bạn chọc thẳng vào Database của nó, bạn đã tự bắn nát hệ thống Hashing thuật toán PBKDF2 của Mật khẩu, phá vỡ hệ thống Caching Infinispan, và đập tan mọi cơ chế Ghi Log Auditing của Security.

**3. "Keycloak được Red Hat bảo trợ". Vậy nếu tôi xài Keycloak miễn phí, tôi có bị vi phạm bản quyền hay bị giới hạn tính năng (Limit 1000 users) như các bản Dùng thử không?**
- **Junior:** Sẽ bị giới hạn, muốn xài mạnh phải mua bản RedHat.
- **Senior:** Đây là sức mạnh của chuẩn **Apache License 2.0 (Open Source)**.
Keycloak phiên bản Cộng đồng (Upstream) HOÀN TOÀN MIỄN PHÍ và MỞ TOÀN BỘ 100% TÍNH NĂNG (Không giới hạn User, Không giới hạn Số lượng Realm). Bạn có thể dùng cho Tập đoàn 100 Triệu người dùng vẫn hợp pháp.
**Bản Red Hat (Red Hat Build of Keycloak / RH-SSO)** thực chất chính là cục mã nguồn của bản Miễn phí, nhưng được Red Hat đem về, khóa phiên bản lại, vá các lỗi nóng (Hotfix), thử nghiệm tính Ổn định dài hạn (LTS), và bán cái DỊCH VỤ HỖ TRỢ (24/7 Support) chứ không phải bán bản quyền phần mềm. Nếu tự tin vào đội IT của mình, xài bản miễn phí là quá đủ.

**4. Tính năng "User Federation" của Keycloak giải quyết bài toán lịch sử gì của các Tập đoàn đã tồn tại 20 năm?**
- **Junior:** Giúp đưa người dùng cũ vào hệ thống mới.
- **Senior:** Tập đoàn lâu đời (Như Hệ thống Điện lực, Ngân hàng) họ đã có một Máy chủ Microsoft Active Directory (AD) chứa 50,000 nhân sự và toàn bộ chính sách Mật khẩu.
Họ muốn chuyển sang dùng App Cloud, dùng Mobile, nhưng AD không hỗ trợ OIDC/JWT. Hơn nữa, họ TUYỆT ĐỐI KHÔNG CHỊU DỜI ĐỔI Database 50,000 người kia ra khỏi con Server AD cũ.
Tính năng **User Federation** của Keycloak giúp giải quyết êm thấm. Keycloak không ép kéo Data về. Keycloak đứng làm trung gian kết nối LDAP tới Server AD. Khi người dùng nhập Pass vào Keycloak, Keycloak sẽ chạy tới Gõ Cửa Server AD để hỏi xem Pass đúng không. Nếu đúng, Keycloak tự sinh JWT và cấp cho điện thoại. Nhân sự vẫn yên ổn nằm ở AD, nhưng Thế giới bên ngoài vẫn giao tiếp mượt mà qua chuẩn REST/JWT hiện đại.

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Official Website:** keycloak.org.
- **Red Hat:** What is Keycloak?.
- **OpenID Foundation:** OpenID Connect Core Specification.
