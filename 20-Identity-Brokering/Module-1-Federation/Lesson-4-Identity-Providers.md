> [!NOTE]
> **Category:** Theory
> **Goal:** Tìm hiểu chi tiết về các loại Identity Providers (IdP) được hỗ trợ trong Keycloak, cách thức cấu hình và cơ chế ánh xạ dữ liệu (Identity Provider Mappers) từ hệ thống ngoài vào Keycloak.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Trong mô hình Identity Brokering, **Identity Provider (IdP)** là một hệ thống bên ngoài quản lý thông tin danh tính của người dùng (như tài khoản, mật khẩu, xác thực đa yếu tố) và chịu trách nhiệm xác thực người dùng đó.
Keycloak hỗ trợ ba nhóm IdPs chính:
1. **Social Networks**: Cung cấp cấu hình tích hợp sẵn cho các nền tảng mạng xã hội phổ biến (Google, Facebook, GitHub, X/Twitter). Dưới nền tảng, chúng thường dựa trên giao thức OAuth 2.0 hoặc OIDC.
2. **OpenID Connect v1.0 (OIDC)**: Hỗ trợ tích hợp với bất kỳ nhà cung cấp nào tuân thủ chuẩn OIDC (như Azure AD, Okta, Auth0, hoặc một Keycloak Server khác).
3. **SAML v2.0**: Chuẩn công nghiệp truyền thống thường thấy trong các hệ thống doanh nghiệp (Enterprise Systems) và tổ chức chính phủ (như Active Directory Federation Services - ADFS).

**Identity Provider Mappers:**
Một trong những tính năng cốt lõi khi kết nối IdP là khả năng dịch thuật dữ liệu (Mappers). Ví dụ: Google trả về `given_name`, nhưng ứng dụng của bạn cần thuộc tính `firstName`. Mapper giúp ánh xạ các claim (trong OIDC) hoặc attribute (trong SAML) từ IdP sang các trường dữ liệu tiêu chuẩn hoặc custom attribute của user cục bộ trong Keycloak.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Khi một IdP được thêm vào Keycloak, hệ thống hoạt động như sau:

```mermaid
flowchart TD
    A[Client Request Login] --> B[Keycloak Redirect to IdP]
    B --> C[IdP Authenticates User]
    C --> D{Loại IdP?}
    D -- Social/OIDC --> E[IdP trả về ID Token (JWT)]
    D -- SAML --> F[IdP trả về SAML Assertion (XML)]
    E --> G[Keycloak xác minh chữ ký (JWKS)]
    F --> H[Keycloak xác minh chữ ký (X.509 Certificate)]
    G --> I[Kích hoạt Identity Provider Mappers]
    H --> I
    I --> J[Trích xuất và chuẩn hóa User Profile]
    J --> K[First/Post Broker Login Flow]
```

**Cơ chế cấp thấp đối với OIDC:**
Keycloak lấy cấu hình động bằng cách truy cập endpoint `/.well-known/openid-configuration` của IdP. Nó tự động nạp các đường dẫn Authorization, Token, UserInfo và JWKS URI. Quá trình kiểm tra token được thực hiện nghiêm ngặt bằng việc đối chiếu chữ ký JWT với public key lấy từ JWKS.

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!WARNING]
> **Cảnh báo về OpenID Connect JWKS**: Khi tích hợp OIDC, bắt buộc phải sử dụng `JWKS URI` thay vì hardcode Public Key. Các nhà cung cấp (như Google/Microsoft) luân chuyển (rotate) khóa của họ liên tục. Hardcode key sẽ gây downtime toàn hệ thống khi khóa bị xoay.

> [!IMPORTANT]
> **Bảo mật SAML XML External Entity (XXE)**: Khi sử dụng SAML, Keycloak phải parse XML do IdP gửi về. Đảm bảo bạn đang sử dụng phiên bản Keycloak mới nhất để tránh các lỗ hổng XXE, và luôn bật tính năng `Validate Signature` trên các SAML Assertions để chống lại các cuộc tấn công giả mạo (Spoofing).

- **Kiểm soát quyền truy cập**: Sử dụng `Post Broker Login Flow` hoặc Mappers để chặn đăng nhập từ các người dùng thuộc một nhóm (Group/Tenant) nhất định trên IdP, tránh tình trạng "ai cũng có thể đăng nhập".

## 4. Cấu hình minh họa thực tế (Configuration Examples)

### Ví dụ Cấu hình Google Social Login:
1. Lấy thông tin từ [Google Cloud Console](https://console.cloud.google.com/):
   - Tạo OAuth 2.0 Client ID.
   - Authorized redirect URIs: `https://<your-keycloak-domain>/realms/<realm>/broker/google/endpoint`
2. Cấu hình trong Keycloak:
   - Vào `Identity Providers` -> Chọn `Google`.
   - Client ID: `<Google Client ID>`
   - Client Secret: `<Google Client Secret>`
   - Default Scopes: `openid profile email`

### Cấu hình Hardcoded Role Mapper (Tự động gán quyền):
Nếu muốn tất cả người dùng đăng nhập từ IdP có tên là "PartnerIdp" đều tự động được gán quyền "partner-role" trong Keycloak.
1. Chuyển sang tab `Mappers` của IdP đó.
2. Create Mapper -> Name: `Assign Partner Role`.
3. Mapper Type: `Hardcoded Role`.
4. Role: `partner-role`.

## 5. Trường hợp ngoại lệ (Edge Cases)

- **CORS block khi Redirect**: Đôi khi ứng dụng SPA cố gắng gọi ngầm Keycloak, và Keycloak lại redirect ngầm sang IdP. Các IdP (như Azure AD) chặn iframe (X-Frame-Options) hoặc không cho phép redirect tự động. Khắc phục: Phải bắt lỗi `login_required` và thực hiện redirect toàn trang thay vì gọi API ngầm.
- **IdP không trả về Email**: Một số hệ thống SAML nội bộ hoặc Social login như GitHub (nếu user ẩn email) không cung cấp email. Vì Keycloak mặc định yêu cầu email, luồng đăng nhập có thể bị kẹt. Khắc phục: Yêu cầu người dùng tự nhập email trong bước `Update Profile` (First Broker Login Flow).

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**Junior Level:**
1. Kể tên 3 nhóm Identity Providers chính mà Keycloak hỗ trợ.
   - *Đáp án:* Social Providers (Google, FB), OpenID Connect v1.0, và SAML 2.0.
2. Tính năng "Identity Provider Mappers" dùng để làm gì?
   - *Đáp án:* Dùng để ánh xạ, chuyển đổi dữ liệu (Claims/Attributes) từ IdP bên ngoài thành dữ liệu người dùng cục bộ (Attributes/Roles/Groups) trong Keycloak.

**Senior Level:**
3. Khi tích hợp một hệ thống SAML nội bộ vào Keycloak, nếu bạn gặp lỗi "Invalid Signature" dù đã tải đúng chứng chỉ số (Certificate), bạn sẽ kiểm tra những yếu tố nào?
   - *Đáp án:* Cần kiểm tra: (1) Certificate đó là để ký (Signing) hay mã hóa (Encryption). (2) IdP ký trên toàn bộ SAML Response hay chỉ ký phần Assertion. (3) Định dạng XML bị thay đổi ngầm (whitespace/newline) bởi proxy làm hỏng tính toàn vẹn chữ ký. (4) Cấu hình NameID format.
4. Trình bày cách bạn triển khai tính năng "Multi-Tenancy" khi chỉ sử dụng một Keycloak Realm nhưng nối với nhiều Azure AD của các đối tác khác nhau?
   - *Đáp án:* Tạo nhiều IdP (Azure AD 1, Azure AD 2) trong cùng một Realm. Sử dụng `kc_idp_hint` trên Client app để định tuyến người dùng đúng Tenant, hoặc viết một Custom Authenticator cho bước đăng nhập đầu tiên (nhập email -> bóc tách domain -> tự động chuyển hướng đến IdP tương ứng - Home Realm Discovery).
5. Làm thế nào để ngăn chặn người dùng từ một IdP cụ thể (Ví dụ: IdP của đối tác) đăng nhập vào ứng dụng dành riêng cho nhân viên nội bộ?
   - *Đáp án:* Sử dụng chức năng Client Scope hoặc Authentication Flow đánh giá điều kiện. Trong Post Broker Login flow, thêm một Execution kiểm tra thuộc tính user hoặc IdP gốc; nếu không thoả mãn sẽ trả về lỗi `Access Denied`.

## 7. Tài liệu tham khảo (References)

- [Keycloak Docs - Identity Providers](https://www.keycloak.org/docs/latest/server_admin/#_identity_broker)
- [OpenID Connect Core 1.0 Specification](https://openid.net/specs/openid-connect-core-1_0.html)
- [Google Identity Platform - OAuth 2.0](https://developers.google.com/identity/protocols/oauth2)
