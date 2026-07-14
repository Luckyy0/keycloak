> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Cấu hình User Federation trong Keycloak để tích hợp và đồng bộ danh tính từ hệ thống OpenLDAP, cho phép người dùng đăng nhập bằng thông tin xác thực sẵn có.

# Lab 1: Tích hợp User Federation với OpenLDAP

## 1. Kịch bản Thực hành (Lab Scenario)
Một doanh nghiệp hiện đang lưu trữ và quản lý tập trung toàn bộ danh tính của nhân viên (username, password, email, phòng ban) bên trong một hệ thống thư mục gốc **OpenLDAP**. 
Khi triển khai một hệ thống quản lý định danh hiện đại sử dụng **Keycloak**, doanh nghiệp không muốn phải tạo lại toàn bộ tài khoản người dùng từ đầu. 
Nhiệm vụ của bạn là cấu hình tính năng **User Federation** trong Keycloak, đóng vai trò như một cầu nối để kết nối đến máy chủ OpenLDAP. Sau khi kết nối thành công, Keycloak sẽ đồng bộ dữ liệu người dùng từ LDAP về Database nội bộ (PostgreSQL) theo cơ chế **Read-Only** (chỉ đọc), giúp người dùng có thể dùng mật khẩu LDAP để đăng nhập vào các ứng dụng thông qua Keycloak (SSO).

## 2. Chuẩn bị Môi trường (Prerequisites)
Để thực hiện bài Lab này, bạn cần có sẵn công cụ **Docker** và **Docker Compose**. Chúng ta sẽ chạy một hệ thống gồm máy chủ Keycloak và một máy chủ OpenLDAP chứa sẵn dữ liệu giả lập.

Nếu bạn chưa có file cấu hình, hãy tạo tệp `docker-compose.yml` với nội dung cơ bản sau:
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    command: start-dev
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - 8080:8080
    depends_on:
      - postgres
  openldap:
    image: osixia/openldap:1.5.0
    environment:
      LDAP_ORGANISATION: "Example Org"
      LDAP_DOMAIN: "example.org"
      LDAP_ADMIN_PASSWORD: "admin"
    ports:
      - 389:389
```

Chạy hệ thống bằng lệnh:
```bash
docker-compose up -d
```
Sau khi các container đã khởi động, bạn có thể truy cập **Keycloak Admin Console** tại `http://localhost:8080` với tài khoản quản trị là `admin` / `admin`. Container OpenLDAP mặc định đã tạo ra Base DN là `dc=example,dc=org` và tài khoản quản trị LDAP là `cn=admin,dc=example,dc=org` cùng mật khẩu `admin`.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Thêm LDAP User Federation Provider
1. Đăng nhập vào **Keycloak Admin Console**.
2. Ở thanh menu bên trái, tìm và chọn mục **User federation**.
3. Tại giao diện chính, nhấn vào nút **Add provider** và chọn **LDAP**.

### Bước 3.2: Cấu hình Kết nối Mạng và Chuẩn Xác Thực (Connection & Authentication)
Khai báo các thông số để Keycloak có thể mở liên kết (TCP socket) tới OpenLDAP:
1. **UI display name**: Nhập `my-company-ldap`.
2. **Vendor**: Chọn `Other` (vì chúng ta đang sử dụng OpenLDAP gốc chứ không phải Active Directory hay Red Hat Directory Server).
3. **Connection URL**: Nhập `ldap://openldap:389`. (Nếu Keycloak và OpenLDAP chạy trong cùng mạng Docker, Keycloak có thể phân giải tên host `openldap`).
4. Nhấn nút **Test connection**. Nếu cấu hình đúng, bạn sẽ nhận được thông báo "Successfully connected to LDAP".

### Bước 3.3: Cấu hình Tìm Kiếm (LDAP searching and updating)
Cuộn xuống phần **LDAP searching and updating** để chỉ định cấu trúc cây thư mục (DIT - Directory Information Tree) mà Keycloak cần đọc:
1. **Users DN**: `dc=example,dc=org` (Tọa độ gốc chứa danh sách người dùng).
2. **Username LDAP attribute**: `cn` (Thuộc tính dùng làm Username).
3. **RDN LDAP attribute**: `cn`.
4. **UUID LDAP attribute**: `entryUUID`.
5. **User object classes**: `inetOrgPerson, organizationalPerson`.

### Bước 3.4: Cấp quyền Đọc Dữ Liệu (Bind Credentials)
Để Keycloak có thể đọc dữ liệu (Search) trong LDAP, nó cần một tài khoản có thẩm quyền (Bind Account):
1. Cuộn xuống mục **Authentication settings**.
2. **Bind type**: Chọn `simple`.
3. **Bind DN**: `cn=admin,dc=example,dc=org`.
4. **Bind credentials**: `admin`.
5. Nhấn nút **Test authentication**. Nếu thành công, bạn sẽ thấy thông báo "Successfully authenticated with LDAP".

### Bước 3.5: Thiết Lập Chế Độ Đồng Bộ (Synchronization settings)
1. Cuộn xuống phần **Synchronization settings**.
2. Bật công tắc **Import Users** thành `ON` (để Keycloak sao chép bản ghi của người dùng về Database cục bộ của nó, giúp tăng tốc độ truy xuất sau này).
3. **Edit mode**: Chọn `READ_ONLY` (Không cho phép Keycloak ghi ngược dữ liệu như đổi tên, đổi mật khẩu vào LDAP).
4. Nhấn nút **Save** ở cuối trang để lưu cấu hình.

### Bước 3.6: Kích Hoạt Đồng Bộ Thủ Công (Trigger Synchronization)
1. Sau khi Save, tại trang cấu hình của provider `my-company-ldap`, cuộn xuống cuối cùng hoặc nhìn phía trên cùng.
2. Nhấn nút **Action** -> **Sync all users** (hoặc nút **Synchronize all users** tùy phiên bản giao diện).
3. Bạn sẽ nhận được thông báo: *"Success! Sync of users finished successfully. 1 imported users, 0 updated users"*. Mặc định image `osixia/openldap` có thể đã tạo sẵn user admin, hoặc nếu bạn đã thêm data giả, số lượng người dùng đồng bộ sẽ hiển thị ở đây.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu
1. Trên Keycloak Admin Console, vào menu **Users**.
2. Nhấn **View all users**. Bạn sẽ thấy người dùng được import từ LDAP xuất hiện trong danh sách (có thể là `admin` của LDAP hoặc các user mà bạn đã populate vào OpenLDAP trước đó).
3. Mở tab trình duyệt ẩn danh (Incognito), truy cập vào **Account Console** của realm hiện tại: `http://localhost:8080/realms/master/account/`.
4. Thực hiện đăng nhập bằng user được đồng bộ từ LDAP và mật khẩu gốc bên LDAP. Nếu hệ thống cho phép đăng nhập thành công, bài Lab đã hoàn thành xuất sắc. Keycloak đã nhận Request, tra cứu user trong PostgreSQL, nhận thấy user này thuộc về LDAP Provider, và đẩy Request xác thực mật khẩu (Bind request) qua LDAP Server để kiểm chứng.

### 4.2. Khắc phục sự cố (Troubleshooting)
- **Lỗi `Connection refused` khi bấm Test connection**: Nguyên nhân do Keycloak không thể kết nối tới port 389 của LDAP. Kiểm tra lại `Connection URL`, đảm bảo tên host `openldap` có thể được phân giải trong Docker network, hoặc thử dùng địa chỉ IP nội bộ của container LDAP.
- **Lỗi `Invalid credentials` khi bấm Test authentication**: Bạn đã nhập sai `Bind DN` hoặc `Bind credentials`. Cần đảm bảo DN của tài khoản quản trị LDAP phải chính xác tuyệt đối, không có khoảng trắng dư thừa.
- **Không tìm thấy User khi Sync**: Kiểm tra lại `Users DN` và `User object classes`. Nếu DIT trong LDAP không dùng `inetOrgPerson` mà dùng class khác (như `posixAccount`), bạn cần thay đổi `User object classes` tương ứng.

> [!TIP]
> Trong môi trường Production (Enterprise), thay vì chọn `simple` Bind và port `389` dạng clear text, hãy luôn sử dụng giao thức LDAPS (LDAP over SSL/TLS) ở port `636` kết hợp với Truststore chứa Certificate của máy chủ LDAP để đảm bảo mật khẩu không bị sniff trên đường truyền.

## 5. Dọn dẹp (Cleanup)
Sau khi hoàn tất bài Lab, xóa bỏ các container để giải phóng tài nguyên hệ thống:
```bash
docker-compose down -v
```
