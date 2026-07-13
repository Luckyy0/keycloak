> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Thiết lập Service Account trên Keycloak và sử dụng thư viện Keycloak Java Admin Client trong Spring Boot để thực hiện các thao tác CRUD (Tạo, Đọc, Cập nhật, Xóa) trên tài khoản người dùng một cách tự động.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn đang phát triển một hệ thống Backend bằng Spring Boot cho chức năng "Đăng ký thành viên". Khi người dùng điền form đăng ký trên Frontend, Backend sẽ tiếp nhận thông tin và cần tạo một tài khoản tương ứng trên hệ thống Keycloak. 
Trong bài Lab này, bạn sẽ không dùng tài khoản quản trị viên thông thường, mà sẽ tạo một **Service Account** để đảm bảo ứng dụng Backend giao tiếp bảo mật (Machine-to-Machine) với Keycloak.

## 2. Chuẩn bị Môi trường (Prerequisites)

- **Keycloak Server** đang chạy (tại `http://localhost:8080`).
- **Tài khoản Admin** của Keycloak để cấu hình (admin/admin).
- Một Project **Spring Boot** (version 3.x) đã được khởi tạo bằng Maven/Gradle.
- Java 17+.
- Kiến thức cơ bản về Dependency Injection trong Spring.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1. Cấu hình Keycloak Client và Service Account

1. Đăng nhập vào giao diện **Keycloak Admin Console** (`http://localhost:8080`).
2. Chọn (hoặc tạo) một Realm, ví dụ `DemoRealm`.
3. Đi đến menu **Clients** và chọn **Create client**.
   - **Client ID:** `spring-backend-client`
   - Bấm **Next**.
   - Bật cờ **Client authentication** sang **ON**.
   - Bật cờ **Service accounts roles** sang **ON**. (Bỏ check phần Standard flow nếu Backend không có giao diện đăng nhập cho user thường).
   - Bấm **Save**.
4. Chọn tab **Credentials** của Client vừa tạo, copy đoạn **Client secret** (Ví dụ: `wXyZ...`).
5. Chuyển sang tab **Service account roles**:
   - Nhấp vào **Assign role**.
   - Trong dropdown `Filter by realm roles`, đổi thành `Filter by clients`.
   - Tìm kiếm và chọn client `realm-management`.
   - Chọn role `manage-users` và nhấp **Assign**.

### Bước 3.2. Cấu hình Spring Boot Project

Thêm thư viện `keycloak-admin-client` vào file `pom.xml` của Spring Boot. Chú ý version của client phải khớp với version của Keycloak server (ví dụ 22.0.0).

```xml
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-admin-client</artifactId>
    <version>22.0.0</version>
</dependency>
```

Thêm cấu hình vào file `application.properties` (hoặc `application.yml`):
```properties
keycloak.server-url=http://localhost:8080
keycloak.realm=DemoRealm
keycloak.client-id=spring-backend-client
keycloak.client-secret=YOUR_COPIED_CLIENT_SECRET
```

### Bước 3.3. Viết mã nguồn tích hợp (Java Code)

Tạo class cấu hình để khởi tạo đối tượng `Keycloak` dưới dạng một Bean Singleton:

```java
import org.keycloak.admin.client.Keycloak;
import org.keycloak.admin.client.KeycloakBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class KeycloakConfig {

    @Value("${keycloak.server-url}")
    private String serverUrl;

    @Value("${keycloak.realm}")
    private String realm;

    @Value("${keycloak.client-id}")
    private String clientId;

    @Value("${keycloak.client-secret}")
    private String clientSecret;

    @Bean
    public Keycloak keycloak() {
        return KeycloakBuilder.builder()
                .serverUrl(serverUrl)
                .realm(realm)
                .grantType("client_credentials")
                .clientId(clientId)
                .clientSecret(clientSecret)
                .build();
    }
}
```

Tạo một Service class thực hiện tạo User:

```java
import org.keycloak.admin.client.Keycloak;
import org.keycloak.representations.idm.CredentialRepresentation;
import org.keycloak.representations.idm.UserRepresentation;
import jakarta.ws.rs.core.Response;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.util.Collections;

@Service
public class KeycloakUserService {

    @Autowired
    private Keycloak keycloak;

    @Value("${keycloak.realm}")
    private String realm;

    public void createUser(String username, String email, String password) {
        UserRepresentation user = new UserRepresentation();
        user.setUsername(username);
        user.setEmail(email);
        user.setEnabled(true);

        CredentialRepresentation credential = new CredentialRepresentation();
        credential.setType(CredentialRepresentation.PASSWORD);
        credential.setValue(password);
        credential.setTemporary(false);

        user.setCredentials(Collections.singletonList(credential));

        Response response = keycloak.realm(realm).users().create(user);
        
        if (response.getStatus() == 201) {
            System.out.println("Tạo user thành công!");
        } else {
            System.err.println("Lỗi tạo user: " + response.getStatusInfo());
        }
        
        // CỰC KỲ QUAN TRỌNG: Luôn phải đóng Response để tránh rò rỉ kết nối
        response.close(); 
    }
}
```

Tạo một Controller đơn giản để gọi thử nghiệm:
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private KeycloakUserService keycloakUserService;

    @PostMapping("/register")
    public String registerUser(@RequestParam String username, 
                               @RequestParam String email, 
                               @RequestParam String password) {
        keycloakUserService.createUser(username, email, password);
        return "Yêu cầu đã được gửi. Kiểm tra console log.";
    }
}
```

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu
1. Khởi động ứng dụng Spring Boot.
2. Mở Terminal (hoặc dùng Postman), gửi request tạo tài khoản:
   ```bash
   curl -X POST "http://localhost:8081/api/users/register?username=labuser&email=lab@example.com&password=labpass"
   ```
3. Kiểm tra log trên console của Spring Boot, bạn phải thấy dòng "Tạo user thành công!".
4. Đăng nhập vào giao diện Keycloak Admin Console, vào Realm `DemoRealm` -> **Users**. Tìm `labuser` và xác nhận user này đã xuất hiện. Thử đăng nhập Account Console bằng `labuser/labpass` để đảm bảo mật khẩu hoạt động.

### 4.2. Troubleshooting (Khắc phục sự cố)
- **Lỗi HTTP 401 Unauthorized:** Spring Boot in ra lỗi này khi gọi Keycloak. Lỗi do cấu hình `client-id` hoặc `client-secret` sai trong file `application.properties`. Hãy copy lại chính xác.
- **Lỗi HTTP 403 Forbidden:** Spring Boot kết nối được, nhưng không thể tạo user. Lỗi do bạn chưa cấp quyền (role) `manage-users` thuộc client `realm-management` cho Service Account ở Bước 3.1.5.
- **Ứng dụng Spring Boot chạy một lúc rồi treo (Timeout/Connection Pool Exhausted):** Do quên gọi lệnh `response.close()` trong hàm `createUser`. Đây là lỗi rất nguy hiểm trên production. Luôn đảm bảo đóng response.
