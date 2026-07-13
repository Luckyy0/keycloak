> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Xây dựng, biên dịch và triển khai (deploy) một Custom REST Endpoint SPI vào hệ thống Keycloak. Học cách viết Java Provider và nạp vào máy chủ Quarkus.

## 1. Kịch bản Thực hành (Lab Scenario)
Doanh nghiệp yêu cầu một Endpoint công khai trả về trạng thái "Ping" nội bộ và thông tin của Realm hiện hành để hệ thống Monitoring sức khỏe (Health Check) bên ngoài có thể gọi liên tục mà không cần xác thực. Bạn cần tạo một `RealmResourceProvider` thực hiện nhiệm vụ này và nhúng vào Keycloak.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Đã cài đặt JDK 17 (hoặc mới nhất).
- Đã cài đặt Maven 3.x.
- Đã cài đặt Keycloak dựa trên Quarkus.
- Bất kỳ Java IDE nào (IntelliJ IDEA, Eclipse, VSCode).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Khởi tạo Project Maven
1. Mở Terminal và tạo thư mục project `kc-custom-rest`.
2. Tạo file `pom.xml` với nội dung khai báo các phụ thuộc của Keycloak (Lưu ý thay version Keycloak trùng với server đang dùng):
   ```xml
   <project xmlns="http://maven.apache.org/POM/4.0.0" ...>
       <modelVersion>4.0.0</modelVersion>
       <groupId>com.company.keycloak</groupId>
       <artifactId>custom-rest-endpoint</artifactId>
       <version>1.0.0</version>
       <dependencies>
           <dependency>
               <groupId>org.keycloak</groupId>
               <artifactId>keycloak-core</artifactId>
               <version>22.0.0</version> <!-- Chỉnh theo thực tế -->
               <scope>provided</scope>
           </dependency>
           <dependency>
               <groupId>org.keycloak</groupId>
               <artifactId>keycloak-server-spi</artifactId>
               <version>22.0.0</version>
               <scope>provided</scope>
           </dependency>
           <dependency>
               <groupId>org.keycloak</groupId>
               <artifactId>keycloak-services</artifactId>
               <version>22.0.0</version>
               <scope>provided</scope>
           </dependency>
       </dependencies>
   </project>
   ```

### Bước 3.2: Viết mã nguồn Provider
1. Tạo class `MyRestResource.java`:
   ```java
   package com.company.keycloak.rest;

   import org.keycloak.models.KeycloakSession;
   import jakarta.ws.rs.GET;
   import jakarta.ws.rs.Path;
   import jakarta.ws.rs.Produces;
   import jakarta.ws.rs.core.MediaType;
   import jakarta.ws.rs.core.Response;

   public class MyRestResource {
       private final KeycloakSession session;

       public MyRestResource(KeycloakSession session) {
           this.session = session;
       }

       @GET
       @Path("/ping")
       @Produces(MediaType.APPLICATION_JSON)
       public Response ping() {
           String realmName = session.getContext().getRealm().getName();
           String json = "{\"status\":\"ok\", \"realm\":\"" + realmName + "\"}";
           return Response.ok(json).build();
       }
   }
   ```

2. Tạo class `MyRestResourceProvider.java`:
   ```java
   package com.company.keycloak.rest;

   import org.keycloak.models.KeycloakSession;
   import org.keycloak.services.resource.RealmResourceProvider;

   public class MyRestResourceProvider implements RealmResourceProvider {
       private KeycloakSession session;

       public MyRestResourceProvider(KeycloakSession session) {
           this.session = session;
       }

       @Override
       public Object getResource() {
           return new MyRestResource(session);
       }

       @Override
       public void close() { }
   }
   ```

3. Tạo class `MyRestResourceProviderFactory.java`:
   ```java
   package com.company.keycloak.rest;

   import org.keycloak.Config;
   import org.keycloak.models.KeycloakSession;
   import org.keycloak.models.KeycloakSessionFactory;
   import org.keycloak.services.resource.RealmResourceProvider;
   import org.keycloak.services.resource.RealmResourceProviderFactory;

   public class MyRestResourceProviderFactory implements RealmResourceProviderFactory {

       public static final String ID = "health-check";

       @Override
       public RealmResourceProvider create(KeycloakSession session) {
           return new MyRestResourceProvider(session);
       }

       @Override
       public void init(Config.Scope config) { }

       @Override
       public void postInit(KeycloakSessionFactory factory) { }

       @Override
       public void close() { }

       @Override
       public String getId() {
           return ID;
       }
   }
   ```

### Bước 3.3: Khai báo SPI và Build JAR
1. Trong cấu trúc thư mục Maven, tạo folder: `src/main/resources/META-INF/services/`.
2. Tạo file có tên chính xác là: `org.keycloak.services.resource.RealmResourceProviderFactory`.
3. Điền nội dung duy nhất vào file đó (là đường dẫn đầy đủ đến class Factory):
   `com.company.keycloak.rest.MyRestResourceProviderFactory`
4. Chạy lệnh: `mvn clean package`. File JAR sẽ được tạo ra tại thư mục `target/`.

### Bước 3.4: Triển khai vào Keycloak
1. Copy file `custom-rest-endpoint-1.0.0.jar` vào thư mục `/opt/keycloak/providers/`.
2. Build lại cấu trúc Quarkus:
   `bin/kc.sh build`
3. Khởi động Keycloak:
   `bin/kc.sh start`

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu:**
- Mở trình duyệt hoặc dùng Postman.
- Gọi vào API: `http://localhost:8080/realms/master/apis/health-check/ping`
- Kết quả nhận được phải là HTTP 200 OK với body JSON: `{"status":"ok", "realm":"master"}`

**Troubleshooting (Khắc phục sự cố):**
- Lỗi 404 Not Found khi gọi API: Keycloak không nhận dạng được Provider. Nguyên nhân hàng đầu là sai tên package trong file cấu hình `META-INF/services/` hoặc quên chưa chạy lệnh `kc.sh build` sau khi copy file JAR.
- Lỗi ClassNotFoundException khi deploy: Có thể bạn đã compile code với version thư viện không đúng với version server Keycloak đang chạy, hoặc dùng scope `compile` làm Maven nhúng lồng các thư viện core của Keycloak vào file JAR. Luôn dùng `<scope>provided</scope>` với các core deps.
