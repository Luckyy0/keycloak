> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Thiết lập và quản lý Client Scopes trong Keycloak. Sinh viên sẽ học cách tạo một Scope tùy chỉnh, cấu hình Protocol Mappers để nhúng dữ liệu (claims) vào Access Token, và kiểm thử sự khác biệt giữa Default Scope và Optional Scope.

## 1. Kịch bản Thực hành (Lab Scenario)

Giả sử ứng dụng của bạn là một Hệ thống Quản trị Nhân sự (`HR-App`). Ứng dụng này khi lấy Access Token cần biết "Phòng ban" (Department) của người dùng để quyết định hiển thị module tương ứng.

Tuy nhiên, bạn không muốn thông tin Phòng ban lúc nào cũng xuất hiện trong mọi Token (để giảm kích thước Token). Bạn quyết định tạo ra một **Client Scope** tên là `department-scope` với cấu hình ánh xạ thuộc tính (User Attribute Mapper) và cấu hình nó ở dạng **Optional Scope**. Nghĩa là ứng dụng Client phải chủ động yêu cầu scope `department-scope` khi đăng nhập thì Token trả về mới có thông tin này.

Nhiệm vụ của bạn: Tạo cấu hình Client Scope, gắn vào Client và kiểm tra sự thay đổi của Token dựa trên các tham số cấu hình.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Máy chủ Keycloak đang chạy ở phiên bản mới nhất.
- Truy cập vào Keycloak Admin Console với quyền quản trị viên.
- Đã có sẵn một Realm (VD: `myrealm`).
- Đã tạo một User (VD: `alice`) và đã cấu hình mật khẩu.
- Đã tạo một Client (VD: `hr-client`) với tính năng **Standard Flow** (Authorization Code) hoặc **Direct Access Grants** (Password) được bật.
- Cài đặt Postman hoặc cURL để giả lập request lấy Token.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Gán dữ liệu thuộc tính cho User

1. Trên Admin Console, chọn Realm `myrealm`.
2. Vào **Users** -> Nhấp vào user `alice`.
3. Chuyển sang tab **Attributes**.
4. Thêm một thuộc tính mới:
   - Key: `department`
   - Value: `IT-Security`
5. Nhấp **Save**.

### Bước 3.2: Tạo Client Scope và Mapper

1. Trình đơn bên trái, chọn **Client Scopes**.
2. Nhấp **Create client scope**.
3. Điền thông tin:
   - **Name**: `department-scope`
   - **Type**: `Default` (ta sẽ đổi ở bước sau khi gắn vào client).
   - **Include in token scope**: Bật (ON) - Đảm bảo tên scope này xuất hiện trong claim `scope` của token.
4. Nhấp **Save**.
5. Sau khi lưu, bạn đang ở trang chi tiết của `department-scope`. Chuyển sang tab **Mappers**.
6. Nhấp **Configure a new mapper** -> Chọn **User Attribute**.
7. Điền thông tin ánh xạ:
   - **Name**: `department-mapper`
   - **User Attribute**: `department` (Khớp đúng với Key đã tạo ở bước 3.1)
   - **Token Claim Name**: `user_department` (Đây là tên trường sẽ xuất hiện trong file JSON Token).
   - **Claim JSON Type**: `String`
   - **Add to ID token**: Bật (ON)
   - **Add to access token**: Bật (ON)
   - **Add to userinfo**: Bật (ON)
8. Nhấp **Save**.

### Bước 3.3: Gắn Client Scope vào Client (Dạng Optional)

1. Trình đơn bên trái, chọn **Clients** -> Nhấp vào client `hr-client`.
2. Chuyển sang tab **Client scopes**.
3. Bạn sẽ thấy danh sách các scopes mặc định (như `email`, `profile`).
4. Nhấp nút **Add client scope**.
5. Tìm `department-scope`, chọn nó, và nhấp **Add -> Optional**.
   *(Giải thích: Optional có nghĩa là client phải gửi kèm tham số `scope=department-scope` trong request thì mapper mới hoạt động).*

### Bước 3.4: Lấy Token thông qua Postman

Để chứng minh sự khác biệt, ta sẽ thực hiện 2 lần lấy Token.

**Lần 1: Lấy Token KHÔNG kèm tham số scope tùy chỉnh**
1. Mở Postman, tạo request POST tới endpoint:
   `http://localhost:8080/realms/myrealm/protocol/openid-connect/token`
2. Tại tab `Body` (chọn `x-www-form-urlencoded`), điền:
   - `client_id`: `hr-client`
   - `grant_type`: `password`
   - `username`: `alice`
   - `password`: `(mật khẩu của alice)`
   - *Không truyền trường scope.*
3. Gửi Request và copy chuỗi `access_token` từ kết quả.
4. Dán chuỗi đó lên [jwt.io](https://jwt.io).
5. Bạn sẽ thấy trong Payload KHÔNG CÓ trường `user_department`.

**Lần 2: Lấy Token CÓ yêu cầu Scope Optional**
1. Quay lại Postman, thêm một tham số vào Body:
   - `scope`: `openid department-scope`
   *(Lưu ý: Luôn truyền `openid` để kích hoạt giao thức OIDC).*
2. Gửi Request và copy `access_token` mới.
3. Giải mã trên jwt.io.
4. Lần này, bạn sẽ thấy Payload xuất hiện thêm thông tin:
   `"user_department": "IT-Security"`

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu (Verification)

Quá trình Lab được xem là thành công nếu:
- Khi request có kèm `scope=department-scope`, JWT Access Token sinh ra có chứa claim `user_department` mang giá trị `IT-Security`.
- Khi request không kèm scope đó, thông tin `user_department` bị lược bỏ. Điều này chứng minh tính chất "Optional" của Scope đã hoạt động đúng như thiết kế, giúp bảo vệ quyền riêng tư và tối ưu kích thước Token.

### 4.2. Khắc phục sự cố (Troubleshooting)

- **Token hoàn toàn không chứa claim `user_department` dù đã gửi đúng scope:**
  - *Nguyên nhân 1:* Cấu hình Mapper bị sai tên User Attribute. Keycloak phân biệt chữ hoa, chữ thường. Hãy chắc chắn attribute của user là `department` chứ không phải `Department`.
  - *Nguyên nhân 2:* Client Scope chưa được gán vào Client. Nếu bạn tạo Scope xong mà quên gán vào Client `hr-client` ở tab Client Scopes, Keycloak sẽ từ chối scope request đó (scope sẽ bị bỏ qua một cách im lặng).
- **Lỗi `invalid_scope` trả về từ API:**
  - *Nguyên nhân:* Tên scope trong Postman gửi lên bị viết sai chính tả (ví dụ gửi lên `department-scopes`).
  - *Khắc phục:* Kiểm tra lại chuẩn xác tên Client Scope.
- **Dữ liệu trả về là null dù đã ánh xạ:**
  - *Nguyên nhân:* User `alice` chưa được định nghĩa giá trị cho attribute `department`, hoặc bạn đang dùng một tài khoản User khác để test.
  - *Khắc phục:* Vào lại hồ sơ User trên Admin Console để kiểm tra tab Attributes.
