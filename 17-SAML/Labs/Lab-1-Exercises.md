> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Thiết lập cấu hình tích hợp SAML 2.0 giữa Keycloak (đóng vai trò là Identity Provider - IdP) và một ứng dụng mẫu (đóng vai trò là Service Provider - SP). Kiểm chứng luồng đăng nhập SSO và phân tích cấu trúc của SAML Assertion.

## 1. Kịch bản Thực hành (Lab Scenario)

Công ty TNHH XYZ đang muốn triển khai Single Sign-On (SSO) cho một ứng dụng nội bộ cũ hỗ trợ chuẩn SAML 2.0. Bạn được giao nhiệm vụ cấu hình Keycloak Server để làm IdP, thiết lập một SAML Client, và kết nối với ứng dụng Service Provider giả lập (SAML SP Test App). Mục tiêu là chứng minh rằng người dùng có thể xác thực tại Keycloak và ứng dụng nhận được đầy đủ thông tin định danh (Tên đăng nhập, Email, Roles) thông qua `SAMLResponse`.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Đã cài đặt và cấu hình Keycloak (phiên bản 21+ trở lên, ví dụ dùng Docker: `quay.io/keycloak/keycloak:latest`).
- Truy cập thành công vào giao diện quản trị Admin Console (`http://localhost:8080/admin`).
- Một Realm đang hoạt động (Ví dụ: realm `test-realm`).
- Một người dùng thử nghiệm (Ví dụ: `user1` với password là `admin`, đã được gán email `user1@example.com` và một số Roles bất kỳ).
- Công cụ giả lập SP: Sử dụng **SAML Tracer** (Tiện ích mở rộng trên Chrome/Firefox) và **SAMLTest.id** hoặc chạy một Docker container SP giả lập như `kristophjunge/test-saml-idp` (Tuy nhiên trong Lab này, ta sẽ dùng [samltest.id](https://samltest.id) làm SP trên môi trường Internet để tiện lợi).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Lấy Metadata của IdP (Keycloak)
1. Đăng nhập vào Keycloak Admin Console.
2. Chọn realm `test-realm`.
3. Nhấp vào tab **Realm settings** ở thanh menu trái.
4. Ở tab **General**, cuộn xuống dưới cùng và tìm phần **Endpoints**.
5. Nhấp vào liên kết **SAML 2.0 Identity Provider Metadata**.
6. Trình duyệt sẽ hiển thị một file XML. Lưu file này vào máy tính với tên `keycloak-idp-metadata.xml`. Bạn sẽ dùng nó để cấu hình cho ứng dụng SP.

### Bước 3.2: Khai báo IdP với ứng dụng giả lập (samltest.id)
1. Truy cập trang web `https://samltest.id/`.
2. Chọn tab **UPLOAD METADATA**.
3. Tải lên file `keycloak-idp-metadata.xml` mà bạn vừa lưu ở Bước 3.1.
4. Hệ thống samltest.id sẽ báo thành công. (Điều này thiết lập Trust Establishment - samltest giờ đã tin tưởng Keycloak của bạn).

### Bước 3.3: Lấy Metadata của ứng dụng giả lập (samltest.id)
1. Tại trang chủ `https://samltest.id/`, chọn tab **DOWNLOAD METADATA**.
2. Lưu file XML về máy tính với tên `samltest-sp-metadata.xml`. Đây là file cấu hình của ứng dụng chứa các Endpoint và chứng chỉ của nó.

### Bước 3.4: Tạo SAML Client trên Keycloak
1. Trở lại Keycloak Admin Console (`test-realm`).
2. Chọn mục **Clients** -> **Import client**.
3. Nhấn **Browse** và chọn file `samltest-sp-metadata.xml` vừa tải. Nhấn **Save**.
4. Keycloak sẽ tự động cấu hình Client ID (thường là `https://samltest.id/saml/sp`), các ACS URL và X.509 Certificate để kiểm tra chữ ký.
5. Kiểm tra cấu hình:
   - Client protocol: Bắt buộc là `saml`.
   - Name ID format: Chọn `email` hoặc `username`.
   - Bật tùy chọn `Force name ID format`.
   - Bật `Sign assertions` (Ký điện tử các khẳng định).
   - Cuộn xuống nhấn **Save**.

### Bước 3.5: Cấu hình Mappers (Thuộc tính gửi kèm)
Mặc định Keycloak chỉ gửi NameID. Ta cần cấu hình thêm để nó gửi User Roles và Email vào `AttributeStatement` của Assertion.
1. Chuyển sang tab **Client scopes**.
2. Nhấn vào liên kết của scope mặc định (ví dụ `samltest.id-dedicated`).
3. Chọn tab **Mappers** -> **Configure a new mapper** -> **User Property**.
   - **Name:** `email_mapper`
   - **Property:** `email`
   - **SAML Attribute Name:** `email`
   - **SAML Attribute NameFormat:** `Basic`
   - Nhấn **Save**.
4. Lặp lại bước tạo Mapper nhưng chọn **Role list** để thêm thuộc tính `Role` cho user.

### Bước 3.6: Thực thi luồng SP-Initiated Login
1. Cài đặt tiện ích mở rộng **SAML-tracer** trên trình duyệt Chrome/Firefox để giám sát gói tin HTTP.
2. Mở tiện ích SAML-tracer (cửa sổ sẽ pop-up).
3. Quay lại trang web `https://samltest.id/`. Chọn tab **TEST YOUR IDP**.
4. Nhập Entity ID của Keycloak IdP (Bạn có thể xem trên Keycloak trong phần Realm Settings, thường là `http://localhost:8080/realms/test-realm`).
5. Nhấn **Login**.
6. Trang web sẽ chuyển hướng sang Keycloak Login form (đây là luồng HTTP-Redirect chứa `SAMLRequest`).
7. Nhập thông tin của `user1` (username/password).
8. Keycloak xác thực thành công và chuyển hướng ngược lại trang `samltest.id` (đây là luồng HTTP-POST chứa `SAMLResponse`).

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu thành công
- Trang `samltest.id` phải hiển thị màn hình báo xanh (Success) hoặc in ra một loạt các thuộc tính của user (Attributes) mà Keycloak đã gửi sang (bao gồm NameID, Email, Roles).

### 4.2. Phân tích gói tin bằng SAML-Tracer
- Mở cửa sổ SAML-tracer đã bật ở Bước 3.6.
- Tìm dòng có chữ `SAML` được tô đậm màu cam. Sẽ có hai dòng: một cho Request, một cho Response.
- Chọn dòng **POST** về `samltest.id/saml/acs`.
- Chuyển sang tab **SAML** trong SAML-tracer. Bạn sẽ đọc được nguyên bản cấu trúc XML.
- Tìm thẻ `<saml:AttributeStatement>` để kiểm chứng mapper đã cấu hình ở Bước 3.5 có hoạt động không (có thẻ chứa giá trị email `user1@example.com` hay không).
- Tìm thẻ `<ds:Signature>` để thấy rằng Assertion đã được mã hóa toàn vẹn.

### 4.3. Khắc phục sự cố thường gặp (Troubleshooting)
- **Lỗi `Invalid redirect uri` trên Keycloak:** Bạn đã cấu hình sai AssertionConsumerService URL trong Keycloak Client. Hãy xóa client và Import lại Metadata từ `samltest.id` một cách chính xác.
- **Lỗi `SAML Assertion signature is invalid` trên SP:** Keycloak dùng sai thuật toán ký hoặc SP không cập nhật đúng Metadata có chứa Public Key của Keycloak. Hãy kiểm tra lại Key Rotation trên Keycloak và tải lại IdP Metadata.
- **Không hiển thị thuộc tính Email:** Kiểm tra lại NameID Format ở Bước 3.4. Nếu cấu hình NameID nhưng User thực tế trong Keycloak chưa được điền thông tin Email, quá trình phát sinh Assertion sẽ gặp lỗi Null Pointer. Đảm bảo user có đầy đủ thông tin cơ bản.
