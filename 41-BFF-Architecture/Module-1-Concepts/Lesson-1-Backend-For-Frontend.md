> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Hiểu sâu về kiến trúc Backend-For-Frontend (BFF), lý do tại sao nó là bắt buộc cho các ứng dụng SPA/Mobile hiện đại thay vì lưu trữ Token trên Client.

## 1. Lý thuyết chuyên sâu (Detailed Theory)
Backend-For-Frontend (BFF) là một mẫu thiết kế (design pattern) nơi một thành phần Backend chuyên trách được tạo ra để phục vụ riêng cho một Frontend cụ thể (SPA, Mobile App).
Thay vì Frontend giao tiếp trực tiếp với các Resource Server (Microservices) hoặc Authorization Server (Keycloak), mọi Request từ Frontend sẽ đi qua BFF.

**Tại sao BFF tồn tại?**
Trong lịch sử, Single Page Applications (SPA) thường sử dụng luồng Implicit Flow hoặc Authorization Code Flow with PKCE để lấy Access Token và lưu trữ tại `localStorage` hoặc `sessionStorage` của trình duyệt. 
Điều này tạo ra lỗ hổng bảo mật nghiêm trọng: XSS (Cross-Site Scripting) có thể đánh cắp Token dễ dàng.
Mẫu BFF giải quyết vấn đề này bằng cách:
- Di chuyển trách nhiệm xác thực (Authentication) từ Frontend xuống BFF.
- Lưu trữ Access Token, Refresh Token an toàn tại Backend (trong Session hoặc Cache).
- Frontend chỉ giao tiếp với BFF thông qua các HTTP-only, Secure, SameSite Cookie. Do Cookie không thể đọc được bằng JavaScript, nguy cơ XSS đánh cắp Token bị loại bỏ hoàn toàn.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

```mermaid
sequenceDiagram
    participant Browser as SPA (Browser)
    participant BFF as BFF (Spring Cloud Gateway)
    participant Keycloak as Keycloak (Auth Server)
    participant Resource as Resource Server

    Browser->>BFF: 1. GET /api/data (No Session)
    BFF-->>Browser: 2. 302 Redirect to Keycloak Login
    Browser->>Keycloak: 3. User Authenticates
    Keycloak-->>Browser: 4. 302 Redirect to BFF with Auth Code
    Browser->>BFF: 5. GET /login/oauth2/code/keycloak
    BFF->>Keycloak: 6. Exchange Auth Code for Tokens (Client ID + Secret)
    Keycloak-->>BFF: 7. Access Token, Refresh Token, ID Token
    BFF->>BFF: 8. Store Tokens in Redis Session
    BFF-->>Browser: 9. Set-Cookie: SESSION=xyz; HttpOnly; Secure
    
    Browser->>BFF: 10. GET /api/data (Cookie: SESSION=xyz)
    BFF->>BFF: 11. Extract Access Token from Session
    BFF->>Resource: 12. GET /api/data (Authorization: Bearer Token)
    Resource-->>BFF: 13. JSON Data
    BFF-->>Browser: 14. JSON Data
```

**Giải thích step-by-step:**
1. Trình duyệt gửi Request tới BFF mà chưa có Session hợp lệ.
2. BFF nhận diện User chưa đăng nhập, kích hoạt OIDC Login và chuyển hướng người dùng đến Keycloak.
3. Người dùng nhập thông tin xác thực tại Keycloak.
4. Keycloak trả về Authorization Code thông qua chuyển hướng trình duyệt.
5. BFF nhận Authorization Code từ URL.
6. BFF sử dụng Back-channel (giao tiếp Server-to-Server) gửi Authorization Code, Client ID, Client Secret để đổi lấy Tokens.
7. Keycloak trả về Access Token, Refresh Token.
8. BFF lưu các Token này vào lưu trữ Session (như Redis).
9. BFF thiết lập một Cookie mã hóa (HttpOnly, Secure) cho trình duyệt.
10. Lần tiếp theo SPA gọi API, trình duyệt tự động đính kèm Cookie.
11. BFF lấy Session, trích xuất Access Token.
12. BFF Proxy Request đến Resource Server, tự động chèn Access Token vào `Authorization` Header.

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!WARNING]
> Không bao giờ để rò rỉ Access Token hoặc ID Token từ BFF xuống Frontend. Frontend không cần biết chi tiết Token, chỉ cần biết trạng thái đăng nhập.

> [!IMPORTANT]
> Cấu hình Cookie phải luôn bật cờ `HttpOnly`, `Secure` (trong môi trường Production), và `SameSite=Strict` hoặc `Lax` để chống lại các tấn công CSRF.

- **Chống CSRF (Cross-Site Request Forgery):** Vì BFF sử dụng Cookie để duy trì trạng thái, nó phải triển khai các cơ chế bảo vệ CSRF như Anti-CSRF Tokens hoặc lợi dụng thuộc tính `SameSite` của Cookie.
- **Quản lý Session:** BFF nên sử dụng kho lưu trữ Session phân tán như Redis nếu triển khai ở cấu hình High Availability (HA) để đảm bảo tính stateless của các Instance BFF.
- **Timeout và Token Expiry:** Khi Access Token hết hạn, BFF phải tự động sử dụng Refresh Token để lấy Token mới trong suốt quá trình xử lý Request, mà không bắt User phải đăng nhập lại.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sử dụng Spring Cloud Gateway làm BFF tích hợp `spring-boot-starter-oauth2-client`:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          keycloak:
            client-id: bff-client
            client-secret: my-secret
            scope: openid, profile, email
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
        provider:
          keycloak:
            issuer-uri: https://auth.example.com/realms/myrealm
```

Cấu hình bảo vệ chống CSRF với Cookie `XSRF-TOKEN`:

```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    http
        .authorizeExchange(exchanges -> exchanges
            .pathMatchers("/", "/login/**").permitAll()
            .anyExchange().authenticated()
        )
        .oauth2Login(withDefaults())
        .csrf(csrf -> csrf.csrfTokenRepository(CookieServerCsrfTokenRepository.withHttpOnlyFalse()));
    return http.build();
}
```

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Refresh Token Expiry:** Nếu Refresh Token bị hết hạn hoặc bị thu hồi tại Keycloak (do Admin ép buộc Log out), yêu cầu làm mới Token của BFF sẽ thất bại. BFF phải xử lý HTTP 401 từ Keycloak và xóa Session hiện tại, sau đó gửi HTTP 401 xuống SPA để SPA chuyển hướng người dùng đến trang Login.
- **CORS (Cross-Origin Resource Sharing):** Nếu SPA và BFF không nằm trên cùng một Domain (Ví dụ: `app.example.com` và `api.example.com`), Cookie có thể bị chặn bởi chính sách bảo mật của trình duyệt. Tốt nhất là cấu hình Reverse Proxy (như Nginx) để phục vụ cả SPA và BFF dưới cùng một Origin.

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. (Junior) Tại sao không nên lưu trữ Access Token trong `localStorage` của trình duyệt?**
*Đáp án:* Vì `localStorage` có thể bị truy cập dễ dàng bằng JavaScript. Nếu trang web bị tấn công XSS, hacker có thể đánh cắp Access Token và giả mạo người dùng.

**2. (Junior) Cookie đính kèm cờ `HttpOnly` có ý nghĩa gì?**
*Đáp án:* Cờ `HttpOnly` chỉ thị cho trình duyệt không cho phép JavaScript (ví dụ: thông qua `document.cookie`) truy cập vào Cookie này, bảo vệ nó khỏi các cuộc tấn công XSS.

**3. (Senior) Trong kiến trúc BFF, làm thế nào để giải quyết vấn đề CSRF khi đã sử dụng Cookie thay vì Bearer Token?**
*Đáp án:* Có hai cách chính: Thiết lập thuộc tính `SameSite=Strict` (hoặc Lax) cho Cookie, và triển khai cơ chế Double Submit Cookie (trả về một Cookie chứa CSRF Token có thể đọc bằng JS, SPA đọc và gửi lại Token này qua Header cho mỗi Request thay đổi trạng thái).

**4. (Senior) Giải thích sự khác biệt giữa Token Handler Pattern và BFF Pattern?**
*Đáp án:* Token Handler thường là một Reverse Proxy tinh gọn (chỉ làm nhiệm vụ trao đổi Session Cookie lấy Token), trong khi BFF có thể là một dịch vụ đầy đủ tính năng chứa logic tổng hợp dữ liệu (Aggregation), định tuyến (Routing), và tối ưu hóa Payload trả về cho Frontend.

**5. (Senior) Làm thế nào để BFF duy trì tính Stateless (không trạng thái) khi nó phải lưu Token cho Frontend?**
*Đáp án:* BFF lưu trữ mapping giữa Session ID và Token vào một hệ thống External Cache như Redis. Như vậy, bất kỳ Instance nào của BFF cũng có thể xác thực Request nếu kết nối cùng một Redis Cluster.

## 7. Tài liệu tham khảo (References)
- [OAuth 2.0 for Browser-Based Apps (IETF Draft)](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps)
- [BFF Pattern - Sam Newman](https://samnewman.io/patterns/architectural/bff/)
- [Spring Security OAuth2 Client Documentation](https://docs.spring.io/spring-security/reference/reactive/oauth2/client/index.html)
