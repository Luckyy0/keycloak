> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Xây dựng một ứng dụng Backend-For-Frontend sử dụng Spring Cloud Gateway tích hợp Keycloak OIDC, bảo vệ SPA an toàn bằng Cookie.

## 1. Kịch bản Thực hành (Lab Scenario)
Bạn được giao nhiệm vụ nâng cấp kiến trúc bảo mật cho hệ thống "E-Commerce SPA". 
Hiện tại, SPA lưu Access Token trực tiếp trên `localStorage`, vi phạm chính sách bảo mật. 
Bạn cần triển khai một BFF (Spring Cloud Gateway) đứng trước SPA và API. Mọi xác thực sẽ do BFF đảm nhiệm thông qua Authorization Code Flow với Keycloak. Frontend chỉ cần gửi Cookie.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Docker và Docker Compose cài đặt trên máy chủ.
- Keycloak đã chạy ở `http://localhost:8080`.
- Java 17 và Maven (hoặc IDE như IntelliJ/Eclipse).
- Trình duyệt web (Chrome/Firefox) có công cụ Developer Tools để kiểm tra Cookie.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 1: Cấu hình Client trên Keycloak**
1. Đăng nhập Keycloak Admin Console.
2. Tạo Realm mới hoặc sử dụng Realm hiện tại (VD: `myrealm`).
3. Điều hướng tới **Clients** -> **Create client**:
   - Client ID: `spring-bff`
   - Client Authentication: `On` (để lấy Client Secret).
   - Standard Flow: `On`.
   - Valid Redirect URIs: `http://localhost:8081/login/oauth2/code/keycloak`
4. Lấy Client Secret trong tab **Credentials**.

**Bước 2: Khởi tạo Project Spring Boot**
Tạo một Spring Boot Project với các dependencies: `spring-boot-starter-webflux`, `spring-cloud-starter-gateway`, `spring-boot-starter-oauth2-client`.

**Bước 3: Cấu hình `application.yml` cho Spring Cloud Gateway**
Mở tệp `application.yml` và cấu hình như sau:

```yaml
server:
  port: 8081

spring:
  cloud:
    gateway:
      default-filters:
        - TokenRelay= # Tự động forward Access Token thành Bearer header
      routes:
        - id: resource-server
          uri: http://localhost:8082 # Địa chỉ Backend API thực tế
          predicates:
            - Path=/api/**
  security:
    oauth2:
      client:
        registration:
          keycloak:
            client-id: spring-bff
            client-secret: <YOUR_CLIENT_SECRET>
            scope: openid, profile
            authorization-grant-type: authorization_code
        provider:
          keycloak:
            issuer-uri: http://localhost:8080/realms/myrealm
```

**Bước 4: Cấu hình Security Configuration**
Tạo class `SecurityConfig.java`:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        http
            .authorizeExchange(exchanges -> exchanges
                .pathMatchers("/", "/index.html").permitAll()
                .anyExchange().authenticated()
            )
            .oauth2Login(withDefaults())
            .csrf(csrf -> csrf.disable()); // Trong thực tế cần bật CSRF và cấu hình Cookie
        return http.build();
    }
}
```

**Bước 5: Khởi chạy và Kiểm tra**
1. Chạy ứng dụng Spring Boot Gateway (`localhost:8081`).
2. Mở trình duyệt, truy cập `http://localhost:8081/api/test`.
3. Spring Gateway sẽ chuyển hướng (302 Redirect) sang giao diện đăng nhập Keycloak.
4. Đăng nhập bằng tài khoản Keycloak.
5. Keycloak trả về Redirect đến `/login/oauth2/code/keycloak`.
6. Gateway nhận Code, lấy Token, lưu Session và chuyển tiếp (Proxy) request đến API.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu (Verification):**
- Mở **Developer Tools** (F12) trên trình duyệt, chuyển sang tab **Application** -> **Cookies**.
- Bạn PHẢI nhìn thấy Cookie tên `SESSION` với flag `HttpOnly` được bật.
- Không có bất kỳ Access Token nào hiển thị trong Cookie hay Local Storage.
- Kiểm tra logs của Backend API (Cổng 8082), API PHẢI nhận được Request đi kèm Header `Authorization: Bearer <token>`.

**Xử lý sự cố (Troubleshooting):**
- **Lỗi HTTP 500 [invalid_token_response]:** Kiểm tra lại Client Secret và Client ID trong file `application.yml` xem có khớp với Keycloak không.
- **Lỗi HTTP 404 (Not Found):** Đảm bảo Resource Server API đang chạy đúng cổng 8082 và cấu hình Route Predicates `Path=/api/**` là chính xác.
- **Vòng lặp Redirect vô tận:** Do cấu hình `issuer-uri` sai. Đảm bảo `issuer-uri` trong cấu hình Spring trùng khớp chính xác 100% với cấu hình Frontend URL của Keycloak.
