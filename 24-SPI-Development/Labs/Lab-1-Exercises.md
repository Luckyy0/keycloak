> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Xây dựng, triển khai và cấu hình một Provider SPI tùy chỉnh (Custom Event Listener SPI) để ghi log các sự kiện đăng nhập và quản trị trên máy chủ Keycloak.

## 1. Kịch bản Thực hành (Lab Scenario)

Trong các hệ thống doanh nghiệp, việc theo dõi hành vi đăng nhập của người dùng là yêu cầu bắt buộc để kiểm toán bảo mật (Audit). Mặc định, Keycloak lưu sự kiện vào Database, nhưng để đẩy log này ra các hệ thống phân tích như ELK Stack (Elasticsearch, Logstash, Kibana) hoặc console chuẩn để cấu hình Fluentd thu thập, chúng ta cần tùy biến một bộ lắng nghe sự kiện (Event Listener).

Trong bài Lab này, bạn sẽ đóng vai trò là một System Developer. Nhiệm vụ của bạn là lập trình một **Custom Event Listener SPI** bằng Java, biên dịch nó thành tệp JAR, triển khai vào Keycloak bằng lệnh Build, và cấu hình trên giao diện Admin Console để hệ thống bắt đầu in thông tin người dùng đăng nhập ra System Console (Log của máy chủ).

## 2. Chuẩn bị Môi trường (Prerequisites)

Để thực hiện bài Lab, bạn cần đảm bảo các công cụ sau đã được cài đặt và cấu hình:

- **Java Development Kit (JDK)**: Phiên bản 17 hoặc 21 (tương thích với Keycloak version bạn đang dùng).
- **Apache Maven**: Công cụ quản lý dự án để biên dịch (version 3.8+).
- **Keycloak Server**: Một instance Keycloak bản Quarkus (tải từ trang chủ `keycloak.org`, giải nén ra thư mục máy tính cục bộ).
- **IDE**: IntelliJ IDEA, Eclipse hoặc VS Code.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Khởi tạo dự án Maven

1. Tạo thư mục dự án mới tên là `custom-event-listener`.
2. Trong thư mục này, tạo file `pom.xml` với nội dung khai báo các dependency của Keycloak.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.keycloak</groupId>
    <artifactId>custom-event-listener</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <keycloak.version>22.0.0</keycloak.version> <!-- Đổi sang version bạn đang dùng -->
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.keycloak</groupId>
            <artifactId>keycloak-server-spi</artifactId>
            <version>${keycloak.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.keycloak</groupId>
            <artifactId>keycloak-server-spi-private</artifactId>
            <version>${keycloak.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.keycloak</groupId>
            <artifactId>keycloak-services</artifactId>
            <version>${keycloak.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```

*(Lưu ý: `scope` được đặt là `provided` vì Keycloak sẽ cung cấp các thư viện này lúc chạy, ta không đóng gói chúng vào JAR để tránh phình dung lượng).*

### Bước 3.2: Lập trình Provider

1. Tạo cấu trúc thư mục Java: `src/main/java/com/example/keycloak/`.
2. Tạo file `CustomEventListenerProvider.java`:

```java
package com.example.keycloak;

import org.jboss.logging.Logger;
import org.keycloak.events.Event;
import org.keycloak.events.EventListenerProvider;
import org.keycloak.events.admin.AdminEvent;
import org.keycloak.models.KeycloakSession;

public class CustomEventListenerProvider implements EventListenerProvider {

    private static final Logger log = Logger.getLogger(CustomEventListenerProvider.class);
    private final KeycloakSession session;

    public CustomEventListenerProvider(KeycloakSession session) {
        this.session = session;
    }

    @Override
    public void onEvent(Event event) {
        log.infof("USER EVENT: Type=%s, Client=%s, User=%s, IP=%s",
                event.getType(), event.getClientId(), event.getUserId(), event.getIpAddress());
    }

    @Override
    public void onEvent(AdminEvent adminEvent, boolean includeRepresentation) {
        log.infof("ADMIN EVENT: Operation=%s, ResourceType=%s, Path=%s",
                adminEvent.getOperationType(), adminEvent.getResourceType(), adminEvent.getResourcePath());
    }

    @Override
    public void close() {
        // Không có kết nối ngoại vi nào cần đóng
    }
}
```

### Bước 3.3: Lập trình Provider Factory

Trong cùng thư mục, tạo file `CustomEventListenerProviderFactory.java`:

```java
package com.example.keycloak;

import org.keycloak.Config;
import org.keycloak.events.EventListenerProvider;
import org.keycloak.events.EventListenerProviderFactory;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.KeycloakSessionFactory;

public class CustomEventListenerProviderFactory implements EventListenerProviderFactory {

    private static final String PROVIDER_ID = "custom-stdout-logger";

    @Override
    public EventListenerProvider create(KeycloakSession session) {
        return new CustomEventListenerProvider(session);
    }

    @Override
    public void init(Config.Scope config) {
        // Khởi tạo một lần
    }

    @Override
    public void postInit(KeycloakSessionFactory factory) { }

    @Override
    public void close() { }

    @Override
    public String getId() {
        return PROVIDER_ID;
    }
}
```

### Bước 3.4: Đăng ký SPI với ServiceLoader

1. Tạo thư mục: `src/main/resources/META-INF/services/`.
2. Tạo một file (không có đuôi mở rộng) có tên chính xác là:
   `org.keycloak.events.EventListenerProviderFactory`
3. Mở file đó và điền nội dung duy nhất là đường dẫn đến class Factory của bạn:
   ```text
   com.example.keycloak.CustomEventListenerProviderFactory
   ```

### Bước 3.5: Biên dịch và Triển khai (Deploy)

1. Mở terminal, trỏ vào thư mục gốc của project (nơi có `pom.xml`) và chạy lệnh:
   ```bash
   mvn clean package
   ```
2. Nếu thành công, bạn sẽ thấy file `custom-event-listener-1.0-SNAPSHOT.jar` trong thư mục `target/`.
3. Copy tệp JAR này vào thư mục `providers/` của Keycloak.
   ```bash
   cp target/custom-event-listener-1.0-SNAPSHOT.jar /path/to/keycloak/providers/
   ```
4. Build lại tối ưu hóa Keycloak:
   ```bash
   cd /path/to/keycloak
   bin/kc.sh build
   ```
5. Khởi động Keycloak:
   ```bash
   bin/kc.sh start-dev
   ```

### Bước 3.6: Kích hoạt trên Admin Console

1. Mở trình duyệt, đăng nhập vào Keycloak Admin Console (ví dụ: `http://localhost:8080/admin`).
2. Ở thanh menu trái, chọn **Realm settings**.
3. Chuyển sang tab **Events**.
4. Tìm đến mục **Event listeners** (Lắng nghe sự kiện). Click vào hộp nhập liệu và bạn sẽ thấy ID SPI của chúng ta: `custom-stdout-logger`.
5. Chọn `custom-stdout-logger` để thêm vào danh sách lắng nghe.
6. Nhấn **Save** (Lưu). Bật công tắc **Save events** nếu nó đang tắt.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu (Verification)

Để kiểm tra SPI đã hoạt động đúng hay chưa:
1. Mở một trình duyệt ẩn danh (Incognito).
2. Truy cập vào trang Account Console của Keycloak (ví dụ: `http://localhost:8080/realms/master/account`).
3. Đăng nhập bằng một tài khoản bất kỳ (hoặc nhập sai mật khẩu).
4. Quan sát cửa sổ Terminal đang chạy máy chủ Keycloak. Bạn PHẢI thấy dòng log in ra từ đoạn mã của bạn với tiền tố `USER EVENT`:
   ```log
   2023-10-10 10:00:00,000 INFO  [com.example.keycloak.CustomEventListenerProvider] (executor-thread-1) USER EVENT: Type=LOGIN_ERROR, Client=account-console, User=admin, IP=127.0.0.1
   ```
   Hoặc nếu tạo một user mới trong Admin console, bạn sẽ thấy `ADMIN EVENT`.

### 4.2. Khắc phục sự cố (Troubleshooting)

- **Lỗi không hiển thị Provider ID trên Admin Console:**
  - *Nguyên nhân:* Quên tạo file trong thư mục `META-INF/services/` hoặc gõ sai tên tệp/tên class bên trong.
  - *Khắc phục:* Kiểm tra lại kỹ tên thư mục, đảm bảo file là `org.keycloak.events.EventListenerProviderFactory` (không có khoảng trắng dư thừa).
- **Lỗi `ClassNotFoundException` khi Keycloak khởi động:**
  - *Nguyên nhân:* Bạn đã đóng gói dư các thư viện `keycloak-server-spi` vào trong JAR (không set `<scope>provided</scope>`).
  - *Khắc phục:* Cập nhật `pom.xml`, chạy `mvn clean package` và deploy lại. Đừng quên chạy lại `bin/kc.sh build`.
- **Thấy log từ Keycloak gốc nhưng không thấy log custom:**
  - *Nguyên nhân:* SPI đã được nạp nhưng chưa được cấu hình.
  - *Khắc phục:* Phải thực hiện Bước 3.6. SPI chỉ chạy khi realm được gán `Event listener` tương ứng. Thử kiểm tra ở realm `master` trước.
