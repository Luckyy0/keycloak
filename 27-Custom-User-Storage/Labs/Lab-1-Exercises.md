> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Triển khai, đóng gói và cấu hình một Custom User Storage Provider trong Keycloak để kết nối và xác thực người dùng từ cơ sở dữ liệu MySQL bên ngoài.

## 1. Kịch bản Thực hành (Lab Scenario)

Công ty bạn đang sử dụng một hệ thống ERP cũ lưu trữ thông tin người dùng trong cơ sở dữ liệu MySQL (bảng `erp_users`). Hệ thống mới sử dụng Keycloak làm IAM (Identity and Access Management). Bạn được yêu cầu cho phép nhân viên sử dụng tài khoản ERP cũ để đăng nhập vào hệ thống mới thông qua Keycloak mà không được phép copy dữ liệu từ MySQL sang cơ sở dữ liệu nội bộ của Keycloak (chính sách Zero Data Duplication).

Bạn sẽ cần lập trình một **User Storage SPI** bằng Java, đóng gói thành file `.jar`, triển khai lên Keycloak và cấu hình kết nối tới MySQL.

## 2. Chuẩn bị Môi trường (Prerequisites)

- **JDK 17** trở lên và **Maven** 3.8+.
- **Docker** và **Docker Compose** (để chạy MySQL và Keycloak).
- Một IDE cho Java (IntelliJ IDEA, Eclipse, hoặc VS Code).
- Mã nguồn mẫu (hoặc tự tạo project Maven).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1. Khởi tạo Database MySQL (External DB)

Sử dụng Docker để chạy một instance MySQL giả lập hệ thống ERP.

Chạy lệnh sau trong Terminal:
```bash
docker run --name erp-mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=erp_db -e MYSQL_USER=erp_user -e MYSQL_PASSWORD=erp_pass -p 3306:3306 -d mysql:8.0
```

Truy cập vào MySQL và tạo bảng người dùng:
```bash
docker exec -it erp-mysql mysql -u erp_user -perp_pass erp_db
```

Chạy script SQL để tạo bảng và dữ liệu mẫu:
```sql
CREATE TABLE erp_users (
    username VARCHAR(50) PRIMARY KEY,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100)
);

-- Mật khẩu ở đây lưu dạng plain-text cho mục đích demo (Trong thực tế phải là hash)
INSERT INTO erp_users (username, password, email) VALUES ('employee1', 'secret123', 'employee1@erp.local');
INSERT INTO erp_users (username, password, email) VALUES ('employee2', 'secret456', 'employee2@erp.local');
```

### Bước 3.2. Viết Custom Provider bằng Java

Tạo một Maven Project với `pom.xml` chứa dependencies của Keycloak (lưu ý version phải khớp với Keycloak đang chạy, ví dụ `22.0.0`).

```xml
<dependencies>
    <dependency>
        <groupId>org.keycloak</groupId>
        <artifactId>keycloak-core</artifactId>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.keycloak</groupId>
        <artifactId>keycloak-server-spi</artifactId>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.keycloak</groupId>
        <artifactId>keycloak-model-legacy</artifactId>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

Tạo class `ErpUserStorageProvider` implement `UserStorageProvider`, `UserLookupProvider`, `CredentialInputValidator`. Viết logic kết nối JDBC đến MySQL bằng `java.sql.Connection` và thực thi truy vấn `SELECT * FROM erp_users WHERE username = ?`. 

Tạo class `ErpUserStorageProviderFactory` implement `UserStorageProviderFactory`.
Đăng ký Factory này trong thư mục:
`src/main/resources/META-INF/services/org.keycloak.storage.UserStorageProviderFactory`
Nội dung file: `com.example.keycloak.ErpUserStorageProviderFactory`

### Bước 3.3. Build và Deploy lên Keycloak

Build project thành file JAR:
```bash
mvn clean package
```
Kết quả ta được file `target/erp-user-storage-provider.jar`.

Khởi chạy Keycloak bằng Docker, đồng thời mount file JAR vào thư mục `/opt/keycloak/providers/`:
```bash
docker run --name keycloak -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin \
  -v $(pwd)/target/erp-user-storage-provider.jar:/opt/keycloak/providers/erp-user-storage-provider.jar \
  quay.io/keycloak/keycloak:22.0.0 start-dev
```

### Bước 3.4. Cấu hình trên Keycloak Admin Console

1. Mở trình duyệt, truy cập `http://localhost:8080` và đăng nhập bằng `admin/admin`.
2. Tạo (hoặc chọn) Realm `ErpRealm`.
3. Nhấp vào menu **User Federation** ở thanh bên trái.
4. Chọn **Add provider** và tìm tên Factory ID của bạn (ví dụ: `erp-user-storage`).
5. Trong cấu hình Provider:
   - Điền JDBC URL: `jdbc:mysql://host.docker.internal:3306/erp_db`
   - DB User: `erp_user`
   - DB Password: `erp_pass`
6. Nhấp **Save**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu
- Mở một trình duyệt ẩn danh, truy cập vào giao diện Account Console của Realm `ErpRealm` (ví dụ: `http://localhost:8080/realms/ErpRealm/account`).
- Đăng nhập bằng `employee1` và password `secret123`.
- Đăng nhập phải thành công. Bạn có thể kiểm tra tab **Users** trong Admin Console, tìm kiếm `employee1` sẽ thấy người dùng hiển thị (với biểu tượng cho biết nó đến từ User Federation).

### 4.2. Troubleshooting (Khắc phục sự cố)
- **Lỗi ClassNotFoundException:** File JAR thiếu dependency (ví dụ: thiếu MySQL JDBC Driver). Hãy chắc chắn bạn đã đóng gói (shade) MySQL driver vào file JAR hoặc cài đặt nó như một module trong Keycloak.
- **Lỗi Connection Refused:** Keycloak trong Docker không thể kết nối tới MySQL trên host. Đảm bảo dùng IP đúng (ví dụ `host.docker.internal` hoặc IP của máy host) thay vì `localhost`.
- **User không thể login:** Kiểm tra log của Keycloak (`docker logs -f keycloak`). Nếu xuất hiện lỗi SQL Exception, hãy kiểm tra lại SQL query trong class Provider và cấu trúc bảng.
