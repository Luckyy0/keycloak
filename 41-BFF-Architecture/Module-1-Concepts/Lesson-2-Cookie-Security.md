> [!NOTE]
> **Category:** Theory
> **Goal:** Nắm vững các tiêu chuẩn bảo mật cấu hình Cookie (Cookie Security) trong Kiến trúc Backend-For-Frontend (BFF), đặc biệt là cách phòng chống tấn công XSS và CSRF khi quản lý phiên đăng nhập (Session) thay thế cho JWT lưu trên Client.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Trong kiến trúc SPA/Mobile hiện đại truyền thống, Access Token (JWT) thường được lưu ở Local Storage hoặc Session Storage của trình duyệt. Tuy nhiên, điều này biến Token thành mục tiêu béo bở của các cuộc tấn công **XSS (Cross-Site Scripting)**, nơi mã độc JavaScript có thể đọc toàn bộ storage và đánh cắp Token.

Để giải quyết triệt để rủi ro này, kiến trúc **Backend-For-Frontend (BFF)** ra đời. Thay vì trả JWT về cho Trình duyệt, BFF (đóng vai trò là một Confidential Client của Keycloak) sẽ giữ JWT trong bộ nhớ (hoặc cache/database) của nó. Sau đó, BFF thiết lập một phiên làm việc (Session) với Trình duyệt thông qua **HTTP Cookie**. Trình duyệt chỉ giữ Cookie, mỗi khi gọi API, Cookie được gửi tự động.

Tuy nhiên, việc dùng Cookie lại mở ra một rủi ro mới: **CSRF (Cross-Site Request Forgery)**. Cookie Security là việc tinh chỉnh các tham số (Flags) của HTTP Cookie để trình duyệt ngăn chặn cả XSS (không cho JS đọc cookie) và CSRF (không gửi cookie từ domain lạ).

Các cờ (Flags) tối quan trọng bao gồm:
- **`HttpOnly`**: Cấm JavaScript (như `document.cookie`) truy cập vào Cookie. Chống lại XSS triệt để.
- **`Secure`**: Cookie chỉ được trình duyệt gửi qua kết nối mã hóa HTTPS. Chống tấn công Man-in-the-Middle (MitM) đánh cắp Cookie trên đường truyền.
- **`SameSite`**: Chỉ thị cho trình duyệt biết lúc nào thì được đính kèm Cookie trong các request gửi qua lại giữa các domain (cross-site).

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Khi BFF nhận được Token từ Keycloak (sau quá trình trao đổi mã Authorization Code), BFF sẽ sinh ra một `Session ID` mã hóa bí mật, tạo Cookie và phản hồi về cho Frontend.

```mermaid
sequenceDiagram
    participant Browser
    participant Hacker as Hacker Site (attacker.com)
    participant BFF as BFF Server (api.mycompany.com)

    Note over Browser, BFF: Đăng nhập thành công, thiết lập Cookie
    BFF-->>Browser: HTTP 200 OK<br/>Set-Cookie: SESSION=xyz123; HttpOnly; Secure; SameSite=Strict
    
    Note over Browser: Kịch bản 1: XSS Attack
    Browser->>Browser: Hacker nhúng JS: console.log(document.cookie)
    Note right of Browser: Cookie bị ẩn (do HttpOnly). XSS thất bại!

    Note over Browser: Kịch bản 2: CSRF Attack
    Browser->>Hacker: User vô tình truy cập trang của Hacker
    Hacker->>Browser: Tự động submit form tới https://api.mycompany.com/transfer
    Browser->>BFF: Gửi POST Request (Cross-Site)
    Note right of Browser: Trình duyệt từ chối đính kèm Cookie SESSION<br/>(Do SameSite=Strict). CSRF thất bại!
```

**Cơ chế đánh giá cờ SameSite cấp thấp trong trình duyệt:**
- `SameSite=Strict`: Trình duyệt chỉ gửi Cookie nếu request bắt nguồn từ CHÍNH XÁC cùng một trang (First-party context). An toàn nhất, nhưng nếu user click link từ email vào web, cookie không được gửi theo (user có thể bị bắt đăng nhập lại).
- `SameSite=Lax`: (Mặc định của hầu hết trình duyệt hiện nay). Trình duyệt cho phép gửi Cookie nếu là các request điều hướng ở top-level (như click link `<a>` hoặc HTTP GET method). Nhưng nó KHÔNG đính kèm cookie cho POST/PUT/DELETE từ domain khác. Đây là sự cân bằng giữa bảo mật và trải nghiệm.
- `SameSite=None`: Bắt buộc gửi cookie trong mọi bối cảnh. Cấu hình này BẮT BUỘC phải đi kèm với cờ `Secure`. Thường chỉ dùng khi ứng dụng của bạn là một iframe nhúng trên trang của đối tác.

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> Đối với kiến trúc BFF bảo vệ SPA (Single Page Application), cấu hình tiêu chuẩn vàng BẮT BUỘC là: `HttpOnly=true`, `Secure=true`, và `SameSite=Strict` (nếu BFF và Frontend chạy cùng domain phụ) hoặc `SameSite=Lax`. Bất kỳ sự thỏa hiệp nào về ba cờ này đều làm hỏng toàn bộ mô hình an ninh của BFF.

> [!WARNING]
> Không nên phụ thuộc hoàn toàn 100% vào `SameSite=Lax` để chống CSRF, bởi vì các trình duyệt cũ (như IE11) không hỗ trợ cờ này. Hãy áp dụng thêm chiến lược **Anti-CSRF Token** (Synchronizer Token Pattern) hoặc **Double Submit Cookie** ngay tại lớp BFF.

- **Cookie Path**: Chỉ định đường dẫn rõ ràng (`Path=/api`) để giới hạn rò rỉ cookie tới các đường dẫn tĩnh (ảnh, css) không cần thiết.
- **Cookie Prefix**: Đặt tiền tố `__Host-` cho tên Cookie (VD: `__Host-SESSION`). Tiền tố này là một tiêu chuẩn trình duyệt buộc Cookie phải có `Secure=true`, không được chỉ định `Domain` attribute (ngăn chia sẻ cookie cho subdomain phụ), và `Path` phải là `/`. Nó khóa chặt cookie vào đúng máy chủ đó.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Ví dụ cấu hình Cookie bảo mật tối đa cho Session lưu tại một ứng dụng Spring Cloud Gateway (đóng vai trò BFF tích hợp OIDC) hoặc Express.js BFF.

**Trường hợp 1: Express.js (Node.js) BFF Configuration**
```javascript
const session = require('express-session');

app.use(session({
  name: '__Host-bff-session', // Sử dụng Cookie Prefix để tăng cường bảo mật
  secret: 'super-secret-key-for-signing-session-id',
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,     // Chống XSS (Không đọc được qua document.cookie)
    secure: true,       // Chống MitM (Chỉ gửi qua HTTPS)
    sameSite: 'strict', // Chống CSRF toàn diện
    path: '/',          // Bắt buộc với __Host- prefix
    maxAge: 3600000     // Hết hạn sau 1 giờ
  }
}));
```

**Trường hợp 2: Spring Boot (Spring Security 6) BFF Configuration**
Cấu hình trong file `application.yml` để tùy biến cookie cho Spring Session:
```yaml
server:
  servlet:
    session:
      cookie:
        name: __Host-SESSION
        http-only: true
        secure: true
        same-site: strict
```

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Localhost Development**: Trong môi trường phát triển (Dev) dưới máy cục bộ chạy qua `http://localhost`, trình duyệt sẽ chặn Cookie nếu bạn set `Secure=true` (vì không phải HTTPS). Riêng đối với `localhost`, Chrome/Firefox có cơ chế ngoại lệ coi nó là bối cảnh an toàn, nhưng nhiều ngôn ngữ Backend mặc định sẽ không gửi `Set-Cookie` nếu request không mã hóa. Bạn cần tách biệt môi trường `development` (Secure=false) và `production` (Secure=true).
- **Subdomain Deployment (BFF ở `api.example.com`, SPA ở `app.example.com`)**: Nếu cấu hình `SameSite=Strict`, trình duyệt có thể coi `api.` và `app.` là cross-site requests trong một số ngữ cảnh Fetch/XHR cụ thể, dẫn đến mất session. Lúc này, bạn phải cấu hình `Domain=.example.com` và chuyển SameSite xuống `Lax`, kết hợp CORS policy chuẩn mực.

## 6. Câu hỏi Phỏng vấn (Interview Questions)

1. **Junior**: Kiến trúc BFF giải quyết lỗ hổng bảo mật nào so với việc lưu trữ Access Token trên SPA?
   - *Đáp án*: BFF giải quyết nguy cơ XSS nhắm vào Local Storage, vì Token không bao giờ được gửi xuống Frontend. Trình duyệt chỉ giữ HttpOnly Cookie (không thể truy cập bằng JavaScript).
2. **Junior**: Tác dụng của cờ `HttpOnly` trên Cookie là gì?
   - *Đáp án*: Nó ngăn chặn mã độc JavaScript (thông qua `document.cookie`) đọc giá trị của Cookie, loại bỏ rủi ro đánh cắp phiên (Session Hijacking) qua lỗ hổng XSS.
3. **Senior**: Tại sao cấu hình `SameSite=None` lại yêu cầu cờ `Secure=true`?
   - *Đáp án*: `SameSite=None` cho phép gửi Cookie ở các request Cross-site (như nhúng iFrame từ tên miền khác). Do rủi ro gửi dữ liệu sang bên thứ ba cao, các nhà phát triển trình duyệt bắt buộc dữ liệu đó phải được truyền qua kênh mã hóa HTTPS (Secure) để tránh bị nghe lén.
4. **Senior**: Tiền tố `__Host-` ở tên Cookie giải quyết bài toán gì?
   - *Đáp án*: Tiền tố này (Cookie Prefixing) ép buộc trình duyệt từ chối tạo cookie nếu nó không đáp ứng 3 điều kiện: Phải là HTTPS, Không chứa thuộc tính `Domain` (không cho phép chia sẻ với Subdomain lân cận dễ bị tấn công), và `Path` phải là `/`. Nó khóa Session vào đúng server gốc (Origin bound).
5. **Senior**: Nếu hệ thống BFF đã cấu hình `SameSite=Strict` thì ứng dụng có an toàn 100% trước CSRF không?
   - *Đáp án*: Khá an toàn nhưng không phải 100%. Nếu ứng dụng có lỗ hổng XSS, kẻ tấn công có thể tiêm JavaScript thực thi mã ngay trên chính domain hiện tại. Lúc đó request xuất phát từ cùng Site, SameSite bị vô hiệu hóa, và Cookie HttpOnly vẫn tự động được gửi đi trong âm thầm. Do đó, chống CSRF không có nghĩa là được phép bỏ qua việc sanitize (làm sạch) chống XSS.

## 7. Tài liệu tham khảo (References)

- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [MDN Web Docs: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [IETF RFC 6265bis: Cookies: HTTP State Management Mechanism](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis)
