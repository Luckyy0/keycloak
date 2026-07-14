> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Nắm vững cơ chế xác thực ứng dụng bằng phương thức `client_secret_basic` theo tiêu chuẩn OAuth 2.0 (RFC 6749), cách mã hóa HTTP Header, và nhận thức được các ưu nhược điểm cũng như chuẩn mực bảo mật khi áp dụng.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Trong hệ sinh thái OAuth 2.0 và OpenID Connect, Authorization Server (Keycloak) cần phải biết và xác thực danh tính của Ứng dụng (Client) mỗi khi Client gọi các API nội bộ như Token Endpoint (để đổi Authorization Code lấy Access Token) hay Introspection Endpoint. Quá trình này được gọi là **Client Authentication**.

`client_secret_basic` là phương thức xác thực Client phổ biến và mặc định nhất. Nó dựa trên cơ chế HTTP Basic Authentication truyền thống (RFC 7617). Trong phương thức này, Client sẽ gộp `client_id` và `client_secret` lại thành một chuỗi duy nhất theo định dạng `client_id:client_secret`. Sau đó, chuỗi này được mã hóa Base64 và đặt vào Header `Authorization` của HTTP Request với tiền tố `Basic`.

Bản chất của phương thức này giải quyết bài toán: Làm thế nào để một Confidential Client (Client bảo mật, ví dụ: Backend chạy trên Server) chứng minh được danh tính của mình mà không cần lộ mật khẩu trong URL hoặc Body của giao thức HTTP theo cách rườm rà. Do sử dụng chuẩn HTTP cơ sở, nó được hỗ trợ bởi hầu hết các thư viện và framework trên thế giới. Tuy nhiên, nó yêu cầu một kết nối an toàn ở tầng mạng (Transport Layer).

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Dưới đây là cơ chế cấp thấp trình bày cách thức một Client xác thực bằng `client_secret_basic` tại Token Endpoint của Keycloak.

```mermaid
sequenceDiagram
    autonumber
    participant Client as Backend Client
    participant Keycloak as Keycloak (Token Endpoint)
    participant DB as Keycloak Database

    Note over Client: 1. Mã hóa URL (URL Encoding)<br>client_id và client_secret
    Note over Client: 2. Nối chuỗi (Concatenate)<br>"client_id:client_secret"
    Note over Client: 3. Mã hóa Base64 (Base64 Encoding)<br>VD: base64("my-client:my-secret") -> "bXktY2xpZW50Om15LXNlY3JldA=="
    
    Client->>Keycloak: HTTP POST /protocol/openid-connect/token<br>Header: Authorization: Basic bXkt...<br>Body: grant_type=client_credentials
    
    Keycloak->>Keycloak: Đọc Header "Authorization"<br>Loại bỏ tiền tố "Basic "<br>Giải mã Base64 (Base64 Decode)
    Keycloak->>Keycloak: Tách chuỗi bằng dấu ":" để lấy<br>client_id và client_secret dạng thô (plain-text)
    
    Keycloak->>DB: Truy vấn Client theo client_id
    DB-->>Keycloak: Trả về thông tin Client & Hashed Secret
    
    Keycloak->>Keycloak: Băm (Hash) client_secret nhận được<br>và so sánh với Hashed Secret trong DB
    
    alt Khớp (Match)
        Keycloak-->>Client: HTTP 200 OK (Cấp Access Token)
    else Không khớp (Mismatch)
        Keycloak-->>Client: HTTP 401 Unauthorized
    end
```

**Giải thích chi tiết các bước cấp thấp:**
1. **Chuẩn bị Dữ liệu (RFC 6749 Section 2.3.1):** Client bắt buộc phải mã hóa dạng `application/x-www-form-urlencoded` cho cả `client_id` và `client_secret` trước khi ghép chúng. Điều này để tránh lỗi nếu `client_id` hoặc `client_secret` vô tình chứa dấu hai chấm (`:`).
2. **Mã hóa Header:** Phép biến đổi Base64 không phải là mã hóa an toàn (Encryption), nó chỉ là cách biểu diễn dữ liệu (Encoding) để truyền qua HTTP an toàn, không bị biến dạng các ký tự.
3. **Phân tích (Parsing) tại Keycloak:** Khi nhận request, Keycloak bóc tách HTTP Header. Nếu header hợp lệ, Keycloak sẽ truy xuất thông tin Client từ cơ sở dữ liệu.
4. **Xác thực Mật khẩu (Secret Validation):** Keycloak không bao giờ lưu trữ `client_secret` dưới dạng plain-text trong Database. Nó lưu phiên bản đã được băm (Salted Hash, thường là PBKDF2). Keycloak sẽ băm cái secret mà Client gửi lên và thực hiện so sánh chuỗi (Constant-time string comparison) để chống lại tấn công định thời gian (Timing Attack).

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!CAUTION]
> Tuyệt đối không sử dụng `client_secret_basic` qua đường truyền HTTP không được mã hóa. Vì Base64 có thể bị giải mã (decode) dễ dàng, kẻ tấn công (Man-in-the-Middle - MitM) có thể đánh cắp toàn bộ quyền truy cập. **Bắt buộc sử dụng HTTPS (TLS).**

> [!WARNING]
> Không bao giờ tích hợp Client Secret vào các ứng dụng Public (Single Page Applications - React/Angular, hoặc Mobile Apps). Các ứng dụng này không thể giữ bí mật mã nguồn, dẫn đến việc lộ Secret. Với Public Client, hãy dùng luồng PKCE thay vì Client Secret.

> [!IMPORTANT]
> - **Luân chuyển (Rotate) Secret định kỳ:** Thiết lập chính sách để thay đổi `client_secret` định kỳ (ví dụ: mỗi 6-12 tháng) nhằm giảm thiểu rủi ro khi Secret bị rò rỉ (leak) trong logs hoặc code repository.
> - **Giám sát Log (Log Monitoring):** Cảnh giác với các HTTP lỗi `401 Unauthorized` liên tục tại Token Endpoint, đây có thể là dấu hiệu của tấn công Brute-Force hoặc Credential Stuffing nhằm dò tìm `client_secret`.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

**Cấu hình trên Keycloak Admin Console:**
1. Mở realm tương ứng, vào menu **Clients**.
2. Chọn Client cần cấu hình, chuyển sang tab **Credentials**.
3. Tại trường **Client Authenticator**, chọn **Client Id and Secret**.
4. (Tuỳ chọn) Bạn có thể sao chép chuỗi Secret được Keycloak sinh ra ngẫu nhiên tại đây.

**Minh họa bằng cURL để lấy Token:**
Giả sử:
- `client_id` = `my-backend-app`
- `client_secret` = `Th1s1sS3cr3t`
- Nối chuỗi: `my-backend-app:Th1s1sS3cr3t`
- Chuỗi Base64: `bXktYmFja2VuZC1hcHA6VGgxczFzUzNjcjN0`

```bash
curl -X POST "https://sso.example.com/realms/my-realm/protocol/openid-connect/token" \
     -H "Authorization: Basic bXktYmFja2VuZC1hcHA6VGgxczFzUzNjcjN0" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=client_credentials"
```

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Ký tự đặc biệt trong Client ID/Secret:** Nếu Client ID có chứa dấu hai chấm `:`, quá trình ghép chuỗi và tách chuỗi của Keycloak có thể bị lỗi, dẫn đến `401 Unauthorized`. *Khắc phục:* RFC yêu cầu phải sử dụng URL encoding cho cả `client_id` và `client_secret` trước khi ghép chúng lại với nhau.
- **Header quá lớn:** Một số WAF (Web Application Firewall) hoặc Nginx/Apache có thể cấu hình kích thước HTTP Header tối đa (ví dụ `large_client_header_buffers`). Mặc dù ít khi xảy ra, nhưng nếu Base64 token quá dài, Nginx có thể chặn yêu cầu với lỗi `413 Payload Too Large` hoặc `431 Request Header Fields Too Large`.
- **Brute-Force & Account Lockout:** Nếu hệ thống bị tấn công Brute-Force vào Client Authenticator, tính năng Brute-Force protection của Keycloak có thể tạm thời khóa Client (Lock), từ chối cả các request hợp lệ, gây ra trạng thái từ chối dịch vụ (Denial of Service).

## 6. Câu hỏi Phỏng vấn (Interview Questions)

1. **(Junior)** Cơ chế `client_secret_basic` khác gì so với việc gửi mật khẩu trên form đăng nhập HTML?
   - *Đáp án:* Gửi mật khẩu form HTML đưa thông tin vào phần Body (Payload) của HTTP POST request. Trong khi đó `client_secret_basic` nhúng thẳng thông tin định danh và mật khẩu vào HTTP Header dưới dạng mã hóa Base64 theo chuẩn HTTP Basic Auth.
2. **(Junior)** Mã hóa Base64 trong `client_secret_basic` có an toàn không? Tại sao?
   - *Đáp án:* Base64 **không an toàn**. Nó chỉ là Encoding (mã hóa để máy tính dễ xử lý, không có khóa), bất kỳ ai lấy được chuỗi Base64 đều có thể đảo ngược lại thành plain-text. Nó chỉ an toàn khi kết hợp với kênh truyền mã hóa như TLS/HTTPS.
3. **(Senior)** So sánh `client_secret_basic` và `client_secret_post`. Khi nào nên dùng loại nào?
   - *Đáp án:* `client_secret_basic` truyền qua Header, còn `client_secret_post` truyền thông tin qua Form Body (application/x-www-form-urlencoded). `basic` là chuẩn chung và được khuyên dùng hơn (RFC 6749). Tuy nhiên một số thư viện hoặc môi trường cũ có thể chặn việc tùy chỉnh Header `Authorization`, khi đó phải dùng `post` như một phương án thay thế.
4. **(Senior)** Nếu Client (như một Single Page App) không thể dùng `client_secret_basic`, ta dùng gì thay thế để an toàn?
   - *Đáp án:* Public Client không thể bảo vệ bí mật. Ta phải bỏ hoàn toàn `client_secret` và sử dụng luồng Authorization Code kết hợp với **PKCE** (Proof Key for Code Exchange).
5. **(Senior)** Theo chuẩn RFC 6749, nếu một Client gửi thông tin xác thực ở cả Header (Basic) và Body (Post) thì Authorization Server phải xử lý thế nào?
   - *Đáp án:* Theo RFC, Server phải từ chối (Reject) yêu cầu nếu tìm thấy nhiều hơn một phương thức xác thực để ngăn chặn các kiểu tấn công lách luật hoặc chèn ép tham số (Parameter Pollution).

## 7. Tài liệu tham khảo (References)
- [RFC 6749: The OAuth 2.0 Authorization Framework - Section 2.3.1 (Client Password)](https://datatracker.ietf.org/doc/html/rfc6749#section-2.3.1)
- [RFC 7617: The 'Basic' HTTP Authentication Scheme](https://datatracker.ietf.org/doc/html/rfc7617)
- [Keycloak Documentation: Server Administration Guide - Client Authentication](https://www.keycloak.org/docs/latest/server_admin/#_client-credentials)
