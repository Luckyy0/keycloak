> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Cấu hình và kiểm thử các phương thức xác thực Client (Client Authentication) khác nhau trong Keycloak bao gồm Client Secret, Private Key JWT và X509 mTLS.

## 1. Kịch bản Thực hành (Lab Scenario)
Trong môi trường OAuth2/OIDC, không chỉ người dùng cần xác thực, mà bản thân các ứng dụng (Clients) cũng cần chứng minh danh tính của mình với Authorization Server (Keycloak) khi yêu cầu cấp Token (ví dụ qua luồng Client Credentials hoặc Authorization Code). Bài lab này sẽ hướng dẫn bạn thiết lập 3 client khác nhau tương ứng với 3 phương thức bảo mật từ cơ bản đến nâng cao.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Keycloak Server đang chạy (phiên bản 20+).
- Công cụ dòng lệnh: `curl`, `openssl`.
- Một Realm mới (ví dụ: `auth-realm`) đã được tạo sẵn trong Keycloak.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 1: Thực hành với Client ID & Secret Basic**
1. Trong `auth-realm`, tạo một Client mới tên là `client-basic`.
2. Bật `Client authentication` (On) và bật `Service accounts roles` (để dùng Client Credentials grant).
3. Trong tab **Credentials**, chọn **Client Authenticator** là `Client Id and Secret`. Copy lại giá trị Secret (ví dụ: `abcd-1234`).
4. Dùng cURL để lấy token:
```bash
curl -X POST http://localhost:8080/realms/auth-realm/protocol/openid-connect/token \
  -u "client-basic:abcd-1234" \
  -d "grant_type=client_credentials"
```
*(Đây là phương thức Basic Auth, mã hóa base64 chuỗi `client_id:client_secret` trong header `Authorization: Basic ...`)*

**Bước 2: Thực hành với Private Key JWT**
Phương thức này an toàn hơn vì không gửi Secret qua mạng, thay vào đó Client ký một JWT bằng Private Key của nó.
1. Tạo một cặp khóa RSA bằng openssl:
```bash
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```
2. Tạo Client mới tên là `client-jwt` (Bật Client authentication & Service accounts roles).
3. Tại tab **Credentials**, đổi **Client Authenticator** thành `Signed Jwt`.
4. Tại tab **Keys**, bật `Use JWKS URL` hoặc upload trực tiếp nội dung file `public.pem` (bằng cách chọn Generate -> hoặc Import Certificate).
5. Để sinh token bằng JWT, bạn cần viết một script nhỏ (bằng Python/NodeJS) để tạo JWT được ký bằng `private.pem`. Sau đó gửi request:
```bash
curl -X POST http://localhost:8080/realms/auth-realm/protocol/openid-connect/token \
  -d "grant_type=client_credentials" \
  -d "client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer" \
  -d "client_assertion=<YOUR_SIGNED_JWT>"
```

**Bước 3: Thực hành với X.509 Certificate (mTLS)**
1. Cấu hình Keycloak để bật HTTPS và yêu cầu Client Certificate (cấu hình `--https-client-auth=request` khi khởi động Keycloak).
2. Tạo Client mới tên là `client-mtls` (Bật Client authentication & Service accounts roles).
3. Tại tab **Credentials**, đổi **Client Authenticator** thành `X509 Certificate`.
4. Nhập Regular Expression để khớp với Subject DN của chứng chỉ client (ví dụ: `(.*?)(CN=my-client)(.*?)`).
5. Gọi cURL kèm theo chứng chỉ:
```bash
curl -X POST https://localhost:8443/realms/auth-realm/protocol/openid-connect/token \
  --cert client.crt --key client.key --cacert ca.crt \
  -d "client_id=client-mtls" \
  -d "grant_type=client_credentials"
```

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)
- **Xác minh thành công:** Cả 3 request cURL ở trên đều phải trả về mã HTTP 200 kèm theo JSON chứa `access_token`.
- **Lỗi 401 Unauthorized với Client Secret:** Đảm bảo bạn đã sử dụng cờ `-u` đúng trong cURL, hoặc truyền đúng `client_id` và `client_secret` trong body (`application/x-www-form-urlencoded`).
- **Lỗi Invalid Client Assertion (JWT):** Xảy ra khi chữ ký JWT tạo từ Private Key không khớp với Public Key đã lưu trên Keycloak, hoặc JWT bị hết hạn (trường `exp`), hoặc sai `aud` (Audience phải là URL của endpoint token).
- **Lỗi mTLS Handshake Failure:** Keycloak sẽ từ chối kết nối ngay ở tầng TCP/TLS nếu chứng chỉ client không được ký bởi CA mà Keycloak tin tưởng (Truststore). Hãy kiểm tra lại cấu hình Truststore của Keycloak.
