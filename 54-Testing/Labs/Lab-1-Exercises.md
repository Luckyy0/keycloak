> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Áp dụng thực tế các kỹ thuật Unit Test (Mockito) và Integration Test (Testcontainers) để kiểm thử một phần mở rộng của Keycloak (Custom Event Listener SPI) từ môi trường cách ly cho đến môi trường giả lập thực tế.

## 1. Kịch bản Thực hành (Lab Scenario)

Giả sử trong hệ thống công ty của bạn, có một yêu cầu bảo mật: **Bất cứ khi nào người dùng thực hiện thay đổi mật khẩu thành công (`UPDATE_PASSWORD`), hệ thống Keycloak phải ghi nhận lại một dòng cảnh báo đặc biệt (Security Audit Log)**.

Bạn được giao nhiệm vụ:
1. Viết mã nguồn cho `CustomAuditEventListenerProvider`.
2. Viết **Unit Test** với Mockito để đảm bảo logic bắt đúng loại sự kiện mà không cần gọi Keycloak Server.
3. Viết **Integration Test** với Testcontainers để đảm bảo provider hoạt động chính xác khi cắm (`deploy`) vào Keycloak thực tế và nhận một request HTTP giả lập.

---

## 2. Chuẩn bị Môi trường (Prerequisites)

- **Môi trường Java:** JDK 17 hoặc 21.
- **Công cụ Build:** Apache Maven 3.8+ hoặc Gradle.
- **Môi trường Ảo hóa:** **Docker Desktop** (hoặc Docker Engine / Podman) đang chạy.
- **IDE:** IntelliJ IDEA (khuyến nghị) hoặc Eclipse.
- **Phiên bản Keycloak:** 24.0.1 (sử dụng Quarkus distribution).

---

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Khởi tạo Project và thêm Dependencies (Maven `pom.xml`)

Tạo một project Maven trống và cấu hình các thư viện thiết yếu:

```xml
<dependencies>
    <!-- Keycloak Core (provided vì nó đã có sẵn trên server) -->
    <dependency>
        <groupId>org.keycloak</groupId>
        <artifactId>keycloak-core</artifactId>
        <version>24.0.1</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.keycloak</groupId>
        <artifactId>keycloak-server-spi</artifactId>
        <version>24.0.1</version>
        <scope>provided</scope>
    </dependency>

    <!-- Testing Dependencies -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.1</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>5.7.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.github.dasniko</groupId>
        <artifactId>testcontainers-keycloak</artifactId>
        <version>3.1.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>5.4.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Bước 2: Viết mã nguồn Provider

Tạo tệp lớp `CustomAuditEventListenerProvider.java` implements interface `EventListenerProvider`.

```java
import org.keycloak.events.Event;
import org.keycloak.events.EventListenerProvider;
import org.keycloak.events.EventType;
import org.keycloak.events.admin.AdminEvent;

public class CustomAuditEventListenerProvider implements EventListenerProvider {

    @Override
    public void onEvent(Event event) {
        // Chỉ xử lý sự kiện đổi mật khẩu
        if (event.getType() == EventType.UPDATE_PASSWORD) {
            System.out.println("SECURITY ALERT: User " + event.getUserId() + " changed password!");
            // Giả lập lưu log đặc biệt
        }
    }

    @Override
    public void onEvent(AdminEvent adminEvent, boolean includeRepresentation) {
        // Bỏ qua admin events
    }

    @Override
    public void close() {
        // Hủy bỏ tài nguyên nếu có
    }
}
```

*Lưu ý: Bạn cũng cần tạo Factory class `CustomAuditEventListenerProviderFactory` và khai báo tệp `META-INF/services/org.keycloak.events.EventListenerProviderFactory` (như cấu trúc SPI chuẩn).*

### Bước 3: Viết Unit Test với Mockito

Tạo tệp `CustomAuditEventListenerProviderTest.java` trong thư mục `src/test/java`.

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.keycloak.events.Event;
import org.keycloak.events.EventType;

import static org.junit.jupiter.api.Assertions.assertDoesNotThrow;

public class CustomAuditEventListenerProviderTest {

    private CustomAuditEventListenerProvider provider;

    @BeforeEach
    public void setUp() {
        provider = new CustomAuditEventListenerProvider();
    }

    @Test
    public void testOnEvent_withUpdatePassword_shouldNotThrowError() {
        // Arrange
        Event event = new Event();
        event.setType(EventType.UPDATE_PASSWORD);
        event.setUserId("user-xyz-123");

        // Act & Assert (Logic đơn giản chỉ in ra dòng log)
        // Dùng assertDoesNotThrow để đảm bảo không bị crash hệ thống
        assertDoesNotThrow(() -> provider.onEvent(event));
    }
}
```

### Bước 4: Viết Integration Test với Testcontainers

Tạo tệp `IntegrationCustomAuditTest.java`. Mục tiêu là khởi chạy nguyên một Keycloak Server, tiêm (inject) file Provider của bạn vào, và dùng `RestAssured` để thực hiện lấy token hoặc mô phỏng lỗi cấu hình.

```java
import dasniko.testcontainers.keycloak.KeycloakContainer;
import org.junit.jupiter.api.Test;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import static io.restassured.RestAssured.given;

@Testcontainers
public class IntegrationCustomAuditTest {

    // Mount thư mục chứa class đã build của project
    @Container
    private static final KeycloakContainer keycloak = new KeycloakContainer("quay.io/keycloak/keycloak:24.0.1")
            .withProviderClassesFrom("target/classes");

    @Test
    public void testKeycloakStartsSuccessfullyWithCustomSPI() {
        // Arrange
        String authUrl = keycloak.getAuthServerUrl();

        // Act & Assert
        // Gọi tới master realm để kiểm tra server có sống và phản hồi không
        given()
            .when()
            .get(authUrl + "/realms/master/.well-known/openid-configuration")
            .then()
            .statusCode(200);
            
        System.out.println("Integration Test Pass. Keycloak URL: " + authUrl);
    }
}
```

---

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### Chạy các bài Test
Chạy lệnh Maven sau trong Terminal/Command Prompt để thực thi toàn bộ luồng kiểm thử:
```bash
mvn clean test
```

**Kỳ vọng (Expected Result):**
- Mockito Test sẽ chạy chớp nhoáng (vài mili-giây).
- Testcontainers sẽ khởi động Docker, bạn sẽ thấy thông báo Pulling Image và tiến trình log của Keycloak. Khoảng 15-30 giây sau, test hoàn tất thành công.

### Khắc phục sự cố thường gặp (Troubleshooting)

1. **Lỗi `DockerClientProviderStrategy - Could not find a valid Docker environment`**
   - **Nguyên nhân:** Testcontainers không thể kết nối tới Docker daemon trên máy tính của bạn.
   - **Cách khắc phục:** Đảm bảo Docker Desktop đã được mở (running). Nếu chạy trên WSL2 hoặc Linux, đảm bảo quyền `sudo usermod -aG docker $USER`.

2. **Lỗi `OOM (Out of Memory)` khiến Container tự chết**
   - **Nguyên nhân:** Máy chủ cấp không đủ bộ nhớ cho JVM của Keycloak trong container (Mặc định nó cần tối thiểu 512MB RAM trống).
   - **Cách khắc phục:** Chỉnh cấu hình `JAVA_OPTS` khi khởi tạo container hoặc tắt các dịch vụ tốn RAM khác trên máy chủ lúc chạy test.

3. **Log không hiển thị SPI đã được Load**
   - **Nguyên nhân:** Testcontainers không tìm thấy classes đã dịch, do bạn chưa chạy `mvn compile` trước khi chạy test, dẫn đến thư mục `target/classes` trống rỗng.
   - **Cách khắc phục:** Hãy luôn chạy `mvn clean test` hoặc `mvn clean package`. Quá trình build sẽ sinh class trước khi test khởi chạy.
