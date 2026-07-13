# Lab 1: Xây dựng và Triển khai Custom EventListener SPI

> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Viết mã nguồn Java để tạo một Custom EventListener SPI. Biên dịch thành JAR thông qua Maven và triển khai lên máy chủ Keycloak bằng kiến trúc Docker.

## 1. Kịch bản Thực hành (Lab Scenario)
Công ty của bạn vừa quyết định tích hợp hệ thống Identity (Keycloak) với nền tảng Gửi Email Marketing (Ví dụ: Mailchimp, SendGrid). Yêu cầu đặt ra là: Bất cứ khi nào một người dùng mới **Đăng ký (Register)** thành công trên Keycloak, hệ thống phải in ra màn hình Console một dòng log có chứa nội dung đặc biệt: `[MARKETING-SYNC] User <Email> has registered. Syncing to CRM...`. Trong thực tế, bạn sẽ dùng HTTP Client để gọi REST API của Mailchimp, nhưng ở bài Lab này, chúng ta tập trung vào việc tạo SPI và in log ra console.

## 2. Chuẩn bị Môi trường (Prerequisites)
- **Java Development Kit (JDK):** Version 17 trở lên.
- **Apache Maven:** Cài đặt Maven để quản lý dependencies và biên dịch.
- **Docker:** Để khởi chạy môi trường giả lập Keycloak (phiên bản 22+).
- Một trình soạn thảo mã nguồn (IntelliJ IDEA, Eclipse, hoặc VS Code).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 1: Khởi tạo Project Maven**
Tạo một thư mục mới có tên `keycloak-custom-listener` và tạo file `pom.xml`:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.company.keycloak</groupId>
    <artifactId>marketing-sync-listener</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.keycloak</groupId>
            <artifactId>keycloak-server-spi</artifactId>
            <version>22.0.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.keycloak</groupId>
            <artifactId>keycloak-server-spi-private</artifactId>
            <version>22.0.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**Bước 2: Viết mã nguồn Java cho Provider**
Tạo cấu trúc thư mục Java `src/main/java/com/company/keycloak/` và file `MarketingEventListenerProvider.java`:
```java
package com.company.keycloak;

import org.keycloak.events.Event;
import org.keycloak.events.EventListenerProvider;
import org.keycloak.events.EventType;
import org.keycloak.events.admin.AdminEvent;

public class MarketingEventListenerProvider implements EventListenerProvider {

    @Override
    public void onEvent(Event event) {
        if (event.getType() == EventType.REGISTER) {
            String email = event.getDetails().get("email");
            System.out.println("[MARKETING-SYNC] User " + email + " has registered. Syncing to CRM...");
        }
    }

    @Override
    public void onEvent(AdminEvent adminEvent, boolean includeRepresentation) {}

    @Override
    public void close() {}
}
```

**Bước 3: Viết mã nguồn cho Provider Factory**
Tạo tiếp file `MarketingEventListenerProviderFactory.java`:
```java
package com.company.keycloak;

import org.keycloak.Config;
import org.keycloak.events.EventListenerProvider;
import org.keycloak.events.EventListenerProviderFactory;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.KeycloakSessionFactory;

public class MarketingEventListenerProviderFactory implements EventListenerProviderFactory {

    @Override
    public EventListenerProvider create(KeycloakSession session) {
        return new MarketingEventListenerProvider();
    }

    @Override
    public void init(Config.Scope config) {}

    @Override
    public void postInit(KeycloakSessionFactory factory) {}

    @Override
    public void close() {}

    @Override
    public String getId() {
        return "marketing-sync-listener";
    }
}
```

**Bước 4: Đăng ký Service**
Tạo thư mục `src/main/resources/META-INF/services/` và tạo file có tên chính xác là `org.keycloak.events.EventListenerProviderFactory`.
Bên trong file đó, ghi đường dẫn tuyệt đối tới lớp Factory của bạn:
```text
com.company.keycloak.MarketingEventListenerProviderFactory
```

**Bước 5: Biên dịch và Build Image Docker**
Mở terminal, chạy lệnh Maven để tạo JAR:
```bash
mvn clean package
```
Sẽ có một file xuất hiện tại `target/marketing-sync-listener-1.0-SNAPSHOT.jar`.
Tiếp theo, tạo file `Dockerfile` ở thư mục gốc:
```dockerfile
FROM quay.io/keycloak/keycloak:22.0.0 AS builder
COPY target/marketing-sync-listener-1.0-SNAPSHOT.jar /opt/keycloak/providers/
RUN /opt/keycloak/bin/kc.sh build

FROM quay.io/keycloak/keycloak:22.0.0
COPY --from=builder /opt/keycloak/ /opt/keycloak/
ENV KEYCLOAK_ADMIN=admin
ENV KEYCLOAK_ADMIN_PASSWORD=admin
ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
CMD ["start-dev"]
```
Build và chạy Docker:
```bash
docker build -t my-custom-keycloak .
docker run --name custom-kc -p 8080:8080 my-custom-keycloak
```

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu:**
1. Truy cập Admin Console `http://localhost:8080`, đăng nhập (admin/admin).
2. Vào **Realm Settings** -> **Events** -> Chuyển qua tab **Event Listeners**.
3. Bạn sẽ thấy `marketing-sync-listener` xuất hiện trong hộp thoại. Thêm nó vào danh sách và lưu lại.
4. Mở cửa sổ ẩn danh, tiến hành đăng ký tài khoản mới (Register) cho người dùng.
5. Quan sát Docker console log (hoặc dùng `docker logs custom-kc`). Bạn phải thấy dòng chữ `[MARKETING-SYNC] User <email> has registered...`.

**Troubleshooting (Khắc phục sự cố):**
- **Không thấy ID của EventListener trên màn hình UI:** Kiểm tra lại file `META-INF/services/...`. Tên file phải khớp với tên Interface của Keycloak, nội dung file phải chứa đường dẫn Package chính xác.
- **Lỗi ClassNotFound khi Build Docker:** Đảm bảo cấu trúc Maven chuẩn, file JAR được sinh ra phải chứa thư mục `com/company/...` thay vì file JAR rỗng.
- **Lỗi Provider Registration Failed:** Do dùng sai phiên bản dependency `keycloak-server-spi`. Phiên bản Keycloak Container chạy bản 22, thì dependency trong Maven cũng phải là 22.
