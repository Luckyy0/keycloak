> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Thiết lập một Spring Cloud Gateway hoạt động dưới vai trò OAuth2 Resource Server, cấu hình Route Security và kích hoạt Token Relay để bảo vệ một ứng dụng Backend nội bộ bằng Keycloak.

## 1. Kịch bản Thực hành (Lab Scenario)
Bạn có một hệ thống gồm:
1. **Keycloak:** Cung cấp dịch vụ cấp phát JWT (Issuer).
2. **Backend Service (Resource Server):** Một ứng dụng Spring Boot chạy ở cổng 8081, có endpoint `/api/protected` yêu cầu xác thực JWT.
3. **API Gateway (Spring Cloud Gateway):** Chạy ở cổng 8080, nhận request từ Client, kiểm tra tính hợp lệ của JWT và định tuyến request đến Backend bằng cách sử dụng `TokenRelay`.

Mục tiêu là Client gọi đến `http://localhost:8080/api/protected` với JWT lấy từ Keycloak. Gateway xác thực Token, chuyển tiếp (Relay) đến Backend. Backend trả về dữ liệu thành công.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Java 17+ và Maven.
- Một Keycloak Server đang chạy. Giả định Keycloak chạy cổng 9090 (để tránh đụng cổng 8080 của Gateway).
- Realm `myrealm` đã được tạo trên Keycloak, Client ID: `gateway-client` (Public hoặc Confidential).
- Tạo một User `testuser` có mật khẩu (ví dụ: `password`).

**Cấu trúc thư mục Lab:**
```text
gateway-lab/
├── gateway-service/ (Spring Boot + Cloud Gateway)
│   ├── pom.xml
│   └── src/main/resources/application.yml
└── backend-service/ (Spring Boot Web)
    ├── pom.xml
    └── src/main/resources/application.yml
```

## 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 1: Khởi chạy và cấu hình Keycloak (Cổng 9090)**
Sử dụng Docker để khởi chạy Keycloak:
```bash
docker run -p 9090:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:latest start-dev
```
- Truy cập `http://localhost:9090/admin`.
- Tạo Realm tên `myrealm`.
- Tạo Client tên `gateway-client` (Client authentication: Off, Valid redirect URIs: `*`).
- Tạo User `testuser`, thiết lập credentials (mật khẩu) `password` và tắt cờ "Temporary".

**Bước 2: Xây dựng Backend Service (Cổng 8081)**
- Tạo dự án Spring Boot (Spring Web, OAuth2 Resource Server).
- Cấu hình `application.yml` cho Backend:
```yaml
server:
  port: 8081
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:9090/realms/myrealm
```
- Tạo REST Controller:
```java
package com.example.backend;

import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class BackendController {
    @GetMapping("/protected")
    public String getProtectedData(@AuthenticationPrincipal Jwt jwt) {
        return "Hello " + jwt.getClaimAsString("preferred_username") + "! You reached the backend.";
    }
}
```

**Bước 3: Xây dựng API Gateway Service (Cổng 8080)**
- Tạo dự án Spring Boot (Gateway, OAuth2 Resource Server).
- Cấu hình `application.yml` cho Gateway:
```yaml
server:
  port: 8080
spring:
  cloud:
    gateway:
      routes:
        - id: backend-route
          uri: http://localhost:8081
          predicates:
            - Path=/api/**
          filters:
            - TokenRelay=
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:9090/realms/myrealm
```
- Cấu hình Security (`SecurityConfig.java`):
```java
package com.example.gateway;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {
    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        http
            .authorizeExchange(exchanges -> exchanges
                .pathMatchers("/api/**").authenticated() // Yêu cầu xác thực
                .anyExchange().permitAll()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt());
        return http.build();
    }
}
```

**Bước 4: Chạy dịch vụ**
Chạy ứng dụng Backend (cổng 8081) và Gateway (cổng 8080) bằng lệnh:
```bash
mvn spring-boot:run
```

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Kiểm tra 1: Lấy Token từ Keycloak**
Mở Terminal, chạy lệnh lấy JWT (Direct Access Grant):
```bash
TOKEN=$(curl -s -X POST "http://localhost:9090/realms/myrealm/protocol/openid-connect/token" \
  -d "client_id=gateway-client" \
  -d "username=testuser" \
  -d "password=password" \
  -d "grant_type=password" | grep -o '"access_token":"[^"]*' | cut -d'"' -f4)
  
echo "Bearer $TOKEN"
```

**Kiểm tra 2: Gọi Backend trực tiếp (Chặn)**
Thử gọi Backend (8081) không có Token. Bạn sẽ nhận lỗi `401 Unauthorized`.
```bash
curl -v http://localhost:8081/api/protected
```

**Kiểm tra 3: Gọi Gateway với Token (Thành công)**
Truyền Token vào Request gọi Gateway (8080):
```bash
curl -H "Authorization: Bearer $TOKEN" http://localhost:8080/api/protected
```
*Kết quả mong muốn:* `Hello testuser! You reached the backend.`

**Lỗi thường gặp (Troubleshooting):**
- *Lỗi 401 Unauthorized tại Gateway:* Nguyên nhân phổ biến là `issuer-uri` trong `application.yml` không khớp chính xác với trường `iss` bên trong Payload của token. Kiểm tra lại việc sử dụng `localhost` hoặc địa chỉ IP. Cả hai phía cấu hình phải giống y hệt nhau.
- *Lỗi 500 Internal Server Error / Connect Timeout:* Có thể Backend (8081) chưa khởi động xong, hoặc cấu hình `uri: http://localhost:8081` trong Gateway sai cổng. Xem console log của Gateway để biết lỗi chi tiết từ Netty.
